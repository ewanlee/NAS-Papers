这篇论文是上交大之前那篇EAS的扩展，同样使用了Net2Net的方法，以及强化学习框架，如下图：

![](http://o7ie0tcjk.bkt.clouddn.com/7sxcxccoipqhkkd1.jpg)

当然只是说框架一样，具体的network transformation操作以及network encoding方式还是很不相同的。

作者认为之前提出的EAS的方法的缺点在于Net2Wider以及Net2Deeper两种操作所设计出的网络过于简单，结构不够丰富，没有
办法设计出类似Inception这样的具有多分支的网络结构（而这些结构是当前流行的且效果最好的）。因此作者已多分支结构为基础，
重新设计了net transformation的操作，不再是wider以及deeper了。

如果想要某一层（卷积层或者全连接层）变成多分支的结构，同时在进行transformation之后还保持原有网络的输出，这一点作者
表示还是挺简单的，下面以两分支为例说明：

![](http://o7ie0tcjk.bkt.clouddn.com/whwuli0gxhb9k3j2.jpg)

上图中Add以及Concat表示作者设定的action，意思就是说如果对分支采用这两种merge方式，应当如何保证网络输出一致。我们可以看出，
对于卷积层来说，Add操作只要对原先输入进行复制，然后每一个分支乘以一个平均系数就可以保证了；对于Concat操作，同样复制输入，
只不过每个分支平分输入的channel就可以。至于全连接层就很相似了，只不过Concat操作的话需要对输入进行split。

以上这种结构其实就定义了Path-Level EAS的搜索空间，可以看到很像一棵树，作者是故意这么设计的。这棵树的node代表{identity，split}
与{add， concat}的组合，edge代表我们一般的卷积层、池化层等等。那么动作空间又是什么呢？

![](http://o7ie0tcjk.bkt.clouddn.com/wvaz6fmhlrxz9l1a.jpg)

从上图可以很清晰的解释动作空间。首先我们有一个搜索起点，这个起点非常简单，就是一个卷积层（当然只是为了简化说明），对于这样一个叶节点来说，
对它进行的操作就是进行扩展，让它产生一个叶节点，自己变成节点，这是第一类操作；然后对于这样一个孩子只有一个叶结点的结点，我们对它进行的
action就是分裂成多分支，分支的数量以及最后分支merge的方式都是可选的action，这构成了第二类action，可以看出第a步的操作其实可以分为两步。
继续往下，对于进行a步操作之后的树来说，是一个父亲结点带有两个孩子结点并且这两个孩子结点是叶节点的结构。第b步进行的操作其实是，对于其左孩子，
进行与第a步相同的操作，即扩展该叶节点并分裂，这样就形成了第三幅图。第c步我们看到将identity操作替换成了一个seq 3×3的操作，这就是第三类
action，第三类action一共有7种选择，都是之前论文中比较偏好的结构：

![](http://o7ie0tcjk.bkt.clouddn.com/42sfsyl0sitlhi9f.jpg)

随后就形成了第4幅图。最后将其转化为比较理论的一个形式。所以对于这样一个理论的形式，数据流向是这样的，首先从顶向下，然后自底向上，形成
最后的输出。这样搜索空间以及动作空间都定义好了，其实搜索空间也就是状态空间，但是由于论文对网络结构通过LSTM进行编码，因此搜索空间只能
说是观察空间，编码之后的网络结构才能说是状态空间。

但是由于这次作者是用一棵树来表示一个网络的，不像EAS那样是一个序列，这样的话就不能简单地用Bi-LSTM来进行编码了。所以作者用了Tree-Structed
LSTM来对网络结构进行编码。作者引用的文章里还分了两类，一类叫做有序的LSTM（N-ary Tree-LSTM）以及无序的LSTM（Child-Sum Tree-LSTM），这
两类正好对应着concat操作以及add操作（完美匹配...），这是处理了两种node，对于edge，作者使用了一般的LSTM进行处理，形式化是这样的：

![](http://o7ie0tcjk.bkt.clouddn.com/vw0tppnd4yxoz5u8.jpg)

然后作者为了致敬Bi-LSTM，所以这样的操作进行了两次：

![](http://o7ie0tcjk.bkt.clouddn.com/the5fqicio7no92n.jpg)

这样一来就可以再次应用最开始提到的强化框架进行网络结构搜索了。实验表明结果要优于NASNet以及EAS，同时只需要200个GPU小时就可以了（比之ENAS还是不行）：

![](http://o7ie0tcjk.bkt.clouddn.com/idq76bh1h8y1ooo6.jpg)

最后搜索出来的两个最优结构以及算法伪代码：

![](http://o7ie0tcjk.bkt.clouddn.com/5bik5lcp1vpx450u.jpg)
![](http://o7ie0tcjk.bkt.clouddn.com/2yhwhjmbzn9a43h2.jpg)
![](http://o7ie0tcjk.bkt.clouddn.com/i42tvacnyky4p1ou.jpg)




