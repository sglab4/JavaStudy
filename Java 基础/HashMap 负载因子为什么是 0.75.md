参考 https://segmentfault.com/a/1190000023308658

一般而言，默认负载因子为0.75的时候在时间和空间成本上提供了很好的折衷。太高了可以减少空间开销，但是会增加查找复杂度。我们设置负载因子尽量减少rehash的操作，但是查找元素的也要有性能保证。

这里以二项分布的原理解释如下：

HashMap 的二项分布如下：

- 实验只有2种结果
  - 往hash表put数据可以转换为key是否会碰撞？碰撞就失败，不碰撞就成功。
- 实验相互独立
- 成功的概率都是一样的

假设共有 n 次事件，已经放置了 s 个位置，那么不发生碰撞概率为

![](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121444344.png)

通常这个概率需大于 0.5，那么有

![image-20220112144519875](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121445892.png)

进一步推导，有

![image-20220112144539297](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121445319.png)

![image-20220112144546082](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121445100.png)

![image-20220112144555991](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121445007.png)

令 s = m + 1，有

![image-20220112144621960](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121446978.png)

![image-20220112144634132](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121446151.png)

![image-20220112144639868](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202201121446889.png)

因为原始容量 n 为 16，那么接近于 0.693 的数字有 0.625（5/8），0.75（3/4），0.875（7/8），因此可选用 0.75。