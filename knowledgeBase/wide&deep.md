# wide & deep

#### 1. 理解wide & deep

---

* wide&deep的理解

  * 记忆与拓展，类别特征，特征交叉，沿着这一脉络，结出了wide&deep，FM/FFM/DeepFM，Deep&Cross Network，Deep Interest Network等果实

    Wide，我们根据人工经验和业务背景，将一些有价值并且显而易见的特征及组合，喂入wide部分

    Deep，通过embedding，和深层交互，学到tag的向量化，模糊查找tag向量，从而使得自己具备良好的“拓展”能力，学习到更深层的推荐

  * 为什么推荐和搜索喜欢类别型的特征？LR和DNN底层还是线性模型，类别化的作用：引入非线性，增强模型的鲁棒性，同时分段数据更符合本质

  * 针对特征交叉，都是对特征进行embedding操作之后，再把特征给wide部分还有deep部分使用，wide部分利用FM做特征组合，deep部分是更深层次的特征隐含表示的组合

    

#### 2. DNNLinearCombinedClassifier实现

---

feature column的实现有以下几种方式：

- feature column允许一个field下有多个tag，允许这多个tag有自己的weight
- feature column自动处理missing value和OOV
- feature column是通过safe_embedding_lookup_sparse来完成embedding的，允许一次性映射多个embedding，并combine。

Feature Column实现了Wide & Deep中用到的所有特征工程方法，比如**token==>id的映射，embedding, hashing trick, feature crossing**等。在掌握了本文所介绍的各类、方法的基本用法之后，阅读Wide & Deep的代码也就水到渠成了。



#### 3. 源码实现

---

- Wide & Deep模型实践起来很简单，但是只有搞清楚了如下4个问题，才算真正理解了它：

  **为什么要deep？**

  **为什么要wide？**

  **什么样的特征喂进deep侧？**

  **什么样的特征喂进wide侧？**

Wide & Deep代码比较清晰，不难理解。看点在于学习**Wide侧是如何处理高维、稀疏的categorical特征**，并把这种技巧应用于自己的代码中。我就是在学习了这种技巧后，开发了**支持多值、稀疏、权重共享的DeepFM**



#### 4. 实战案例

---

红酒价格预测

特征：国家，红酒描述，称号/名称，得分，价格，省份，区域1，区域2，类别 ，酒厂



**wide部分的输入：**

类别型的特征（国家，红酒描述，称号/名称，省份，区域1，区域2，类别 ，酒厂），应该是需要做embedding之后再输入，数值型的特征（得分），标签：价格

对于红酒描述来说，是text_to_matrix；对于类别，是one-hot，然后把这两个数据输入到wide部分

wide部分在keras中应该就是一层全连接，神经元为1的全连接网络



**deep部分的输入：**

对红酒的描述做word2vec的embedding，然后flatten扁平化之后输入到全连接神经网络中