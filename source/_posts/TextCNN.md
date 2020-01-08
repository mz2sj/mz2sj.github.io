---
title: TextCNN
date: 2020-01-07 19:50:28
tags: [CNN,文本分类]
---



学了点pytorch，最近在知乎刷到了一个repo主要是介绍文本分类的，感觉还不错，打算用来入门，自己将会比较细致的实现代码，分析代码的结构。天下大事必做于易，天下大事必做于细。寒假来了，自己基础较差，只能多花点时间了。

## 基础网络

作者自己没有直接调用torch的api，而是在其基础上进行实现。一些模块继承自```nn.Module```层，并且实现了初始化和前向计算。嘿呀，等我熟练了，我也要自己做一个工具箱。不多说上代码

### Conv1d

```python
import torch.nn.Functional as F

class Conv1d(nn.Module):
    def __init__(self,in_channels,out_channels,filter_sizes):
        super(Conv1d,self).__init__()
        self.convs=nn.ModuleList([
            nn.Conv1d(in_channels,out_channels,kernel_size=fs)
            for fs in filter_sizes
        ])
        self.init_params()
        
    def init_params(self):
        for m in self.convs:
            nn.init.xavier_uniform_(m.weight.data)
            nn.init.constant_(m.bias.data,0.1)
            
    def forward(self,x):
        return [F.relu(conv(x))for conv in self.convs]
```

让我们来细细品味这段代码，继承自 ```nn.Module``` 这不用说了，表明这个模块可以和torch自带的模块混用。```__init__```函数的参数有讲究，除了必传的```self```外，还传入了一些指定卷积层形状的参数，例如通道数：```in_channels、out_channels```,还有卷积核集合。这里传入的是卷积核集合，而不是单单一个卷积核需要注意。接下来是继承自父类的初始化，```super()```第一个参数一般为函数名，第二个是self,后面再接```__init__()```。在```__init__()```函数中放置要用到的模块或参数等，不涉及到具体的操作，在初始化层调用了后面定义的参数 初始化函数。

第二部分是初始化层，```init_params()```。初始化层比较好理解，初始化```init```层的参数就好。前面要初始化的只有```ModuleList```中的卷积层，卷积的函数改天专门总结一下。卷积层参数分为weight和bias两部分,bias部分初始化为0。

第三部分是前向传播，```forward()```。参数有输入x，由于有多个卷积核，将多个卷积核的卷积结果放到列表中。再调用```F.relu()```activation。不得不说```torch.nn.Functional``` 超好用。

## Linear

Linear层和Conv1d结构上差不多，不过Linear层的初始化参数将卷积层的channel换成了feature，并且少了卷积核。代码如下：

```python
class Linear(nn.Module):
    def __init__(self,in_features,out_features):
        super(Linear,self).__init__()
        self.linear=nn.Linear(in_features=in_features,out_features=out_features)
        self.init_params()
        
    def init_parmas(self):
        nn.init.kaiming_normal_(self.linear.weight)
        nn.init.constant_(self,linear.bias,0)
        
    def forward(self,x):
        x=self.linear(x)
        return x
```

## TextCNN

到了最最关键的环节了，TextCNN层:

```python
class TextCNN(nn.Module):
    def __init__(self,embedding_dim,n_filters,filter_sizes,output_dim,dropout,
                 pretrained_embeddings):
        super(TextCNN,self).__init__()
        #导入预训练的embedding frezee指定权重是否更新
        self.embedding=nn.Embedding.from_pretrained(pretrained_embeddings,freeze=False)
        #卷积核参数 embedding维数，卷积核个数，卷积核大小
        self.convs=Cov1d(embedding_dim,n_filters,filter_sizes)
        #前向运算 输入维度 预测输出
        self.fc=Linear(len(filter_sizes)*fliters,output_dim)
        self.dropout=nn.Dropout(dropout)
        
    def forward(self,x):
        #text:[sent_len,batch_size] 两个参数分别是行数列数
        text,_=x
        #维度换位 [batch_size,sent_len]
        text=text.permute(1,0)
        # [batch_size,sent_len,embedding_len]
        embedded=self.embedding(text)
        #[batch_size,emb_dim,sent_len]
        embedded=embedded.permute(0,2,1)
        conved=self.convs(embedded)
        ##[batch_size,n_filters,sent_len-filter_sizes[n]+1]
        pooled=[F.max_pool1d(conv,conv.shape[2])
               for conv in conved]
        cat=self.dropout(torch.cat(pooled,dim=1))
        cat=cat.reshae((cat.shape[0],-1))
        return self.fc(cat)
```

TextCNN层中的张量维度变换比较复杂，我是一下子反应不过来的，后面自己会debug彻底弄清楚每一步是怎么卷积的，维度怎么变换的。

## argparse

代码还有一个比较值得借鉴的点就是使用了argparse模块对各种参数进行管理，不是一个个手动去定义变量，更加清晰。

``` python
import argparse

def get_args(data_dir, cache_dir, embedding_folder, model_dir, log_dir):
    parser = argparse.ArgumentParser(description='SST')

    # data_util
    parser.add_argument('--model_name', default='TextCNN', type=str, help='参数所属模型名')
    parser.add_argument('--seed', default=47, type=int, help='随机种子')
    parser.add_argument('--data_path', default=data_dir, type=str, help='SST2数据集位置')
    parser.add_argument('--cache_path', default=cache_dir, type=str, help='数据集地址')
    parser.add_argument('--sequence_length', default=60, type=int, help='句子长度')

    # 输出文件名
    parser.add_argument('--model_dir', default=model_dir + 'TextCNN/', type=str, help='输出模型地址')
    parser.add_argument('--log_dir', default=log_dir + 'TextCNN/', type=str, help='日志文件地址')
    parser.add_argument('--do_train', action='store_true', help='Whether to run training')
    parser.add_argument('--print_step', default=100, type=int, help='多少步存储一次模型')

    # 优化参数
    parser.add_argument('--batch_size', default=64, type=int)
    parser.add_argument('--epoch_num', default=5, type=int)
    parser.add_argument('--dropout', default=0.4, type=float)

    # 模型参数
    parser.add_argument('--output_dim', default=2, type=int)

    # TextCNN参数
    parser.add_argument('--filter_num', default=200, type=int, help='filter数量')
    parser.add_argument('--filter_sizes', default='1 2 3 4 5', type=str, help='filter的size')

    # word embdding
    parser.add_argument('--glove_word_file', default=embedding_folder + 'glove.6B.50d.txt',
                        type=str, help='path of embedding file')
    parser.add_argument('--glove_words_size', default=int(2.2e6), type=int, help='Corpus size for Glove')
    parser.add_argument('--glove_word_dim', default=50, type=int, help='word embedding size(default:300)')
    config = parser.parse_args([])
    return config
```

注意的一点就是parse_args()在notebook中需要加入[]作为参数，否则会报错。

## device

指定设备device，这个很容易理解

```python
def get_device():
    device=torch.device('cuda:0' if torch.cuda.is_available() else 'cpu')
    n_gpu=torch.cuda.device_count()
    if torch.cuda.is_available():
        print('device is cuda,# cuda is:',n_gpu)
    else:
        print('device is cpu,not recommend')
    return device,n_gpu
```

统计gpu数代码：```torch.cuda.device_count()```

指定device代码：```torch.device('cuda:0')```

## 加载数据集

对api还不太熟悉，这里也比较复杂。

```python
def load_sst2(path,text_field,label_field,batch_size,device,embedding_file,cache_dir):
    train,dev,test=data.TabularDataset.splits(
    	path=path,train='train.tsv',validationj='dev.tsv',
        test='test.tsv',format='tsv',skip_header=True,
        fields=[('text',text_field),('label',label_field)]
    )
    print('the size of train:{},dev:{},test:{}'.format(
        len(train.examples), len(dev.examples), len(test.examples)
    ))
    #将词向量放到缓存目录中，下次直接从换从目录中加载
    vectors=vocab.Vectors(embedding_file,cache_dir)
    
    #使用之前构建的词向量映射文本序列，将子序列转化为数字序列
    text_field.build_vocab(
    	train,dev,test,max_size=25000,
        vectors=vectors,unk_init=torch.Tensor.normal_
    )
    label_field.build_vocab(train,dev,test)
    
    train_iter,dev_iter,test_iter=data.BucketIterator.splits(
    	(train,dev,test),batch_sizes=(batch_size,len(dev),len(test)),
        sort_key=lambda x:len(x.text),sort_within_batch=True,repeat=False,
        shuffle=True,device=device
    )
    return train_iter,dev_iter,test_iter
```

得总结一下套路，看起来懵懵的

## 训练

```python
def train(epoch_num,model,train_dataloader,dev_dataloader,optimizer,criterion,
          label_list,out_model_file,log_dir,print_step,data_type='word'):
    #首先转为训练模式
    model.train()
    writer=SummaryWriter(log_dir=log_dir+time.strftime('%H%M%S',time.gmtime()))

    global_step=0
    best_dev_loss=float('inf')
    
    for epoch in range(int(epoch_num)):
        print(f'--------------Epoch:{epoch+1:02}--------------')
        
        epoch_loss=0
        train_step=0
        
        all_preds=np.array([],dtype=int)
        all_labels=np.array([],dtype=int)
        
        for step,batch in enumerate(tqdm(train_dataloader,desc='Iteration')):
            #首先梯度归零
            optimzier.zero_grad()
            logits=model(batch.text)
            
            #计算损失
            loss=criterion(logits.view(-1,len(label_list)),batch.label)
            labels=batch.label.detach().cpu().numpy()
            preds=np.argmax(logits.detach().cpu().numpy(),axis=1)
            
            #计算梯度
            loss.backward()
            optimizer.step()
            global_step+=1
            
            epoch_loss+=loss.item()
            train_steps+=1
            
            all_preds=np.append(all_preds,preds)
            all_labels=np.append(all_labels,labels)
            
            #打印
            if global_step%print_step==0:
                train_loss=epoch_loss/train_steps
                train_acc,train_report=classification_metric(
                    all_preds,all_labels,label_list)
                dev_loss,dev_acc,dev_report=evaluate(
                    model,dev_dataloader,criterion,label_list,data_type
                )
                c=global_step/print_step

                writer.add_scalar('loss/train',train_loss,c)
                writer.add_scalar('loss/dev',dev_loss,c)

                writer.add_scalar('acc/train',train_acc,c)
                writer.add_scalar('acc/dev',dev_acc,c)

                for label in label_list:
                    writer.add_scalar(label+':'+'f1/train',
                                train_report[label]['f1-score'],c)
                    writer.add_scalar(label+':'+'f1/dev',
                                dev_report[label]['f1-score'],c)
                print_list=['macro avg','weighted avg']
                for label in print_list:
                    writer.add_scalar(label+':'+'f1/train',
                                train_report[label]['f1-score'],c)
                    writer.add_scalar(label+':'+'f1/dev',
                                dev_report[label]['f1-score'],c)
                if dev_loss<best_dev_loss:
                    best_dev_loss=dev_loss
                    torch.save(model.state_dict(),out_model_file)
                model.train()
    writer.close()
```

训练函数有几个点总结，传入参数有：迭代次数、model、训练集、验证集，optimzer、损失函数、打印的epoch数，等等。

## evaluate

用于在训练过程中对训练结果进行测试

```python
def evaluate(model,iterator,criterion,label_list,data_type='word'):
    model.eval()
    epoch_loss=0
    all_preds=np.array([],dtype=int)
    all_labels=np.array([],dtype=int)

    with torch.no_grad():
        for batch in iterator:
            if data_type=='word':
                with torch.no_grad():
                    logits=model(batch.text)
            elif data_type=='highway':
                with torch.no_grad():
                    logits=model(batch.text_word,batch.text_char)

            loss=criterion(logits.view(-1,len(label_list)),batch.label)

            labels=batch.label.detach().cpu().numpy()
            preds=np.argmax(logits.detach().cpu().numpy(),axis=1)

            all_preds=np.append(all_preds,preds)
            all_labels=np.append(all_labels,labels)
            epoch_loss+=loss.item()
    acc,report=classification_metric(all_preds,all_labels,label_list)
    return epoch_loss/len(iterator),acc,report
```

测试过程中记得要切换到eval模式，并且关闭梯度追踪```with torch.no_grad()```。

## main函数

````python
def main(config):
    if not os.path.exists(config.model_dir):
        os.makedirs(config.model_dir)

    if not os.path.exists(config.log_dir):
        os.makedirs(config.log_dir)

    print('\t \t \t the model name is {}'.format(config.model_name))
    device, n_gpu = get_device()

    # 指定torch 随机种子
    torch.manual_seed(config.seed)
    # 指定numpy 随机种子
    np.random.seed(config.seed)
    torch.manual_seed(config.seed)
    if n_gpu > 0:
        # 指定cuda随机种子
        torch.cuda.manual_seed_all(config.seed)
        # CuDNN的卷积操作就是每次一样,实验可重复
        torch.backends.cudnn.deterministic = True

    # sst2 数据准备
    text_field = data.Field(tokenize='toktok', lower=True, include_lengths=True, fix_length=config.sequence_length)
    label_field = data.LabelField(dtype=torch.long)

    train_iterator, dev_iterator, test_iterator = load_sst2(config.data_path, text_field, label_field,
                                                            config.batch_size,
                                                            device, config.glove_word_file, config.cache_path)

    # 词向量准备 获取词向量按index顺序排序
    pretrained_embeddings = text_field.vocab.vectors
    model_file = config.model_dir + 'model1.pt'

    # 模型准备 卷积核大小
    filter_sizes = [int(val) for val in config.filter_sizes.split()]
    model = TextCNN(config.glove_word_dim, config.filter_num, filter_sizes,
                    config.output_dim, config.dropout, pretrained_embeddings)
    optimizer = torch.optim.Adam(model.parameters())
    criterion = nn.CrossEntropyLoss()

    model = model.to(device)
    criterion = criterion.to(device)

    if not config.do_train:
        train(config.epoch_num, model, train_iterator, dev_iterator, optimizer, criterion,
              ['0', '1'], model_file, config.log_dir,config.print_step, 'word')
    model.load_state_dict(torch.load(model_file))

    test_loss, test_acc, test_report = evaluate(model, test_iterator, criterion, ['0', '1'], 'word')
    print('-----------Test----------')
    print('\t Loss:{}|Acc:{}|Macro avg F1{}|Weighted avg F1{}'.format(
        test_loss, test_acc, test_report['macro avg']['f1-score'], test_report['weighted avg']['f1-score']
    ))
````

太多啦，懒得再记录鸟~下面分析一下卷积的过程吧。

## 卷积的过程

代码中用了5个卷积核，卷积核的大小分别是[1,2,3,4,5]。一维卷积层的输入是词向量的拼接，本代码中使用的词向量是50维，每个样本有60个词，因此卷积的输入是50x60的矩阵$X$。一维卷积核的输入通道数就是$X$的宽50，输出通道数是卷积核的个数200，则卷积核[1,2,3,4,5]对应的输出分别为[200*x60,200x59,200x58,200x57,200x56]。再分别对行进行最大池化，得到的输出为5个[200x1]的向量，将这5个向量拼接起来成[1000x1]的向量，输入前馈神经网络[1000x2]，即可进行二分类。老实说，一维卷积的过程自己没太看懂。代码中将卷积核的个数作为一维卷积的输出通道数，embedding维度作为卷积的输入通道数。一直以来看到的都是二维卷积，一维卷积有点懵，下面画个草图理解下吧。

[![l2WFPS.md.jpg](https://s2.ax1x.com/2020/01/08/l2WFPS.md.jpg)](https://imgchr.com/i/l2WFPS)