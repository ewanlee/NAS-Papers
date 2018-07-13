这篇论文主要借鉴了Net2Net的思想，并将其应用到以强化学习为基础的NAS中。其主要思想是通过应用Net2Net之后，每次进行结构
搜索时都不用重新训练网络，因为其动作空间是在之前的网络上“加宽”或者“加深”，之前训练好的参数直接无损地复制（说复制不太恰当
，具体后面解释）过来，这样可以大幅度地加速训练。

它和NAS还有一个主要区别在于并不是直接用网络结构作为state，而是将每一层网络结构进行编码之后输入到一个Bi-LSTM中得到整个网络
的一个编码，以这个编码作为state。整体框架如下所示：

![](http://o7ie0tcjk.bkt.clouddn.com/8yfe98ufthbs4kr6.jpg)

这个架构中最重要的还是两种Actor Network，分别是Net2Wider以及Net2Deeper，下面分别解释这两种操作。

首先是Net2Wider，这里的加宽对于全连接层来说就是增加维度，对于卷积层来说就是增加filter的个数。以卷积层来举例，假设第i层一共有
l(i)个filter，进行加宽之后变成了m(i)个filter，为了保证网络的输出不变，需要进行的操作就是这m(i)个filter中前l(i)个filter的
参数直接从原来的复制过来，剩余的m(i)-l(i)个filter的参数从原先的l(i)个中随机采样。这还没完，因为这m(i)个filter中有些参数被
复制了多次，因此第(i+1)层的filter相对应的就要除去重复次数，这样才能保证网络输出不变。

Net2Wider Actor的具体结构如下所示：

![](http://o7ie0tcjk.bkt.clouddn.com/wmkyqyc1nv9kmd33.jpg)

这里的softmax分类器是所有层共享的，其首先输出每一层是否要加宽，如果是的话就把该层的维度或者filter个数从当前值增加到预先定义好的
下一个级别，这里的动作空间都是离散的。

对于Net2Deeper操作就要简单些，只要增加一个identity层就可以。Net2Deeper Actor的结构如下：

![](http://o7ie0tcjk.bkt.clouddn.com/094qwgbnpsdqofhq.jpg)

这里的block是以pooling层分隔的。论文中还提出了一个针对DenseNet结构的搜索空间，具体可参看论文。
