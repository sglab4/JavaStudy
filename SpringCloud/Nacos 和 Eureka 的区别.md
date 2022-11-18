Eureka 应用启动时的注册原理

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202208271551121.jpeg)

**Nacos与eureka的共同点**

1. 都支持服务注册和服务拉取
2. 都支持服务提供者心跳方式做健康检测

**Nacos 和 Eureka 区别**

**1、注册实例类型**

Nacos 可注册临时和非临时实例，临时实例（AP），非临时实例（CP）

Eureka 只可注册临时实例

**2、范围不同**

Nacos的阈值是针对某个具体Service的，而不是针对所有服务的；但Eureka的自我保护阈值是针对所有服务的。nacos支持CP和AP两种；eureka只支持AP。nacos使用netty，是长连接；eureka是短连接，定时发送

**3、保护方式不同**

Eureka保护方式：当在短时间内，统计续约失败的比例，如果达到一定阈值，则会触发自我保护的机制，在该机制下，Eureka Server不会剔除任何的微服务，等到正常后，再退出自我保护机制。自我保护开关（eureka.server.enable-self-preservation: false)。

Nacos保护方式：当域名健康实例（Instance)占总服务实例（Instance)的比例小于阈值时，无论实例（Instance)是否健康，都会将这个实例（Instance)返回给客户端。这样做虽然损失了一部分流量，但是保证了集群的剩余健康实例（Instance)能正常工作。

**4、连接方式不同**

nacos支持动态刷新，在控制器（controller）上加＠RefreshScope注解即可，采用Netty连接，是长连接；eureka本身不支持动态刷新，需要配合MQ完成动态刷新，且是短连接，是定时发送。

**Nacos 中的 CAP 模式切换**

AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例

CP模式下则支持注册持久化实例