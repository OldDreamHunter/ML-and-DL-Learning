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

案例中的做法：

对于descrption，先是建立tokenize, 然后利用tokenize.texts_to_matrix当作输入

对于variety，利用label_encoder，transform之后，利用to_categorical做one-hot

改进：

仅仅只有这两部分还不够，应该把评分加入进去，酒庄等特征都加进去



**deep部分的输入：**

对红酒的描述做word2vec的embedding，然后flatten扁平化之后输入到全连接神经网络中



#### 5. 参数影响

---

batch-size 和 epoch，其中batch-size越小就越影响梯度的随机性，一定程度上能提高优化，但是batch-size太小又会造成不收敛，epoch越大则权重更新的次数越多，容易造成过拟合。

神经网络参数的优化一般是选取梯度下降的方法，梯度下降主要有三种方法，1. 批量（batch）；2. 随机（stochastic）; 3. 小批量（mini-batch）。

* 随机梯度下降

  对每一条数据单独做梯度下降，类似与梯度下降的在线机器学习算法；

  优点：1. 更新频繁可以及时反馈更新效果；2. 避免局部最优；

  缺点：1. 计算消耗更大；2. 更新产生的噪声导致模型参数和误差来回跳动；3. 噪声更新让算法难以稳定收敛

* 批量梯度下降（全量数据在一个batch中，每次对全量数据进行求解）

  对训练集的每一个数据计算误差，然后所有训练数据计算完成之后才更新模型

  对训练集的一次训练过程称为一代，因此，批量梯度下降就是更新一个epoch之后的模型更新的结果

  优点：更稳定的误差梯度和收敛，误差计算和模型更新分离

  缺点：会导致模型收敛到一个局部最优；累积所有训练数据误差之后再更新梯度；

* 小批量梯度下降（选取mini-batch来计算梯度，每次梯度更新就在mini-batch的数据中更新）

  将训练集划分为很多批，每一批数据计算误差并更新参数；

  对batch的梯度进行累加或者计算均值，减少梯度的方差；

  在随机梯度下降的鲁棒性和批量梯度下降的效率之间取得平衡；

  优点：比批量梯度下降更快的更新频率，有利于稳定收敛，避免局部最优；相比于随机梯度下降的效率更高

  缺点：增加了一个超参数batch-size；和批量梯度下降一样，需要累加每个batch的误差

**mini-batch梯度下降如何配置？**

主要是选择batch-size，batch-size越小，会使得收敛过程更快，但是产生的噪声多，如果batch-size越大，则会导致收敛较慢，但是能准确地计算每次误差梯度。

建议：

batch-size的默认值最好是32；调节batch-size时，最好观察模型在不同的batch-size下的训练时间和验证误差的曲线；调整其他超参数之后最后再调整学习率和batch-size；