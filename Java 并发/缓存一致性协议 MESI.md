参考 https://www.cnblogs.com/gunduzi/p/13590528.html

# 1、概述

由于内存的运行速度和CPU的运行速度相差太多，所以现代计算机CPU都不是直接操作内存，而是直接操作寄存器和高速缓存，如果只有一个CPU这个事情就很简单，但是如果计算机中有多个核，那每个CPU都从主内存中读取了同一个变量，如何保证缓存的一致性，就变得非常麻烦，现在常用的解决办法有两种。

1. **总线锁定**：当某个CPU需要修改某个数据的时候，通过锁住内存总线，使得别的CPU无法访问内存中的数据，从而保证缓存的一致性,但这种实现方式会导致CPU执行效率降低，现在很少被使用。
2. **缓存锁**：当一个CPU要修改缓存中的变量时，会对缓存加锁，同时会通过总线通知别的CPU，让他们的变量副本失效，这样同样可以保证一次只有一个CPU修改变量的值，从而保证缓存一致性。

# 2、高速缓存结构

高速缓存的结构和 jdk 中的 HashMap 的结构有点类似，都是采用数组进行分桶，之后采用拉链法挂到对应的桶上，具体结构如下：

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111291141235.png)

链表中节点名称叫做cache entry,下面看一下cache entry的结构：

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111291141401.png)

其中tag用来定位cache entry,data block用来保存缓存的数据，flag就是重点了，这个标识就是用来标注当前节点的状态，对应下面要介绍的MESI协议的四种状态，分别为 M，E，S，I。

# 3、MESI 协议介绍

MESI是四个单词的首字母缩写，Modified修改,Exclusive独占,Shared共享,Invalid无效，下面就简要介绍一下这四种状态。

M：表示当前CPU的高速缓存中的变量副本是独占的，而且和主存中的变量值不一致，而且别的CPU的flag不可能是这个状态。如果别的CPU想要读取变量的值，不能直接读主内存中的值，而是需要将处于M状态的变量刷新回主内存才可以。

E：表示当前CPU的高速缓存中的变量副本是独占的，别的CPU高速缓存中该变量的副本不能处于该状态，但是，处于E状态的高速缓存变量的值和主内存中的变量值是一致的。

S：处于S状态表示CPU中的变量副本和主存中数据一致，而且多个CPU都可以处于S状态，举例，当多个CPU读取主内存的值的时候高速缓存的flag就处于S状态。

I：表示当前CPU的高速缓存的变量副本处于不合法状态，不可以直接使用，需要从主内存重新读取，flag的初始状态就是I。

# 4、MESI 状态转换

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111291427519.png)

 说明：

- local read(本地读取)：本地cache读取本地cache
- local write(本地写入)：本地cache写入本地cache
- remote read(远端读取)：其他cache读取本地cache
- remote write(远端写入)：其他cache写入本地cache

|       状态        |                         触发本地读取                         |                         触发本地写入                         |                         触发远端读取                         |                         触发远端写入                         |
| :---------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| **M状态（修改）** |             本地cache:M 触发cache:M 其他cache:I              |             本地cache:M 触发cache:M 其他cache:I              | 本地cache:M→E→S 触发cache:I→S 其他cache:I→S 同步主内存后修改为E独享,同步触发、其他cache后本地、触发、其他cache修改为S共享 | 本地cache:M→E→S→I 触发cache:I→S→E→M 其他cache:I→S→I 同步和读取一样,同步完成后触发cache改为M，本地、其他cache改为I |
| **E状态（独享）** |             本地cache:E 触发cache:E 其他cache:I              | 本地cache:E→M 触发cache:E→M 其他cache:I 本地cache变更为M,其他cache状态应当是I（无效） | 本地cache:E→S 触发cache:I→S 其他cache:I→S 当其他cache要读取该数据时，其他、触发、本地cache都被设置为S(共享) | 本地cache:E→S→I 触发cache:I→S→E→M 其他cache:I→S→I 当触发cache修改本地cache独享数据时时，将本地、触发、其他cache修改为S共享.然后触发cache修改为独享，其他、本地cache修改为I（无效），触发cache再修改为M |
|  **S状态(共享)**  |             本地cache:S 触发cache:S 其他cache:S              | 本地cache:S→E→M 触发cache:S→E→M 其他cache:S→I 当本地cache修改时，将本地cache修改为E,其他cache修改为I,然后再将本地cache为M状态 |             本地cache:S 触发cache:S 其他cache:S              | 本地cache:S→I 触发cache：S→E→M 其他cache:S→I 当触发cache要修改本地共享数据时，触发cache修改为E（独享）,本地、其他cache修改为I（无效）,触发cache再次修改为M(修改) |
| **I状态（无效）** | 本地cache:I→S或者I→E 触发cache:I→S或者I →E 其他cache:E、M、I→S、I 本地、触发cache将从I无效修改为S共享或者E独享，其他cache将从E、M、I 变为S或者I |  本地cache:I→S→E→M 触发cache:I→S→E→M 其他cache:M、E、S→S→I   |           既然是本cache是I，其他cache操作与它无关            |           既然是本cache是I，其他cache操作与它无关            |

- 本地cache，可以认为就是其中的一个CPU中的cache
- 触发cache，可以认为是触发了read或者写操作的CPU的cache，如果触发cache就是本地cache，那这两个相同
- 其他cache，除了上面介绍的两个CPU的cache（如果本地cache就是触发cache，就只有一个）其他的CPU的cache

# 5、MESI 消息

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111291429547.png)

# 6、MESI 协议举例

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/202111291429655.png)

说明：图中有两个CPU，主存中有一个变量 X = 0，下面就介绍一个CPU A和CPU B读写X的过程。

1. 初始状态，cache A和cache B中都没有X的副本，CPU A发起read请求向主内存，主内存会向总线发送ReadResponse,之后CPU A将Cache A的状态更改位E，表示独占。
2. CPU B发起Read请求，此时CPU A和CPU B同时嗅探总线，发现主内存的变量X不只有一个副本，此时CPU A将Cache A的状态更改位S，CPU B收到ReadResponse之后，更改状态位S。
3. 假如这时CPU A要修改变量X的值为1，这时CPU A先发起Invalidate请求，当CPU B嗅探该请求之后，会将Cache B的状态更新为I，之后回复Invalidate Acknowledge，当CPU A收到CPU B发送的ack之后，才会更改变量X的值为1，之后更新Cache A的状态为M。
4. 如果此时CPU B要读取变量X，发现自己缓存的状态为I，则会发起Read请求，这时CPU A嗅探到Read请求，会将Cache A中的X的值刷新回主内存，然后会将自己的状态更新为E，之后，CPU A会将X的值同步给CPU B，在之后两者的状态都更新为S。 