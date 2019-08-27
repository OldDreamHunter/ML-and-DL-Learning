# Seq2Seq & Attention

![img](http://plm-images.oss-cn-hongkong.aliyuncs.com/image/nlp/am/01-encoder-decoder-n)

通过Encoder将语句encode成语义编码，再用Decoder对语义编码进行解析，从而得出最后的结果

其中Encoder和Decoder具体选择什么模型都可以，CNN，RNN，BiRNN，LSTM等

缺点：

y1,y2..,yn都是通过同一个语义编码得到的，因此称为注意力不集中的模型，其实wide and deep就是一个Encoder-Decoder的结构，但是存在的问题就是注意力不集中。

**Attention基本架构**



![img](http://plm-images.oss-cn-hongkong.aliyuncs.com/image/nlp/am/02-encoder-decoder-am-n)

每个y1都有自己的c1去对应计算，这个c1是怎么计算的呢，通过生成对应的关注概率计算

而对应的关注概率就是x1和y1去通过对齐概率计算的





# Attention

https://www.jianshu.com/p/0f0c674837e3

https://www.sohu.com/a/242214491_164987

https://arxiv.org/abs/1706.03762

