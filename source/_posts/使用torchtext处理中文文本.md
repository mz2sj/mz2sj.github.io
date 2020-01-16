---
title: 使用torchtext处理中文文本
date: 2020-01-08 19:45:08
tags: torchtext
---





torchtext是pytorch中专门用来处理文本的包，使用torchtext可以灵活的帮我们生成batch和词向量。网上介绍的大多是基于英文的版本，自己在中文文本上进行了尝试，将之前的英文文本分类改成中文文本分类。下面开始介绍。

## 数据格式

| text                   | label |
| ---------------------- | ----- |
| 又要写博客了，好开心啊 | 0     |
| 程序出bug了，唉        | 1     |

数据格式可以是csv、tsv、json文件，自己比较喜欢处理csv、tsv文件两者是一样的，json文件还没怎么接触。数据的内容如上所示，包括文本列和标签列，一般会分为train、dev、test三个集合。本文用到train.tsv、dev.tsv、test.tsv。

## Field

我也不知道这个该叫啥，针对我们要处理的不同数据分别创建Field对象。

```python
import torchtext.data as data
import jieba

def tokenizer(x):
    return list(jieba.cut(x))

text_field=data.Field(tokenize=tokenizer,fix_length=60)
label_field=data.LabelField(dtype=torch.long)
```

针对我们需要处理的text列和label列，我们创建了两个Field对象，分别用来处理对应列。因为我们需要对text进行分词，所以需要指定分词器，我这里用的结巴。分词器的返回结果需要是关于词语的列表，可以根据自己的需求指定。fix_length指定单个序列的长度，torchtext会自动帮我们进行padding和裁剪操作。label列是分类的类别列，torchtext相应提供了LabelField，需要指定数据类型。看别人的代码，使用Field也能用，自己还没实验过。

Field中还有其他参数

```python
def generate_bigrams(x):
    n_grams = set(zip(*[x[i:] for i in range(2)]))
    for n_gram in n_grams:
        x.append(' '.join(n_gram))
    return x

text_field=data.Field(tokenize=tokenizer,fix_length=60,include_lengths=True,
                     preprocessing=generate_bigrams)
```

include_lengths,返回的minibatch中包含padded后的text和序列长度，序列长度对于我们处理变长序列有用。这样操作后，在后面的每个batch中，batch.text返回的就是一个（text，text_length）的元组。

preprocessing,添加预处理函数，tokenize后的结果返回的是列表类型数据，将这个列表数据再通过preprocessing指定的函数进行处理。

## TabularDataset

针对我们需要处理的表格数据，torch提供了TabularDataset帮助我们按行进行划分，按照表格对应列的顺序指定Field对象构成元组```fields=[('text', text_field)，('label', label_field), ]``` ,label和text是别名，可以任意，我们后面可以通过这个别名获取batch的数据。除此之外还要指定数据的路径，格式，以及是否跳过表头等超参数。

```
train,dev,test=data.TabularDataset.splits(path,train='trian.tsv',validation='dev.tsv',test='test.tsv',skip_header=True,fields=[('label',label_field),('text',text_field)],format='tsv')
```



生成的数据格式如下

[![l2IfgO.md.png](https://s2.ax1x.com/2020/01/08/l2IfgO.md.png)](https://imgchr.com/i/l2IfgO)



## 构建映射和导入词向量

前面将数据按行进行划分了，但是并没有建立词和索引的对应关系，也没有建立索引和词向量的对应关系，下一步就是建立词和索引和向量的关系

```python
import torchtext.vocab as vocab

vectors=vocab.Vectors('embedding_path','cachepath')
text_field.build_vocab(train,dev,test,vectors,unk_init=torch.Tensor.normal_)
label_field.build_vocab(train,dev,test)
```

torchtext.vocab提供了一个缓存词向量的方法，在第一次读取词向量时将词向量缓存到指定目录，下次再读取词向量时直接从缓存目录中读取，这样就不用每次加载词向量了，加快了读取速度。未知词和padding词词语和索引对应关系为：{<unk>:0,<pad>:1}。其索引可以通过如下 方式获取：

```python
PAD_IDX=text_field.vocab.stoi[text_field.pad_token]
UNK_IDX=text_field.vocab.stoi[text_field.unk_token]
```

未知词和padding词通常对我们的任务没有帮助，但`unk_init=torch.Tensor.normal_`却正太分布初始化了，获取了这两个词的索引可以帮助我们在后面的embedding层将其词向量重置为0

建立text中词和索引、词向量的映射关系时，需要输入要处理的数据、词向量，指定未识别词的初始化方式。

建立label中类别和索引的关系就比较简单了，输入之前生成的数据就好了。看一眼生成的词表和词向量

[![l2TJf0.md.png](https://s2.ax1x.com/2020/01/08/l2TJf0.md.png)](https://imgchr.com/i/l2TJf0)

## 生成batch

前面我们已经将每行数据进行了分词，建立了词表、索引等对应关系，接下来就是生成batch啦

```python
train_iter,dev_iter,test_iter=data.BucketIterator.splits(
	(train,dev,test),batch_sizes=(64,len(dev),len(test)),
    sort_key=lambda x:len(x.text),sort_within_batch,repeat=False,
    shuffle=True,device=device
)
```

首先指定要处理的数据（train，dev，test），接着指定各个数据对应要生成 的batch的batch_size大小，注意这里是多个数据集的batch_size,参数是batch_sizes=（64，len（dev),len(test)）,可以看到这里对dev和test我们的batchsize大小就是他们所有数据，所以这些数据只会生成一个batch。通常对训练集和验证集的数据要按数据长度进行排序，我们要指定排序的键sort_key，这里使用了匿名函数，text就是我们之前指定的别名。还包括其他一些参数，不一一介绍。至此我们就生成了text和label的batch集合，两者是合在一起的，我们可以通过batch的别名获得各自值。sort_within_batch,在一个batch内对数据进行排序。

## embedding

如何获得词表对应的embedding呢？

```python
pretrained_embeddings=text_field.vocab.vectors
```

可以和torch的embedding模块结合使用

```python
embedding = nn.Embedding.from_pretrained(pretrained_embeddings, freeze=False)
embedded=embedding(batch.text)
```

embedding的输入需要是之前处理过的batch数据，torchtext会自动帮我们处理好对应关系。

那么我们如何处理边长序列呢？通常为了数据的通用性，我们对较短的数据会进行padding，让所有序列具有相同的长度。当使用RNN等网络时，经过padding的数据产生的hidden state、outputs对于我们并无帮助，我们主需要真是序列长产生的hidden state和outputs即可。torchtext提供了pack_padded_sequence和pad_packed_sequence两个方法，对embedded的结果进行处理加入序列长度。

```python
embedded=self.embedding(text) #添加了padding的embedding结果
packed_embedded=nn.utils.pack_padded_embedding(embedded,text_lengths)
packed_output,(hidden,cell)=self.rnn(packed_output) #没有padding的输出
output,output_lengths=nn.utils.rnn.pad_packed_sequence(packed_output) #返回输出，和没有padding的句子长度
```

对于unk和pad词我们可以将其词向量初始化为0向量

```python
model.embedding.weight.data[UNK_IDX]=torch.zeros(EMBEDDING_DIM)
model.embedding.weight.data[PAD_IDX]=torch.zeros(EMBEDDING_DIM)
```

其实在调用field.build_coab时会自动将pad和unk初始化为0

还可以换种方式指定embedding

```python
embedding=nn.Embedding(vocab_size,embedding_dim,pad_idx=pad_idx)
```

先指定好embedding的形状，再在模型搭建好后指定embedding

```
embedding=model.embedding.weight.data.copy_(pretrained_embeddings)
```

收工~