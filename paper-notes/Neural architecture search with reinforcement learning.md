这篇论文应该算是用强化学习进行网络结构搜索的鼻祖了（就目前看过的论文看来，欢迎指正），毕竟用800个GPU训练28天这种事情也就Google
能干得出来。

网络结构搜索用强化学习来解决，其主要问题在于如何把结构搜索这个问题应用到强化的结构中，也就是状态空间、动作空间以及reward的定义问题。
论文中定义的方法按我的理解整理如下：
- 状态空间
  对于论文中状态空间的定义其实有两种方式。如果按照论文中的公式来看，其状态转移方程就没有涉及到状态，只有action的转移概率。按照这种
  定义来理解的话，结构搜索这个问题只有一个状态，有种bandit问题的感觉了；另外一种观点是，由于论文采用RNN来进行网络结构的搜索，在每个
  时间步网络以前一步的hidden state作为输入，然后这一步的sofmax产生输出（类似图片描述生成的模式，只用前一步的hidden state作为输入）
  即为action。这样看来，state可以表示为之前所有的action的一个joint distribution
- 动作空间
  论文其实一共搜索了三种网络结构，分别是一般的CNN、带skip connection的CNN（Resnet)以及LSTM。对于这三种结构分别设有不同的动作空间，
  其实也就是不同的网络组成模块，CNN的话就有filter height/width、stride height/width等等。skip connection就在之前的基础上新加
  了一个是否connect到后面layer的二元softmax。至于LSTM则比较有意思，论文设计了一种更通用的类LSTM结构，我就在后面直接贴图好了
- reward
  reward的定义就很自然了，将搜索出的网络在验证集上进行训练，最后的accuracy经过一些预处理后作为reward

所以整个的流程可以表示如下：

![nas_overview](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/nas_overview.png)

对于三种不同网络结构的搜索空间，可以由下面三张图看出来，首先是普通的CNN, 可以看到只有卷积层:

![cnn](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/cnn.png)

然后是带有skip connection的CNN:

![cnn_sc](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/cnn_sc.png)

最后是LSTM：

![lstm](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/lstm.png)

在强化学习算法上，论文选取了带有baseline的REINFORCE算法，baseline是之前架构accuracy的exponential moving average。

在实现的时候为了加速训练，采用了如下的并行架构：

![parallel](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/parallel.png)
