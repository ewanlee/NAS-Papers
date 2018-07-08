这篇论文是NAS的后续工作，主要改进在于设计了一个新的搜索空间NASNet search space，借鉴了Inception
以及Resnet的网络结构。主要实验流程，首先在CIFAR10上进行预训练，接着在ImageNet上进行fine-tune。
同时对于回归任务，使用了在以上任务中提取出的feature结合Faster-RCNN在COCO数据集上进行了detection。

搜索空间可简单概括如下，其整个网络结构中只有两种Cell(一般意义上的layer，这里的layer类似于Inception网络),
即Normal Cell以及Reduction Cell，前者输入与输入的feature map的size相同，后者输出feature map的size
高度和宽度缩小为输入的1/2。然后整个网络都是这两种Cell的交替堆叠，其中每个Cell都可以重复堆叠多次。值得注意的是，
所有同种类的Cell内部结构都是完全相同的。如下图所示：

![architecture](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/arch.png)

这两种Cell均由5个Block组成，每个Block又由五部分组成：
- 输入1 (从前两层layer输出中选择或者从本层前面Block的输出选择或者是输入图片)
- 输入2 (同上)
- 针对输入1的操作
- 针对输入2的操作
- 针对操作之后两个输出的合并操作(elementwise add或者depthwise concat)

具体可选操作如下图所示：

![opera](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/opera.png)

总结起来就是每种Cell共需要5B个softmax输出。论文中同时学习两种Cell的结构，采用的方法是网络设定为10B个softmax层
输出，前五个给Normal Cell，后五个给Reduction Cell，总结如下图：

![control_model](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/control_model.png)

附上最优网络结构：

![best_arcg](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/best_arch.png)

另外比较有意思的一点是论文中的baseline选择为random search算法，最后的效果和PPO只差了一个点左右，作者解释说是因为
搜索空间设计的比较精妙，所以随机策略也能取得很好的效果。

![random_search](http://o7ie0tcjk.bkt.clouddn.com/nas-paper-notes/random_search.png)

在论文附录部分提到了一个比较重要的细节，就是如果操作选择为seperable convelution的时候，需要将这个操作对输入进行两次，
实验结果表明这样设计会使得效果提升很多。除了这一点之外，附录部分还提到了很多实现细节，值得仔细研究。
