# 认识微服务

## 单体架构

**单体架构**：将业务的所有功能集中在一个项目中开发，打成一个包部署。

![image-20220412145334805](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121453844.png)

**优点：**架构简单，部署成本低

**缺点：**耦合度高（维护困难、升级困难）

## 分布式架构

**分布式架构**：根据业务功能对系统做拆分，每个业务功能模块作为独立项目开发，称为一个服务。

![image-20220412145408660](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121454709.png)

**优点：**降低服务耦合，有利于服务升级和拓展

**缺点：**服务调用关系错综复杂

**问题**：

- 服务拆分的粒度如何界定？
- 服务之间如何调用？
- 服务的调用关系如何管理？

## 微服务

微服务的架构特征：

- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责
- 自治：团队独立、技术独立、数据独立，独立部署和交付
- 面向服务：服务提供统一标准的接口，与语言和技术无关
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

![image-20220412145543232](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121455276.png)

其中在 Java 领域最引人注目的就是 SpringCloud 提供的方案了。

国内最著名的是 SpringCloud 和 Dubbo

## 微服务的内容

![image-20220412152300702](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121523776.png)

## 技术栈对比

![image-20220412153135838](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121531915.png)

SpringCloudAlibaba 同时兼容 Dubbo 和 SpringCloud 两种架构

微服务可能遇到的技术栈四种情况：

![image-20220412153308586](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121533629.png)

## SpringCloud

SpringCloud 是目前国内使用最广泛的微服务框架。官网地址：https://spring.io/projects/spring-cloud。

SpringCloud 集成了各种微服务功能组件，并基于 SpringBoot 实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

其中常见的组件包括：

![image-20220412153416028](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121534073.png)

另外，SpringCloud 底层是依赖于 SpringBoot 的，并且有版本的兼容关系，如下：

![image-20220412153604996](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121536034.png)

# 服务拆分

**服务拆分注意事项**

单一职责：不同微服务，不要重复开发相同业务

数据独立：不要访问其它微服务的数据库

面向服务：将自己的业务暴露为接口，供其它微服务调用

![image-20220412153823488](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121538529.png)

在各个微服务中，若一个服务想要访问另一个服务，则可采用 http 访问的方式，Spring 提供了一个 RestTemplate 工具，只需要把它创建出来即可。（即注入 Bean）

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

发送请求，自动序列化为 Java 对象。

```java
@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private RestTemplate restTemplate;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        String url = "http://localhost:8081/user/" + order.getUserId();
        // 有 get 和 post 请求，第一个参数为 url 地址，第二个参数为转换后的 class 对象
        User user = restTemplate.getForObject(url, User.class);
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```

服务提供者：暴露接口给其他服务调用

服务消费者：调用其他微服务的接口

在上面代码的 url 中，我们可以发现调用服务的地址采用硬编码，这在后续的开发中肯定是不理想的，这就需要服务注册中心（Eureka）来帮我们解决这个事情。

# Eureka 注册中心

最广为人知的注册中心就是 Eureka，其结构如下：

Eureka 分为了 Eureka-server（注册中心）、Eureka-Client（客户端），Eureka-Client 又分为了消费者和服务者

![image-20220412170654983](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121706082.png)

**order-service 如何得知 user-service 实例地址？**

- user-service 服务实例启动后，将自己的信息注册到 eureka-server(Eureka服务端)，叫做**服务注册**
- eureka-server 保存服务名称到服务实例地址列表的映射关系
- order-service 根据服务名称，拉取实例地址列表，这个叫**服务发现**或服务拉取

**order-service 如何从多个 user-service 实例中选择具体的实例？**

- order-service从实例列表中利用**负载均衡算法**选中一个实例地址，向该实例地址发起远程调用

**order-service 如何得知某个 user-service 实例是否依然健康，是不是已经宕机？**

- user-service 会**每隔一段时间(默认30秒)向 eureka-server 发起请求**，报告自己状态，称为**心跳**
- 当超过一定时间没有发送心跳时，eureka-server 会认为微服务实例故障，将该实例从服务列表中剔除
- order-service 拉取服务时，就能将故障实例排除了

## 搭建注册中心

1、配置依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

2、main 函数

```java
// Eureka 的开关
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

3、配置文件

```yml
server:
  port: 10086 # Eureka 的服务端口
spring:
  application:
    name: eureka-server # Eureka 的服务名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka  # Eureka 自己的地址信息
```

其中 `default-zone` 是因为前面配置类开启了注册中心所需要配置的 eureka 的**地址信息**，因为 eureka 本身也是一个微服务，这里也要将自己注册进来，当后面 eureka **集群**时，这里就可以填写多个，使用 “,” 隔开。

启动完成后，访问 http://localhost:10086/

![image-20220412172506900](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121725967.png)

## 服务注册

1、在客户端的 pom 文件中加入依赖

在需要添加到 Eureka 的客户端中的 pom 文件，引入 SpringCloud 为 eureka 提供的 starter 依赖，注意这里是用 **client**

在本例中，在 user-service 的 pom 文件中添加

```xml
<!--Eureka-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2、在启动类上添加注解：`@EnableEurekaClient`

3、配置 application.yml 文件

```yml
spring:
  application:
      #name：orderservice
    name: userservice
eureka:
  client:
    service-url: 
      defaultZone: http:127.0.0.1:10086/eureka
```

在 userservice 和 orderservice 中配置完成后，启动 3 个项目，访问 http://localhost:10086/

![image-20220412185902933](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204121859980.png)

## 服务拉取

> 在 order-service 中完成服务拉取，然后通过负载均衡挑选一个服务，实现远程调用

步骤如下：

1、首先给 `RestTemplate` 这个 Bean 添加一个 `@LoadBalanced` **注解**，用于开启**负载均衡**。（后面会讲）

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

2、修改 OrderService 访问的url路径，用**服务名**代替ip和端口号：

![image-20220413100516029](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131005109.png)

# Ribbon 负载均衡

我们添加了 `@LoadBalanced` 注解，即可实现负载均衡功能，这是什么原理呢？

**SpringCloud 底层提供了一个名为 Ribbon 的组件，来实现负载均衡功能。**

![image-20220413101326286](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131013366.png)

## 源码跟踪

为什么我们只输入了 service 名称就可以访问了呢？为什么不需要获取ip和端口，这显然有人帮我们根据 service 名称，获取到了服务实例的ip和端口。它就是`LoadBalancerInterceptor`，这个类会在对 RestTemplate 的请求进行拦截，然后从 Eureka 根据服务 id 获取服务列表，随后利用负载均衡算法得到真实的服务地址信息，替换服务 id。

我们进行源码跟踪：

![image-20220413101434457](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131014555.png)

这里的 `intercept()` 方法，拦截了用户的 HttpRequest 请求，然后做了几件事：

- `request.getURI()`：获取请求uri，即 http://user-service/user/8
- `originalUri.getHost()`：获取uri路径的主机名，其实就是服务id `user-service`
- `this.loadBalancer.execute()`：处理服务id，和用户请求

这里的 `this.loadBalancer` 是 `LoadBalancerClient` 对象

继续跟入 `execute()` 方法：

![image-20220413101638723](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131016795.png)

- `getLoadBalancer(serviceId)`：根据服务id获取 `ILoadBalancer`，而 `ILoadBalancer` 会拿着服务 id 去 eureka 中获取服务列表。
- `getServer(loadBalancer)`：利用内置的负载均衡算法，从服务列表中选择一个。在图中**可以看到获取了8082端口的服务**

可以看到获取服务时，通过一个 `getServer()` 方法来做负载均衡:

![image-20220413101731307](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131017354.png)

我们继续跟入：

![image-20220413101751652](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131017704.png)

继续跟踪源码 `chooseServer()` 方法，发现这么一段代码：

![image-20220413101804830](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131018882.png)

我们看看这个 `rule` 是谁：

![image-20220413101818837](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131018895.png)

这里的 rule 默认值是一个 `RoundRobinRule`（轮询） ，还可以有 `RandomRule` （随机）

## 流程总结

SpringCloud Ribbon 底层采用了一个拦截器，拦截了 RestTemplate 发出的请求，对地址做了修改。

基本流程如下：

- `LoadBalancerInterceptor` 拦截我们的 `RestTemplate` 请求 http://userservice/user/1
- `RibbonLoadBalancerClient` 会从请求url中获取服务名称，也就是 user-service
- `DynamicServerListLoadBalancer` 根据 user-service 到 eureka 拉取服务列表
- eureka 返回列表，localhost:8081、localhost:8082
- `IRule` 利用内置负载均衡规则，从列表中选择一个，例如 localhost:8081
- `RibbonLoadBalancerClient` 修改请求地址，用 localhost:8081 替代 userservice，得到 http://localhost:8081/user/1，发起真实请求

![image-20220413102016132](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131020219.png)

## 负载均衡策略

负载均衡的规则都定义在 IRule 接口中，而 IRule 有很多不同的实现类：

![image-20220413102826563](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131028653.png)

不同规则的含义如下：

| **内置负载均衡规则类**    | **规则描述**                                                 |
| :------------------------ | :----------------------------------------------------------- |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略：（1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。 （2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule 规则的客户端也会将其忽略。并发连接数的上限，可以由客户端设置。 |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| **ZoneAvoidanceRule**     | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。 |
| BestAvailableRule         | 忽略那些短路的服务器，并选择并发数较低的服务器。             |
| RandomRule                | 随机选择一个可用的服务器。                                   |
| RetryRule                 | 重试机制的选择逻辑                                           |

默认的实现就是 `ZoneAvoidanceRule`，**是一种轮询方案**。

## 自定义策略

通过定义 IRule 实现可以修改负载均衡规则，有两种方式：

1、代码方式：在配置类中定义一个新的 IRule

![image-20220413103246225](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131032265.png)

2、配置文件方式：在 order-service 的 application.yml 文件中，添加新的配置也可以修改规则：

```yml
userservice: # 给需要调用的微服务配置负载均衡规则，orderservice服务去调用userservice服务
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则 
```

二者区别在于第一种方式是全局修改，所有服务者都会采用修改后的负载均衡方式，而第二种可以修改部分服务者的负载均衡策略。

**注意**：一般用默认的负载均衡规则，不做修改。

## 饥饿加载

当我们启动 orderservice，第一次访问时，时间消耗会大很多，这是因为 Ribbon 懒加载的机制，即第一次访问时才会去创建 LoadBalanceClient，拉取集群地址，所以请求时间会很长。

![image-20220413103844717](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131038765.png)

而饥饿加载则会在项目启动时创建 LoadBalanceClient，降低第一次访问的耗时，通过下面配置开启饥饿加载：

```yml
ribbon:
  eager-load:
    enabled: true
    clients: 
       - userservice # 项目启动时直接去拉取userservice的集群
```

# Nacos 注册中心

SpringCloudAlibaba 推出了一个名为 Nacos 的注册中心，在国外也有大量的使用。

![image-20220413104716939](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131047018.png)

解压启动 Nacos，在 bin 目录下打开 cmd

```cmd
startup.cmd -m standalone
```

或者双击 startup.cmd

启动完成后，访问：http://localhost:8848/nacos/

![image-20220413104802077](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131048133.png)

## 服务注册

Nacos 本身就是一个 SprintBoot 项目，这点从启动的控制台打印就可以看出来，所以就不再需要去额外搭建一个像 Eureka 的注册中心。

1、引入依赖

在 cloud-demo 父工程中引入 SpringCloudAlibaba 的依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

然后在 user-service 和 order-service 中的pom文件中引入 nacos-discovery 依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2、配置 nacos 服务地址

在 user-service 和 order-service 的 application.yml 中添加 nacos 地址：

```yml
spring:
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
```

项目重新启动后，可以看到三个服务都被注册进了 Nacos

![image-20220413113429603](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131134656.png)

## 分级存储模型

一个**服务**可以有多个**实例**，例如我们的 user-service，可以有:

- 127.0.0.1:8081
- 127.0.0.1:8082
- 127.0.0.1:8083

假如这些实例分布于全国各地的不同机房，例如：

- 127.0.0.1:8081，在上海机房
- 127.0.0.1:8082，在上海机房
- 127.0.0.1:8083，在杭州机房

Nacos就将同一机房内的实例，划分为一个**集群**。

![image-20220413114951407](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131149505.png)

微服务互相访问时，应该尽可能访问同集群实例，因为本地访问速度更快。**当本集群内不可用时，才访问其它集群。**例如：杭州机房内的 order-service 应该优先访问同机房的 user-service。

![image-20220413115004631](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131150704.png)

## 配置集群

接下来我们给 user-service **配置集群**

修改 user-service 的 application.yml 文件，添加集群配置：

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称，可自定义
```

重启两个 user-service 实例后，我们再去启动一个上海集群的实例，将 application.yml 配置文件中的集群名称改为 SH，或者在配置中更改集群名称

```yml
-Dserver.port=8083 -Dspring.cloud.nacos.discovery.cluster-name=SH
```

![image-20220413115141588](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131151641.png)

查看 nacos 控制台：

![image-20220413115155012](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204131151093.png)

## NacosRule

Ribbon的默认实现 `ZoneAvoidanceRule` 并不能实现根据同集群优先来实现负载均衡，需要将规则改成 **NacosRule **。我们是用 orderservice 调用 userservice，所以在 orderservice 配置规则。

第一种方法：

```java
@Bean
public IRule iRule(){
    //默认为轮询规则，这里自定义为随机规则
    return new NacosRule();
}
```

第二种方法：

```yaml
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule #负载均衡规则 
```

对 orderservice 配置集群：

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

启动四个服务：

- orderservice - HZ
- userservice - HZ
- userservice1 - HZ
- userservice2 - SH

访问地址：http://localhost:8080/order/101

在访问中我们发现，只有同在一个 HZ 集群下的 userservice、userservice1 会被调用，并且是**随机**的。

我们试着把 userservice、userservice2 停掉。依旧可以访问。

在 userservice3 控制台可以看到发出跨集群查询的警告，因为 orderservice 本身是在 HZ 集群的，这波 HZ 集群没有了 userservice，就会去别的集群找。

![image-20220415153507318](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151535481.png)

NacosRule 源码分析 https://blog.csdn.net/weixin_38019299/article/details/121663362

## 权重配置

实际部署中会出现这样的场景：

服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。但默认情况下 NacosRule 是同集群内随机挑选，不会考虑机器的性能问题。

因此，Nacos 提供了**权重配置来控制访问频率**，0~1 之间，权重越大则访问频率越高，权重修改为 0，则该实例永远不会被访问。

在 Nacos 控制台，找到 user-service 的实例列表，点击编辑，即可修改权重。

![image-20220415154328041](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151543113.png)

![image-20220415154432122](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151544174.png)

另外，在服务升级的时候，有一种较好的方案：我们也可以通过调整权重来进行平滑升级，例如：先把 userservice 权重调节为 0，让用户先流向 userservice2、userservice3，升级 userservice后，再把权重从 0 调到 0.1，让一部分用户先体验，用户体验稳定后就可以往上调权重了。

## 环境隔离

Nacos 提供了 namespace 来实现环境隔离功能。

- Nacos 中可以有多个 namespace
- namespace 下可以有 group、service 等
- 不同 namespace 之间**相互隔离**，例如不同 namespace 的服务互相不可见

![image-20220415155552823](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151555947.png)

### 创建 namespace

默认情况下，所有 service、data、group 都在同一个 namespace，名为 public(保留空间)：

![image-20220415155614848](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151556901.png)

我们可以点击页面新增按钮，添加一个 namespace：

![image-20220415155625614](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151556664.png)

![image-20220415155634963](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151556014.png)

### 配置namespace

给微服务配置 namespace 只能通过修改配置文件来实现。

修改 order-service 的 application.yml 文件：

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ
        namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间ID
```

重启 order-service 后，访问控制台。

**public**

![image-20220415155735378](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151557418.png)

**dev**

![image-20220415155744292](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151557332.png)

此时访问 order-service，因为 namespace 不同，会导致找不到 userservice，控制台会报错：

![image-20220415155759059](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151557125.png)

## Nacos 注册原理分析

![image-20220413104716939](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207101844507.png)

Nacos 的服务实例分为两种类型：

- **临时实例**：如果实例宕机超过一定时间，会从服务列表剔除，**默认的类型**。
- 非临时实例：如果实例宕机，不会从服务列表剔除，也可以叫永久实例。

和临时实例相比，非临时实例对服务器压力比较大。

配置是否为临时示例：

```yaml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false # 设置为非临时实例
```

另外，Nacos 集群**默认采用 AP 方式(可用性)**，当集群中存在非临时实例时，**采用 CP 模式(一致性)**；而 Eureka 采用 AP 方式，不可切换。（这里说的是 CAP 原理，后面会写到）

# Nacos 配置中心

Nacos 除了可以做注册中心，同样可以做配置管理来使用。

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。**我们需要一种统一配置管理方案，可以集中管理所有实例的配置。**

![image-20220415161429062](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151614129.png)

Nacos 一方面可以将配置集中管理，另一方可以在配置变更时，及时通知微服务，**实现配置的热更新。**

## 创建配置

在 Nacos 控制面板中添加配置文件

![image-20220415161812479](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151618561.png)

![image-20220415161823802](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151618871.png)

**注意：**项目的核心配置，需要**热更新**的配置才有放到 nacos 管理的必要。**基本不会变更**的一些配置(例如数据库连接)还是保存在微服务本地比较好。

## 拉取配置

首先我们需要了解 Nacos 读取配置文件的环节是在哪一步，在没加入 Nacos 配置之前，获取配置是这样：

![image-20220415161955239](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151619297.png)

加入 Nacos 配置，它的读取是在 application.yml 之前的：

![image-20220415162013138](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151620207.png)

这时候如果把 nacos 地址放在 application.yml 中，显然是不合适的，**Nacos 就无法根据地址去获取配置了。**

因此，nacos 地址必须放在优先级最高的 bootstrap.yml 文件。

![image-20220415162055553](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151620640.png)

步骤如下：

**1、引入 nacos-config 依赖**

首先，在 user-service 服务中，引入 nacos-config 的客户端依赖：

```xml
<!--nacos 的配置管理依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**2、添加 bootstrap.yml**

然后，在 user-service 中添加一个 bootstrap.yml 文件，内容如下：

```yaml
spring:
  application:
    name: userservice # 服务名称
  profiles:
    active: dev #开发环境，这里是dev 
  cloud:
    nacos:
      server-addr: localhost:8848 # Nacos地址
      config:
        file-extension: yaml # 文件后缀名
```

根据 spring.cloud.nacos.server-addr 获取 nacos地址，再根据`${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`作为文件id，来读取配置。

在这个例子例中，就是去读取 `userservice-dev.yaml`

![image-20220415163716409](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151637470.png)

使用代码来验证是否拉取成功

在 user-service 中的 UserController 中添加业务逻辑，读取 pattern.dateformat 配置并使用：

![image-20220415165006930](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151650986.png)

启动服务后，访问：http://localhost:8081/user/now

![image-20220415165028486](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151650521.png)

## 配置热更新

我们最终的目的，是修改 nacos 中的配置后，微服务中无需重启即可让配置生效，也就是**配置热更新**。

有两种方式：1. 用 `@Value` 读取配置时，搭配 `@RefreshScope`；2. 直接用 `@ConfigurationProperties` 读取配置

优先使用 `@ConfigurationProperties` 读取配置

### @RefreshScope

方式一：在 `@Value` 注入的变量所在类上添加注解 `@RefreshScope`

![image-20220415165649869](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151656916.png)

### @ConfigurationProperties

方式二：使用 `@ConfigurationProperties` 注解读取配置文件，就不需要加 `@RefreshScope` 注解。

在 user-service 服务中，添加一个 PatternProperties 类，读取 `patterrn.dateformat` 属性

```java
@Data
@Component
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    public String dateformat;
}
```

![image-20220415170737825](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151707863.png)

```java
@Autowired
private PatternProperties patternProperties;

@GetMapping("now2")
public String now2(){
    //格式化时间
    return LocalDateTime.now().format(DateTimeFormatter.ofPattern(patternProperties.dateformat));
}
```

## 配置共享

其实在服务启动时，nacos 会读取多个配置文件，例如：

- `[spring.application.name]-[spring.profiles.active].yaml`，例如：userservice-dev.yaml
- `[spring.application.name].yaml`，例如：userservice.yaml

这里的 `[spring.application.name].yaml` 不包含环境，**因此可以被多个环境共享**。

**添加一个环境共享配置**

我们在 nacos 中添加一个 userservice.yaml 文件：

![image-20220415172008875](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151720926.png)



对 PatternProperties 继续修改

![image-20220415172043894](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151720932.png)

在 Controller 中添加验证方法

```java
@GetMapping("prop")
public PatternProperties prop() {
    return properties;
}
```

访问后，得到

![image-20220415172128221](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151721260.png)

### 配置优先级

当 nacos、服务本地同时**出现相同属性时**，优先级有高低之分。

![image-20220415172221479](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151722535.png)

![image-20220415181115848](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151811934.png)

## Nacos 集群

![image-20220415193234193](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204151932258.png)

# Feign 远程调用

我们以前利用 RestTemplate 发起远程调用的代码：

![image-20220418095432390](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204180954456.png)

- 代码可读性差，编程体验不统一
- 参数复杂URL难以维护

Feign 是一个声明式的 http 客户端，其作用就是帮助我们**优雅的实现 http 请求的发送**，解决上面提到的问题。

Feign 默认实现了 Ribbon 负载均衡。

## Feign 使用

**1、引入依赖**

我们在 order-service 引入 feign 依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**2、添加注解**

在 order-service 启动类添加注解开启 Feign

```java
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

`@FeignClient("userservice")`：其中参数填写的是微服务名

`@GetMapping("/user/{id}")`：其中参数填写的是请求路径

这个客户端主要是基于 SpringMVC 的注解 `@GetMapping` 来声明远程调用的信息

Feign 可以帮助我们发送 http 请求，无需自己使用 RestTemplate 来发送了。

经测试，可根据订单 id 访问到用户 id 信息，并且由于 feign 中已经集成了 Ribbon 负载均衡策略，因此会自动进行负载均衡。

![image-20220418101055506](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181010542.png)

feign 的 start 文件如下：

![image-20220418101034199](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181010241.png)

## 自定义配置

Feign 可以支持很多的自定义配置，如下表所示：

| 类型                   | 作用             | 说明                                                   |
| :--------------------- | :--------------- | :----------------------------------------------------- |
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign.Contract         | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign.Retryer          | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的 @Bean 覆盖默认 Bean 即可，通常修改的是日志级别。

两种配置方法，可以基于配置文件修改，也可以基于 Java 代码修改。

1、配置文件

修改 order 中的配置文件

针对单个服务：

```yaml
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

针对所有服务：

```yaml
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

而日志的级别分为四种：

- NONE：不记录任何日志信息，这是默认值。
- BASIC：仅记录请求的方法，URL以及响应状态码和执行时间
- HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息
- FULL：记录所有请求和响应的明细，包括头信息、请求体、元数据

2、基于 Java 代码修改

先声明一个类，然后声明一个 Logger.Level 的对象

```java
public class DefaultFeignConfiguration {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```

如果要**全局生效**，将其放到启动类的 `@EnableFeignClients` 这个注解中：

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration.class) // 全局有效
```

如果是**局部生效**，则把它放到对应的 `@FeignClient` 这个注解中：

```java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration.class) 
```

## 性能优化

Feign 底层发起 http 请求，依赖于其它的框架。其底层客户端实现有：

- **URLConnection**：默认实现，不支持连接池
- **Apache HttpClient** ：支持连接池
- **OKHttp**：支持连接池

因此提高 Feign 性能的主要手段就是使用**连接池**代替默认的 URLConnection

另外，日志级别应该尽量用 basic/none，可以有效提高性能。

**这里我们用 Apache 的HttpClient来演示连接池。**

步骤：

1、引入依赖

在 order-service 的 pom 文件中引入 HttpClient 依赖

```xml
<!--httpClient的依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

2、配置连接池

```yaml
feign:
  client:
    config:
      default: # default全局的配置
        loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息
  httpclient:
    enabled: true # 开启feign对HttpClient的支持
    max-connections: 200 # 最大的连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
```

在 FeignClientFactoryBean 中的 loadBalance 方法中打断点

![image-20220418102954111](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181029218.png)

Debug 方式启动 order-service 服务，可以看到这里的 client，底层就是 Apache HttpClient

![image-20220418103009139](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181030214.png)

## 最佳实践

![image-20220418105253512](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181052562.png)

两种最佳实践方案：

**1、继承方式**

一样的代码可以通过继承来共享：

1）定义一个 API 接口，利用定义方法，并基于 SpringMVC 注解做声明

2）Feign 客户端、Controller 都集成该接口

![image-20220418105536952](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181055026.png)

优点

- 简单
- 实现了代码共享

缺点

- 服务提供方、服务消费方紧耦合
- 参数列表中的注解映射并不会继承，因此 Controller 中必须再次声明方法、参数列表、注解

**2、抽取方式**

将 FeignClient 抽取为独立模块，并且把接口有关的 POJO、默认的 Feign 配置都放到这个模块中，提供给所有消费者使用。

例如：将 UserClient、User、Feign 的默认配置都抽取到一个 feign-api 包中，所有微服务引用该依赖包，即可直接使用。

![image-20220418105615387](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181056476.png)

接下来我们就用该方法在代码中实现

首先创建一个 module，命名为 feign-api

![image-20220418105918583](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181059632.png)

在 feign-api 中然后引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

order-service中的 UserClient、User 都复制到 feign-api 项目中

![image-20220418105951101](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181059161.png)

**在order-service中使用 feign-api**

首先，删除 order-service 中的 UserClient、User

在 order-service 中引入 feign-api

```xml
<dependency>
    <groupId>com.xn2001.feign</groupId>
    <artifactId>feign-api</artifactId>
    <version>1.0</version>
</dependency>
```

```java
@EnableFeignClients(basePackages = "com.xn2001.feign.clients")
```

## Feign 远程调用原理

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171745379.webp)

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171739530.webp)

**1、基于面向接口的动态代理方式生成实现类**

使用 jdk 动态代理基于面向接口的动态代理方式生成实现类，将请求调用委托到动态代理实现类

![image-20220717174041114](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171740203.png)

**2、根据接口类的注解声明规则，解析出底层的 MethodHandler**

**3、基于 RequestBean，动态生成Request**

根据传入的Bean对象和注解信息，从中提取出相应的值，来构造Http Request 对象

**4、使用 Encoder 将 request bean转换成 Http报文正文（消息解析和转码逻辑）**

Feign 最终会将请求转换成Http 消息发送出去，传入的请求对象最终会解析成消息体

![image-20220717174319317](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171743425.png)

**5、拦截器负责对请求和返回进行装饰处理**

在请求转换的过程中，Feign 抽象出来了拦截器接口，用于用户自定义对请求的操作，比如，如果希望Http消息传递过程中被压缩，可以定义一个请求拦截器。

**6、记录日志**

**7、基于重试器发送 http 请求**

Feign 内置了一个重试器，当HTTP请求出现IO异常时，Feign会有一个最大尝试次数发送请求

**8、发送 http 请求**

Feign 真正发送HTTP请求是委托给 feign.Client 来做的

Feign 默认底层通过JDK 的 java.net.HttpURLConnection 实现了feign.Client接口类,**在每次发送请求的时候，都会创建新的HttpURLConnection 链接**，这也就是为什么默认情况下Feign的性能很差的原因。可以通过拓展该接口，使用Apache HttpClient 或者OkHttp3等基于连接池的高性能Http客户端。

## Feign 性能优化

**1、GZIP压缩**

gzip是一种数据格式，采用deflate算法压缩数据。当Gzip压缩到一个纯文本数据时，可以减少70％以上的数据大小。

gzip作用：网络数据经过压缩后实际上降低了网络传输的字节数，最明显的好处就是可以加快网页加载的速度。

**2、替换为HttpClient客户端（使用HTTP连接池提供性能）**

Feign的HTTP客户端支持3种框架，分别是；HttpURLConnection、HttpClient、OKHttp。Feign中默认使用HttpURLConnection。

- HttpURLConnection是JDK自带的HTTP客户端技术，并不支持连接池，如果要实现连接池的机制，还需要自己来管理连接对象。对于网络请求这种底层相对复杂的操作，如果有可用的其他方案，也没有必要自己去管理连接对象。
- Apache提供的**HttpClient**框架相比传统JDK自带的HttpURLConnection，它封装了访问http的请求头，参数，内容体，响应等等；它不仅使客户端发送HTTP请求变得容易，而且也方便了开发人员测试接口（基于Http协议的），即提高了开发的效率，也方便提高代码的健壮性；另外高并发大量的请求网络的时候，**还是用“HTTP连接池”提升吞吐量**。
- OKHttp是一个处理网络请求的开源项目,是安卓端最火热的轻量级框架。**OKHttp拥有共享Socket,减少对服务器的请求次数，通过连接池,减少了请求延迟等技术特点**。

# Gateway 网关

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等响应式编程和事件流技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Gateway 网关是我们服务的守门神，**所有微服务的统一入口。**

网关的**核心功能特性**：

- 请求路由
- 权限控制
- 限流

![image-20220418110937026](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181109109.png)

**权限控制**：网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**路由和负载均衡**：一切请求都必须先经过 gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**限流**：当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大。

在 SpringCloud 中网关的实现包括两种：

- gateway
- zuul

Zuul 是基于 Servlet 实现，属于阻塞式编程。而 Spring Cloud Gateway 则是基于 Spring5 中提供的WebFlux，属于响应式编程的实现，具备更好的性能。

## 入门使用

1. 创建 SpringBoot 工程 gateway，引入网关依赖
2. 编写启动类
3. 编写基础配置和路由规则
4. 启动网关服务进行测试

1、新建模块 gateway，引入依赖

```xml
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2、创建 application.yml 文件，内容如下：

```yaml
server:
  port: 10010
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: localhost:8848  # nacos 地址
    gateway:
      routes:
        - id: userservice  # 路由标识，必须唯一
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
        - id: orderservice
          uri: lb://orderservice
          predicates:
            - Path=/order/**
```

我们将符合`Path` 规则的一切请求，都代理到 `uri`参数指定的地址。

上面的例子中，我们将 `/user/**` 开头的请求，代理到 `lb://userservice`，其中 lb 是负载均衡(LoadBalance)，根据服务名拉取服务列表，实现负载均衡。

重启网关，访问 http://localhost:10010/order/101 时，符合 `/order/**` 规则，请求转发到 uri：http://orderservice/order/101

![image-20220418112908007](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181129057.png)

## 流程图

![image-20220418113002856](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181130938.png)

网关只起到规则判断和转发作用，路由配置包括：

1. 路由id：路由的唯一标示
2. 路由目标（uri）：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. 路由断言（predicates）：判断路由的规则
4. 路由过滤器（filters）：对请求或响应做处理

## 断言工厂

我们在配置文件中写的断言规则只是字符串，这些字符串会被 Predicate Factory 读取并处理，转变为路由判断的条件。

例如 `Path=/user/**` 是按照路径匹配，这个规则是由

`org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory` 类来处理的，像这样的断言工厂在 Spring Cloud Gateway 还有十几个

| 名称       | 说明                           | 示例                                                         |
| :--------- | :----------------------------- | :----------------------------------------------------------- |
| After      | 是某个时间点后的请求           | - After=2037-01-20T17:42:47.789-07:00[America/Denver]        |
| Before     | 是某个时间点之前的请求         | - Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]        |
| Between    | 是某两个时间点之前的请求       | - Between=2037-01-20T17:42:47.789-07:00[America/Denver], 2037-01-21T17:42:47.789-07:00[America/Denver] |
| Cookie     | 请求必须包含某些cookie         | - Cookie=chocolate, ch.p                                     |
| Header     | 请求必须包含某些header         | - Header=X-Request-Id, \d+                                   |
| Host       | 请求必须是访问某个host（域名） | - Host=`**.somehost.org`, `**.anotherhost.org`               |
| Method     | 请求方式必须是指定方式         | - Method=GET,POST                                            |
| Path       | 请求路径必须符合指定规则       | - Path=/red/{segment},/blue/**                               |
| Query      | 请求参数必须包含指定参数       | - Query=name, Jack或者- Query=name                           |
| RemoteAddr | 请求者的ip必须是指定范围       | - RemoteAddr=192.168.1.1/24                                  |
| Weight     | 权重处理                       |                                                              |

一般的，我们只需要掌握 Path，加上官方文档的例子，就可以应对各种工作场景了。

```yaml
predicates:
  - Path=/order/**
  - After=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]
```

上面规则要求访问满足 2031-04-13 15:14:47 后的请求才可访问，很明显不满足，将不会进行转发

## 过滤器

### 过滤器工厂

GatewayFilter 是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理。

![image-20220418114548923](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181145012.png)

Spring 提供了31种不同的路由过滤器工厂。

> 官方文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

| **名称**             | **说明**                     |
| :------------------- | :--------------------------- |
| AddRequestHeader     | 给当前请求添加一个请求头     |
| RemoveRequestHeader  | 移除请求中的一个请求头       |
| AddResponseHeader    | 给响应结果中添加一个响应头   |
| RemoveResponseHeader | 从响应结果中移除有一个响应头 |
| RequestRateLimiter   | 限制请求的流量               |

**需求**：给所有进入 userservice 的请求添加一个请求头：`sign=Kurisu is time`

只需要修改 gateway 服务的 application.yml文件，添加路由过滤即可。

```yaml
spring:
  cloud:
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
          filters:
            - AddRequestHeader=sign, Kurisu is time  # 响应头 - 响应头内容
```

如何验证，我们修改 userservice 中的一个接口

```java
@GetMapping("/{id}")
public User queryById(@PathVariable("id") Long id, @RequestHeader(value = "sign", required = false) String sign) {
    log.warn(sign);
    return userService.queryById(id);
}
```

重启两个服务，访问：http://localhost:10010/user/1

可以看到控制台打印出了这个请求头

![image-20220418143211067](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204181432130.png)

当然，Gateway 也是有**全局过滤器**的，如果要**对所有的路由都生效**，则可以将过滤器工厂写到 default-filters 下：

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - AddRequestHeader=sign, Kurisu is time # 添加请求头
```

若全局过滤器和局部过滤器都进行了配置，则**优先局部过滤器**。

### 全局过滤器

上面介绍的过滤器工厂，网关提供了 31 种，但每一种过滤器的作用都是固定的。**如果我们希望拦截请求，做自己的业务逻辑则没办法实现**。

全局过滤器的作用也是处理一切进入网关的请求和微服务响应，**与 GatewayFilter 的作用一样**。区别在于 GlobalFilter 的逻辑可以**写代码来自定义规则**；而 GatewayFilter 通过配置定义，处理逻辑是固定的。

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1、获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();
        // 2、获取 authorization 参数
        String auth = params.getFirst("authorization");
        // 3、校验
        if ("admin".equals(auth)) {
            // 4、是，放行
            return chain.filter(exchange);
        }
        // 5、否，拦截
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### 过滤器顺序

请求进入网关会碰到三类过滤器：DefaultFilter、当前路由的过滤器、GlobalFilter；

请求路由后，会将三者合并到一个过滤器链（集合）中，排序后依次执行每个过滤器.

![image-20220419102357811](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191023938.png)

排序的规则是什么呢？

- 每一个过滤器都必须指定一个 int 类型的 order 值，**order 值越小，优先级越高，执行顺序越靠前**。

- GlobalFilter 通过实现 Ordered 接口，或者使用 @Order 注解来指定 order 值，由我们自己指定。

- 路由过滤器和 defaultFilter 的 order 由 Spring 指定，默认是按照声明顺序从1递增。

  - 路由过滤器

    ```yaml
    filters:
      - AddRequestHeader=sign, Kurisu is time  # 响应头 - 响应头内容     // 1
      - AddRequestHeader=sign, Kurisu is time  # 响应头 - 响应头内容     // 2
      - AddRequestHeader=sign, Kurisu is time  # 响应头 - 响应头内容     // 3
    ```

  - defaultFilter 过滤器

    ```yaml
    default-filters:
      - AddRequestHeader=sign, KoitoYuu is time  # 响应头 - 响应头内容     // 1
      - AddxxxtHeader=sign, KoitoYuu is time  # 响应头 - 响应头内容        // 2
      - AddxxxestHeader=sign, KoitoYuu is time  # 响应头 - 响应头内容      // 3
    ```

    二者是分开 order 的

- 当过滤器的 order 值一样时，**会按照 defaultFilter > 路由过滤器 > GlobalFilter 的顺序执行。**

## 跨域问题

跨域：

- 域名不同
- 域名相同，端口号不同

跨域问题：**浏览器**禁止请求发起者和服务端发生跨域的 **ajax 请求**，请求被浏览器拦截。

解决方案：CORS

```yaml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]': # 对哪些请求进行跨域拦截
            allowedOrigins: # 允许哪些网站的跨域请求 allowedOrigins: “*” 允许所有网站
              - "http://localhost:8090"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```

## 限流

**Gateway 的限流原理**

参考 https://blog.csdn.net/qq_41432730/article/details/122022749

Spring Cloud Gateway官方就提供了RequestRateLimiterGatewayFilterFactory这个类，使用Redis和lua脚本实现了令牌桶的方式。具体实现逻辑在RequestRateLimiterGatewayFilterFactory类中，lua脚本在如下图所示的文件夹中：

![image-20211219124620273](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209121622461.png)

接下来我们使用redis作为令牌桶来实现限流：

步骤如下：

- 引入依赖
- 创建限流标识
- 配置限流速率

1、引入依赖

```xml
<!--基于Redis实现限流-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

2、添加限流标识

限流通常要根据某一个参数值作为参考依据来限流，比如每个IP每秒钟只能访问2次，此时是以IP为参考依据，我们创建根据IP限流的对象，该对象需要实现接口`KeyResolver`：

```java
public class IpKeyResolver implements KeyResolver {

    /***
     * 根据IP限流
     * @param exchange
     * @return
     */
    @Override
    public Mono<String> resolve(ServerWebExchange exchange) {
        return Mono.just(exchange.getRequest().getRemoteAddress().getAddress().getHostAddress());
    }
}
```

指定限流方式：我们如果需要根据IP限流，则需要将`IpKeyResolver`的实例交给Spring容器管理。直接将其放在启动类下即可。

```java
@Bean(name = "ipKeyResolver")
public KeyResolver userIpKeyResolver() {
    return new IpKeyResolver();
}
```

3、在配置文件中配置限流

在 application.yml 核心配置文件中添加RequestRateLimiter配置：

```yml
spring:
	cloud:
        gateway:
          routes:
            #商品服务
            - id: goods_route
              uri: lb://mall-goods
              predicates:
                - Path=/mall/brand/**
              filters:
                - StripPrefix=1
                # 指定过滤器
                - name: RequestRateLimiter
                  args:
                    # 指定限流标识
                    key-resolver: '#{@ipKeyResolver}'
                    # 速率限流
                    redis-rate-limiter.replenishRate: 1
                    # 能容纳的并发流量总数
                    redis-rate-limiter.burstCapacity: 2
```

在上面的配置文件，配置了RequestRateLimiter的限流过滤器，该过滤器需要配置三个参数：

- burstCapacity，令牌桶总容量。
- replenishRate，令牌桶每秒填充平均速率。
- key-resolver，用于限流的键的解析器的 Bean 对象的名字。它使用 SpEL 表达式根据#{@beanName}从 Spring 容器中获取 Bean 对象。

此时我们请求限流的接口不停刷新进行测试，效果如下：

![image-20211219125339926](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209121622277.png)

# Docker

## 什么是 Docker

大型项目运行项目很复杂，部署时经常会遇到一些问题：

- 依赖关系复杂，容易出现兼容性问题
- 开发测试环境有差异

Docker 如何解决兼容性问题：

- 将应用的函数库，依赖进行打包
- 将每个应用放到一个隔离的容器去运行，避免相互干扰

 在不同操作系统下，Docker 如何运行，内核和硬件交互，提供操作硬件的指令，系统应用将内核指令封装成为函数，供程序员调用，用户基于操作系统函数实现功能。

若二者系统应用不同，那么将会报错，Docker 如何解决？

- Docker 将用户程序和所需的系统函数库一起打包
- Docker 运行不同操作系统时，基于打包的函数库，借助 Linux 内核来运行

![image-20220419104434489](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191044582.png)

因此 Docker 打包好的程序包可运行在任意 Linux 内核下。

## 初识 Docker

Docker 与虚拟机

虚拟机是在操作系统中模拟硬件设备，然后运行另一个操作系统。

![image-20220419104956758](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191049844.png)

## Docker 架构

镜像：Docker 将应用程序所需依赖、函数库、配置环境等打包，称为镜像，镜像为只读，不可被污染。

容器：镜像中的应用程序运行所形成的进程就是容器，Docker 只会对容器做隔离，对外不可见。

镜像共享可通过 DockerHub

Docker 是一个 CS 架构的程序，分为两部分：

- 服务端：Docker 守护线程，负责处理 Docker 指令
- 客户端：通过命令向 Docker 服务端发送指令。

![image-20220419105814075](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191058160.png)

## 安装 Docker

在 VMware 中安装 CentOS7，需要先联网，联网参考 https://www.jianshu.com/p/a516a28b5f04，然后再安装 Docker

## Docker 基本操作

`sudo service docker start`：启动 docker

`sudo service docker restart`：重启 docker

`sudo service docker stop`：关闭 docker

`sudo systemctl start docker`：开机启动 docker

`docker -v`：查看 docker 版本

`docker ps -a`：查看所有的镜像（包括运行和停止）

`docker rm 删除指定的容器`：删除容器

`docker restart 指定容器`：重启指定容器

镜像由两部分组成：[repository]:[tag]

若没有指定 tag，则默认为最新版本镜像

![image-20220419163236338](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191632391.png)

镜像操作命令：

![image-20220419163448228](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191634305.png)

可使用 `docker -push --help` 查看帮助文档

容器相关命令：

![image-20220419165040834](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191650898.png)

创建一个 nginx 容器：

```shell
docker run --name containerName -p 80:80 -d nginx
```

解读如下：

- docker run：创建并运行一个容器
- --name：给容器命名
- -p：将宿主机端口和容器端口进行映射，左侧为宿主机端口，右侧为容器端口
- -d：后台运行
- nginx：镜像名称（没有版本号为最新版本）

![image-20220419165752617](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191657675.png)

在启动后，直接访问

![image-20220419170349604](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191703669.png)

## 数据卷

容器和数据存在的耦合问题：

1. 不便于修改
2. 数据不可用
3. 升级维护困难

数据卷：是一个虚拟目录，指向宿主机文件系统中的某个目录

![image-20220419192117210](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191921277.png)

数据卷命令：

`docker volume create html` ：创建名为 html 的数据卷

`docker volume ls`：查看所有数据卷

`docker volume inspect html`：查看 html 数据卷存放的内存地址

`docker volume prune`：删除未使用的数据卷

`docker volume rm html`：删除 html 数据卷

查看容器对应元数据 `docker inspect 容器id`，可以在 Mounts节点查看建立的数据卷信息。

![image-20220627103506541](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202206271035635.png)

> 数据卷挂载文件目录

第一步：`docker run --name mynginx -v html:/usr/share/nginx/html -p 80:80 -d nginx`  用于在创建的 mynginx 容器上加上数据卷，其中 -v 表示添加数据卷，将宿主机挂载到容器中，前者为主机地址，后者为容器文件夹地址

第二步：进入 html 数据卷所在位置，并修改内容

1. docker volume inspect html  # 查找 html 数据卷所在位置
2. cd /var/lib/docker/volumes/html/_data # 进入该目录
3. vi index.html # 修改文件内容

> 运行 MySQL 容器，将宿主机目录挂载到容器中

```shell
docker run \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=123 \  # MySQL 的必要配置（密码）
  -p 3306:3306 \
  -v /tmp/mysql/conf/hmy.cnf:/etc/mysql/conf.d/hmy.cnf # 配置文件
  -v /tmp/mysql/data:/var/lib/mysql # 数据文件
  -d mysql:5.7.25
```

## 自定义镜像

### 镜像结构

镜像是将应用程序所需要的系统函数库、环境、配置、依赖打包。

镜像结构：

![image-20220419195806010](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204191958114.png)

镜像是一个分层结构，每层称为一个 layer

- BaseImage 层：包括基本的系统函数库、环境变量、文件系统
- Entrypoint：入口，镜像启动命令
- 其他：在 BaseImage 基础上添加依赖、安装程序、完成整个应用的安装和配置

### Dockerfile

Dockerfile 是一个文本文件，其中包含一个个指令，用来说明执行什么操作来构建镜像，每个指令都会形成一个 layer

常用命令：

![image-20220419200248249](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204192002314.png)

> 基于 ubuntu 构建 Java 项目

```shell
# 指定基础镜像
FROM ubuntu:16.04
# 配置环境变量，JDK的安装目录
ENV JAVA_DIR=/usr/local

# 拷贝jdk和java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar

# 安装JDK
RUN cd $JAVA_DIR \
 && tar -xf ./jdk8.tar.gz \
 && mv ./jdk1.8.0_144 ./java8

# 配置环境变量
ENV JAVA_HOME=$JAVA_DIR/java8
ENV PATH=$PATH:$JAVA_HOME/bin

# 暴露端口
EXPOSE 8090
# 入口，java项目的启动命令
ENTRYPOINT java -jar /tmp/app.jar
```

上面可发现，除了 `COPY ./docker-demo.jar /tmp/app.jar` 在不同 Java 项目不同外，其余都是配置 Java 的 jdk 环境和配置环境变量，那么，可将相同的内容构建为一个镜像，由此减少配置步骤。

> 基于 java:8-alpine 构建 Java 项目

```shell
# 指定基础镜像
FROM java:8-alpine

# java项目的包
COPY ./jdk8.tar.gz $JAVA_DIR/
COPY ./docker-demo.jar /tmp/app.jar

# 暴露端口
EXPOSE 8090
# 入口，java项目的启动命令
ENTRYPOINT java -jar /tmp/app.jar
```

总结：

1. Dockerfile 本质上是一个文件，用来描述镜像的构建过程
2. Docker 第一行为 from，指定基础镜像
3. 基础镜像可以为基本的操作系统，例如 ubuntu，也可以为他人构建好的镜像，例如 java:8-alpine

## DockerCompose

DockerCompose 可以基于 Compose 文件快速部署微服务，通过指令定义集群中的每个容器如何执行。分为 version 和 services 两部分，services 中用来描述各个服务启动时所需要的镜像、环境、端口号、数据卷挂载等内容。

![image-20220420110820906](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201108981.png)

```yaml
version: "3.2"

services:
  nacos: # nacos 微服务
    image: nacos/nacos-server
    environment:
      MODE: standalone
    ports:
      - "8848:8848"
  mysql: # MySQL 微服务
    image: mysql:5.7.25
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "$PWD/mysql/data:/var/lib/mysql"
      - "$PWD/mysql/conf:/etc/mysql/conf.d/"
  userservice: # userserivce 微服务
    build: ./user-service
  orderservice: # orderservice 微服务
    build: ./order-service
  gateway: # gateway 微服务
    build: ./gateway
    ports:
      - "10010:10010"
```

## 镜像仓库

分为公共和私有两种

- 公共仓库：例如 Docker 官方的 DockerHub，国内的例如网易云镜像仓库，阿里云镜像仓库等
- 除了公共仓库外，也可自己构造私有仓库 Docker Registry

总结：

1. 推送本地镜像到仓库前必须重命名镜像，以镜像仓库地址为前缀
2. 镜像仓库推送前需要将仓库地址配置到 docker 服务的 daemon.json 中，被 docker 信任
3. 推送使用 docker push 命令
4. 拉取使用 docker pull 命令

# RabbitMQ

## 同步异步通信

**微服务间通信有同步和异步两种方式**

同步通信：就像打电话，需要实时响应。

异步通信：就像发邮件，不需要马上回复。

![image-20220420150133255](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201501325.png)

两种方式各有优劣，打电话可以立即得到响应，但是你却不能跟多个人同时通话。发送邮件可以同时与多个人收发邮件，但是往往响应会有延迟。

我们之前学习的 **Feign 调用**就属于**同步方式**，虽然调用可以实时得到结果，但存在下面的问题：

![image-20220420150311367](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201503451.png)

**同步调用的优点：**

- 时效性较强，可以立即得到结果

**同步调用的缺点：**

- 耦合度高
- 性能和吞吐能力下降
- 有额外的资源消耗
- 有级联失败问题

**异步调用：**

在事件模式中，支付服务是事件发布者（publisher），在支付完成后只需要发布一个支付成功的事件（event），事件中带上订单id。订单服务和物流服务是事件订阅者（Consumer），订阅支付成功的事件，监听到事件后完成自己业务即可。

为了解除事件发布者与订阅者之间的耦合，两者并不是直接通信，而是有一个中间人（Broker）。发布者发布事件到Broker，不关心谁来订阅事件。订阅者从Broker订阅事件，不关心谁发来的消息。

![image-20220420150821576](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201508644.png)

![image-20220420150840592](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201508643.png)

Broker 是一个像数据总线一样的东西，所有的服务要接收数据和发送数据都发到这个总线上，这个总线就像协议一样，让服务间的通讯变得标准和可控。

**异步调用优点：**

- 吞吐量提升：无需等待订阅者处理完成，响应更快速
- 故障隔离：服务没有直接调用，不存在级联失败问题
- 调用间没有阻塞，不会造成无效的资源占用
- 耦合度极低，每个服务都可以灵活插拔，可替换
- 流量削峰：不管发布事件的流量波动多大，都由 Broker 接收，订阅者可以按照自己的速度去处理事件

**异步调用缺点：**

- 架构复杂了，业务没有明显的流程线，不好管理
- 需要依赖于 Broker 的可靠、安全、性能

## MQ消息队列

MQ，中文是消息队列（MessageQueue），字面来看就是存放消息的队列，也就是事件驱动架构中的 Broker

比较常见的 MQ 实现：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka

几种常见MQ的对比：

|            | **RabbitMQ**            | **ActiveMQ**                      | **RocketMQ** | **Kafka**  |
| :--------- | :---------------------- | :-------------------------------- | :----------- | :--------- |
| 公司/社区  | Rabbit                  | Apache                            | 阿里         | Apache     |
| 开发语言   | Erlang                  | Java                              | Java         | Scala&Java |
| 协议支持   | AMQP、XMPP、SMTP、STOMP | OpenWire、STOMP、REST、XMPP、AMQP | 自定义协议   | 自定义协议 |
| 可用性     | 高                      | 一般                              | 高           | 高         |
| 单机吞吐量 | 一般                    | 差                                | 高           | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                            | 毫秒级       | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                              | 高           | 一般       |

各消息队列的适应场景：

**1.Kafka**

Kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，适合产生大量数据的互联网服务的数据收集业务。大型公司建议可以选用，如果有日志采集功能，肯定是首选kafka了。

**2.RocketMQ**

天生为金融互联网领域而生，对于可靠性要求很高的场景，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ在稳定性上可能更值得信赖，这些业务场景在阿里双11已经经历了多次考验，如果你的业务有上述并发场景，建议可以选择RocketMQ。

**3.RabbitMQ**

RabbitMQ :结合erlang语言本身的并发优势，性能较好，社区活跃度也比较高，但是不利于做二次开发和维护。不过，RabbitMQ的社区十分活跃，可以解决开发过程中遇到的bug。如果你的数据量没有那么大，小公司优先选择功能比较完备的RabbitMQ。

以 RabbitMQ 为例，我们在 Centos7 虚拟机中使用 Docker 来安装

1、在线拉取镜像

```sh
docker pull rabbitmq:3-management
```

2、执行下面的命令来运行MQ容器

```sh
docker run \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=123456 \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

启动成功后访问地址：http://192.168.197.128:15672/

**RabbitMQ 中的一些角色**

- publisher：生产者
- consumer：消费者
- exchange：交换机，负责消息路由
- queue：队列，存储消息
- virtualHost：虚拟主机，**隔离不同租户**的 exchange、queue、消息的隔离

**MQ 的基本结构**

![image-20220420154036273](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201540415.png)

## 消息模式

RabbitMQ 的五种消息模式：

RabbitMQ 官方提供了 5 个不同的 Demo 示例，对应了不同的消息模型。

![image-20220420154133085](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201541196.png)

**Hello World 模型**

![image-20220420154147678](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201541743.png)

官方的 HelloWorld 是基于最基础的消息队列模型来实现的，只包括三个角色：

- publisher：消息发布者，将消息发送到队列queue
- queue：消息队列，负责接受并缓存消息
- consumer：订阅队列，处理队列中的消息

### publisher实现

- 建立连接
- 创建 channel
- 声明队列
- 发送消息
- 关闭连接和 channel

```java
public class PublisherTest {
    @Test
    public void testSendMessage() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("192.168.197.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("123456");
        // 1.2.建立连接
        Connection connection = factory.newConnection();
        // 2.创建通道Channel
        Channel channel = connection.createChannel();
        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);
        // 4.发送消息
        String message = "Hello RabbitMQ！";
        channel.basicPublish("", queueName, null, message.getBytes());
        System.out.println("发送消息成功：" + message);
        // 5.关闭通道和连接
        channel.close();
        connection.close();
    }
}
```

### consumer实现

- 建立连接
- 创建 channel
- 声明队列（需要注意的是，consumer 接收 publisher 发送的消息时，声明的队列名应该相同）
- 订阅消息

```java
public class ConsumerTest {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("192.168.197.128");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("123456");
        // 1.2.建立连接
        Connection connection = factory.newConnection();
        // 2.创建通道Channel
        Channel channel = connection.createChannel();
        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);
        // 4.订阅消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) {
                // 5.处理消息
                String message = new String(body);
                System.out.println("接收到消息：" + message);
            }
        });
        System.out.println("等待接收消息中");
    }
}
```

## SpringAMQP

SpringAMQP 是基于 RabbitMQ 封装的一套模板，并且还利用 SpringBoot 对其实现了自动装配，使用起来非常方便。

SpringAMQP 的官方地址：https://spring.io/projects/spring-amqp

![image-20220420161521307](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201615382.png)

![image-20220420161530674](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201615747.png)

SpringAMQP 提供了三个功能：

- 自动声明队列、交换机及其绑定关系
- 基于注解的监听器模式，异步接收消息
- 封装了 RabbitTemplate 工具，用于发送消息

### BasicQueue

第一步：添加依赖

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

第二步：在 publisher、consumer 服务中的 application.yml 中添加配置

```yaml
spring:
  rabbitmq:
    host: 192.168.197.128 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: 123456 # 密码
```

第三步，在 consumer 服务中添加监听队列

```java
@Component
public class RabbitMQListener {
    @RabbitListener(queues = "simple.queue") // 指定队列名称
    public void listenSimpleQueueMessage(String msg) throws InterruptedException {
        System.out.println("消费者接收到消息：【" + msg + "】");
    }
}
```

第四步，在 publisher 服务中添加发送消息的测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAmqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    public void testSimpleQueue() {
        // 队列名称
        String queueName = "simple.queue";
        // 消息
        String message = "你好啊！";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```

注意：消息一旦消费就会从消息队列中删除，RabbitMQ 没有消息回溯功能

### WorkQueue

Work queues，也被称为（Task queues），任务模型。简单来说就是**让多个消费者绑定到一个队列，共同消费队列中的消息**。

![image-20220420171954584](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204201719652.png)

当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。

此时就可以使用 work 模型，多个消费者共同处理消息处理，速度就能大大提高了。

我们循环发送，模拟大量消息堆积现象，在 publisher 服务中的 SpringAmqpTest 类中添加一个测试方法：

```java
/**
 * workQueue
 * 向队列中不停发送消息，模拟消息堆积。
 */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "simple.queue";
    // 消息
    String message = "hello, message_";
    for (int i = 0; i < 50; i++) {
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20);
    }
}
```

**消息接收**

要模拟多个消费者绑定同一个队列，我们在 consumer 服务的 RabbitMQListener 中添加2个新的方法：

```java
@RabbitListener(queues = "simple.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20);
}

@RabbitListener(queues = "simple.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200);
}
```

启动 ConsumerApplication 后，在执行 publisher 服务中刚刚编写的发送测试方法 testWorkQueue

可以看到消费者1很快完成了自己的25条消息。消费者2却在缓慢的处理自己的25条消息。

这是因为 RabbitMQ 默认有一个 **消息预取机制**，将消息 **平均分配给每个消费者**，并没有考虑到消费者的处理能力，我们需要的是去限制每次只能取一条消息，解决这个问题。

在 spring 中有一个简单的配置，设置 prefetch 属性，我们修改 consumer 服务的 application.yml 文件，添加配置

```yml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```

Work 模型的使用：

- 多个消费者绑定到一个队列，同一条消息只会被一个消费者处理
- 通过设置 prefetch 来控制消费者预取的消息数量

### 发布/订阅

![image-20220420202548947](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202025019.png)

图中可以看到，在订阅模型中，多了一个 exchange 角色，而且过程略有变化

- Publisher：生产者，也就是要发送消息的程序，但是不再发送到队列中，**而是发给 exchange（交换机）**
- Consumer：消费者，与以前一样，订阅队列，没有变化
- Queue：消息队列也与以前一样，接收消息、缓存消息
- Exchange：交换机，一方面，接收生产者发送的消息；另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于 Exchange 的类型。Exchange 有以下3种类型：
  - Fanout：广播，将消息交给所有绑定到交换机的队列
  - Direct：定向，把消息交给符合指定 routing key 的队列
  - Topic：通配符，把消息交给符合 routing pattern（路由模式） 的队列

**Exchange（交换机）只负责转发消息，不具备存储消息的能力**，因此如果没有任何队列与 Exchange 绑定，或者没有符合路由规则的队列，那么消息会丢失！

#### Fanout

![image-20220420204431765](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202044830.png)

在广播模式下，消息发送流程是这样的：

- 可以有多个队列
- 每个队列都要绑定到 Exchange（交换机）
- 生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定
- 交换机把消息发送给绑定过的所有队列
- 订阅队列的消费者都能拿到消息

接下里我们用 SpringAMQP 来简单实现 FanoutExchange

1. 在 consumer 服务中，利用代码声明队列、交换机，并将两者绑定
2. 在 consumer 服务中，编写两个消费者方法，分别监听 fanout.queue1 和 fanout.queue2
3. 在 publisher 中编写测试方法，向 fanout发送消息

**声明队列和交换机**

Spring 提供了一个接口 Exchange，来表示所有不同类型的交换机。

![image-20220420204534017](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202045089.png)

在 consumer 中创建一个类，声明队列、交换机、**绑定对象 Binding**

```java
@Configuration
public class FanoutConfig {

    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        // 交换机名称
        return new FanoutExchange("xn2001.fanout");
    }

    /**
     * 声明队列
     * @return Queue
     */
    @Bean
    public Queue fanoutQueue1(){
        // 队列名称
        return new Queue("fanout.queue1");
    }
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(FanoutExchange fanoutExchange, Queue fanoutQueue1){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
    @Bean
    public Binding bindingQueue2(FanoutExchange fanoutExchange, Queue fanoutQueue2){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }

}
```

![image-20220420204652068](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202046162.png)

在 consumer 服务的 SpringRabbitListener 中添加三个方法，作为消费者

```java
@RabbitListener(queues = "fanout.queue1")
public void listenFanoutQueue1(String msg) throws InterruptedException {
    System.out.println("接收到fanout.queue1的消息：【" + msg + "】" + LocalTime.now());
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String msg) throws InterruptedException {
    System.err.println("接收到fanout.queue2的消息：【" + msg + "】" + LocalTime.now());
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(value = "fanout.queue3"),
    exchange = @Exchange(value = "xn2001.fanout",type = "fanout")
))
public void listenFanoutQueue3(String msg) {
    System.out.println("接收到fanout.queue3的消息：【" + msg + "】" + LocalTime.now());
}
```

在 publisher 服务的 SpringAmqpTest 类中添加测试方法

```java
/**
 * fanout
 * 向交换机发送消息
 */
@Test
public void testFanoutExchange() {
    // 交换机名称
    String exchangeName = "xn2001.fanout";
    // 消息
    String message = "hello, everybody!";
    rabbitTemplate.convertAndSend(exchangeName, "", message);
}
```

![image-20220420204804875](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202048922.png)

交换机的作用是什么？

- 接收 publisher 发送的消息
- 将消息按照规则路由到与之绑定的队列
- 不能缓存消息，路由失败，消息丢失
- FanoutExchange 会将所有消息路由到每个绑定的队列

#### Direct

在 Fanout 模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到 DirectExchange

![image-20220420212639538](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202126636.png)

在 Direct 模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`（路由key）
- 消息的发送方向 Exchange发送消息时，也必须指定消息的 `RoutingKey`。
- Exchange 不再把消息交给每一个绑定的队列，而是根据消息的`Routing Key`进行判断，只有队列的`Routingkey` 与消息的 `Routing key`完全一致，才会接收到消息

通过之前 `@Bean` 的方式来声明比较麻烦，其实我们也是可以直接通过 `@RabbitListener` 注解来完成的，代码如下：

消费者：

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "direct.queue4"),
        exchange = @Exchange(value = "xn2001.direct", type = ExchangeTypes.DIRECT),
        key = {"a", "c"}
))
public void listenFanoutQueue6(String msg) {
    System.out.println("接收到fanout.queue3的消息：【" + msg + "】" + LocalTime.now());
}

@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "direct.queue5"),
        exchange = @Exchange(value = "xn2001.direct", type = ExchangeTypes.DIRECT),
        key = {"a","b"}
))
public void listenDirectQueue7(String msg){
    System.out.println("接收到direct.queue1的消息：【" + msg + "】" + LocalTime.now());
}
```

生产者：

```java
@Test
public void testDirectExchangeToA() {
    // 交换机名称
    String exchangeName = "xn2001.direct";
    // 消息
    String message = "hello, i am direct to a!";
    rabbitTemplate.convertAndSend(exchangeName, "a", message);
}
```

当 bindingKey="a" 时，"direct.queue4" 和 "direct.queue5" 队列都可接收到交换机 exchange 转发的消息；当 bindingKey="c" 时，只有 "direct.queue4" 可接收到 exchange 转发的消息。

![image-20220420213436773](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202134841.png)

#### Topic

`Topic `与 `Direct` 很类似，都是可以根据`RoutingKey`把消息路由到不同的队列。只不过`Topic `类型可以让队列在绑定`Routing key` 的时候使用**通配符**

通配符规则：

`#`：匹配一个或多个词

`*`：只能匹配一个词

例如：

```txt
item.#`：能够匹配`item.spu.insert` 或者 `item.spu
item.*`：只能匹配`item.spu
```

![image-20220420214007038](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202140144.png)

消费者：

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "topic.queue1"),
        exchange = @Exchange(value = "k.topic", type = ExchangeTypes.TOPIC),
        key = {"china.#"}
))
public void listenTopicQueue1(String msg){
    System.out.println("接收到topic.queue1的消息：【" + msg + "】" + LocalTime.now());
}

@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "topic.queue2"),
        exchange = @Exchange(value = "k.topic", type = ExchangeTypes.TOPIC),
        key = {"#.news"}
))
public void listenTopicQueue2(String msg){
    System.out.println("接收到topic.queue2的消息：【" + msg + "】" + LocalTime.now());
}
```

生产者：

```java
@Test
public void testTopicExchange() {
    // 交换机名称
    String exchangeName = "k.topic";
    // 消息
    String message1 = "hello, i am topic form china.news";
    String message2 = "hello, i am topic form china.weather";
    rabbitTemplate.convertAndSend(exchangeName, "china.news", message1);
    rabbitTemplate.convertAndSend(exchangeName, "china.weather", message2);
}
```

## 消息转换器

Spring 会把你发送的消息序列化为字节发送给 MQ，接收消息的时候，还会把字节反序列化为 Java 对象。

**默认情况下 Spring 采用的序列化方式是 JDK 序列化。**

我们可以去试一下效果

```java
@Test
public void testSendMap()  {
    // 准备消息
    Map<String,Object> msg = new HashMap<>();
    msg.put("name", "Jack");
    msg.put("age", 21);
    // 发送消息
    rabbitTemplate.convertAndSend("object.queue", msg);
}
```

![image-20220420220212281](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202202347.png)

JDK序列化存在下列问题：

- 数据体积过大
- 有安全漏洞
- 可读性差

我们推荐可以使用 JSON 来序列化

在 publisher 和 consumer 两个服务中都引入依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

配置消息转换器。

在各自的启动类中添加一个 Bean 即可

```java
@Bean
public MessageConverter jsonMessageConverter(){
    return new Jackson2JsonMessageConverter();
}
```

![image-20220420220533392](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202204202205468.png)

# Elasticsearch

## 初识 Elasticsearch

Elasticsearch 是一款非常强大的开源搜索引擎，具备非常多强大功能，可以帮助我们从海量数据中快速找到需要的内容，可以用来实现搜索、日志统计、分析、系统监控等功能。

![image-20220429163515091](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291635176.png)

elasticsearch 的底层实现为 Luceus，为 Java 语言的搜索引擎类库。

## 倒排索引

**首先，倒排索引的概念是对于 MySQL 这样的正向索引而言的。**

那么我们先讲何为正向索引。例如给下表（tb_goods）中的 id 创建索引

![image-20220429164721128](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291647230.png)

如果是根据 id 查询，那么直接走索引，查询速度非常快。但如果是基于 title 做模糊查询，只能是逐行扫描数据，流程如下：

1. 用户搜索数据，条件是 title 符合 `"%手机%"`
2. 逐行获取数据，比如 id 为 1 的数据
3. 判断数据中的 title 是否符合用户搜索条件
4. 如果符合则放入结果集，不符合则丢弃。然后回到步骤1

逐行扫描，也就是全表扫描，随着数据量增加，其查询效率也会越来越低。

而倒排索引中有两个非常重要的概念：

- 文档（`Document`）：在倒排索引中，用来搜索的数据，每一条数据就是一个文档。
- 词条（`Term`）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条

对某个字段创建倒排索引，实现如下：

- 将每一个文档的数据利用算法分词，得到一个个词条
- 创建表，每行数据包括词条、词条所在文档 id、位置等信息
- 因为词条唯一性，可以给词条创建索引，例如 hash 表结构索引

![image-20220429164901754](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291649830.png)

**倒排索引的搜索流程**如下（以搜索"华为手机"为例）

1. 用户输入条件`"华为手机"`进行搜索
   1. 对用户输入内容**分词**，得到词条：`华为`、`手机`

2. 拿着词条在倒排索引中查找，可以得到包含词条的文档 id 有 1、2、3
3. 拿着文档 id 到正向索引中查找具体文档

**虽然要先查询倒排索引，再查询正向索引，但是词条和文档 id 都建立了索引，查询速度非常快！无需全表扫描。**

![image-20220429164926053](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291649140.png)

**正向索引**是最传统的，根据 id 索引的方式。但根据词条查询时，必须先逐条获取每个文档，然后判断文档中是否包含所需要的词条，是**根据文档找词条的过程**

**倒排索引**则相反，是先找到用户要搜索的词条，根据得到的文档 id 获取该文档。是**根据词条找文档的过程**

## 文档和字段

elasticsearch 是面向**文档（Document）**存储的，文档数据会被序列化为 json 格式后存储在 elasticsearch

![image-20220429165513645](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291655719.png)

而 JSON 文档中往往包含很多的**字段（Field）**，类似于数据库中的列。

## 索引和映射

**索引（Index）**，就是相同类型的文档的集合，类似于数据库中的表

![image-20220429165553030](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291655092.png)

数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。

**mysql 与 elasticsearch**

| **MySQL** | **Elasticsearch** | **说明**                                                     |
| :-------- | :---------------- | :----------------------------------------------------------- |
| Table     | Index             | 索引(index)，就是文档的集合，类似数据库的表(table)           |
| Row       | Document          | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是JSON格式 |
| Column    | Field             | 字段（Field），就是JSON文档中的字段，类似数据库中的列（Column） |
| Schema    | Mapping           | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
| SQL       | DSL               | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

- Mysql：擅长事务类型操作，可以确保数据的安全和一致性
- Elasticsearch：擅长海量数据的搜索、分析、计算

因此在企业中，往往是两者结合使用：

- 对安全性要求较高的写操作，使用 MySQL 实现
- 对查询性能要求较高的搜索需求，使用 ELasticsearch 实现
- 两者再基于某种方式，实现数据的同步，保证一致性

![image-20220429165709113](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291657183.png)

## 安装 Elasticsearch

首先创建一个网络

```shell
docker network create es-net 
```

将 es.tar 复制到 linux 中，然后加载

```shell
docker load -i es.tar
```

安装

```sh
docker run -d \
--name es \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
-v es-data:/usr/share/elasticsearch/data \
-v es-plugins:/usr/share/elasticsearch/plugins \
--privileged \
--network es-net \
-p 9200:9200 \
-p 9300:9300 \
elasticsearch:7.12.1
```

命令解释：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为 es-net 的网络中
- `-p 9200:9200`：端口映射配置

访问地址：[http://192.168.197.128:9200](http://192.168.197.128:9200/) 即可看到 elasticsearch 的响应结果

![image-20220429191609026](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204291916090.png)

## 安装 kibana

kibana 可以给我们提供一个 elasticsearch 的可视化界面。

和之前一样，使用 docker 开启 kibana。首先将 kibana.tar 导入到 linux 中，然后将其加载到容器中

```shell
docker load -i kibana.tar
```

最后调用如下命令启动 kibana

```shell
docker run -d \
    --name kibana \
    -e ELASTICSEARCH_HOSTS=http://es:9200 \
    --network=es-net \
    -p 5601:5601  \
    kibana:7.12.1
```

- `--network es-net` ：加入一个名为 es-net 的网络中，与 elasticsearch 在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置 elasticsearch 的地址，因为 kibana 已经与 elasticsearch 在一个网络，因此可以用容器名直接访问 elasticsearch
- `-p 5601:5601`：端口映射配置

访问 http://192.168.197.128:5601/

![image-20220430165520036](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301655149.png)

![image-20220430165540440](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301655509.png)



![image-20220430165609654](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301656754.png)

## 安装 IK 分词器

es 默认提供的分词器对于中文的支持较差，因此采用 ik 分词器。

安装插件需要知道 elasticsearch 的 plugins 目录位置，而我们用了数据卷挂载，因此需要查看 elasticsearch 的数据卷目录，通过下面命令查看

```shell
docker volume inspect es-plugins
```

结果

```txt
[
    {
        "CreatedAt": "2022-04-30T16:39:35+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]
```

说明 plugins 目录被挂载到了 `/var/lib/docker/volumes/es-plugins/_data `这个目录中

![image-20220430170602385](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301706465.png)

重启容器

```shell
# 4、重启容器
docker restart es

# 查看es日志
docker logs -f es
```

IK分词器包含两种模式：

- `ik_smart`：智能切分，粗粒度
- `ik_max_word`：最细切分，细粒度

在 kibana 中的 Dev Tools 中进行测试

![image-20220430170439339](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301704412.png)

![image-20220430170814335](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301708432.png)

## 扩展词词典

在上面的IK分词器我们可以随着热点词来扩展，可以自己添加，例如最新的网络热词，也可以设置停止对一些词汇进行分词，例如敏感词汇。

打开IK分词器 config 目录是 `IKAnalyzer.cfg.xml`，添加一个文件名，我们以 `ext.dic` 文件名为例。

![image-20220430172306474](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301723531.png)

由于配置文件的目录下没有 ext.dic，因此自己创建一个 ext.dic 文件，然后添加需要拓展的词汇

![image-20220430172357837](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301723903.png)

对 stopword.dic 中的停止分词的词汇进行拓展

![image-20220430172442514](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301724581.png)

重启 es 容器，在 kibana 中进行测试

```dsl
GET /_analyze
{
  "analyzer": "ik_smart",
  "text": "白嫖视频是不好的哦"
}
```

分词结果

```txt
{
  "tokens" : [
    {
      "token" : "白嫖",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "视频",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "是",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "不好",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    }
  ]
}
```

## 索引库操作

### Mapping属性映射

索引库就类似数据库表，**mapping 映射就类似表的结构**

我们要向 es 中存储数据，必须先创建“库”和“表”

mapping 是对索引库中文档的约束，常见的 mapping 属性包括：

- type：字段数据类型，常见的简单类型有：
  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  - 数值：long、integer、short、byte、double、float、
  - 布尔：boolean
  - 日期：date
  - 对象：object
- **index：是否创建索引，默认为 true**
- analyzer：使用哪种分词器
- properties：该字段的子字段

我们以需要存储下面的 JSON 为例来讲解

```json
{
    "age": 21,
    "weight": 52.1,
    "isMarried": false,
    "info": "钟老师真菜",
    "email": "jialna@qq.com",
    "score": [99.1, 99.5, 98.9],
    "name": {
        "firstName": "湖",
        "lastName": "心"
    }
}
```

首先对应的每个字段映射（mapping）情况如下：

- age：类型为 integer；参与搜索，index 为 true；无需分词器
- weight：类型为 float；参与搜索，index 为 true；无需分词器
- isMarried：类型为boolean；参与搜索，index 为 true；无需分词器
- info：类型为字符串，需要分词，因此是 text；参与搜索，index为true；分词器可以用 ik_smart
- email：类型为字符串，但是不需要分词，因此是 keyword；不参与搜索，index 为 false；无需分词器
- score：虽然是数组，**但是我们只看元素的类型**，类型为 float；参与搜索，index 为 true；无需分词器
- name：类型为 object，需要定义多个子属性
  - name.firstName：类型为字符串，不需要分词，keyword；参与搜索，index 为 true；无需分词器
  - name.lastName：类型为字符串，不需要分词，keyword；参与搜索，index 为 true；无需分词器

### 创建索引库和映射

上面我们了解了 Mapping 属性映射，接下来我们就去看看如何创建索引库及映射。

```txt
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      }
      // ...略
    }
  }
}
```

```txt
PUT /kurisu
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": "false"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

![image-20220430195243749](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture/202204301952837.png)

mapping 需要考虑的问题：字段名、数据类型、是否参与搜索、是否分词（若分词，则采用什么分词器）

特殊字段说明：

- location：地理坐标，里面包含精度、纬度
- all：一个组合字段，其目的是将多字段的值利用 `copy_to` 合并，提供给用户搜索，这样一来就只需要搜索一个字段就可以得到结果，性能更好。

### 修改索引库

倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改 mapping**

虽然无法修改 mapping 中已有的字段，但是却允许添加新的字段到 mapping 中，不会对倒排索引产生影响。

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```

### 删除索引库

```json
DELETE /索引库名
```

### 查询索引库

```json
GET /索引库名
```

## DSL文档操作

### 新增文档

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    }
    // ...
}
POST /xn2001/_doc/1
{
    "info": "我不会Java",
    "email": "jiawa@qq.com",
    "name": {
        "firstName": "钟",
        "lastName": "弟弟"
    }
}
```

### 查询文档

```json
GET /{索引库名称}/_doc/{id}
```

### 删除文档

```json
DELETE /{索引库名}/_doc/{id}
```

### 修改文档

修改文档有两种方式：

- 全量修改：直接覆盖原来的文档
- 增量修改：修改文档中的部分字段

**全量修改**是覆盖原来的文档，其本质是：

- 根据指定的 id 删除文档
- 新增一个相同 id 的文档

**注意**：如果根据 id 删除时，id 不存在，第二步的新增也会执行，也就是变成了新增操作

```json
PUT /{索引库名}/_doc/id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```

```json
PUT /kurisu/_doc/1
{
    "info": "今天晚上不吃饭",
    "email": "abc@qq.com",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

**增量修改** 是只修改指定 id 匹配的文档中的部分字段

```json
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```

```json
POST /kurisu/_update/1
{
  "doc": {
    "name": {
      "firstName": "Kurisu",
      "lastName": "Makise"
    }
  }
}
```

## RestClient 文档操作

ES 官方提供了各种不同语言的客户端，用来操作 ES。这些客户端的本质就是组装 DSL 语句，通过 http 请求发送给 ES。官方文档地址：https://www.elastic.co/guide/en/elasticsearch/client/index.html

其中的Java Rest Client又包括两种：

- Java Low Level Rest Client
- Java High Level Rest Client

在给定表 tb_hotel 下，创建 mapping 映射

![image-20220520152522938](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201525030.png)

- 字段名、字段数据类型，可以参考数据表结构的名称和类型
- 是否参与搜索要分析业务来判断，例如图片地址，就无需参与搜索
- 是否分词呢要看内容，内容如果是一个整体就无需分词
- 分词器，我们可以统一使用 `ik_max_word`

```dsl
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to": "all"
      },
      "address":{
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "integer"
      },
      "score":{
        "type": "integer"
      },
      "brand":{
        "type": "keyword",
        "copy_to": "all"
      },
      "city":{
        "type": "keyword",
        "copy_to": "all"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type": "keyword"
      },
      "location":{
        "type": "geo_point"
      },
      "pic":{
        "type": "keyword",
        "index": false
      },
      "all":{
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```

特殊字段说明：

- location：地理坐标，里面包含精度、纬度
- all：一个组合字段，其目的是将多字段的值利用 `copy_to` 合并，提供给用户搜索，这样一来就只需要搜索一个字段就可以得到结果，性能更好。

ES中支持两种地理坐标数据类型：

- geo_point：由纬度（latitude）和经度（longitude）确定的一个点。例如："32.8752345, 120.2981576"
- geo_shape：有多个 geo_point 组成的复杂几何图形。例如一条直线，"LINESTRING (-77.03653 38.897676, -77.009051 38.889939)"

### 初始化RestClient

在 elasticsearch 提供的 API 中，elasticsearch 一切交互都封装在一个名为 RestHighLevelClient 的类中，必须先完成这个对象的初始化，建立与 elasticsearch 的连接。

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

SpringBoot 默认的 ES 版本是 7.6.2，我们需要将 pom 中 elasticsearch 的版本和虚拟机中开启的 elasticsearch 版本相同

```xml
<properties>
    <java.version>1.8</java.version>
    <elasticsearch.version>7.12.1</elasticsearch.version>
</properties>
```

初始化 RestHighLevelClient，初始化的代码如下：

```java
RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
        HttpHost.create("http://192.168.197.128:9200") // 当有集群多个地址时，可加入多个
));
```

我们创建一个测试类 HotelIndexTest，然后将初始化的代码编写在 `@BeforeEach` 方法

```java
public class HotelIndexTest {

    private RestHighLevelClient restHighLevelClient;

    @Test
    void testInit(){
        System.out.println(this.restHighLevelClient);
    }

    @BeforeEach
    void init(){
        this.restHighLevelClient = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.197.128:9200")
        ));
    }

    @AfterEach
    void down() throws IOException {
        this.restHighLevelClient.close();
    }
}
```

### 创建索引库

```java
@Test
void createHotelIndex() throws IOException {
    //指定索引库名
    CreateIndexRequest hotel = new CreateIndexRequest("hotel");
    //写入JSON数据，这里是Mapping映射。注意观察下面的 MAPPING_TEMPLATE，不包含索引库名
    hotel.source(HotelConstants.MAPPING_TEMPLATE, XContentType.JSON);
    //创建索引库， .indices() 返回的是操作 es 的 API，里面有创建、删除、修改等方法
    restHighLevelClient.indices().create(hotel, RequestOptions.DEFAULT);
}
```

```java
public class HotelConstants {
    public static String MAPPING_TEMPLATE = "{\n" +
            "  \"mappings\": {\n" +
            "    \"properties\": {\n" +
            "      \"id\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"name\":{\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"address\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"price\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"score\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"brand\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"city\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"starName\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"business\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"location\":{\n" +
            "        \"type\": \"geo_point\"\n" +
            "      },\n" +
            "      \"pic\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"all\":{\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\"\n" +
            "      }\n" +
            "    }\n" +
            "  }\n" +
            "}";
}
```

### 删除索引库

```java
@Test
void deleteHotelIndex() throws IOException {
    DeleteIndexRequest hotel = new DeleteIndexRequest("hotel");
    restHighLevelClient.indices().delete(hotel,RequestOptions.DEFAULT);
}
```

### 判断索引库是否存在

```java
@Test
void existHotelIndex() throws IOException {
    GetIndexRequest hotel = new GetIndexRequest("hotel");
    boolean exists = restHighLevelClient.indices().exists(hotel, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

### 新增文档

```java
@SpringBootTest
public class HotelDocumentTest {

    private RestHighLevelClient restHighLevelClient;

    @Autowired
    private HotelService hotelService;

    @Test
    void testInit() {
        System.out.println(this.restHighLevelClient);
    }

    @Test
    void createHotelIndex() throws IOException {
        Hotel hotel = hotelService.getById(61083L);
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 1.准备Request对象
        IndexRequest hotelIndex = new IndexRequest("hotel").id(hotelDoc.getId().toString());
        // 2.准备Json文档
        hotelIndex.source(JSON.toJSONString(hotelDoc), XContentType.JSON);
        // 3.发送请求
        restHighLevelClient.index(hotelIndex, RequestOptions.DEFAULT);
    }
    
    @BeforeEach
    void init() {
        this.restHighLevelClient = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.197.128:9200")
        ));
    }

    @AfterEach
    void down() throws IOException {
        this.restHighLevelClient.close();
    }
}
```

![image-20220520155651280](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201556379.png)

### 查询文档

```java
@Test
void testGetDocumentById() throws IOException {
    // 1.准备Request
    GetRequest hotel = new GetRequest("hotel", "61083");
    // 2.发送请求，得到响应
    GetResponse hotelResponse = restHighLevelClient.get(hotel, RequestOptions.DEFAULT);
    // 3.解析响应结果
    String hotelDocSourceAsString = hotelResponse.getSourceAsString();
    // 4.json转实体类
    HotelDoc hotelDoc = JSON.parseObject(hotelDocSourceAsString, HotelDoc.class);
    System.out.println(hotelDoc);
}
```

### 删除文档

```java
@Test
void testDeleteDocumentById() throws IOException {
    DeleteRequest hotel = new DeleteRequest("hotel", "61083");
    restHighLevelClient.delete(hotel,RequestOptions.DEFAULT);
}
```

### 修改文档

前面我们说过，修改文档有两种方式：

- 全量修改：直接覆盖原来的文档
- 增量修改：修改文档中的部分字段

在 RestClient 的 API 中，全量修改与新增的 API 完全一致，判断依据是 ID

- 如果新增时，ID已经存在，则修改
- 如果新增时，ID不存在，则新增

所以全量修改写法与新增文档一样，下面我们主要是介绍增量修改。

```java
@Test
void testUpdateDocument() throws IOException {
    // 1.准备Request
    UpdateRequest request = new UpdateRequest("hotel", "61083");
    // 2.准备请求参数
    request.doc(
        "price", "952",
        "starName", "四钻"
    );
    // 3.发送请求
    restHighLevelClient.update(request, RequestOptions.DEFAULT);
}
```

### 批量导入文档

批量处理 BulkRequest，其本质就是将多个普通的 CRUD 请求组合在一起发送。

因此Bulk中添加了多个IndexRequest，就是批量新增功能了。示例：

![image-20220520161542286](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201615400.png)

利用这一点，我们可以写出自己需要的代码，如下

```java
@Test
void testBulk() throws IOException {
    BulkRequest bulkRequest = new BulkRequest();
    List<Hotel> hotelList = hotelService.list();
    for (Hotel item : hotelList) {
        HotelDoc hotelDoc = new HotelDoc(item);
        bulkRequest.add(new IndexRequest("hotel")
                .id(hotelDoc.getId().toString())
                .source(JSON.toJSONString(hotelDoc), XContentType.JSON));
    }
    restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
}
```

## DSL 文档查询

Elasticsearch 提供了基于 JSON 的 DSL([Domain Specific Language](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html))来定义查询。常见的查询类型包括：

**查询所有**：查询出所有数据，一般测试用。例如：match_all

```dsl
GET /hotel/_search //返回所有统计信息
```

![image-20220520162610042](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201626123.png)

**全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如：

- match_query
- multi_match_query

**精确查询**：根据精确词条值查找数据，一般是查找 keyword、数值、日期、boolean 等类型字段。例如：

- ids
- range
- term

**地理（geo）查询**：根据经纬度查询。例如：

- geo_distance
- geo_bounding_box

**复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：

- bool
- function_score

### 查询所有

```dsl
# 查询所有 get /索引库名称/_search
GET /kurisu/_search
{
  "query": {
    "match_all": {
    }
  }
}
```

### 全文检索

使用场景：全文检索查询的基本流程如下：

- 对用户搜索的内容做分词，得到词条
- 根据词条去倒排索引库中匹配，得到文档id
- 根据文档id找到文档，返回给用户

比较常用的场景包括：

- 商城的输入框搜索
- 百度输入框搜索

例如京东：

因为是拿着词条去匹配，因此参与搜索的字段也必须是可分词的text类型的字段。

常见的全文检索查询包括：

- match 查询：单字段查询
- multi_match 查询：多字段查询，任意一个字段符合条件就算符合查询条件

match 查询语法如下：

```json
GET /indexName/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  }
}
```

mulit_match 查询语法如下：

```json
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "TEXT",
      "fields": ["FIELD1", " FIELD12"]
    }
  }
}
```

因为我们将 brand、name、business 值都利用 **copy_to** 复制到了 **all** 字段中，你根据三个字段搜索，和根据 all 字段搜索效果是一样的。

```json
GET /hotel/_search
{
  "query": {
    "match": {
      "all": "7天酒店"
    }
  }
}
GET /hotel/_search
{
  "query": {
    "multi_match": {
      "query": "7天酒店",
      "fields": ["brand","name"]
    }
  }
}
```

**搜索字段越多，对查询性能影响越大，因此建议采用 copy_to 将多个字段合并为一个，然后使用单字段查询的方式。**

### 精准查询

精确查询一般是查找 keyword、数值、日期、boolean 等类型字段。所以**不会**对搜索条件分词。

- term：根据词条精确值查询
- range：根据值的范围查询

#### term查询

因为精确查询的字段搜是不分词的字段，因此查询的条件也必须是**不分词**的词条。查询时，用户输入的内容跟自动值完全匹配时才认为符合条件。如果用户输入的内容过多，反而搜索不到数据。

语法说明：

```json
// term查询
GET /indexName/_search
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```

示例：

```json
GET /hotel/_search
{
  "query": {
    "term": {
      "brand": {
        "value": "7天酒店"
      }
    }
  }
}
```

#### range查询

范围查询，一般应用在对数值类型做范围过滤的时候。比如做价格范围过滤。

基本语法：

```json
// range查询
GET /indexName/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10, // 这里的gte代表大于等于，gt则代表大于
        "lte": 20 // lte代表小于等于，lt则代表小于
      }
    }
  }
}
```

示例：

![image-20220520163003253](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201630362.png)

### 地理坐标查询

地理坐标查询，其实就是根据经纬度查询，官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html

常见的使用场景包括：

- 携程：搜索我附近的酒店
- 滴滴：搜索我附近的出租车
- 微信：搜索我附近的人

查询方式有：

1. 矩形范围查询
2. 附近查询

#### 矩形范围查询

矩形范围查询，也就是 `geo_bounding_box` 查询，查询坐标落在某个矩形范围的所有文档

![image-20220520163532780](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201635880.png)

查询时，需要指定矩形的**左上**、**右下**两个点的坐标，然后画出一个矩形，落在该矩形内的都是符合条件的点。

```dsl
// geo_bounding_box查询
GET /indexName/_search
{
  "query": {
    "geo_bounding_box": {
      "FIELD": {
        "top_left": { // 左上点
          "lat": 31.1,
          "lon": 121.5
        },
        "bottom_right": { // 右下点
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```

#### 附近查询

附近查询，也叫做距离查询（geo_distance）：查询到指定中心点小于某个距离值的所有文档

在地图上找一个点作为圆心，以指定距离为半径，画一个圆，落在圆内的坐标都算符合条件：

![image-20220520163634835](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201636963.png)

```dsl
// geo_distance 查询
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance": "15km", // 半径
      "FIELD": "31.21,121.5" // 圆心
    }
  }
}
```

![image-20220520163925462](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201639535.png)

在查询中加入 sort

```dsl
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance" : {
          "location": "31.034661,121.612282", //圆心
          "order" : "asc", //升序
          "unit" : "km" //单位
      }
    }
  ]
}
```

结果：

```txt
"hits" : [
    {
        "_index" : "hotel",
        "_type" : "_doc",
        "_id" : "2056298828",
        "_score" : null,
        "_source" : {
            ...
        },
        "sort" : [
            4.8541199685347785 //这里的结果为离圆心的距离
        ]
    },
```

注意：输出结果中的 **sort** 为距离，比较常用。

排序完成后，页面还要获取我附近每个酒店的具体**距离**值，这个值在响应结果中是独立的：

![image-20220520164156933](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201641026.png)

### 复合查询

复合（compound）查询：复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑。

- fuction score：算分函数查询，可以控制文档相关性算分，控制文档排名
- bool query：布尔查询，利用逻辑关系组合多个其它的查询，实现复杂搜索

#### 相关性算分

当我们利用 match 查询时，文档结果会根据与搜索词条的关联度打分（_score），返回结果时按照分值降序排列。例如，我们搜索 "虹桥如家"，结果如下：

elasticsearch 早期使用的打分算法是 **TF-IDF 算法**，公式如下：

![image-20220520164650720](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201646797.png)

在后来的5.1版本升级中，elasticsearch 将算法改进为 **BM25 算法**，公式如下：

![image-20220520164801113](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201648220.png)

TF-IDF 算法有一各缺陷，就是词条频率越高，文档得分也会越高，单个词条对文档影响较大。而 BM25 则会让单个词条的算分有一个上限，曲线更加平滑：

![image-20220520164846145](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201648260.png)

#### 算分函数查询

根据相关度打分是比较合理的需求，但有时候也不能够满足我们的需求。

以百度为例，你搜索的结果中，并不是相关度越高排名越靠前，而是谁给的钱多排名就越靠前。

**要想认为控制相关性算分，就需要利用 elasticsearch 中的 function score 查询了**

![image-20220520165029624](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201650747.png)

function score 查询中包含四部分内容：

- **原始查询**条件：query 部分，基于这个条件搜索文档，并且基于BM25算法给文档打分，**原始算分**（query score)
- **过滤条件**：filter 部分，符合该条件的文档才会**重新算分**
- **算分函数**：符合 filter 条件的文档要根据这个函数做运算，得到的**函数算分**（function score），有四种函数
  - weight：函数结果是常量
  - field_value_factor：以文档中的某个字段值作为函数结果
  - random_score：以随机数作为函数结果
  - script_score：自定义算分函数算法
- **加权模式**：算分函数的结果、原始查询的相关性算分，两者之间的运算方式，包括：
  - multiply：相乘
  - replace：用 function score 替换 query score
  - sum、avg、max、min

function score 的运行流程如下：

1. 根据**原始条件**查询搜索文档，并且计算相关性算分，称为**原始算分**（query score）
2. 根据**过滤条件**，过滤文档
3. 符合**过滤条件**的文档，基于**算分函数**运算，得到**函数算分**（function score）
4. 将**原始算分**（query score）和**函数算分**（function score）基于**运算模式**做运算，得到最终结果，作为相关性算分。

因此，其中的关键点是

- 过滤条件：决定哪些文档的算分被修改
- 算分函数：决定函数算分的算法
- 运算模式：决定最终算分结果

例子：

![image-20220520165659958](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201657077.png)

添加了算分函数后，如家得分就提升了

![image-20220520165807333](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201658455.png)

### 布尔查询

布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must_not：必须不匹配，**不参与算分**，类似“非”
- filter：必须匹配，**不参与算分**

**每一个不同的字段，其查询的条件、方式都不一样，必须是多个不同的查询，而要组合这些查询，就必须用 bool查询了。**

需要注意的是，搜索时，参与**打分的字段越多，查询的性能也越差**。因此这种多条件查询时，建议这样做：

- 搜索框的关键字搜索，是全文检索查询，使用 must 查询，参与算分
- 其它过滤条件，采用 filter 查询，不参与算分

```dsl
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"city": "上海" }}
      ],
      "should": [
        {"term": {"brand": "皇冠假日" }},
        {"term": {"brand": "华美达" }}
      ],
      "must_not": [
        { "range": { "price": { "lte": 500 } }}
      ],
      "filter": [
        { "range": {"score": { "gte": 45 } }}
      ]
    }
  }
}
```

bool 查询的几种逻辑关系

- must：必须匹配的条件，可以理解为“与”
- should：选择性匹配的条件，可以理解为“或”
- must_not：必须不匹配的条件，不参与打分
- filter：必须匹配的条件，不参与打分

## 搜索结果处理

### 排序

elasticsearch 默认是根据相关度算分（_score）降序排序，但是也支持自定义方式对搜索结果排序。当指定自定义排序规则后，原本默认的 \_score 将会失效。可以排序字段类型有：keyword 类型、数值类型、地理坐标类型、日期类型等

```dsl
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "FIELD": "desc"  // 排序字段、排序方式ASC、DESC
    }
  ]
}
```

排序条件是一个数组，也就是可以写多个排序条件。按照声明的顺序，当第一个条件相等时，再按照第二个条件排序。

![image-20220520170621074](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201706182.png)

![image-20220520170941523](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201709629.png)

### 分页

elasticsearch 默认情况下只返回 top10 的数据。而如果要查询更多数据就需要修改分页参数了。

elasticsearch 通过修改 from、size 参数来控制要返回的分页结果：

- from：从第几个文档开始
- size：总共查询几个文档

```dsl
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```

> **深度分页问题**

现在，我要查询990~1000的数据，查询逻辑要这么写

```dsl
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
  "from": 990, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```

这里是查询990开始的数据，也就是 第990~第1000条 数据。

**注意**：elasticsearch 内部分页时，必须先查询 0~1000条，然后截取其中的 990 ~ 1000 的这10条

![image-20220520171448251](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201714355.png)

查询TOP1000，如果 es 是单点模式，这并无太大影响。

但是 elasticsearch 将来一定是集群，例如我集群有5个节点，我要查询 TOP1000 的数据，并不是每个节点查询200条就可以了。节点A的 TOP200，在另一个节点可能排到10000名以外了。

**因此要想获取整个集群的 TOP1000，必须先查询出每个节点的 TOP1000，汇总结果后，重新排名，重新截取 TOP1000。**

![image-20220520171949426](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201719542.png)

**当查询分页深度较大时，汇总数据过多，对内存和CPU会产生非常大的压力，因此 elasticsearch 会禁止from+ size 超过10000的请求。**

针对深度分页，ES提供了两种解决方案，[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)：

- scroll：原理将排序后的文档id形成快照，保存在内存。官方已经不推荐使用。

  原理上是对某次查询生成一个游标 scroll_id ， 后续的查询只需要根据这个游标去取数据，直到结果集中返回的 hits 字段为空，就表示遍历结束。scroll_id 的生成可以理解为建立了一个临时的历史快照，在此之后的增删改查等操作不会影响到这个快照的结果。每次发送scroll请求，我们还需要指定一个scroll参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了。

  每次请求两条。可以定制 scroll = 5m意味着该窗口过期时间为5分钟。

  ```dsl
  GET /student/student/_search?scroll=5m
  {
    "query": {
      "match_all": {}
    },
    "size": 2
  }
  ```

- search after：分页时需要排序，原理是从上一次的排序值开始，查询下一页数据。官方推荐使用的方式。

  scroll能够解决深度分页的问题，但是其无法实现实时查询，即当scroll_id生成后无法查询到之后数据的变更，因为其底层原理是生成数据的快照。这时 search_after应运而生。

  search_after 根据上一页的最后一条数据来确定下一页的位置，同时在分页请求的过程中，如果有索引数据的增删改查，这些变更也会实时的反映到游标上。为了找到每一页最后一条数据，每个文档必须有一个全局唯一值，官方推荐使用 _uid 作为全局唯一值，但是只要能表示其唯一性就可以。

分页查询的常见实现方案以及优缺点

- `from + size`
  - 优点：支持随机翻页
  - 缺点：深度分页问题，默认查询上限（from + size）是10000
  - 场景：百度、京东、谷歌、淘宝这样的随机翻页搜索
- `search after`
  - 优点：没有查询上限（单次查询的size不超过10000）
  - 缺点：只能向后逐页查询，不支持随机翻页
  - 场景：没有随机翻页需求的搜索，例如手机向下滚动翻页
- `scroll`
  - 优点：没有查询上限（单次查询的size不超过10000）
  - 缺点：会有额外内存消耗，并且搜索结果是非实时的
  - 场景：海量数据的获取和迁移。从ES7.1开始不推荐，建议用 after search方案。

### 高亮

我们在百度，京东搜索时，关键字会变成红色，比较醒目，这叫高亮显示：

![image-20220520172236635](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201722743.png)

高亮显示的实现分为两步：

- 1）给文档中的所有关键字都添加一个标签，例如`<em>`标签
- 2）页面给`<em>`标签编写CSS样式

**注意：**

- 高亮是对关键字高亮，因此**搜索条件必须带有关键字**，而不能是范围这样的查询。
- 默认情况下，**高亮的字段，必须与搜索指定的字段一致**，否则无法高亮
- 如果要对非搜索字段高亮，则需要添加一个属性：`required_field_match=false`

![image-20220520193832767](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201938882.png)

> DSL 总体结构如下：

![image-20220520193913498](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205201939636.png)

## RestClient文档查询

### 发起查询请求

![image-20220521151049856](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211510940.png)

```java
@SpringBootTest
public class HotelSearchTest {

    private RestHighLevelClient restHighLevelClient;

    @Autowired
    private IHotelService hotelService;

    @Test
    public void match_All() throws IOException {
        SearchRequest request = new SearchRequest("hotel");
        request.source()
                .query(QueryBuilders.matchAllQuery());
        SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
    }

    @BeforeEach
    void init() {
        this.restHighLevelClient = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.211.128:9200")
        ));
    }

    @AfterEach
    void down() throws IOException {
        this.restHighLevelClient.close();
    }
}
```

- 第一步，创建`SearchRequest`对象，指定索引库名
- 第二步，利用`SearchRequest.source()`构建 DSL，DSL 中可以包含查询、分页、排序、高亮等
  - `query()`：代表查询条件，利用 `QueryBuilders.matchAllQuery()` 构建一个 match_all 查询的 DSL
- 第三步，利用 `client.search()` 发送请求，得到响应

关键的 API 有两个，一个是 `request.source()`，其中包含了查询、排序、分页、高亮等所有功能

![image-20220521151349278](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211513427.png)

另一个是 `QueryBuilders`，用来构造查询条件，其中包含 match、term、function_score、bool 等各种查询

![image-20220521151355738](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211513843.png)

### 解析查询响应

逐层解析文档

![image-20220521151445791](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211514893.png)

Elasticsearch 返回的结果是一个 JSON 字符串，结构包含

- `hits`：命中的结果
  - `total`：总条数，其中的value是具体的总条数值
  - `max_score`：所有结果中得分最高的文档的相关性算分
  - `hits`：搜索结果的文档数组，其中的每个文档都是一个 json 对象
    - `_source`：文档中的原始数据，也是 json 对象

因此，我们解析响应结果，就是逐层解析 JSON 字符串，流程如下

- `SearchHits`：通过 `response.getHits()` 获取，就是 json 中的最外层的 hits，代表命中的结果
  - `SearchHits.getTotalHits().value`：获取总条数信息
  - `SearchHits.getHits()`：获取 SearchHit 数组，也就是文档数组
    - `SearchHit.getSourceAsString()`：获取文档结果中的 `_source`，也就是原始的 json 文档数据

```java
@Test
public void match_All() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source()
            .query(QueryBuilders.matchAllQuery());
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

### match查询

![image-20220521153119216](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211531312.png)

```java
@Test
public void matchQuery() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source()
            .query(QueryBuilders.matchQuery("all","如家"));
    SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}

@Test
public void multiMatchQuery() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source()
            .query(QueryBuilders.multiMatchQuery("如家","name","brand"));
    SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

### 精确查询

精确查询主要是两者

- term：词条精确匹配
- range：范围查询

![image-20220521153358766](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211533837.png)

### 布尔查询

布尔查询是用 must、must_not、filter等方式组合其它查询，代码示例如下

![image-20220521153515236](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211535332.png)

```java
@Test
void testBool() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    request.source()
            .query(
                    QueryBuilders.boolQuery()
                            .must(QueryBuilders.termQuery("city", "上海"))
                            .filter(QueryBuilders.rangeQuery("price").lte(300))
            );
    // 3.发送请求
    SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

### 排序、分页

搜索结果的排序和分页是与 query 同级的参数，因此同样是使用 `request.source()` 来设置。

对应的API如下

![image-20220521153917902](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211539986.png)

```java
@Test
void testPageAndSort() throws IOException {
    // 页码，每页大小
    int page = 1, size = 5;

    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    // 2.1.query
    request.source().query(QueryBuilders.matchAllQuery());
    // 2.2.排序 sort
    request.source().sort("price", SortOrder.ASC);
    // 2.3.分页 from、size
    request.source().from((page - 1) * size).size(5);
    // 3.发送请求
    SearchResponse response = restHighLevelClient.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

### 高亮

- 查询的 DSL：其中除了查询条件，还需要添加高亮条件，同样是与 query 同级。
- 结果解析：结果除了要解析 `_source` 文档数据，还要解析高亮结果

![image-20220521154147439](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211541532.png)

```java
@Test
void testHighlight() throws IOException {
    // 1.准备Request
    SearchRequest request = new SearchRequest("hotel");
    // 2.准备DSL
    // 2.1.query
    request.source().query(QueryBuilders.matchQuery("all", "如家"));
    // 2.2.高亮
    request.source().highlighter(new HighlightBuilder().field("name").requireFieldMatch(false));
    // 3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    System.out.println("hits.getTotalHits().条数 = " + searchHits.getTotalHits().value);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        HotelDoc hotelDoc = JSON.parseObject(sourceAsString, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

**高亮结果解析**

高亮的结果与查询的文档结果默认是分离的，并不在一起。

![image-20220521155002908](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211550979.png)

因此解析高亮的代码需要额外处理：

![image-20220521154531526](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211545650.png)

- 第一步：从结果中获取 source。`hit.getSourceAsString()`，这部分是非高亮结果，json 字符串，需要反序列为 HotelDoc 对象
- 第二步：获取高亮结果。`hit.getHighlightFields()`，返回值是一个 Map，key 是高亮字段名称，值是HighlightField 对象，代表高亮值
- 第三步：从 map 中根据高亮字段名称，获取高亮字段值对象 HighlightField
- 第四步：从 HighlightField 中获取 Fragments，并且转为字符串。**这部分是真正的高亮字符串**
- 第五步：用高亮的结果替换 HotelDoc 中的非高亮结果

```java
private void handleResponse(SearchResponse response) {
    // 4.解析响应
    SearchHits searchHits = response.getHits();
    // 4.1.获取总条数
    long total = searchHits.getTotalHits().value;
    System.out.println("共搜索到" + total + "条数据");
    // 4.2.文档数组
    SearchHit[] hits = searchHits.getHits();
    // 4.3.遍历
    for (SearchHit hit : hits) {
        // 获取文档source
        String json = hit.getSourceAsString();
        // 反序列化
        HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
        // 获取高亮结果
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if (!CollectionUtils.isEmpty(highlightFields)) {
            // 根据字段名获取高亮结果
            HighlightField highlightField = highlightFields.get("name");
            if (highlightField != null) {
                // 获取高亮值
                String name = highlightField.getFragments()[0].string();
                // 覆盖非高亮结果
                hotelDoc.setName(name);
            }
        }
        System.out.println("hotelDoc = " + hotelDoc);
    }
}
```

### 地理坐标查询

![image-20220521155403577](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211554679.png)

### 相关性得分

function_score 查询结构如下

![image-20220521155503547](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211555681.png)

对应的 JavaAPI 如下

![image-20220521155610289](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211556397.png)

## DSL数据聚合

**聚合**可以让我们极其方便的实现对数据的统计、分析、运算。

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

在 Elasticsearch 实现这些统计功能比数据库的 sql 要方便的多，而且查询速度非常快，可以实现近实时搜索效果。

聚合常见的有三类

- **桶（Bucket）**聚合：用来对文档做分组
  - TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组
- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
  - Avg：求平均值
  - Max：求最大值
  - Min：求最小值
  - Stats：同时求 max、min、avg、sum 等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合

**注意：**参加聚合的字段必须是keyword、日期、数值、布尔类型

### Bucket聚合语法

例如：我们要统计所有数据中的酒店品牌有几种，其实就是按照品牌对数据分组。此时可以根据酒店品牌的名称做聚合，也就是 Bucket 聚合。

```dsl
GET /hotel/_search
{
  "size": 0,  // 分页参数，设置size为0，结果中不包含文档，只包含聚合结果
  "aggs": { // 定义聚合
    "brandAgg": { //给聚合起个名字
      "terms": { // 聚合的类型，按照品牌值聚合，所以选择term
        "field": "brand", // 参与聚合的字段
        "size": 20 // 希望获取的聚合结果数量，即使可获得 100 个聚合结果，也只会显示前 20 个
      }
    }
  }
}
```

![image-20220521160501012](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211605105.png)

> **聚合结果排序**

默认情况下，Bucket 聚合会统计 Bucket 内的文档数量，记为 `_count`，并且按照 `_count` 降序排序。

我们可以指定 order 属性，自定义聚合的排序方式

```dsl
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "order": {
          "_count": "asc" // 按照_count升序排列
        },
        "size": 20
      }
    }
  }
}
```

> **限定聚合范围**

默认情况下，Bucket 聚合是对索引库的所有文档做聚合，但真实场景下，用户会输入搜索条件，因此聚合必须是对搜索结果聚合。那么聚合必须添加限定条件。

我们可以限定要聚合的文档范围，只要添加 query 条件即可；

```dsl
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200 // 只对200元以下的文档聚合
      }
    }
  }, 
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20
      }
    }
  }
}
```

![image-20220521160749948](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211607077.png)

### Metric聚合语法

上面，我们对酒店按照品牌分组，形成了一个个桶。现在我们需要对桶内的酒店做运算，获取每个品牌的用户评分的 min、max、avg 等值。

这就要用到 Metric 聚合了，例如 stats 聚合：就可以获取 min、max、avg 等结果

```dsl
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": { 
      "terms": { 
        "field": "brand", 
        "size": 20
      },
      "aggs": { // 是brands聚合的子聚合，也就是分组后对每组分别计算
        "score_stats": { // 聚合名称
          "stats": { // 聚合类型，这里stats可以计算min、max、avg等
            "field": "score" // 聚合字段，这里是score
          }
        }
      }
    }
  }
}
```

这次的 score_stats 聚合是在 brandAgg 的**聚合内部嵌套的子聚合**。因为我们需要在每个桶分别计算。

另外，我们还可以给聚合结果做个排序，例如按照每个桶的酒店平均分做排序

![image-20220521161142150](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211611257.png)

## RestAPI数据聚合

聚合条件与 query 条件同级别，因此需要使用 `request.source()` 来指定聚合条件

![image-20220521161406255](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211614363.png)

聚合的结果也与查询结果不同，API 也比较特殊。不过同样是 JSON 逐层解析

![image-20220521161657661](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211616810.png)

```java
@Test
public void testAggregation() throws IOException {
    // 1、准备 request
    SearchRequest request = new SearchRequest("hotel");
    // 2、准备 dsl
    request.source().aggregation(
            AggregationBuilders
                    .terms("brandAgg")
                    .field("brand")
                    .size(20)
    );
    // 3、发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4、解析结果
    Terms brandAgg = response.getAggregations().get("brandAgg");
    List<? extends Terms.Bucket> buckets = brandAgg.getBuckets();
    for (Terms.Bucket bucket : buckets) {
        String key = bucket.getKeyAsString();
        System.out.println("key = " + key);
    }
}
```

## 自动补全

当用户在搜索框输入字符时，我们应该提示出与该字符有关的搜索项，提示完整词条的功能，就是自动补全了。

### 拼音分词器

使用 `docker volume inspect es-plugins` 查看插件目录，将下载的文件解压上传，再使用 `docker restart es` 重启 Elasticsearch

测试用法如下：

```json
POST /_analyze
{
  "text": "炸鸡腿很好吃",
  "analyzer": "pinyin"
}
```

![image-20220521162743044](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211627138.png)

### 自定义分词器

默认的拼音分词器会将每个汉字单独分为拼音，而我们希望的是每个词条形成一组拼音，需要对拼音分词器做个性化定制，形成自定义分词器。

elasticsearch 中分词器（analyzer）的组成包含三部分：

- character filters：在 tokenizer 之前对文本进行处理。例如删除字符、替换字符
- tokenizer：将文本按照一定的规则切割成词条（term）。例如 keyword，就是不分词；还有 ik_smart
- tokenizer filter：将 tokenizer 输出的词条做进一步处理。例如大小写转换、同义词处理、拼音处理等

文档分词时会依次由这三部分来处理文档：

![image-20220521163724396](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211637516.png)

过滤器的参数配置可参考官方文档 https://github.com/medcl/elasticsearch-analysis-pinyin 

```dsl
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器
        "my_analyzer": {  // 分词器名称
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true, // 是否保留原始输入（原始输入为中文），默认 false
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",   // 创建倒排索引使用 my_analyzer 分词器
        "search_analyzer": "ik_smart" // 搜索时的分词器为 ik_smart
      }
    }
  }
}
```

![image-20220521164111673](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211641853.png)

注意：**为了避免搜索到同音字，搜索时不要使用拼音分词**

![image-20220521164251057](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211642168.png)

### 自动补全查询

elasticsearch 提供了 [Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-suggesters.html) 查询来实现自动补全功能。这个查询会匹配以用户输入内容开头的词条并返回；为了提高补全查询的效率，对于文档中字段的类型有一些约束

- 参与补全查询的字段必须是 completion 类型。
- 字段的内容一般是用来补全的多个词条形成的数组。

```dsl
// 创建索引库
PUT test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "completion"
      }
    }
  }
}
```

然后插入下面的数据

```json
// 示例数据
POST test/_doc
{
  "title": ["Sony", "WH-1000XM3"]
}
POST test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST test/_doc
{
  "title": ["Nintendo", "switch"]
}
```

查询的 DSL 语句如下

```json
// 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 补全查询的字段
        "skip_duplicates": true, // 跳过重复的
        "size": 10 // 获取前10条结果
      }
    }
  }
}
```

### JavaAPI

![image-20220521165534135](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211655256.png)

解析响应的代码如下

![image-20220521165614403](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205211656520.png)

## 数据同步

elasticsearch 中的数据来自于 mysq l数据库，因此 mysql 数据发生改变时，elasticsearch 也必须跟着改变，这个就是 elasticsearch 与 mysql 之间的**数据同步**。

在微服务中，MySQL 数据库和 elasticsearch 可能部署在不同服务器上，对于这种环境下的数据同步问题，常见的数据同步方案有三种

- 同步调用
- 异步通知
- 监听 binlog

### 同步调用

方案一：同步调用

![image-20220522142643527](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221426606.png)

存在的问题：hotel-admin 和 hotel-demo 数据耦合问题

### 异步通知

方案二：异步通知

![image-20220522142836373](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221428454.png)

异步通知依赖于 MQ 消息队列的可靠性

### 监听binlog

方案三：监听binlog

![image-20220522142935095](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221429178.png)

依赖于中间件和 MySQL 的 binlog。大致方案：

1、mysq发生数据变更，写binlog

2、canal伪装成mysql的从库，拉取binlog

3、canal根据配置过滤内容，将符合过滤条件的变更内容投递到kafka

4、业务服务从kafka拉取消息，触发更新索引信息（1、发更新mq。2、新增修改部分属性的接口，调用该接口修改数据）

### 三者对比

方式一：同步调用

- 优点：实现简单，粗暴
- 缺点：业务耦合度高

方式二：异步通知

- 优点：低耦合，实现难度一般
- 缺点：依赖 mq 的可靠性

方式三：监听binlog

- 优点：完全解除服务间耦合
- 缺点：开启 binlog 增加数据库负担、实现复杂度高

### JavaAPI

我们以**异步通知**为例，使用 MQ 消息中间件

MQ结构如图：

![image-20220522145338993](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221453082.png)

hotel-admin 用于连接 MySQL 数据库，hotel-demo 用于连接 elasticsearch。步骤如下：

**1、引入依赖**，在 hotel-admin、hotel-demo 中引入 rabbitmq 的依赖：

```xml
<!--amqp-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**2、声明队列交换机**

```java
public class MqConstants {
    /**
     * 交换机
     */
    public final static String HOTEL_EXCHANGE = "hotel.topic";
    /**
     * 监听新增和修改的队列
     */
    public final static String HOTEL_INSERT_QUEUE = "hotel.insert.queue";
    /**
     * 监听删除的队列
     */
    public final static String HOTEL_DELETE_QUEUE = "hotel.delete.queue";
    /**
     * 新增或修改的RoutingKey
     */
    public final static String HOTEL_INSERT_KEY = "hotel.insert";
    /**
     * 删除的RoutingKey
     */
    public final static String HOTEL_DELETE_KEY = "hotel.delete";
}
```

3、消息接收方

```java
// 声明交换机、消息队列，将交换机和消息队列实现绑定
@Configuration
public class MqConfig {
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(MqConstants.HOTEL_EXCHANGE, true, false);
    }

    @Bean
    public Queue insertQueue() {
        return new Queue(MqConstants.HOTEL_INSERT_QUEUE, true);
    }

    @Bean
    public Queue deleteQueue() {
        return new Queue(MqConstants.HOTEL_DELETE_QUEUE, true);
    }

    @Bean
    public Binding insertQueueBinding() {
        return BindingBuilder.bind(insertQueue()).to(topicExchange()).with(MqConstants.HOTEL_INSERT_KEY);
    }

    @Bean
    public Binding deleteQueueBinding() {
        return BindingBuilder.bind(deleteQueue()).to(topicExchange()).with(MqConstants.HOTEL_DELETE_KEY);
    }
}
```

controller

```java
@Component
public class HotelListener {

    @Autowired
    private HotelService hotelService;

    /**
     * 监听酒店新增或修改的业务
     *
     * @param id 酒店id
     */
    @RabbitListener(queues = MqConstants.HOTEL_INSERT_QUEUE)
    public void listenHotelInsertOrUpdate(Long id) {
        hotelService.insertById(id);
    }

    /**
     * 监听酒店删除的业务
     *
     * @param id 酒店id
     */
    @RabbitListener(queues = MqConstants.HOTEL_DELETE_QUEUE)
    public void listenHotelDelete(Long id) {
        hotelService.deleteById(id);
    }
}
```

service

```java
@Override
public void insertById(Long id) {
    try {
        // 根据id查询酒店数据
        Hotel hotel = getById(id);
        // 转换为文档类型
        HotelDoc hotelDoc = new HotelDoc(hotel);

        IndexRequest request = new IndexRequest("hotel").id(hotel.getId().toString());
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);
        client.index(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

@Override
public void deleteById(Long id) {
    try {
        DeleteRequest request = new DeleteRequest("hotel", id.toString());
        client.delete(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

4、消息发送方

controller

```java
rabbitTemplate.convertAndSend(MQConstants.HOTEL_EXCHANGE, MQConstants.HOTEL_INSERT_KEY, hotel.getId());

rabbitTemplate.convertAndSend(MQConstants.HOTEL_EXCHANGE, MQConstants.HOTEL_DELETE_KEY, id);
```

## Elasticsearch集群

单机的 Elasticsearch 做数据存储，必然面临两个问题：海量数据存储问题、单点故障问题。

解决方案：

- 海量数据存储问题：将索引库从逻辑上拆分为N个分片（shard），存储到多个节点
- 单点故障问题：将分片数据在不同节点备份（replica ）

**ES集群相关概念**：

- 集群（cluster）：一组拥有共同的 cluster name 的节点。
- 节点（node) ：集群中的一个 Elasticearch 实例
- 分片（shard）：索引可以被拆分为不同的部分进行存储，称为分片。**在集群环境下，一个索引的不同分片可以拆分到不同的节点中，解决数据量太大，单点存储量有限的问题。**

![image-20220522150843411](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221508492.png)

此处，我们把数据分成3片：shard0、shard1、shard2

主分片（Primary shard）：相对于副本分片的定义。

副本分片（Replica shard）每个主分片可以有一个或者多个副本，数据和主分片一样。

**数据备份**可以保证高可用，但是每个分片备份一份在节点上，所需要的节点数量就会翻倍，成本太高。为了在高可用和成本间寻求平衡

- 首先对数据分片，存储到不同节点
- 然后对每个分片进行备份，放到对方节点，**完成互相备份**

这样可以大大减少所需要的服务节点数量，如图，我们以3分片，每个分片备份一份为例：

![image-20220522151026717](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221510840.png)

现在，每个分片都有1个备份，存储在3个节点：

- node0：保存了分片0和1
- node1：保存了分片0和2
- node2：保存了分片1和2

### 部署集群

#### 搭建Elasticsearch

我们会在单机上利用 Docker 容器运行多个 Elasticsearch 实例来模拟集群。

可以直接使用 docker-compose 来完成，由于每个 Elasticsearch 默认有 512M 内存空间，因此部署三个 Elasticsearch 要求Linux虚拟机至少有**4G**以上的内存空间。

#### 集群状态监控

kibana 可以监控 Elasticsearch 集群，但是 kibana 只能监视一个 Elasticsearch，并且配置较为复杂，因此更推荐使用 [cerebro](https://github.com/lmenezes/cerebro)

下载解压打开 /bin/cerebro.bat

访问 [http://localhost:9000](http://localhost:9000/) 即可进入管理界面

![image-20220522151938750](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221519855.png)

输入任意节点的地址和端口，点击 connect

![image-20220522152134114](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221521220.png)

绿色的线条代表集群处于绿色（健康状态）

#### 创建索引库

我们还可以通过 cerebro 创建索引库，当然你需要使用 kibana 也可以。

![image-20220522152212339](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221522415.png)

填写索引库信息

![image-20220522152308828](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221523913.png)

回到首页，即可查看索引库分片效果

![image-20220522152609036](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221526154.png)

## 集群职责划分

Elasticsearch 中集群节点有不同的职责划分

![image-20220522152846036](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221528125.png)

**默认情况下，集群中的任何一个节点都同时兼职上述四种角色。**

真实的集群一定要将集群职责分离

- master 节点：对 CPU 要求高，但是内存要求低
- data 节点：对 CPU 和内存要求都高
- coordinating 节点：对网络带宽、CPU 要求高

职责分离可以让我们根据不同节点的需求分配不同的硬件去部署。而且避免业务之间的互相干扰。

![image-20220522153205548](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221532654.png)

## 集群脑裂问题

脑裂是因为网络问题，导致集群中的节点失联引起的。

例如一个集群中，主节点 node1 与其它节点失联。

![image-20220522153307223](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221533329.png)

此时，node2 和 node3 认为 node1 宕机，就会重新选主。当 node3 当选后，集群继续对外提供服务，node2 和 node3 自成集群，node1 自成集群，两个集群数据不同步，出现数据差异。

当网络恢复后，因为集群中有两个 master 节点，集群状态的不一致，出现脑裂的情况。

![image-20220522153330169](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221533290.png)

解决脑裂的方案是，要求选票大于等于 **(eligible节点数量+1)/2** 才能当选为 master，因此 eligible 节点数量最好是奇数。对应配置项是`discovery.zen.minimum_master_nodes`，在版本 7.0 以后，已经成为默认配置，因此一般不会发生脑裂问题。

例如：3个节点形成的集群，选票必须大于等于 (3+1)/2 ，也就是 2 票。node3 得到 node2 和 node3 的选票，当选为 master。node1 只有自己 1 票，没有当选。集群中依然只有1个主节点，没有出现脑裂。

## 集群分布式存储

当新增文档时，应该保存到不同分片，保证数据均衡，那么 coordinating node 如何确定数据该存储到哪个分片呢？

Elasticsearch 会通过 hash 算法来计算文档应该存储到哪个分片

![image-20220522154129077](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221541148.png)

- _routing 默认是文档的 id
- 算法与分片数量有关，因此索引库一旦创建，分片数量不能修改！因为查询时也会根据这个公式到对应的分片中查询，一旦更改分片数量，查询时将不能根据 id 查询到

新增文档的流程如下图，

1. 新增一个 id=1 的文档
2. 对 id 做 hash 运算，假如得到的是 2，则应该存储到 shard-2
3. shard-2 的主分片在 node3 节点，将数据路由到 node3，node3 保存文档
4. 同步给 shard-2 的副本分片2(R-2)，在 node2 节点
5. 返回结果给 coordinating-node 节点(node1)

![image-20220522154256964](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221542075.png)

## 集群分布式查询

当使用 match 进行查询时，由于没有传入文档 id，Elasticsearch 不知道该到哪个分片查询文档，那么 Elasticsearch 查询分成两个阶段

- scatter phase：分散阶段，coordinating node 会把请求分发到每一个分片。
- gather phase：聚集阶段，coordinating node 汇总 data node 的搜索结果，并处理为最终结果集返回给用户。

![image-20220522154358961](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221543085.png)

## 集群故障转移

集群的 master 节点会监控集群中的节点状态，如果发现有节点宕机，会立即将宕机节点的分片数据迁移到其它节点，确保数据安全，这个叫做故障转移。

例如一个集群结构如图，三个都是健康的。

![image-20220522154650707](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221546823.png)

现在，node1 是主节点，其它两个节点是从节点。突然，node1 发生了故障，宕机后的第一件事，需要重新选主，例如选中了 node2。node2 成为主节点后，会检测集群监控状态，将 node1 上的数据迁移到 node2、node3，确保数据依旧正常访问。

![image-20220522154723691](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221547816.png)

# Sentinel 流量组件

## 雪崩问题

> 微服务之间相互调用，因为调用链中的一个服务故障，引起整个链路都无法访问的情况。

微服务中，服务间调用关系错综复杂，一个微服务往往依赖于多个其它微服务。服务器支持的线程和并发数有限，到来的请求一直阻塞，会导致服务器资源耗尽，**从而导致所有其它服务都不可用**。

综上，依赖于当前服务的其它服务随着时间的推移，最终也都会变的不可用，形成级联失败，导致整个微服务无法使用，这就是雪崩问题。

![image-20220522160106581](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221601708.png)

解决雪崩问题的常见方式有四种

1. 超时处理：设定超时时间，请求超过一定时间没有响应就返回错误信息，不会无休止等待。但是当请求速度大于超时时间时，随着时间推移，必然会导致服务器的资源耗尽，没有解决根本问题。

2. 线程隔离：是一种舱壁模式，我们可以限定每个业务能使用的线程数，避免耗尽整个 tomcat 的资源，因此也叫线程隔离。这种模式的问题会空耗服务器资源，服务 C 已经宕机，但是还是有请求不断访问服务 C，空耗资源。

   ![image-20220522160344249](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221603348.png)

3. 降级熔断：是一种断路器模式，由**断路器**统计业务执行的异常比例，如果超出阈值则会**熔断**该业务，拦截访问该业务的一切请求。

   ![image-20220522160535308](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221605395.png)

4. 限流

   **流量控制**：限制业务访问的 QPS，避免服务因流量的突增而故障。

   ![image-20220522160628237](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221606363.png)

**限流**是对服务的保护，避免因瞬间高并发流量而导致服务故障，进而避免雪崩。是一种**预防**措施。

**超时处理、线程隔离、降级熔断**是在部分服务故障时，将故障控制在一定范围，避免雪崩。是一种**补救**措施。

|                  | Sentinel                                       | Hystrix                       |
| :--------------- | :--------------------------------------------- | ----------------------------- |
| **隔离策略**     | 信号量隔离                                     | 线程池隔离/信号量隔离         |
| **熔断降级策略** | 基于慢调用比例或异常比例                       | 基于失败比率                  |
| 实时指标实现     | 滑动窗口                                       | 滑动窗口（基于 RxJava）       |
| 规则配置         | 支持多种数据源                                 | 支持多种数据源                |
| 扩展性           | 多个扩展点                                     | 插件的形式                    |
| 基于注解的支持   | 支持                                           | 支持                          |
| **限流**         | 基于 QPS，支持基于调用关系的限流               | 有限的支持                    |
| **流量整形**     | 支持慢启动、匀速排队模式                       | 不支持                        |
| 系统自适应保护   | 支持                                           | 不支持                        |
| **控制台**       | 开箱即用，可配置规则、查看秒级监控、机器发现等 | 不完善                        |
| 常见框架的适配   | Servlet、Spring Cloud、Dubbo、gRPC 等          | Servlet、Spring Cloud Netflix |

## 初识 Sentinel

Sentinel是阿里巴巴开源的一款微服务流量控制组件。官网地址：https://sentinelguard.io/zh-cn/index.html

Sentinel 具有以下特征

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的** **SPI** **扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

## 整合 Sentinel

下载后 jar 包后，运行代码：`java -jar sentinel-dashboard-1.8.1.jar`

如果要修改 Sentinel 的默认端口、账户、密码，可以通过下列配置：

| **配置项**                       | **默认值** | **说明**   |
| :------------------------------- | :--------- | :--------- |
| server.port                      | 8080       | 服务端口   |
| sentinel.dashboard.auth.username | sentinel   | 默认用户名 |
| sentinel.dashboard.auth.password | sentinel   | 默认密码   |

例如，修改端口：

```sh
java -Dserver.port=8090 -jar sentinel-dashboard-1.8.1.jar
```

我们在 order-service 中整合 Sentinel，并连接 Sentinel 的控制台，步骤如下

1）引入 Sentinel 依赖

```xml
<!--sentinel-->
<dependency>
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2）配置控制台

修改 application.yml 文件，添加下面内容：

```yaml
server:
  port: 8088
spring:
  cloud: 
    sentinel:
      transport:
        dashboard: localhost:8080
```

3）访问 order-service 的任意端点

打开浏览器，访问 http://localhost:10010/order/101，多访问几次，多点几次刷新，这样才能触发 Sentinel 的监控。

然后再访问 Sentinel 的控制台，查看效果。

![image-20220522165323575](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221653699.png)

## 流量控制

雪崩问题虽有四种方案，但是**限流是避免服务因突发的流量而发生故障，是对微服务雪崩问题的预防。**

### 限流算法

#### 1、滑动窗口

在介绍滑动窗口之前，先介绍一种**计数器算法**

计数器算法是限流算法里最简单也是最容易实现的一种算法。比如我们规定，对于A接口来说，我们1分钟的访问次数不能超过100个。那么我们可以这么做：在一开 始的时候，我们可以设置一个计数器counter，每当一个请求过来的时候，counter就加1，如果counter的值大于100并且该请求与第一个 请求的间隔时间还在1分钟之内，那么说明请求数过多；如果该请求与第一个请求的间隔时间大于1分钟，且counter的值还在限流范围内，那么就重置 counter，具体算法的示意图如下：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311122494.jpeg)

这个算法虽然简单，但是有一个十分致命的问题，那就是临界问题：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311123000.jpeg)

从上图中我们可以看到，假设有一个恶意用户，他在0:59时，瞬间发送了100个请求，并且1:00又瞬间发送了100个请求，那么其实这个用户在 1秒里面，瞬间发送了200个请求。我们刚才规定的是1分钟最多100个请求，也就是每秒钟最多1.7个请求，用户通过在时间窗口的重置节点处突发请求， 可以瞬间超过我们的速率限制。用户有可能通过算法的这个漏洞，瞬间压垮我们的应用。

刚才的问题其实是因为我们统计的精度太低，为了解决这个问题，引出滑动窗口算法。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311126838.jpeg)

在上图中，整个红色的矩形框表示一个时间窗口，在我们的例子中，一个时间窗口就是一分钟，一分钟内最大访问数为 100，即一个时间窗口的最大访问数为 100。然后我们将时间窗口进行划分，比如图中，我们就将滑动窗口 划成了6格，所以每格代表的是10秒钟。每过10秒钟，我们的时间窗口就会往右滑动一格。每一个格子都有自己独立的计数器counter，比如当一个请求 在0:35秒的时候到达，那么0:30~0:39对应的counter就会加1。

同样是临界问题，若0:59到达的100个请求落在灰色的格子中，而1:00到达的请求落在橘黄色的格 子中。当时间到达1:00时，我们的窗口会往右移动一格，那么此时时间窗口内的总请求数量一共是200个，超过了限定的100个，所以此时能够检测出来触 发了限流。

我们可以发现，计数器算法其实就是滑动窗口算法。只是它没有对时间窗口做进一步地划分，所以只有1格。

由此可见，当滑动窗口的格子划分的越多，那么滑动窗口的滚动就越平滑，限流的统计就会越精确。

#### 2、令牌桶算法

**按一定额定的速率产生令牌，存入令牌桶，桶有最大容量（应该为微服务最大承载）；服务过来时需要请求到一个令牌才可以进入服务执行；服务里就可以保持基本不会超过承载值。【保护服务】**

1）、所有的请求在处理之前都需要拿到一个可用的令牌才会被处理；
2）、根据限流大小，设置按照一定的速率往桶里添加令牌；
3）、桶设置最大的放置令牌限制，当桶满时、新添加的令牌就被丢弃或者拒绝；
4）、请求达到后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处理完业务逻辑之后，将令牌直接删除；
5）、令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删除令牌，以此保证足够的限流；

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311130390.png)

#### 3、漏桶算法

**维持一个队列，所有请求先进队列，然后从队列取出请求的速率是固定。**【**保护请求**】

漏桶算法其实很简单，可以粗略的认为就是注水漏水过程，往桶中以一定速率流出水，以任意速率流入水，当水超过桶流量则丢弃，因为桶容量是不变的，保证了整体的速率。

当请求进来时，相当于水倒入漏斗，然后从下端小口慢慢匀速的流出。不管上面流量多大，下面流出的速度始终保持不变。不管服务调用方多么不稳定，通过漏桶算法进行限流，每10毫秒处理一次请求。因为处理的速度是固定的，请求进来的速度是未知的，可能突然进来很多请求，没来得及处理的请求就先放在桶里，既然是个桶，肯定是有容量上限，如果桶满了，那么新进来的请求就丢弃。漏桶算法存在一个缺陷，无法应对短时间的突发流量，比如双十一抢购、秒杀活动开始。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311131291.png)

### 簇点链路

当请求进入微服务时，首先会访问 DispatcherServlet，然后进入 Controller、Service、Mapper，这样的一个调用链就叫做 **簇点链路**。

**簇点链路中被监控的每一个接口就是一个资源**。默认情况下 Sentinel 会监控 SpringMVC 的每一个端点（Endpoint，也就是 Controller 中的方法），因此 SpringMVC 的每一个端点（Endpoint）就是调用链路中的一个资源。

例如，我们刚才访问的 order-service 中的 OrderController 中的端点：/order/{orderId}

![image-20220522172057288](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221720383.png)

流控、熔断等都是针对簇点链路中的资源来设置的，因此我们可以点击对应资源后面的按钮来设置规则：

- 流控：流量控制
- 降级：降级熔断
- 热点：热点参数限流，是限流的一种
- 授权：请求的权限控制

点击资源 /order/{orderId} 后面的流控按钮，就可以弹出表单。

![image-20220522172207186](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221722300.png)

![image-20220522172219344](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221722416.png)

其含义是限制 /order/{orderId} 这个资源的单机 QPS 为 1，即每秒只允许 1 次请求，超出的请求会被拦截并报错。

### 流控模式

在添加限流规则时，点击高级选项，可以选择三种**流控模式**

- 直接：统计当前资源的请求，触发阈值时对当前资源直接限流，也是默认的模式
  - 直接对当前资源限流
- 关联：统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流
  - 相当于高优先级资源触发阈值，对低优先级资源限流。
- 链路：统计从指定链路访问到本资源的请求，触发阈值时，对指定链路限流

#### 直接模式

统计当前资源的请求，触发阈值时对当前资源直接限流，也是默认的模式

#### 关联模式

统计与当前资源相关的另一个资源，触发阈值时，对**当前资源限流**。

**使用场景**：比如用户支付时需要修改订单状态，同时用户要查询订单。查询和修改操作会争抢数据库锁，产生竞争。业务需求是优先支付和更新订单的业务，因此当修改订单业务触发阈值时，需要对查询订单业务限流。

满足下面条件可使用关联模式：

- 两个有竞争关系的资源
- 一个优先级高，一个优先级低

![image-20220522172634153](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205221726257.png)

#### 链路模式

只针对从指定链路访问到本资源的请求做统计，判断是否超过阈值。

例如有两条请求链路

- /test1 --> /common
- /test2 --> /common

如果只希望统计从 /test2 进入到 /common 的请求，则可以这样配置

![image-20220523095340543](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205230953616.png)

默认情况下，OrderService 中的方法是不被 Sentinel 监控的，需要我们自己通过注解来标记要监控的方法。

给 OrderService 的 queryGoods 方法添加 `@SentinelResource` 注解。

```java
@SentinelResource("goods")
public void queryGoods(){
    System.err.println("查询商品");
}
```

链路模式中，是对不同来源的两个链路做监控。但是 Sentinel 默认会给进入 SpringMVC 的所有请求设置同一个 root 资源，会导致链路模式失效。我们需要关闭这种对 SpringMVC 的资源聚合，修改 order-service 服务的 application.yml 文件

```yaml
spring:
  cloud:
    sentinel:
      web-context-unify: false # 关闭context整合
```

重启服务，访问 /order/query 和 /order/save，可以查看到 Sentinel 的簇点链路规则中，出现了新的资源

![image-20220523095502949](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205230955019.png)

添加新的流控规则如下

![image-20220523095536488](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205230955621.png)

只统计从 /order/query 进入 /goods 的资源，QPS 阈值为 2，超出则被限流。

## 流控效果

流控效果是指请求达到流控阈值时应该采取的措施，包括三种

- 快速失败：达到阈值后，新的请求会被立即拒绝并抛出 FlowException 异常，是默认的处理方式。**滑动时间窗口算法**
- Warm Up：预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一个较小值逐渐增加到最大阈值。**漏桶算法**
- 排队等待：让所有的请求按照先后次序排队执行，两个请求的间隔不能小于指定时长。**令牌桶算法**

### 快速失败

超过设置的 QPS 直接拒绝

### Warm up

阈值一般是一个微服务能承担的最大 QPS，但是一个服务刚刚启动时，一切资源尚未初始化（**冷启动**），如果直接将 QPS 跑到最大值，可能导致服务瞬间宕机。

Warm Up 也叫**预热模式**，是应对服务冷启动的一种方案。请求阈值初始值是 `maxThreshold / coldFactor`，持续指定时长后，逐渐提高到 maxThreshold 值。而 coldFactor 的默认值是 3.

例如，我设置 QPS 的 maxThreshold 为 10，预热时间为 5 秒，那么初始阈值就是 10 / 3 = 3，然后在 5 秒后逐渐增长到 10

![image-20220523100339713](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231003838.png)

### 排队等待

当请求超过 QPS 阈值时，「快速失败」和 「Warm Up」会拒绝新的请求并抛出异常。

而排队等待则是让所有请求进入一个队列中，然后按照阈值允许的时间间隔依次执行。后来的请求必须等待前面执行完成，如果请求预期的等待时间超出最大时长，则会被拒绝。这种方式严格控制了请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。

例如：QPS = 5，意味着每 200ms 处理一个队列中的请求；timeout = 2000，意味着**预期等待时长**超过 2000ms 的请求会被拒绝并抛出异常。

比如现在一下子来了 12 个请求，因为每 200ms 执行一个请求，那么预期等待时长就是：

- 第6个请求的**预期等待时长** = 200 * (6 - 1) = 1000ms
- 第12个请求的预期等待时长 = 200 * (12-1) = 2200ms

又比如下图：

![image-20220523100418021](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231004138.png)

现在，第 1 秒同时接收到 10 个请求，但第 2 秒只有 1 个请求，此时 QPS 的曲线这样的

![image-20220523100435576](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231004668.png)

如果使用排队等待的流控效果，所有进入的请求都要排队，以固定的 200ms 的间隔执行，QPS 会变的很平滑

![image-20220523100443780](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231004880.png)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。起到了流量整形效果。

## 热点参数限流

之前的限流是统计访问某个资源的所有请求，判断是否超过 QPS 阈值。而「热点参数限流」是**分别统计参数值相同的请求**，判断是否超过 QPS 阈值。

### 全局参数限流

例如，一个根据 id 查询商品的接口

访问/goods/{id} 的请求中，id 参数值会有变化，「热点参数限流」会根据参数值分别统计 QPS，统计结果：

![image-20220523101911763](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231019846.png) 

当 id=1 的请求触发阈值被限流时，id值不为1的请求则不受影响。

配置示例：对 hot 这个资源的 0 号参数（也就是第一个参数）做统计，每 1s **相同参数值**的请求数不能超过 5

![image-20220523102217175](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231022267.png)

### 热点参数限流

在实际开发中，可能部分商品是热点商品，例如秒杀商品，我们希望这部分商品的 QPS 限制与其它商品不一样，高一些。那就需要配置「热点参数限流」的高级选项了。

![image-20220523102307025](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231023140.png)

结合上一个配置，这里的含义是对 0 号的 long 类型参数限流，每 1 个相同参数的 QPS 不能超过 5，有如下两个例外

- 如果参数值是 100，则每 1s 允许的 QPS 为 10
- 如果参数值是 101，则每 1s 允许的 QPS 为 15

**注意事项**：热点参数限流对默认的 SpringMVC 资源无效，需要利用 `@SentinelResource` 注解在 controller 对应方法上标记资源。

![image-20220523102445492](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231024567.png)

## 隔离和降级

限流只是一种预防措施，虽然限流可以尽量避免因高并发而引起的服务故障，但服务还会因为其它原因而故障。

而要将这些故障控制在一定范围，避免雪崩，就要靠**线程隔离**（舱壁模式）和**熔断降级**手段了。Sentinel 支持的雪崩解决方案为线程隔离和熔断降级。

**线程隔离**：调用者在调用服务提供者时，给每个调用的请求分配独立线程池，出现故障时，最多消耗这个线程池内资源，避免把调用者的所有资源耗尽。

![image-20220523102757711](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231027823.png)

**熔断降级**：是在调用方这边加入断路器，统计对服务提供者的调用，如果调用的失败比例过高，则熔断该业务，不允许访问该服务的提供者了。

![image-20220523102813561](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231028682.png)

可以看到，不管是线程隔离还是熔断降级，都是对**客户端**（调用方）的保护。需要在**调用方**发起远程调用时做线程隔离、或者服务熔断。

而我们的微服务远程调用都是基于 Feign 来完成的，因此我们需要将 Feign 与 Sentinel 整合，在 Feign 里面实现线程隔离和服务熔断。

### Feign 整合 Sentinel

SpringCloud中，微服务调用都是通过Feign来实现的，因此做客户端保护必须整合 Feign 和 Sentinel

修改配置，开启 Sentinel 功能，修改 OrderService 的 application.yml 文件，开启 Feign 的 Sentinel 功能

```yaml
feign:
  sentinel:
    enabled: true # 开启feign对sentinel的支持
```

服务降级：访问失败后，服务**编写失败降级逻辑代码**，业务失败后，不能直接报错，而应该返回用户一个友好提示或者默认结果，这个就是失败降级逻辑。

给 FeignClient 编写失败后的降级逻辑

①方式一：FallbackClass，但无法对远程调用的异常做处理。

②方式二：FallbackFactory，可以对远程调用的异常做处理，通常使用这种

**步骤一**：

编写 FallbackFactory 接口的实现类 UserClientFallbackFactory

![image-20220523103812722](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231038815.png)

```java
@Slf4j
public class UserClientFallbackFactory implements FallbackFactory<UserClient> {
    @Override
    public UserClient create(Throwable throwable) {
       return userClient -> {
           log.error("查询用户失败",throwable);
           return new User();
       };
    }
}
```

**步骤二**：

将 UserClientFallbackFactory  注册为 bean

```java
@Bean
public UserClientFallbackFactory userClientFallbackFactory(){
    return new UserClientFallbackFactory();
}
```

**步骤三**：

在 client 接口上添加 **fallbackFactory** 选项

```java
@FeignClient(value = "userservice", fallbackFactory = UserClientFallbackFactory.class)
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

![image-20220523103959839](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231039967.png)

### 线程隔离

线程隔离（舱壁模式）有两种方式实现

- 线程池隔离：给每个服务调用业务分配一个线程池，利用线程池本身实现隔离效果。
- 信号量隔离（Sentinel默认采用）：不创建线程池，而是计数器模式，记录业务使用的线程数量，达到信号量上限时，禁止新的请求。

![image-20220523104649521](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231046650.png)



两者的优缺点

- 线程池隔离：基于线程池模式，有额外开销，但隔离控制更强
- 信号量隔离：基于计数器模式，简单，开销小

![image-20220523104729049](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231047171.png)

**Sentinel 使用的是信号量隔离**，而 Hystrix 则两种线程隔离都可以，18 年Hystrix已经停止更新。

如何使用呢，在添加限流规则时，可以选择两种阈值类型

![image-20220523104849793](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231048866.png)

- QPS：就是每秒的请求数，之前已经演示过。
- 线程数：是该资源能使用的 Tomcat 线程数的最大值。也就是通过限制线程数量，实现**线程隔离**（舱壁模式）。

### 熔断降级

熔断降级是解决雪崩问题的重要手段。其思路是由**断路器**统计服务调用的异常比例、慢请求比例，如果超出阈值则会**熔断**该服务。即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求。

断路器控制熔断和放行是通过状态机来完成的，如下图就是一个断路器的状态机

![image-20220523105117350](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231051440.png)

状态机包括三个状态

- closed：关闭状态，断路器放行所有请求，并开始统计异常比例、慢请求比例。会去判断是否达到熔断条件，这一步我们叫做「熔断策略」，达到该条件则切换到 open 状态，打开断路器。
- open：打开状态，服务调用被**熔断**，访问被熔断服务的请求会被拒绝，快速失败，直接走降级逻辑。Open 状态 5 秒后会进入 half-open 状态。
- half-open：半开状态，会一段时间放行一次请求，根据执行结果来判断接下来的操作。请求成功：则切换到 closed 状态；请求失败：则切换到 open 状态。

断路器熔断策略有三种：慢调用、异常比例、异常数

#### 慢调用

业务的响应时长（RT）大于指定时长的请求认定为慢调用请求。在指定时间内，如果请求数量超过设定的最小数量，慢调用比例大于设定的阈值，则触发熔断。

例如下图，设置 RT 超过 500ms 的调用是慢调用，统计最近 10000ms 内的请求，如果请求量超过 10 次，并且慢调用比例不低于 0.5，则触发熔断，熔断时长为 5s，然后进入 half-open 状态，放行一次请求做测试。

![image-20220523105343121](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231053241.png)

#### 异常

**异常比例或异常数**：统计指定时间内的调用，如果调用次数超过指定请求数，并且**出现异常**的比例达到设定的比例阈值（或超过指定异常数），则触发熔断。

例如，异常比例设置如下，统计最近 1000ms 内的请求，如果请求量超过 10 次，并且**异常比例**不低于 0.4，则触发熔断。

![image-20220523105738233](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231057308.png)

异常数设置如下，统计最近 1000ms 内的请求，如果请求量超过 10 次，并且**异常数**不低于 2 次，则触发熔断。

![image-20220523105808490](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231058586.png)

## 授权规则

之前在 SpringCloud 网关中，可通过网关来判断访问是否可通行，但是若微服务的内部地址泄露，直接通过地址访问，便绕过了网关，这时网关无法判断。授权规则可以对请求方来源做判断和控制。授权规则可以对调用方的来源做控制，有白名单和黑名单两种方式。

比如我们希望控制对资源 `test` 的访问设置白名单，只有来源为 `appA` 和 `appB` 的请求才可通过。

- 白名单：来源（origin）在白名单内的调用者允许访问
- 黑名单：来源（origin）在黑名单内的调用者不允许访问

点击左侧菜单的授权，可以看到授权规则

![image-20220523110915119](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231109207.png)

- 资源名：就是受保护的资源，例如 /order/{orderId}
- 流控应用：是来源者的名单
  - 如果是勾选白名单，则名单中的来源被许可访问。
  - 如果是勾选黑名单，则名单中的来源被禁止访问。

比如我们允许请求从 gateway 到 order-service，不允许浏览器访问 order-service，那么白名单中就要填写**网关的来源名称（origin）**。

![image-20220523110945031](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231109157.png)



Sentinel 是通过 RequestOriginParser 这个接口的 parseOrigin() 方法来获取请求的来源的。

这个方法的作用就是从 request 对象中，获取请求者的 origin 值并返回。默认情况下，Sentinel 不管请求者从哪里来，返回值永远是 default，也就是说一切请求的来源都被认为是一样的值 default

因此，我们需要自定义这个接口的实现，让**不同的请求，返回不同的 origin**

例如 order-service 服务中，我们定义一个 RequestOriginParser 的实现类

```java
@Component
public class HeaderOriginParser implements RequestOriginParser {
    @Override
    public String parseOrigin(HttpServletRequest request) {
        // 1.获取请求头
        String origin = request.getHeader("origin");
        // 2.非空判断
        if (StringUtils.isEmpty(origin)) {
            origin = "blank";
        }
        return origin;
    }
}
```

我们必须让**所有从 Gateway 路由到微服务的请求都带上 origin 头**。这个需要利用之前学习的一个 GatewayFilter 来实现，使用 AddRequestHeaderGatewayFilter，修改 Gateway 服务中的 application.yml

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - AddRequestHeader=origin,gateway #origin前面不能带空格，这表示 origin 的值为 gateway
```

这样，从 Gateway 路由的所有请求都会带上 origin 头，值为 gateway。而从其它地方到达微服务的请求一般没有这个头。

接下来，我们添加一个授权规则，放行 origin 值为 gateway 的请求。那么跳过网关直接访问将会被 Sentinel 拦截，通过网关访问的将会放行。

![image-20220523111201119](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231112233.png)

## 自定义异常结果

默认情况下，发生限流、降级、授权拦截时，都会抛出异常到调用方。异常结果都是 flow limmiting（限流）。这样不够友好，无法得知是限流还是降级还是授权拦截。

而如果要自定义异常时的返回结果，需要实现 BlockExceptionHandler 接口

```java
public interface BlockExceptionHandler {
    /**
     * 处理请求被限流、降级、授权拦截时抛出的异常：BlockException
     */
    void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception;
}
```

这个方法有三个参数：

- HttpServletRequest request：request 对象
- HttpServletResponse response：response 对象
- BlockException e：被 Sentinel 拦截时抛出的异常

这里的 BlockException 包含多个不同的子类

| **异常**             | **说明**           |
| :------------------- | :----------------- |
| FlowException        | 限流异常           |
| ParamFlowException   | 热点参数限流的异常 |
| DegradeException     | 降级异常           |
| AuthorityException   | 授权规则异常       |
| SystemBlockException | 系统规则异常       |

自定义异常处理，下面我们就在 order-service 定义一个自定义异常处理类

```java
@Component
public class SentinelExceptionHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        String msg = "未知异常";
        int status = 429;

        if (e instanceof FlowException) {
            msg = "请求被限流了";
        } else if (e instanceof ParamFlowException) {
            msg = "请求被热点参数限流";
        } else if (e instanceof DegradeException) {
            msg = "请求被降级了";
        } else if (e instanceof AuthorityException) {
            msg = "没有权限访问";
            status = 401;
        }

        response.setContentType("application/json;charset=utf-8");
        response.setStatus(status);
        response.getWriter().println("{\"msg\": " + msg + ", \"status\": " + status + "}");
    }
}
```

## 规则持久化

现在，Sentinel 的所有规则都是内存存储，重启后所有规则都会丢失。在生产环境下，我们必须确保这些规则的持久化，避免丢失。

规则是否能持久化，取决于规则管理模式，Sentinel 支持三种规则管理模式

- 原始模式：Sentinel 的默认模式，将规则保存在内存，重启服务会丢失。
- pull 模式
- push 模式

### pull模式

pull 模式：控制台将配置的规则推送到 Sentinel 客户端，而客户端会将配置规则保存在本地文件或数据库中。以后会定时去本地文件或数据库中查询，更新本地规则。

![image-20220523112224007](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231122150.png)

### push模式

push 模式：控制台将配置规则推送到远程配置中心，例如 Nacos，Sentinel 客户端监听 Nacos，获取配置变更的推送消息，完成本地配置更新。这种模式是比较好的一种。

![image-20220523112240588](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231122713.png)

# 分布式事务

## 概述

随着互联网的快速发展，软件系统由原来的单体应用转变为分布式应用，下图描述了单体应用向微服务的演变：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231641079.jpeg)

分布式系统会把一个应用系统拆分为可独立部署的多个服务，因此需要服务与服务之间远程协作才能完成事务操作，这种分布式系统环境下由不同的服务之间通过网络远程协作完成事务称之为**分布式事务**，例如用户注册送积分事务、创建订单减库存事务，银行转账事务等都是分布式事务。

在分布式环境下，如下场景：

```sql
begin transaction；
    //1.本地数据库操作：张三减少金额
    //2.远程调用：让李四增加金额
commit transation;
```

可以设想，当远程调用让李四增加金额成功了，由于网络问题远程调用并没有返回，此时本地事务提交失败就回滚了张三减少金额的操作，此时张三和李四的数据就不一致了。

## 分布式事务产生的场景

### 1、跨JVM进程产生分布式事务

典型的场景就是微服务架构：微服务之间通过远程调用完成事务操作。比如：订单微服务和库存微服务，下单的同时订单微服务请求库存微服务减少库存。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231643241.jpeg)

### 2、跨数据库实例产生分布式事务

单体系统访问多个数据库实例当单体系统需要访问多个数据库（实例）时就会产生分布式事务。比如：用户信息和订单信息分别在两个MySQL实例存储。

![image-20220523164419606](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231644681.png)

### 3、多服务访问同一个数据库实例

订单微服务和库存微服务即使访问同一个数据库也会产生分布式事务，原因就是跨JVM进程，两个微服务持有了不同的数据库链接进行数据库操作，此时产生分布式事务。

![image-20220523164724399](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231647476.png)

## 分布式事务基础理论

### CAP理论

CAP 是 Consistency、Availability、Partition tolerance 三个单词的缩写，分别表示一致性、可用性、分区容忍性。

如下图，是商品信息管理的执行流程：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231649973.jpeg)

整体执行流程如下

1. 商品服务请求主数据库写入商品信息（添加商品、修改商品、删除商品）
2. 主数据库向商品服务响应写入成功
3. 商品服务请求从数据库读取商品信息

### C - Consistency

  一致性是指写操作后的读操作可以读取到最新的数据状态，当数据分布在多个节点上，从任意结点读取到的数据都是最新的状态。

上图中，商品信息的读写要满足一致性就是要实现如下目标：

1. 商品服务写入主数据库成功，则向从数据库查询新数据也成功。
2. 商品服务写入主数据库失败，则向从数据库查询新数据也失败。

如何实现一致性？

1. 写入主数据库后要将数据同步到从数据库。
2. 写入主数据库后，在向从数据库同步期间要将从数据库锁定，待同步完成后再释放锁，以免在新数据写入成功后，向从数据库查询到旧的数据。

分布式系统一致性的特点：

1. **由于存在数据同步的过程，写操作的响应会有一定的延迟。**
2. **为了保证数据一致性会对资源暂时锁定，待数据同步完成释放锁定资源。**
3. **对于请求数据同步失败的结点则会返回错误信息，一定不会返回旧数据。**

### A - Availability

可用性是指任何事务操作都可以得到响应结果，且不会出现响应超时或响应错误。

上图中，商品信息读取满足可用性就是要实现如下目标：

1. 从数据库接收到数据查询的请求则立即能够响应数据查询结果。
2. 从数据库不允许出现响应超时或响应错误。

如何实现可用性

1. 写入主数据库后要将数据同步到从数据库。
2. 由于要保证从数据库的可用性，不可将从数据库中的资源进行锁定。
3. 即时数据还没有同步过来，从数据库也要返回要查询的数据，哪怕是旧数据，如果连旧数据也没有则可以按照约定返回一个默认信息，但不能返回错误或响应超时。

分布式系统可用性的特点：**所有请求都有响应，且不会出现响应超时或响应错误**

### P - Partition tolerance

通常分布式系统的各个结点部署在不同的子网，这就是网络分区，不可避免的会出现由于网络问题而导致结点之间通信失败，此时仍可对外提供服务，这叫分区容忍性。

上图中，商品信息读写满足分区容忍性就是要实现如下目标：

1. 主数据库向从数据库同步数据失败不影响读写操作。
2. 其一个结点挂掉不影响另一个结点对外提供服务。

如何实现分区容忍性？

1. 尽量使用异步取代同步操作，例如使用异步方式将数据从主数据库同步到从数据，这样结点之间能有效的实现松耦合。
2. 添加从数据库结点，其中一个从结点挂掉其它从结点提供服务。

分布式分区容忍性的特点：**分区容忍性分是分布式系统具备的基本能力**

### CAP 组合方式

**在所有分布式事务场景中不会同时具备 CAP 三个特性，因为在具备了P的前提下C和A是不能共存的**

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231702878.jpeg)

满足分区容忍性，含义是：

1. 主数据库通过网络向从数据库同步数据，可以认为主从数据库部署在不同的分区，通过网络进行交互。
2. 当主数据库和从数据库之间的网络出现问题不影响主数据库和从数据库对外提供服务。
3. 其中一个节点挂掉不影响另一个节点对外提供服务。

如果要实现 C 则必须保证**数据一致性**，在数据同步的时候为防止向从数据库查询不一致的数据则需要将从数据库数据锁定，待同步完成后解锁，如果同步失败从数据库要返回错误信息或超时信息。

如果要实现 A 则必须保证**数据可用性**，在主数据库向从数据库进行同步信息时不能加同步锁，不管任何时候都可以向从数据查询数据，则不会响应超时或返回错误信息。通过分析发现在满足P的前提下 C 和 A 存在矛盾性。

1. **AP**
   放弃一致性，追求分区容忍性和可用性。这是很多分布式系统设计时的选择。
   例如：上边的商品管理，完全可以实现 AP，前提是只要用户可以接受所查询到的数据在一定时间内不是最新的即可。
   通常实现 AP 都会保证最终一致性，后面将的 **BASE** 理论就是根据 AP 来扩展的，一些业务场景比如：订单退款，今日退款成功，明日账户到账，只要用户可以接受在一定的时间内到账即可。
2. **CP**
   放弃可用性，追求一致性和分区容错性，zookeeper 其实就是追求的强一致，又比如跨行转账，一次转账请求要等待双方银行系统都完成整个事务才算完成。
3. **CA**
   放弃分区容忍性，即不进行分区，不考虑由于网络不通或结点挂掉的问题，则可以实现一致性和可用性。那么系统将不是一个标准的分布式系统，最常用的关系型数据就满足了 CA。

CAP 是一个已经被证实的理论，一个分布式系统最多只能同时满足：一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）这三项中的两项。对于多数大型互联网应用的场景，结点众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，要达到良好的响应性能来提高用户体验，因此一般都会做出如下选择：保证 P 和 A ，舍弃 C 强一致，保证最终一致性。

### BASE 理论

**强一致性和最终一致性**

CAP 理论告诉我们一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容忍性（Partition tolerance）这三项中的两项，其中AP在实际应用中较多，AP 即舍弃一致性，保证可用性和分区容忍性，但是在实际生产中很多场景都要实现一致性，比如前边我们举的例子主数据库向从数据库同步数据，即使不要一致性，但是最终也要将数据同步成功来保证数据一致，这种一致性和 CAP 中的一致性不同，CAP 中的一致性要求 在任何时间查询每个结点数据都必须一致，它强调的是强一致性，但是最终一致性是允许可以在一段时间内每个结点的数据不一致，但是经过一段时间每个结点的数据必须一致，它强调的是最终数据的一致性。

**Base 理论介绍**

BASE 是 Basically Available（基本可用）、Soft state（软状态）和 Eventually consistent （最终一致性）三个短语的缩写。BASE 理论是对 CAP 中 AP 的一个扩展，通过牺牲强一致性来获得可用性，当出现故障允许部分不可用但要保证核心功能可用，允许数据在一段时间内是不一致的，但最终达到一致状态。满足BASE理论的事务，我们称之为“**柔性事务**”。

1. **基本可用**：分布式系统在出现故障时，允许损失部分可用功能，保证核心功能可用。如电商网站交易付款出现问题了，商品依然可以正常浏览。
2. **软状态**：由于不要求强一致性，所以BASE允许系统中存在中间状态（也叫**软状态**），这个状态不影响系统可用性，如订单的"支付中"、“数据同步中”等状态，待数据最终一致后状态改为“成功”状态。
3. **最终一致**：最终一致是指经过一段时间后，所有节点数据都将会达到一致。如订单的"支付中"状态，最终会变 为“支付成功”或者"支付失败"，使订单状态与实际交易结果达成一致，但需要一定时间的延迟、等待。

## 分布式事务解决方案 2PC

### 原理

针对不同的分布式场景业界常见的解决方案有 2PC、3PC、TCC、可靠消息最终一致性、最大努力通知这几种。

2PC 即两阶段提交协议，是将整个事务流程分为两个阶段，准备阶段（Prepare phase）、提交阶段（commit phase），2 是指两个阶段，P 是指准备阶段，C 是指提交阶段。

举例：张三和李四好久不见，老友约起聚餐，饭店老板要求先买单，才能出票。这时张三和李四分别抱怨近况不如意，囊中羞涩，都不愿意请客，这时只能AA。只有张三和李四都付款，老板才能出票安排就餐。但由于张三和李四都是铁公鸡，形成了尴尬的一幕：

准备阶段：老板要求张三付款，张三付款。老板要求李四付款，李四付款。

提交阶段：老板出票，两人拿票纷纷落座就餐。

例子中形成了一个事务，若张三或李四其中一人拒绝付款，或钱不够，店老板都不会给出票，并且会把已收款退回。

整个事务过程由事务管理器和参与者组成，店老板就是事务管理器，张三、李四就是事务参与者，事务管理器负责决策整个分布式事务的提交和回滚，事务参与者负责自己本地事务的提交和回滚。

在计算机中部分关系数据库如 Oracle、MySQL 支持两阶段提交协议，如下图：

1. 准备阶段（Prepare phase）：事务管理器给每个参与者发送 Prepare 消息，每个数据库参与者在本地执行事务，并写本地的 Undo/Redo 日志，此时事务没有提交。（Undo 日志是记录修改前的数据，用于数据库回滚，Redo 日志是记录修改后的数据，用于提交事务后写入数据文件）
2. 提交阶段（commit phase）：如果事务管理器收到了参与者的执行失败或者超时消息时，直接给每个参与者发送回滚（Rollback）消息；否则，发送提交（Commit）消息；参与者根据事务管理器的指令执行提交或者回滚操作，并释放事务处理过程中使用的锁资源。注意：**必须在最后阶段释放锁资源**。

下图展示了2PC的两个阶段，分成功和失败两个情况说明：

成功情况

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231721919.jpeg)

失败情况

![image-20220523172155310](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231721403.png)

### 解决方案

#### XA 方案

2PC 的传统方案是在数据库层面实现的，如 Oracle、MySQL 都支持 2PC 协议。国际开放标准组织 Open Group 定义了分布式事务处理模型 **DTP**。

以新用户注册送积分为例：

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205231728930.jpeg)

执行流程如下：

1. 应用程序（AP）持有用户库和积分库两个数据源。
2. 应用程序（AP）通过 TM 通知用户库 RM 新增用户，同时通知积分库RM为该用户新增积分，RM 此时并未提交事务，此时用户和积分资源锁定。
3. TM 收到执行回复，只要有一方失败则分别向其他 RM 发起回滚事务，回滚完毕，资源锁释放。
4. TM 收到执行回复，全部成功，此时向所有 RM 发起提交事务，提交完毕，资源锁释放。

DTP 模型定义如下角色：

- **AP**（Application Program）：即应用程序，可以理解为使用 DTP 分布式事务的程序。
- **RM**（Resource Manager）：即资源管理器，可以理解为事务的参与者，一般情况下是指一个数据库实例，通过资源管理器对该数据库进行控制，资源管理器控制着分支事务。
- **TM**（Transaction Manager）：事务管理器，负责协调和管理事务，事务管理器控制着全局事务，管理事务生命周期，并协调各个 RM。**全局事务**是指分布式事务处理环境中，需要操作多个数据库共同完成一个工作，这个工作即是一个全局事务。

DTP 模型定义TM和RM之间通讯的接口规范叫 **XA**，简单理解为数据库提供的 2PC 接口协议，**基于数据库的 XA 协议来实现 2PC 又称为 XA 方案**

以上三个角色之间的交互方式如下：

1. TM 向 AP 提供 应用程序编程接口，AP 通过 TM 提交及回滚事务。
2. TM 交易中间件通过 XA 接口来通知 RM 数据库事务的开始、结束以及提交、回滚等。

**总结**

整个 2PC 的事务流程涉及到三个角色 AP、RM、TM。AP 指的是使用 2PC 分布式事务的应用程序；RM 指的是资源管理器，它控制着分支事务；TM 指的是事务管理器，它控制着整个全局事务。

（1）**准备阶段**： RM 执行实际的业务操作，但不提交事务，资源锁定

（2）**提交阶段**： TM 会接受 RM 在准备阶段的执行回复，只要有任一个RM执行失败，TM 会通知所有 RM 执行回滚操作，否则，TM 将会通知所有 RM 提交该事务。提交阶段结束资源锁释放。

**XA方案的问题**

1. 需要本地数据库支持XA协议。
2. 资源锁需要等到两个阶段结束才释放，性能较差。

#### Seata 方案

Seata 是由阿里中间件团队发起的开源项目 Fescar，后更名为 Seata，它是一个是开源的分布式事务框架。

传统 2PC 的问题在 Seata 中得到了解决，它通过对本地关系数据库的分支事务的协调来驱动完成全局事务，是工作在应用层的中间件。主要优点是性能较好，且不长时间占用连接资源，它以高效并且对业务 0 侵入的方式解决微服务场景下面临的分布式事务问题，它目前提供 AT 模式（即 2PC）及 TCC 模式的分布式事务解决方案。

**Seata 的设计思想如下：**

Seata 把一个分布式事务理解成一个包含了若干**分支事务**的**全局事务**。全局事务的职责是协调其下管辖的分支事务达成一致，要么一起成功提交，要么一起失败回滚。此外，通常分支事务本身就是一个关系数据库的本地事务，下图是全局事务与分支事务的关系图：

![image-20220524113046020](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205241130107.png)

与传统 2PC 的模型类似，Seata 定义了 3 个组件来协议分布式事务的处理过程：

- Transaction Coordinator（TC）：事务协调器，它是独立的中间件，需要独立部署运行，它维护全局事务的运行状态，接收 TM 指令发起全局事务的提交与回滚，负责与 RM 通信协调各各分支事务的提交或回滚。
- Transaction Manager（TM）： 事务管理器，TM 需要嵌入应用程序中工作，它负责开启一个全局事务，并最终向 TC 发起全局提交或全局回滚的指令。
- Resource Manager（RM）：控制分支事务，负责分支注册、状态汇报，并接收事务协调器 TC 的指令，驱动分支（本地）事务的提交和回滚。

**Seata实现2PC与传统2PC的差别：**

架构层次方面：传统 2PC 方案的 RM 实际上是在数据库层，RM 本质上就是数据库自身，通过 XA 协议实现，而 Seata 的 RM 是以 jar 包的形式作为中间件层部署在应用程序这一侧的。

两阶段提交方面：传统 2PC无论第二阶段的决议是 commit 还是 rollback ，事务性资源的锁都要保持到 Phase2 完成才释放。而 Seata 的做法是在 Phase1 就将本地事务提交，这样就可以省去 Phase2 持锁的时间，整体提高效率。

#### 小结

对于传统 2PC（基于数据库 XA 协议）和 Seata 实现 2PC 的两种方案，于 Seata 的 0 侵入性并且解决了传统 2PC 长期锁资源的问题，推荐采用 Seata 实现 2PC。

## 分布式事务解决方案 TCC

### 原理

TCC 是 Try、Conﬁrm、Cancel 三个词语的缩写，TCC 要求每个分支事务实现三个操作：预处理 Try、确认 Conﬁrm、撤销 Cancel。Try 操作做业务检查及资源预留，Conﬁrm 做业务确认操作，Cancel 实现一个与 Try 相反的操作即回滚操作。

TM 首先发起所有的分支事务的 Try 操作，任何一个分支事务的Try操作执行失败，TM 将会发起所有分支事务的 Cancel 操作，若 Try 操作全部成功，TM 将会发起所有分支事务的 Conﬁrm 操作，其中 Conﬁrm/Cancel 操作若执行失败，TM 会进行重试。

TCC 分为三个阶段：

1. **Try** 阶段是做完业务检查（一致性）及资源预留（隔离），此阶段仅是一个初步操作，它和后续的 Conﬁrm 一起才能真正构成一个完整的业务逻辑。
2. **Confirm** 阶段是做确认提交，Try 阶段所有分支事务执行成功后开始执行 Conﬁrm。通常情况下，采用 TCC 则认为 Conﬁrm 阶段是不会出错的。即：只要 Try 成功，Conﬁrm 一定成功。若 Conﬁrm 阶段真的出错了，需引入重试机制或人工处理。
3. **Cancel** 阶段是在业务执行错误需要回滚的状态下执行分支事务的业务取消，预留资源释放。通常情况下，采用 TCC 则认为 Cancel 阶段也是一定成功的。若 Cancel 阶段真的出错了，需引入重试机制或人工处理。

### TCC 异常处理

TCC需要注意三种异常处理分别是**空回滚**、**幂等**、**悬挂**

**空回滚**

在没有调用 TCC 资源 Try 方法的情况下，调用了二阶段的 Cancel 方法，Cancel 方法需要识别出这是一个空回滚，直接返回成功。

出现原因是当一个分支事务所在服务宕机或网络异常，分支事务调用记录为失败，这个时候其实是没有执行 Try 阶段，当故障恢复后，分布式事务进行回滚则会调用二阶段的 Cancel 方法，从而形成空回滚。

解决思路是关键就是要识别出这个空回滚。思路很简单就是需要知道一阶段是否执行，如果执行了，那就是正常回滚；如果没执行，那就是空回滚。

前面已经说过 TM 在发起全局事务时生成全局事务记录，全局事务 ID 贯穿整个分布式事务调用链条。再额外增加一张分支事务记录表，其中有全局事务 ID 和分支事务 ID，第一阶段 Try 方法里会插入一条记录，表示一阶段执行了。Cancel 接口里读取该记录，如果该记录存在，则正常回滚；如果该记录不存在，则是空回滚。

**幂等**

通过前面介绍已经了解到，为了保证 TCC 二阶段提交重试机制不会引发数据不一致，要求 TCC 的二阶段 Try、Conﬁrm 和 Cancel 接口保证幂等，这样不会重复使用或者释放资源。如果幂等控制没有做好，很有可能导致数据不一致等严重问题。

解决思路在上述"分支事务记录"中增加执行状态，每次执行前都查询该状态。

**悬挂**

悬挂就是对于一个分布式事务，其二阶段 Cancel 接口比 Try 接口先执行，导致 Try 执行后预留的业务资源无法被继续处理。

出现原因是在 RPC 调用分支事务 Try 时，先注册分支事务，再执行 RPC 调用，如果此时 RPC 调用的网络发生拥堵，通常 RPC 调用是有超时时间的，RPC 超时以后，TM 就会通知 RM 回滚该分布式事务，可能回滚完成后，RPC 请求才到达参与者真正执行，而一个 Try 方法预留的业务资源，只有该分布式事务才能使用，该分布式事务第一阶段预留的业务资源就再也没有人能够处理了，对于这种情况，我们就称为悬挂，即业务资源预留后没法继续处理。

解决思路是如果二阶段执行完成，那一阶段就不能再继续执行。在执行一阶段事务时判断在该全局事务下，"分支事务记录"表中是否已经有二阶段事务记录，如果有则不执行 Try。

### 小结

如果拿 TCC 事务的处理流程与 2PC 两阶段提交做比较，2PC 通常都是在跨库的 DB 层面，而 TCC 则在应用层面的处理，需要通过业务逻辑来实现。这种分布式事务的实现方式的优势在于，可以让**应用自己定义数据操作的粒度，使得降低锁冲突、提高吞吐量成为可能**。

而不足之处则在于对应用的侵入性非常强，业务逻辑的每个分支都需要实现 Try、Conﬁrm、Cancel 三个操作。此外，其实现难度也比较大，需要按照网络状态、系统故障等不同的失败原因实现不同的回滚策略。

## 分布式事务解决方案 可靠消息最终一致性

### 原理

可靠消息最终一致性方案是指当事务发起方执行完成本地事务后并发出一条消息，事务参与方（消息消费者）一定能够接收消息并处理事务成功，此方案强调的是只要消息发给事务参与方最终事务要达到一致。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205241152675.png)

可靠消息最终一致性方案要解决以下几个问题：

1、**本地事务与消息发送的原子性问题**
本地事务与消息发送的原子性问题即：事务发起方在本地事务执行成功后消息必须发出去，否则就丢弃消息。即实现本地事务和消息发送的原子性，要么都成功，要么都失败。

2、**事务参与方接收消息的可靠性**

事务参与方必须能够从消息队列接收到消息，如果接收消息失败可以重复接收消息。

3、**消息重复消费的问题**

由于网络2的存在，若某一个消费节点超时但是消费成功，此时消息中间件会重复投递此消息，就导致了消息的重复消费。

### 解决方案

#### 本地消息表

通过本地事务保证数据业务操作和消息的一致性，然后通过定时任务将消息发送至消息中间件，待确认消息发送给消费方成功再将消息删除。

本地消息表与业务数据表处于同一个数据库中，这样就能利用本地事务来保证在对这两个表的操作满足事务特性，并且使用了消息队列来保证最终一致性。核心思想是将分布式事务拆分成本地事务进行处理。

本地消息表顾名思义就是会有一张存放本地消息的表，一般都是放在数据库中，然后在执行业务的时候 **将业务的执行和将消息放入消息表中的操作放在同一个事务中**，这样就能保证消息放入本地表中业务肯定是执行成功的。

1. 在分布式事务操作的一方完成写业务数据的操作之后向本地消息表发送一个消息，本地事务能保证这个消息一定会被写入本地消息表中。
2. 之后将本地消息表中的消息转发到消息队列中，如果转发成功则将消息从本地消息表中删除，否则继续重新转发。
3. 在分布式事务操作的另一方从消息队列中读取一个消息，并执行消息中的操作。

**优点：** 一种非常经典的实现，避免了分布式事务，实现了最终一致性。在 .NET中 有现成的解决方案。

**缺点：** 消息表会耦合到业务系统中，如果没有封装好的解决方案，会有很多杂活需要处理。

### 小结

可靠消息最终一致性就是保证消息从生产方经过消息中间件传递到消费方的一致性：

1. **本地事务与消息发送的原子性**问题。
2. 事务参与方接收消息的可靠性。

可靠消息最终一致性事务适合执行周期长且实时性要求不高的场景。引入消息机制后，同步的事务操作变为基于消息执行的异步操作, 避免了分布式事务中的同步阻塞操作的影响，并实现了两个服务的解耦。

## 分布式事务解决方案 最大努力通知

### 原理

以充值为例

![image-20220524120056882](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205241200973.png)

1. 账户系统调用充值系统接口
2. 充值系统完成支付处理向账户发起充值结果通知，若通知失败，则充值系统按策略进行重复通知
3. 账户系统接收到充值结果通知修改充值状态
4. 账户系统未接收到通知会主动调用充值系统的接口查询充值结果

最大努力通知方案的目标：**发起通知方通过一定的机制最大努力将业务处理结果通知到接收方**。

具体包括：

1. 有一定的消息重复通知机制。因为接收通知方可能没有接收到通知，此时要有一定的机制对消息重复通知
2. 消息校对机制。如果尽最大努力也没有通知到接收方，或者接收方消费消息后要再次消费，此时可由接收方主动向通知方查询消息信息来满足需求。

最大努力通知与可靠消息一致性有什么不同？

1. **解决方案思想不同**
   可靠消息一致性，发起通知方需要保证将消息发出去，并且将消息发到接收通知方，消息的可靠性关键由发起通知方来保证。最大努力通知，发起通知方尽最大的努力将业务处理结果通知为接收通知方，但是可能消息接收不到，此时需要接 收通知方主动调用发起通知方的接口查询业务处理结果，通知的可靠性关键在接收通知方。
2. **两者的业务应用场景不同**
   可靠消息一致性关注的是交易过程的事务一致，以异步的方式完成交易。最大努力通知关注的是交易后的通知事务，即将交易结果可靠的通知出去。
3. **技术解决方向不同**
   可靠消息一致性要解决消息从发出到接收的一致性，即消息发出并且被接收到。最大努力通知无法保证消息从发出到接收的一致性，只提供消息接收的可靠性机制。可靠机制是，最大努力的将消息通知给接收方，当消息无法被接收方接收时，由接收方主动查询消息（业务处理结果）

### 解决方案

#### 消息队列的 ack 机制

采用 MQ 的 ack 机制就可以实现最大努力通知。

方案1：

1. 发起方将通知发给 MQ。使用普通消息机制将通知发给MQ。
   注意：**如果消息没有发出去可由接收方主动请求发起通知方查询业务执行结果。**（后边会讲）
2. 接收方监听 MQ。
3. 接收方接收消息，业务处理完成回应 ack。
4. 接收方若没有回应 ack 则 MQ 会重复通知。
   MQ会按照间隔 1min、5min、10min、30min、1h、2h、5h、10h的方式，逐步拉大通知间隔，直到达到通知要求的时间窗口上限。
5. 接收方可通过消息校对接口来校对消息的一致性。

方案 2：

1. 发起通知方将消息发给 MQ。

2. 通知程序监听 MQ，接收 MQ 的消息。

   方案 1 中接收通知方直接监听 MQ，方案 2 中由通知程序监听 MQ。通知程序若没有回应 ack 则 MQ 会重复通知。

3. 通知程序通过互联网接口协议（如 http、webservice）调用接收通知方案接口，完成通知。

4. 接收通知方可通过消息校对接口来校对消息的一致性。

**方案1和方案2的不同点**：

1. 方案 1 中接收方监听 MQ，此方案主要应用与内部应用之间的通知。
2. 方案 2 中由通知程序监听 MQ，收到 MQ 的消息后由通知程序通过互联网接口协议调用接收方。此方案主要应用于外部应用之间的通知，例如支付宝、微信的支付结果通知。

## 小结

**2PC**：最大的诟病是一个阻塞协议。这种设计不能适应随着事务涉及的服务数量增加而扩展的需要，很难用于并发较高以及子事务生命周期较长 的分布式服务中。

**TCC**：典型的使用场景：满减，登录送优惠券等。

**可靠消息最终一致性**：典型的使用场景：注册送积分，登录送优惠券等。

**最大努力通知**：典型的使用场景：银行通知、支付结果通知等。

![image-20220524144914214](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205241449323.png)

参考资料：

- https://www.bilibili.com/video/BV1LQ4y127n4
- https://www.xn2001.com/archives/663.html
- https://zhuanlan.zhihu.com/p/263555694

# Redis 分布式缓存

Redis 支持三种集群方案

- 主从复制模式
- Sentinel（哨兵）模式
- Cluster 模式

## 主从复制原理

### 基本原理

主从复制模式中包含一个主数据库实例（master）与一个或多个从数据库实例（slave）

![image-20220530105855523](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301058652.png)

客户端可对主数据库进行读写操作，对从数据库进行读操作，主数据库写入的数据会实时自动同步给从数据库。

具体工作机制为：

- slave启动后，向master发送SYNC命令，master接收到SYNC命令后通过bgsave保存快照，并使用缓冲区记录保存快照这段时间内执行的写命令
- master将保存的快照文件发送给slave，并继续记录执行的写命令
- slave接收到快照文件后，加载快照文件，载入数据
- master快照发送完后开始向slave发送缓冲区的写命令，slave接收命令并执行，完成复制初始化
- 此后master每次执行一个写命令都会同步发送给slave，保持master与slave之间数据的一致性

### 优缺点

优点：

- master能自动将数据同步到slave，可以进行读写分离，分担master的读压力
- master、slave之间的同步是以非阻塞的方式进行的，同步期间，客户端仍然可以提交查询或更新请求

缺点：

- 不具备自动容错与恢复功能，master或slave的宕机都可能导致客户端请求失败，需要等待机器重启或手动切换客户端IP才能恢复
- master宕机，如果宕机前数据没有同步完，则切换IP后会存在数据不一致的问题
- 难以支持在线扩容，Redis的容量受限于单机配置

## Sentinel（哨兵）模式

### 基本原理

哨兵模式基于主从复制模式，只是引入了哨兵来监控与自动处理故障

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301145966.png)

哨兵顾名思义，就是来为Redis集群站哨的，一旦发现问题能做出相应的应对处理。其功能包括

- 监控master、slave是否正常运行
- 当master出现故障时，能自动将一个slave转换为master
- 多个哨兵可以监控同一个Redis，哨兵之间也会自动监控

当 master 宕机时，有如下动作

- 将宕机的master下线
- 找一个slave作为master
- 通知所有的slave连接新的master
- 全量数据或者部分数据同步

Sentinel（哨兵）是Redis 的高可用性解决方案：由一个或多个 Sentinel 实例组成的 Sentinel 系统可以监视任意多个主服务，以及这些主服务器属下的所有从服务，并在被监视的主服务进入下线（不可服务）状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器。

**哨兵作用**

- **集群监控**
- **消息通知**
- **自动故障转移**

> 注意：哨兵也是一台Redis服务器，只是不提供数据服务，通常哨兵配置的数量为单数。

#### 集群监控

1、哨兵 1 连接到 Redis 集群

- 发送info命令到master，并建立cmd连接；
- 哨兵端保存哨兵状态（SentinelStatus），保存所有哨兵状态，主节点和从节点的信息；master端会记录 redis 实例的信息（SentinelRedisInstance）；
- 哨兵根据master中获取的每个slave信息，去连接每个slave，发送同样也是info命令。

2、其他哨兵连接到 Redis 集群

- 发送info命令到master节点，并建立cmd连接；
- 发现master中存在其他哨兵节点的信息，新加入的哨兵将会保存其他哨兵信息
- 为了每个哨兵的信息都一致它们之间建立了一个发布订阅。为了哨兵之间的信息长期对称它们之间也会互发 ping 命令。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301156291.png)

- Sentinel会向master、slave以及其他Sentinel获取状态
- Sentinel之间会组建“对应频道”，大家一起发布信息、订阅信息、收信息、同步信息等。

#### 消息通知

1）Sentinel节点会通过master/slave 节点建立的cmd连接获取其工作状态

2）Sentinel收到反馈结果之后，会在哨兵内部进行信息的互通

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301158606.png)

#### 故障转移

关于故障转移，严格来讲可划分两个步骤：**故障判定**、**故障转移**。

**Q1：如何判断一个节点出现故障？**

- 哨兵会一直给主节点发送 publish sentinel：hello

> 直到主节点故障，哨兵报出 sdown，同时此哨兵还会向其他哨兵发布消息说这个主节点挂了。发送的指令是 sentinel is-master-down-by-address-port。

- 其余的哨兵接收到指令后，主节点挂了吗？让我去看看到底挂没挂。发送的信息也是 hello。

> 其余的哨兵也会发送他们收到的信息并且发送指令 sentinel is-master-down-by-address-port 到自己的内网，确认一下第一个发送 sentinel is-master-down-by-address-port 的哨兵说你说的对，这个家伙确实挂了。

- 当所有人都认为主节点挂了后就会修改其状态为 odown。

> 当一个哨兵认为主节点挂了标记的是 sdown，当半数哨兵都认为挂了其标记的状态是 odown。

一个哨兵认为master节点挂了称为主观下线（sdown），超半数哨兵认为master节点挂了则称为客观下线（odown）。

**Q2：如何进行故障转移？**

**1）首先，哨兵选举出哨兵Leader去处理故障转移**

**2）其次，哨兵Leader从所有的slave节点找出一个作为master节点**

主要的规则：

- 选择在线的节点，pass掉已下线的节点；
- 选择响应速度快的，pass掉响应慢的节点
- 选择与原master断开时间短的，pass掉断开时间较长的；

假如以上优先级均一致，会考虑其他优先原则：

- 偏移量较大

> 假如说 slave1 的 offset 为 50，slave2 偏移量为 55，则哨兵就会选择 slave2 为新的主节点。

- runid偏大的

> 这点类似于职场中的论资排辈，也就说根据 runid 的创建时间来判断，时间早的先上位。

**3）数据转移**

- 新master上任：Sentinel向新的master发送slaveof no one
- 其他slave周知：向其他slave发送slaveof 新master IP端口

### 优缺点

优点：

- 哨兵模式基于主从复制模式，所以主从复制模式有的优点，哨兵模式也有
- 哨兵模式下，master挂掉可以自动进行切换，系统可用性更高

缺点：

- 同样也继承了主从模式难以在线扩容的缺点，Redis的容量受限于单机配置
- 需要额外的资源来启动sentinel进程，实现相对复杂一点，同时slave节点作为备份节点不提供服务

## Cluster模式

### 基本原理

Redis哨兵模式实现了高可用，读写分离，但是其主节点仍然只有一个，即写入操作都是在主节点中，这也成为了性能的瓶颈。存在难以在线扩容，Redis容量受限于单机配置的问题。Cluster模式实现了Redis的分布式存储，即每台节点存储不同的内容，来解决在线扩容的问题。

![image-20220530110636724](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301106830.png)

Cluster采用无中心结构,它的特点如下：

- 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽
- 节点的fail是通过集群中超过半数的节点检测失效时才生效
- 客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
- 相比较sentinel模式，多个master节点保证主要业务（比如master节点主要负责写）稳定性，不需要搭建多个sentinel实例监控一个master节点；
- 相比较一主多从的模式，不需要手动切换**，具有自我故障检测，故障转移的特点**；
- 相比较其他两个模式而言，**对数据进行分片（sharding），不同节点存储的数据是不一样的**；
- 从某种程度上来说，Sentinel模式主要针对高可用（HA），**而Cluster模式是不仅针对大数据量，高并发，同时也支持HA。**

Cluster模式的具体工作机制：

- 在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383
- 当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的主节点，然后进行存取操作
- 为了保证高可用，Cluster模式也引入主从复制模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点
- 当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么该集群就无法再提供服务了

### Redis Cluser是如何保证高可用

Redis Cluster保证高可用主要还是依靠：**故障检测与故障转移两种策略**

#### 故障转移

举例说明：包含7000、7001、7002、7003四个主节点的集群，我们此时加入7004、7005两个节点，并当做7000的主节点的两个从节点。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301132804.png)

如果此时主节点7000下线（宕机），则 7000 的从节点将从他们中间选择一个作为主节点，用来处理客户端的读写请求。如下，7004 作为主节点，7005 作为了 7004 的从节点

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301133655.png)

若 7000 再次上线，则他将作为 7004 的从节点

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205301134897.png)

当一个从节点发现自己正在复制的主节点下线时，从节点将开始对下线主节点进行故障转移：

　　1） 在该下线主节点的所有从节点中，选择一个做主节点

　　2） 被选中的从节点会执行SLAVEOF no one命令，成为新的主节点；

　　3） 新的主节点会撤销对所有对已下线主节点的槽指派，并将这些槽全部派给自己。

　　4） 新的主节点向集群广播一条PONG消息，让其他节点知道“我已经变成主节点了，并且我会接管已下线节点负责的处理的槽”；

　　5） 新主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。

选举新的主节点步骤：

　　1）集群里的每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。

　　2）当从节点发现自己正在复制的主节点进入已下线状态时，从节点会向集群广播消息：要求所有收到这条消息、并且具有投票权的主节点向这个从节点投票。如果一个主节点具有投票权，并且这个主节点尚未投票跟其它从节点，则返回给从节点 ACK 消息，表示投票给该从节点

　　3）每个主节点只有一次投票机会，所有有N个主节点的话，那么具有大于N/2+1张支持票的从节点只有一个，成为新的主节点

　　4）如果在一个配置纪元里没有从节点能收集到足够多的支持票，那么集群进入一个新的配置纪元，并再次进行选举，直到选出新的主节点为止。

#### 故障检测

集群中每个节点都会定期地向集群中的其他节点发送PING消息，以此检测对方是否在线；如果接收PING消息的节点没有在规定的时间内，向发送PING消息的节点返回PONG消息，那么发送PING消息的节点就会将PING消息节点标记为疑似下线（possible fail，PFAIL）。

如果在集群中，超过半数以上负责处理槽的主节点都将某个节点X标记为PFAIL，则某个主节点就会将这个主节点X就会被标记为已下线（FAIL），并且广播到这条消息，这样其他所有的节点都会立即将主节点X标记为FAIL。

# 分布式锁

## 基本介绍

一个应用被部署到多个机器上做负载均衡。为了保证一个方法或属性在高并发情况下的同一时间只能被同一个线程执行，我们该如何解决这个问题呢？

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311025402.png)

在传统单体应用单机部署的情况下，可以使用并发处理相关的功能（如Java并发处理相关的API：ReentrantLcok或synchronized）进行互斥控制来解决。但是，随着业务的发展，系统架构也会逐步优化升级，原本单体单机部署的系统被演化成分布式集群系统，由于分布式系统多线程、多进程并且分布在多个不同机器上，这将使原单机部署情况下的并发控制锁策略无法满足，并不能提供分布式锁的能力。为了解决这个问题就需要一种跨机器的互斥机制来控制共享资源的访问，需要使用分布式锁。

1、处理效率提升：应用分布式锁，可以减少重复任务的执行，避免资源处理效率的浪费；

2、数据准确性保障：使用分布式锁可以放在数据资源的并发访问，避免数据不一致情况，甚至数据损失等。

## 实现方式

关于分布式锁的实现，可以分别控制在不同的环节。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311027360.jpeg)

### 1、开源组件锁控制：ZooKeeper

ZooKeeper 是一个分布式协调服务的开源框架。主要用来解决分布式集群中应用系统的一致性的问题，例如怎样避免同时操作同一数据造成脏读的问题。ZooKeeper 本质上是一个分布式的小文件存储系统。提供基于类似于文件系统的目录树方式的数据存储，并且可以对树中的节点进行有效管理。

![image-20220424152244612](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202205311028181.png)

1、客户端获取锁时，在 lock 节点（节点名可自定义）下创建**临时顺序**节点。

2、客户端获取到 lock 节点下的所有子节点，判断自己创建的临时顺序节点的序号是否最小，若是，则表示此客户端获取到了该锁。使用完后将其删除。

3、若发现自己创建的临时顺序节点序号不是最小，则说明未获取到锁，此时客户端监听比自己小的节点，对其注册事件监听器，**监听删除事件**。

4、若发现比自己小的节点被删除，则 Watcher 会受到相应通知，再次判断临时顺序节点的序号是否最小，根据上面的规则获取锁。

以上步骤中，应该注意的是，对于监听节点来说，当最小节点被删除后，之后监听该节点的 Watcher 会受到相应通知，其余不会收到通知。例如上图中，/lock/2 监听 /lock/1，当 /lock/1 删除后，/lock/3 不会收到通知。

ZooKeeper分布式锁方式，性能相对Redis方式较差，主要原因是写操作（获取锁释放锁）都需要在Leader上执行，然后同步到follower。

### 2、任务处理锁控制：Redis

优势：

- 性能极高 – Redis能读的速度是11w+次/s,写的速度是8w+次/s
- 丰富的数据类型 – Redis主要支持 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型
- 原子性 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性执行
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

Redis实现简单分布式锁过程：

（1）获取锁：使用 setnx 加锁，并使用 expire 命令为锁添加一个超时时间，超过该时间则自动释放锁，通常 value 为一个随机生成的UUID，通过此在释放锁的时候进行判断。若未获取到锁，设置一个获取的超时时间，若超过这个时间则放弃获取锁。

```sql
SETNX key value // 并没有设置过期时间
set key val EX 过期时间(秒) nx
```

当且仅当 key 不存在时，将 key 的值设为 value。

返回整数，具体为

- \- 1，当 key 的值被设置
- \- 0，当 key 的值没被设置

多个进程执行以下Redis命令：

SETNX lock.foo <current Unix time + lock timeout + 1>

- 如果 SETNX 返回1，说明该进程获得锁，SETNX将键 lock.foo 的值设置为锁的超时时间（当前时间 + 锁的有效时间）。
- 如果 SETNX 返回0，说明其他进程已经获得了锁，进程不能进入临界区。进程可以在一个循环中不断地尝试 SETNX 操作，以获得锁。

（2）释放锁：通过UUID判断是不是该锁，若是该锁，则执行delete进行锁释放。

利用Redis实现分布式锁，实现可能存在的缺点：

1、redis 过期时间问题：在执行delete进行释放锁的时候，假如操作删除锁动作失败，那此 Key-Value 过期时间则不好控制，可能会一直存在，可能对后续数据验证造成影响。

2、使用 set 设置 key - value 和过期时间应为原子操作，这个可以使用 `set key value Ex 过期时间 nx` 解决

3、当需要删除 key 时，由于 value 为 uuid，因此先对比是否为本线程加的锁，即对比 get 到的 uuid，是否相同，但是仍然存在问题，例如 redis 中的过期时间为 10s，业务代码执行了 9.5s，当从 redis 中 get 到本线程 uuid 后，花费 0.5s，那么此时 redis 将自动删除 key，之后若其他线程将 key - value 存入，此时第一个线程将会执行 delete 删除 key，那么删除了第二个线程 key - value，出现错误。这需要 get 和 delete 为原子操作，需要使用 lua 脚本进行原子解锁，参考官网文档，如下：

```java
String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
// 原子解锁
stringRedisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class), Arrays.asList("lock"), uuid);
```

上面由自己实现貌似有点麻烦，因此可使用 Redission 在 Redis 的基础上实现 Java 内存数据网络，提供分布式服务，提供 Redis 的分布式锁。也就是说，使用 Redission 实现 Redis 分布式锁的控制。参考 https://github.com/redisson/redisson

![image-20220702171548931](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207021715027.png)

#### Redission 如何实现 Redis 分布式锁

**Redission 的 Lock 方法**

Redission 的加锁方法 lock 继承了 juc 包下的 Lock，因此使用和可重入锁 ReentrantLock 相同，Redission 的 getLock("锁的名字") 进行分布式并发控制，这里锁的名字可以用来控制锁的粒度。

Redission 使用看门狗实现了锁的自动续期，因为 Redission 基于 Redis 的 setnx 实现，将此线程的 key - value 存入到 redis 中，若业务代码执行时间较长，则会默认更新 TTL 为 30s，即使在业务代码程序崩溃，没有手动解锁，也会由于超时时间（默认 30s）redis 自动删除 key，不会出现死锁问题。

当然我们也可使用 redission.lock(10, TimeUnit.SECONDS) 来自己指定时间，但是这种需要注意，这种方式 Redission 不会自动对 TTL 进行续期，也就是说，指定的时间必须大于业务方法的执行时间。

通常我们使用指定时间的方式加锁和手动解锁来显式控制。

**Redission 的信号量**

在 juc 包下，有信号量 Semaphore，因此 Redission 也有信号量的方法  getSemaphore("这个 Semaphore 的名称")，和 juc 包下的 Semaphore 相同，用来限流。

**Redission 的 CountDownLatch**

在 Redisssion 中，和 juc 相同的 CountDownLatch 为 getCountDownLatch("这个CountDownLatch 的名称")

注意，这些分布式并发锁都是基于 Redis 实现的，将特定的 key-val 存入到 Redis 中用来控制并发过程。

### 3、数据写入锁控制：MySQL

数据库层面是最终数据写入的时候，对数据做写入控制处理，算是分布式锁的最终末端环节。

实现方式一：唯一索引

```sql
UNIQUE KEY `uidx_name` (`name`) USING BTREE；
```

上述case中，我们对 `name` 字段做了索引的唯一性约束，当存在多个新增数据请求同时提交到数据库的话，数据库自身则会利用唯一索引，来保证数据的唯一性。

实现方式二：排他锁

执行以下SQL：

```sql
SELECT status FROM users WHERE id = 3 FOR UPDATE；
```

假如，在另一个事务中再次执行：

```sql
SELECT status FROM users WHERE id = 3 FOR UPDATE；
```

则第二个事务会一直等待上一个事务的提交，此时第二个查询处于阻塞的状态。

排它锁的应用：

在进行事务操作时，通过 “FOR UPDATE” 语句，MySQL会对查询结果集中每行数据都添加排他锁，其他线程对该记录的更新与删除操作都会阻塞。排他锁包含行锁、表锁。

实现方式三：乐观锁

实现逻辑：乐观锁每次在执行数据修改操作时，都会带上一个数据版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1 操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现 ABA 问题。

> 除了 version 以外，还可以使用时间戳，因为时间戳天然具有顺序递增性。

比较麻烦的一点：就是在操作业务前，需要先查询出当前的 version 版本。

数据库分布式锁实现可能存在的缺点：

- DB操作性能较差，并且有锁表的风险；
- 非阻塞操作失败后，需要轮询，占用cpu资源;
- 长时间不commit或者长时间轮询，可能会占用较多连接资源

### 比较

1. 理解的难易程度 数据库 > Redis > Zookeeper
2. 实现的复杂程度 Zookeeper >= Redis > 数据库
3. 性能高低 Redis > Zookeeper > 数据库
4. 可靠性 Zookeeper > Redis > 数据库

# 分布式 Session

参考 https://www.cnblogs.com/study-everyday/p/7853145.html

## 分布式 Session 存在的问题

单服务器web应用中，session信息只需存在该服务器中，因此单个服务下服务器可在一次会话中获得 session 存储的内容，但是在分布式场景中，可能存在通过负载均衡方式路由到不同的服务器中，那么就存在 session 存储在其中某一台服务器而其他服务器不知道该 session 的情况。存在 session 一致性的问题。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171716042.png)

## Session 一致性的解决方案

### 1、Session 复制（同步）

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171718384.png)

**思路**：多个web-server之间相互同步session，这样每个web-server之间都包含全部的session

**优点**：web-server支持的功能，应用程序不需要修改代码

**不足**：

- session的同步需要数据传输，占**内网带宽**，有时延
- 所有web-server都包含所有session数据，数据量受内存限制，无法水平扩展

### 2、客户端存储

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171718387.png)

**思路**：服务端存储所有用户的 session，内存占用较大，可以将session存储到浏览器cookie中，每个端只要存储一个用户的数据了

**优点**：服务端不需要存储

**缺点**：

- 每次http请求都携带session，占**外网带宽**
- 数据存储在端上，并在网络传输，存在泄漏、篡改、窃取等安全隐患
- session存储的数据大小受cookie限制

“端存储”的方案虽然不常用，但确实是一种思路。

### 3、反向代理 hash

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171719254.png)

**思路**：web-server为了保证高可用，有多台冗余，反向代理服务器通过一些策略让 **同一个用户的请求保证落在一台web-server**

有如下方案：

**方案一：四层代理 hash**

反向代理层使用用户ip来做hash，以保证同一个ip的请求落在同一个web-server上

**方案二：七层代理 hash**

反向代理使用http协议中的某些业务属性来做hash，例如sid，city_id，user_id等，能够更加灵活的实施hash策略，以保证同一个浏览器用户的请求落在同一个web-server上

**优点**：

- 只需要改nginx配置，不需要修改应用代码
- 负载均衡，只要hash属性是均匀的，多台web-server的负载是均衡的
- 可以支持web-server水平扩展（session同步法是不行的，受内存限制）

**不足**：

- 如果web-server重启，一部分session会丢失，产生业务影响，例如部分用户重新登录
- 如果web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session

### 4、后端统一集中存储

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202207171721378.png)

**思路**：将session存储在web-server后端的存储层，数据库或者缓存

**优点**：

- 没有安全隐患
- 可以水平扩展，数据库/缓存水平切分即可
- web-server重启或者扩容都不会有session丢失

**不足**：增加了一次网络调用，并且需要修改应用代码

对于db存储还是cache，个人推荐后者：session读取的频率会很高，数据库压力会比较大。如果有session高可用需求，cache可以做高可用，但大部分情况下session可以丢失，一般也不需要考虑高可用。
