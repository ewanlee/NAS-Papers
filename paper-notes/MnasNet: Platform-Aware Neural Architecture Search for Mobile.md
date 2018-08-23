亮点在于直接使用移动设备的延迟作为reward的一部分来考虑，而不是理论上计算FLOPS

![](http://o7ie0tcjk.bkt.clouddn.com/nz5p2uutf89hn5sk.jpg)

然后搜索空间比较简单（因为是Mobile环境，网络结构减少碎片化可以大幅度提升效率）：

![](http://o7ie0tcjk.bkt.clouddn.com/6yoqj3hpw3k2ubar.jpg)

实验结果：

![](http://o7ie0tcjk.bkt.clouddn.com/emquy8rax7ns3bi9.jpg)

![](http://o7ie0tcjk.bkt.clouddn.com/md5pyebz7yujuzt1.jpg)

可以看到在延迟减少的情况下性能还有所提升。

下面是最终的网络结构：

![](http://o7ie0tcjk.bkt.clouddn.com/lj6yfjv8z6rmfbvn.jpg)

然后是最佳网络结构变种的耗时情况：

![](http://o7ie0tcjk.bkt.clouddn.com/2yol4q23buj4qsx4.jpg)

最后作者关于网络结构有一些结论：
1. 5×5的kernel会更快一些
2. layer的多样性比较重要（现存的network的layers都比较单一）
