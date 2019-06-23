# DEEP FM



1. 显式的交叉LAYER层（隐式的交叉，对于高度稀疏的特征来说，并不一定有用）；
2. 借助LR/FM等线性模型，可以帮助DNN学习更好的特征表示；

**基本原理**

FM是一种Embedding，特征之间的关联性通过embedding vector的内积来表示，就是一种显式的特征组合。
