SpringBoot 启动主要包括三个部分：

1. SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器；
2. 应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块；
3. 自动化配置模块，该模块作为springboot自动配置核心

启动：

每个SpringBoot程序都有一个主入口，也就是main方法，main里面调用SpringApplication.run()启动整个spring-boot程序，该方法所在类需要使用@SpringBootApplication注解，以及@ImportResource注解(if need)，

@SpringBootApplication包括三个注解，功能如下：

1. @EnableAutoConfiguration：SpringBoot根据应用所声明的依赖来对Spring框架进行自动配置
2. @SpringBootConfiguration(内部为@Configuration)：被标注的类等于在spring的XML配置文件中(applicationContext.xml)，装配所有bean事务，提供了一个spring的上下文环境
3. @ComponentScan：组件扫描，可自动发现和装配Bean，默认扫描SpringApplication的run方法里的Booter.class所在的包路径下文件，所以最好将该启动类放到根包路径下

![image-20211117103726727](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191602231.png)

进入 run 方法：

![image-20211117103804830](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191602913.png)

run方法中去创建了一个SpringApplication实例，在该构造方法内，我们可以发现其调用了一个初始化的initialize方法

![image-20211117103823905](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191602450.png)

在 initialize 方法中，主要是为SpringApplication对象赋一些初值。构造函数执行完毕后，我们回到run方法

![image-20211117103849774](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191602158.png)

**该方法中实现了如下几个关键步骤：**

1.创建了应用的监听器SpringApplicationRunListeners并开始监听

2.加载SpringBoot配置环境(ConfigurableEnvironment)，如果是通过web容器发布，会加载StandardEnvironment，其最终也是继承了ConfigurableEnvironment，并且 xxxEnvironment最终都实现了PropertyResolver接口，我们平时通过environment对象获取配置文件中指定Key对应的value方法时，就是调用了propertyResolver接口的getProperty方法

3.配置环境(Environment)加入到监听器对象中(SpringApplicationRunListeners)

4.创建run方法的返回对象：ConfigurableApplicationContext(应用配置上下文)

5.回到run方法内，prepareContext方法将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联

6.接下来的refreshContext(context)方法(初始化方法如下)将是实现spring-boot-starter-*(mybatis、redis等)自动化配置的关键，包括spring.factories的加载，bean的实例化等核心工作。

**refresh方法**

配置结束后，Springboot做了一些基本的收尾工作，返回了应用环境上下文。回顾整体流程，Springboot的启动，主要创建了配置环境(environment)、事件监听(listeners)、应用上下文(applicationContext)，并基于以上条件，在容器中开始实例化我们需要的Bean，至此，通过SpringBoot启动的程序已经构造完成，接下来我们来探讨自动化配置是如何实现。

------

参考 https://www.cnblogs.com/Narule/p/14253754.html

**主要流程如下**

0.启动main方法开始

1.**初始化配置**：首先创建 SpringApplication 对象，在 Spring Application 中通过类加载器，（loadFactories）读取classpath下所有的spring.factories配置文件，创建一些初始配置对象；通知监听者应用程序启动开始，创建环境对象 environment，用于读取环境配置 如 application.yml

2.**创建应用程序上下文**-createApplicationContext，创建 bean工厂对象

3.**刷新上下文（启动核心）**

3.1 配置工厂对象，包括上下文类加载器，对象发布处理器，beanFactoryPostProcessor

3.2 注册并实例化bean工厂发布处理器，并且调用这些处理器，对包扫描解析(主要是class文件)

3.3 注册并实例化bean发布处理器 beanPostProcessor

3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat服务器）

3.5 实例化所有bean工厂缓存的bean对象（剩下的）

3.6 发布通知-通知上下文刷新完成（启动tomcat服务器）

4.**通知监听者-启动程序完成**

启动中，大部分对象都是BeanFactory对象通过反射创建

