## 3.textCNN
### 基础问题
    3.1卷积的作用？
    3.2和图片卷积的区别？　图像的卷积核高宽都是一样的，textcnn宽等于词向量维度，长可自定义为了保证上下文和语序
    3.3卷积核
    3.4nparray是什么？

### 结构
    预测：crf分词 输入限定1000字 同一个vocab 词表40w多 是倒序放置的 输入为分词term在词表的位置
    模型：词向量的维度在300
    输入层：
        把vocab词表分term2id和id2term，categories同理，用预训练的word2vec生成vocab的id2vec，训练的输入是label的id +文档的在vocab的id，用　tf.nn.embedding_lookup(embeddings, self.X)　对应得到embedded输入(嵌入层) x是词限制1000，y是类别；

    卷积层：
        卷积主要用不同高度和做点乘，叠加偏执，用激活函数激活（Relu? 作用？防止特征爆炸和消失）得到特征，对边缘采用置零处理，
        https://blog.csdn.net/John_xyz/article/details/79210088
        https://blog.csdn.net/vivian_ll/article/details/80831509

    num_filters = 128 # 卷积核数量
    kernel_size = 5 # 卷积核大小
    batch_size = 128　＃　每次训练采用样本的数量

    dropout_keep_prob = 0.5　# Dropout层每个元素被保留的概率，防止过拟合　https://bbs.csdn.net/topics/392287483

    池化层：
        按最大取，
        池化层将所有卷积核特征拼接，一般选最大、ｋ大或者分片最大拼接
        tf.reduce_max(conv, reduction_indices=[1], name='gmp')

    池化层和全连接层可以做droupOut，每层被保留的概率，防止过拟合

    全连接：
        relu激活函数　＋　softMax分类器　得到最终结果
    预测结果：np.argmax(a, axis=1)　返回　行/列　最大值的索引　１：行　０：列 ，主要对比准确率啥的
    score下的softmax_logits直接返回概率（这里只做了二分类
    softmax就是ｎ维的概率，和为１，每列就是每个概率

### 优点：
    TextCNN最大优势网络结构简单 ,在模型网络结构如此简单的情况下，通过引入已经训练好的词向量依旧有很不错的效果，在多项数据数据集上超越benchmark。
    网络结构简单导致参数数目少, 计算量少, 训练速度快，在单机单卡的v100机器上，训练165万数据, 迭代26万步，半个小时左右可以收敛。