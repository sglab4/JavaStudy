# 1、概述

1、Spring 全家桶：spring，springmvc，spring boot，spring cloud，主要用来管理模块之间和类之间的关系，实现模块之间，类之间的解耦合。

2、Spring 核心技术：**IoC 和 AOP**

3、依赖：类 a 使用了类 b 的属性或方法，则类 a 依赖于 b

4、框架学习的内容：

1. 框架的作用
2. 框架的语法
3. 框架的内部实现
4. 实现一个框架

# 2、IoC

## 2.1 IoC 概述

IoC：控制反转，是一个思想理论，描述的是把对象的创建、赋值和管理工作交给代码之外的**容器**实现，由外部资源完成。IoC 底层使用的是 java 的反射机制。

IoC 能够实现业务对象之间的解耦合，例如 service 和 dao 之间的解耦合。

控制：创建对象，对象的属性赋值，对象之间的关系管理。

反转：把原来开发人员管理，创建对象的权限转交给代码之外的容器实现，由容器代替开发人员管理对象，创建对象，给对象赋值。

正转：开发人员使用 new 创建对象，主动创建和管理对象

容器：是一个服务器软件，是一个框架（Spring）

反转的目的：减少对代码的改动，来实现不同的功能，实现解耦合。

> **java 中创建对象的方式**

1. 构造方法：new
2. 反射
3. 反序列化
4. 克隆
5. IoC
6. 动态代理

> IoC 的体现

Servlet：

1. 创建类继承 HttpServlet

2. 在 web.xml 中注册 Servlet：

   ```xml
   <servlet-name>myservlet</servlet-name>
   <servlet-class>com.servlet.MyServlet</servlet-class>
   ```

3. 在程序中没有创建过 MyServlet，但可以使用，原因是 Tomcat 服务器创建了 Servlet，Tomcat 也被称为容器

> IoC 的技术实现

DI 是 IoC 的技术实现。

DI（dependency injection）：依赖注入，只需在程序中提供要使用的对象名称即可，至于对象如何在容器中创建、赋值、查找都由容器内部实现。

Spring 使用 DI 实现了 IoC 功能，Spring 的底层使用的是反射机制来创建对象。

> 使用 IoC 由 spring 创建对象的步骤

1. 创建 maven 项目
2. 导入依赖，包括 Spring 依赖和 Junit 依赖
3. 创建类（接口和实现类）
4. 创建 Spring 所需的配置文件（和 Servlet 中的配置文件类似）
5. 测试 Spring 创建的对象

pom 配置为文件中的依赖包：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.spring-demo</groupId>
  <artifactId>spring-test-3</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>spring-test-3</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <dependencies>
    <!--spring 框架依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.9</version>
    </dependency>

    <!--测试依赖包-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

Spring 的配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

说明如下：

1. beans 是根标签，Spring 将 java 对象称为 bean
2. spring-beans.xsd：是约束文件

以上是 Spring 的标准格式

使用 Spring 创建对象：

```xml
<bean id="someService" class="com.springdemo.service.impl.SomeServiceImpl"></bean>
```

其中，id 为对象的自定义名称，唯一值，spring 通过这个名称找到对象，class 为类的全限定名称（不能是接口，因为 spring 通过反射机制来创建对象），配置文件中 id 不可重复，class 可重复。和 Servlet 中的配置类似。

以上配置好后就相当于

```java
SomeService someService = new SomeServiceImpl();
```

Spring 创建好对象后将其放在 map 中，Spring 框架中有个 map 来存储对象，`springMap.put(id 值, 对象);`，例如：

```java
springMap.put("someService", new SomeServiceImpl);
```

因此便可通过 id 来获取到对象，由于使用的是 Map 来存储对象，因此 id 不可重复。

配置好了之后，使用 Spring 创建对象的步骤如下：

1. 指定 Spring 配置文件对象名称

2. 创建 Spring 容器，ApplicationContext，ApplicationContext 便为 spring 容器，可由此来获取对象

   ApplicationContext 为一个接口，实现类有 FileSystemXmlApplicationContext 和 ClassPathXmlApplicationContext，不过通常使用 ClassPathXmlApplicationContext，ClassPathXmlApplicationContext 表示采用类 class 来创建对象，因此说 Spring 采用的是反射机制来创建对象。

   ![image-20210913161847550](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210913161908.png)

   ![image-20210913162635881](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210913162636.png)

3. 调用容器的方法从容器中获取对象，方法为 getBean(x) 参数为配置文件中 bean 的 id 值

```java
@Test
public void test2() {
    //1
    String config = "beans.xml";
    //2
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //3
    SomeService someService = (SomeService) applicationContext.getBean("someService");
    //使用创建好的对象
    someService.doSome();
}
```

以上代码中，完成第二步创建容器后，Spring 配置文件中配置的对象通过反射便被创建，Spring 构造对象，默认调用的是无参构造方法，以 Map 形式存储在容器中，在第三步通过 getBean 得到 id 的 val 值，为 java 对象。

Spring 容器的 ApplicationContext 容器有两个方法` applicationContext.getBeanDefinitionCount();` 和 `applicationContext.getBeanDefinitionNames();` 分别获得配置文件中 bean 的数量和 id，`getBeanDefinitionNames` 获取到的 id 为字符串数组。

> spring 是否可创建非自定义类的对象

可以，在 spring 配置文件中配置

```xml
<bean id="myArrayList" class="java.util.ArrayList"></bean>
```

测试类中

```java
@Test
public void test4() {
    String config = "beans.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    ArrayList<Integer> myArrayList = (ArrayList<Integer>) applicationContext.getBean("myArrayList");
    myArrayList.add(1);
    myArrayList.add(2);
    System.out.println("myArrayList = " + Arrays.toString(myArrayList.toArray()));
}
```

## 2.2 DI 的实现

1. 在 Spring **配置文件**中，使用标签和属性完成，称为基于 xml 的 DI 实现
2. 使用 Spring 中的**注解**，完成属性赋值，称为基于注解的 DI 实现

### 2.2.1 基于 XML 的 DI

DI 语法分类：

1. set 注入（设值注入）：使用类的 set 方法，实现属性的赋值
2. 构造注入：调用类的有参构造方法，在构造方法中实现赋值

> set 注入

在 java 中，int、char 等关键字修饰的和其包装类、String 等都属于**基本数据类型**，基本数据类型的 set 注入方式采用 property 标签

```xml
<bean id="myStudent" class="com.springdemo.pag1.Student">
    <property name="name" value="Kurisu" /> <!-- setName("Kurisu") -->
    <property name="age" value="19" /> <!-- setAge(19) -->
</bean>
```

以上的配置，value = kurisu 相当于调用了 Student 类中的 setName 方法，形参为 "Kurisu"，因此在类 Student 中必须有 setter 方法。

1. 若在 Studen 中没有 setter 方法，则会报错；

2. 若在 setter 中，将 this.name = name 删除，则同样会调用 setName 方法，不会报错，但不会给 name 赋值，因此可发现，Spring 只负责调用 setter 方法，setter 方法中的内容可自行定义；

3. setter 方法运行发生在构造方法之后

4. 若在 Student 类中，没有属性 email，但是手动写了 setEmail 方法，spring 同样会执行 setEmail 方法，不会报错，可发现，Spring 在此标签下只执行的是 setter 方法，而不管是否在类中是否有属性。也就是说，对于类中的方法 setEmail(String email)，在配置文件中配置如下，则 spring 容器只会寻找 email 对应的 set 方法，即 setEmail 方法。 

   ```xml
   <property name="email" value="abcd"/> <!-- setAge("abcd") -->
   ```

5. 不论方法的参数是什么类型，在 xml 中赋值时，均需要加引号，这是 xml 文件规范。

6. 同样，对于非自定义类（java API），也符合以上规则

在 set 注入中，若类中的属性为**引用类型**，如 Student 类中有自定义类 School，可采用 ref 标签属性引用

```xml
<bean id="myStudent" class="com.springdemo.pag2.Student">
    <property name="name" value="Kurisu" />
    <property name="age" value="19" />
    <property name="school" ref="mySchool" />
</bean>

<bean id="mySchool" class="com.springdemo.pag2.School">
    <property name="name" value="ABCD" />
    <property name="address" value="abcd" />
</bean>
```

> 构造注入

使用构造注入，需要类中有有参构造方法，在创建对象的同时使用构造方法给属性赋值。注意，当类中有有参构造时，在 `<bean>` 中必须配置 `constructor-arg` 标签给参数赋值。

构造注入采用 `<constructor-arg name="name" value="Kurisu" />` name 标签属性表示类中的有参构造中的参数名称，value 表示给该参数赋的值。

```xml
<bean id="myStudent" class="com.springdemo.pag3.Student">
    <constructor-arg name="name" value="Kurisu" />
    <constructor-arg name="age" value="19" />
</bean>
```

同理，若有参构造中的参数为引用类型，则 value 应该替换为 ref，和 set 注入相同。

若将 constructor-arg 标签中的 name 替换为 index，则 index 表示有参构造的第 0,1,2... 个位置。

```xml
<bean id="myStudent2" class="com.springdemo.pag3.Student">
    <constructor-arg index="0" value="Kurisu" />
    <constructor-arg index="1" value="19" />
</bean>
```

不论采用 name 还是 index 标签属性，前后位置可换。

使用构造注入创建系统类 File

```xml
<bean id="myFile" class="java.io.File">
    <constructor-arg name="parent" value="路径"/>
    <constructor-arg name="child" value="文件名"/>
</bean>
```

便可由此来创建 File

> **引用类型**的**自动注入** 

引用类型的自动注入通常使用的规则是 byName 和 byType

**byName**：java 中**引用类型**的属性名和 Spring 容器中 bean 的 id 名称相同，且数据类型一致，则 spring 可通过 byName 自动完成赋值，若在此 bean 中设置为了 byName，则所有引用类型都采用 byName 赋值。

例如：在 Student 类中，有属性 School，名为 school，那么

```xml
<bean id="myStudent" class="com.springdemo.pag4.Student" autowire="byName">
    <property name="name" value="Kurisu" />
    <property name="age" value="19" />
</bean>

<bean id="school" class="com.springdemo.pag4.School">
    <property name="name" value="abcd" />
    <property name="address" value="ABCD" />
</bean>
```

调用 getBean 创建 id 为 myStudent 的对象，得到结果为：

```json
myStudent = Student{name='Kurisu', age=19, school=School{name='abcd', address='ABCD'}}
```

**byType**：java 中**引用类型**的数据类型和 Spring 容器中 bean 的 class 属性是同源关系，则可按 byType 完成自动赋值。

同源指的是：

1. java 中引用类型的数据类型和 bean 中的 class 相同
2. java 中引用类型的数据类型和 bean 中的 class 为父子关系，java 引用类型为父类， bean 中为子类
3. java 中引用类型的数据类型和 bean 中的 class 为接口和接口实现关系

```xml
<bean id="myStudent" class="com.springdemo.pag4.Student" autowire="byType">
    <property name="name" value="Kurisu" />
    <property name="age" value="19" />
</bean>

<bean id="school" class="com.springdemo.pag4.School">
    <property name="name" value="abcd" />
    <property name="address" value="ABCD" />
</bean>
```

在 byType 中，若在 spring 容器中出现了多个同源的 bean，则会报错

```xml
<!--编译报错-->
<bean id="myStudent" class="com.springdemo.pag4.Student" autowire="byType">
    <property name="name" value="Kurisu" />
    <property name="age" value="19" />
</bean>

<bean id="school" class="com.springdemo.pag4.School">
    <property name="name" value="abcd" />
    <property name="address" value="ABCD" />
</bean>

<bean id="school2" class="com.springdemo.pag4.School">
    <property name="name" value="abcd" />
    <property name="address" value="ABCD" />
</bean>
```

当项目中含有的类很多时，可将一个配置文件转换为多个配置文件，优势如下：

1. 每个配置文件小很多
2. 避免多人带来的冲突

> 多配置文件分配

1. 按功能模块
2. 按类的功能

在进行分类后，需要主配置文件 spring-total.xml，一般不用来定义对象，语法：` <import resource="其他配置文件位置" /> `，关键字` classpath`：类路径（class 文件所在的位置，位与 target 包中）

在 spring-student.xml 中：

```xml
<bean id="student" class="com.springdemo.pag5.Student" autowire="byType">
    <property name="name" value="ccc" />
    <property name="age" value="2" />
</bean>
```

在 spring-school.xml 中：

```xml
<bean id="school" class="com.springdemo.pag5.School">
    <property name="name" value="aaa" />
    <property name="address" value="abc" />
</bean>
```

在 spring-total.xml 中：

```xml
<import resource="classpath:pag5/spring-student.xml" />
<import resource="classpath:pag5/spring-school.xml" />
```

在使用 spring 容器创建对象时，和之前 getBean 相同

```java
@Test
public void test() {
    String config = "pag5/spring-total.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    Student student = (Student) applicationContext.getBean("student");
    System.out.println("student = " + student);
}
```

在包含关系的配置文件中，还可使用通配符

将 spring-total.xml 文件名改为 total.xml，import 改为如下，表示包含所有 spring- 开头的 xml 文件

```xml
<import resource="classpath:pag5/spring-*.xml" />
```

在使用通配符时，total.xml 不能符合包含的通配符的范围，不然将会造成死循环，自己包含自己。若使用通配符包含，则所有配置文件应该包含在同一级目录下，即同一个包中。

### 2.2.2 基于注解的 DI

使用注解需要使用 spring-aop 的依赖，在之前添加 spring-context 时 spring-aop 已经添加。

1. @Component
2. @Repository
3. @Service
4. @Controller
5. @Value
6. @Autowired
7. @Resource

使用注解完成对象创建、属性赋值，代替 xml 文件的依赖注入的步骤：

1. 加入依赖
2. 创建类，在类中加入注解
3. 创建 spring 配置文件，声明组件扫描器
4. 使用注解创建对象

>@Component

@Component 用来创建对象，等同于 spring 标签中的 `<bean>` 功能，其中 @Component 中的属性 value 表示创建的对象的名称，即 `<bean>` 中 id 的值，此值是唯一的，在 spring 容器中只有一个。

```java
@Component(value = "myStudent")
```

等同于

```xml
<bean id="myStudent" class="com.springdemo.pag1.Student" />
```

配置 spring 配置文件中的组件扫描器 `context:component-scan`，`base-package` 表示应该扫描的包，在 spring 工作时，会将指定包中的所有类扫描一遍，然后根据配置了的注解，进行创建对象和属性赋值。

```xml
<context:component-scan base-package="com.springdemo.bag1" />
```

在加入 `context:component-scan` 标签后，Spring会去自动扫描 base-package 的值所表示的包的位置中的 java 文件，如果扫描到有 @Component、@Controller、@Service、@Repository 等类似注解的类，会将这个类注册为 Bean。

在注解中配置 value，可将其省略

```java
@Component("myStudent")
```

在注解中不指定对象名称，由 spring 提供默认名称，spring 提供的默认名称为首字母小写，如 Student 类提供的默认名称为 student

```java
@Component
```

>@Repository

放在持久层的 dao 类上，可访问数据库。

>@Service

放在业务层上，service 实现类上，创建 service 对象，做业务处理、事务等功能

> @Controller

放在控制器上，接收用户提交的参数，显示用户请求的结果（Servlet 功能）

@Repository、@Service、@Controller 来给项目进行分层，和 @Component 用法相同，以上功能相同，但用的地方不同，表示不同含义。

若有多个包需要进行组件扫描，则可使用如下方法：

1. 使用多次组件扫描器

   ```xml
   <context:component-scan base-package="com.springdemo.bag1" />
   <context:component-scan base-package="com.springdemo.bag2" />
   ```

2. 使用分隔符，逗号或分号

   ```xml
   <context:component-scan base-package="com.springdemo.bag1;com.springdemo.bag2" />
   ```

3. 指定父包

   ```xml
   <context:component-scan base-package="com.springdemo" />
   ```

   不要指定的父包范围太大，避免扫描时耗时太长。

> @Value

**简单类型**属性赋值，属性为 value，为 String 类型，表示简单属性赋值

用法：

1. 用在属性定义上面，不需要 setter 方法

   ```java
   @Value(value = "Kurisu") //可省略 
   private String name;
   @Value("19")
   private int age;
   ```

2. 用在 setter 方法上面

   ```java
   @Value("Kurisu")
   public void setName(String name) {
       this.name = name;
   }
   ```

>@Autowired

**引用类型**的属性赋值，自动注入原理，支持 byName，byType，@Autowired 默认使用的是 byType。可用在属性定义的上面，无需 setter 方法；也可用在 setter 方法的上面。

School.java 类：

```java
@Component("mySchool")
public class School {
    @Value("ABCD")
    private String name;
    @Value("abcd")
    private String address;
    
	setter ....
        
    getter ....
}
```

Student.java 类：

```java
@Component("myStudent")
public class Student {
    @Value(value = "Kurisu")
    private String name;
    @Value(value = "18")
    private int age;
    @Autowired
    private School school;

	setter ....
    
    getter ....
}
```

若要使用 byName 的方式，则

1. 在属性名上加 @Autowired
2. 在属性名上加 @Qualifier(value = "bean 的 id 名字")，value 可省略 @Qualifier("bean 的 id 名字")

@Autowired 中有一个属性为 required，默认为 true，若为 true，则表明引用赋值失败，程序出错，终止执行；若 required 为 false，则表示若引用类型赋值失败，则赋值为 null，程序正常执行。

> @Resource

@Resource 为 JDK 的注解，Spring 框架中提供了对这个注解的功能支持，可使用 @Resource 对引用类型赋值，使用的也是自动注入原理，支持 byName，byType，默认使用的是 byName。

使用方式：

1. 在属性定义上面加 @Resource
2. 在 setter 方法上面加 @Resource

@Resource 先使用 byName 自动注入，若赋值失败，则使用 byType 注入。若只想使用 byName，则在 @Resource 中加入一个属性 name，用来指定创建的 bean 的 id 名称。

### 2.2.3 注解和 xml 的对比

xml 优点：

1：xml 是集中式的元数据，不需要和代码绑定的，实现完全解耦合

2：使用 xml 配置可以让软件更具有扩展性；

3：对象之间的关系一目了然；

4：基于 xml 配置的时候，只需要修改 xml 即可，不需要对现有的程序进行修改。

5：容易与其他系统进行数据交互。数据共享方便

xml 缺点：

1：xml 配置文件过多，会导致维护变得困难

2：开发的时候，既要维护代码又要维护配置文件，使得开发的效率降低；

3：在程序编译期间无法对其配置项的正确性进行验证，只能在运行期发现。

注解优点：

1：注解的解析可以不依赖于第三方库，可以之间使用 Java 自带的反射

2：注解和代码在一起的，之间在类上，降低了维护两个地方的成本

3：注解如果有问题，在编译期间，就可以验证正确性，如果出错更容易找

4：使用注解开发能够提高开发效率。不用多个地方维护

注解缺点：

1：在程序中注解太多的话，会影响代码质量，对原有的 java 代码具有一定侵入性

2：处理业务类之间的复杂关系，不然 xml 那样容易修改，也不及xml那样明了

总结：对于经常修改的属性值，采用 xml 注入，对于不经常修改的属性值，采用注解注入

## 2.3 动态代理

代理(Proxy)是一种设计模式,提供了对目标对象另外的访问方式;即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能.
这里使用到编程中的一个思想：不要随意去修改别人已经写好的代码或者方法,如果需改修改,可以通过代理的方式来扩展该方法

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211022154122.png)

可以在程序执行过程中，创建代理对象，通过代理对象执行方法，给目标类增加额外的功能（功能增强）

反射和动态代理放有一定的相关性，但单纯的说动态代理是由反射机制实现的，其实是不够全面不准确的，动态代理是一种功能行为，而它的实现方法有很多。详见 https://www.jianshu.com/p/de5958318165

> JDK 动态代理

JDK 动态代理实现步骤：

1. 创建目标类（需要增加额外增强的类）
2. 创建 InvocationHandler 接口的实现类，在这个类实现给目标方法增加功能
3. 使用 JDK 中的类 Proxy，创建代理对象，实现创建对象的能力。

使用如上的动态代理实现时，必须加对象的定义的接口。

目标类：

```java
package com.service;

public interface SomeService {
    void doSomeService();
    void doOtherService();
}
```

```java
package com.service.impl;

import com.service.SomeService;

public class SomeServiceImpl implements SomeService {
    @Override
    public void doSomeService() {
        System.out.println("doSomeService....");
    }

    @Override
    public void doOtherService() {
        System.out.println("doOtherService....");
    }
}
```

创建 InvocationHandler 接口的实现类，来实现需要增加的功能：

```java
package com.handler;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.Date;

public class MyIncationHandler implements InvocationHandler {
    //目标类
    private Object target;

    public MyIncationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //代理对象执行方法时，会调用 invoke
        Object res = null; //用来记录返回结果
        System.out.println("Time :" + new Date()); //额外增加的功能
        //使用 Method 类执行目标类的方法，参数为目标类，参数...
        res = method.invoke(target, args);
        System.out.println("trans..."); //额外增加的功能
        //执行返回结果
        return res;
    }
}
```

以上解释如下：target 表示要增加功能的目标类，同时定义了一个构造方法，用来接收目标类；核心 invoke 方法，第一个参数表示代理类的实例，第二个参数表示代理实现的目标对象的方法，第三个参数表示目标方法可能需要的传入的参数。在 invoke 方法中，使用 res 表示返回值，使用 Method 的 invoke 的方法，实现动态代理执行目标类的方法，在方法前后可增加需要增强的额外功能。

```java
package com;

import com.handler.MyIncationHandler;
import com.service.SomeService;
import com.service.impl.SomeServiceImpl;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class MyTest {
    public static void main(String[] args) {
        //创建目标对象
        SomeService someService = new SomeServiceImpl();
        //创建 InvocationHandler 对象
        InvocationHandler handler = new MyIncationHandler(someService);
        //使用 Proxy 的 newProxyInstance 创建代理对象
        //newProxyInstance 的三个参数：1、目标对象的类加载器；2、目标对象的接口；3、InvocationHandler 对象
        SomeService proxyInstance = (SomeService) Proxy.newProxyInstance(someService.getClass().getClassLoader(), someService.getClass().getInterfaces(), handler);
        //使用代理对象执行 doSomeService 方法
        proxyInstance.doSomeService();

    }
}
```

测试主函数中，首先创建目标对象，然后再创建 InvocationHandler 来实现动态代理，在 InvocationHandler 中，使用了刚刚自定义的实现类，并传入目标对象，然后再使用 Proxy 的 newProxyInstance 方法创建代理，newProxyInstance 方法中有三个参数，分别为 1、目标对象的类加载器；2、目标对象的接口；3、自己实现的InvocationHandler 接口创建的 InvocationHandler 对象，然后使用动态代理执行指定方法。

以上 MyIncationHandler 实现 InvocationHandler 接口中，没有对传入的方法进行区分，可使用 method.getName(); 获取传入的方法名称，来控制不同的方法增加不同的功能。

> CGLIB 动态代理

JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。为了解决这个问题，可以用 CGLIB 动态代理机制来避免。

在 CGLIB 动态代理机制中 `MethodInterceptor` 接口和 `Enhancer` 类是核心。通过 `Enhancer`类来动态获取被代理类，当代理类调用方法的时候，实际调用的是 `MethodInterceptor` 中的 `intercept` 方法。

使用步骤：

1. 定义一个类；
2. 自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法，和 JDK 动态代理中的 `invoke` 方法类似；
3. 通过 `Enhancer` 类的 `create()`创建代理类；

下面例子参考 https://www.cnblogs.com/wyq1995/p/10945034.html

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/202202221612948.png)

```xml
<dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>2.2.2</version>
</dependency>
```

目标类：

```java
package com.wyq.day527;

public class Dog{
    
    final public void run(String name) {
        System.out.println("狗"+name+"----run");
    }
    
    public void eat() {
        System.out.println("狗----eat");
    }
}
```

方法拦截器：

```java
package com.wyq.day527;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class MyMethodInterceptor implements MethodInterceptor{

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("这里是对目标类进行增强！！！");
        //注意这里的方法调用，不是用反射哦！！！
        Object object = proxy.invokeSuper(obj, args);
        return object;
    }  
}
```

测试类：

```java
package com.wyq.day527;

import net.sf.cglib.core.DebuggingClassWriter;
import net.sf.cglib.proxy.Enhancer;

public class CgLibProxy {
    public static void main(String[] args) {
        //创建Enhancer对象，类似于JDK动态代理的Proxy类，下一步就是设置几个参数
        Enhancer enhancer = new Enhancer();
        //设置目标类的字节码文件
        enhancer.setSuperclass(Dog.class);
        //设置回调函数
        enhancer.setCallback(new MyMethodInterceptor());
        
        //这里的creat方法就是正式创建代理类
        Dog proxyDog = (Dog)enhancer.create();
        //调用代理类的eat方法
        proxyDog.eat();       
    }
}
```

# 3、AOP

## 3.1 概念

AOP （面向切面编程） 的底层采用的是动态代理模式实现，就是动态代理的规范化，采用了两种代理：JDK 动态代理和 CGLIB 动态代理。

- JDK 动态代理要求目标类必须实现接口。

- cglib 动态代理是第三方工具库，创建代理对象，原理是继承，通过继承目标类，创建子类，子类就是代理对象，要求类和方法均不为 final。


动态代理作用：

1. 源代码不改变的情况下增加功能
2. 减少代码重复
3. 专注业务逻辑代码
4. 实现解耦合，使得业务功能和非业务功能分离

AOP 中**切面**（Aspect）的理解：给目标类增加非业务功能（统计信息、日志信息），称为切面。处理逻辑、计算等称为业务。处理非业务方法，称为切面，切面特点：一般面向非业务方法，独立使用。

- 分析项目功能时，找出切面
- 合理安排切面执行时间（目标方法前，还是目标方法后）
- 合理安排切面执行位置，在哪个类哪个方法中增加功能

JoinPoint **连接点**：连接业务方法和切面的位置，也就是某个类中的业务方法。连接点指可以被切面织入的具体方法，通常业务接口中的方法均为连接点。

PointCut **切入点**：指多个连接点方法的集合。为多个方法。

**目标对象**：给哪个类增加功能，该类就称为目标对象。

**Active 通知**：通知切面功能执行的时间

切面的三要素：

1. 切面的功能
2. 切面的执行位置
3. 切面执行时间，在目标方法之前，还是在目标方法之后

## 3.2 实现

AOP 是一个规范，标准。

AOP 技术的实现框架：

1. Spring：主要处理使用时使用 Spring 的 AOP，很少使用，比较笨重
2. aspectJ：开源 AOP 框架，因为 Spring 中已经集成了 aspectJ 框架，因此可通过 Spring 使用 aspectJ 框架，aspectJ 实现 AOP 的两种方式：
   1. 使用 xml 配置（配置全局事务）
   2. 使用注解

### 3.2.1 aspectJ 

aspectJ 框架的使用：

1. 切片执行时间，称为 Advice，在 aspectJ 框架中使用注解来进行表示：

   - @Before
   - @AfterReturning
   - @Around
   - @AfterThrowing
   - @After

   除了使用注解外，还可使用 xml 进行配置。

**aspectJ 切入点表达式：**

![image-20210917093626693](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210917093635.png)

以上表达式分为 4 个部分：**execution(访问权限 方法返回值 方法声明（参数） 异常类型)**，其中，方法返回值和方法声明不可省略，切入点表达式可进行如下匹配：

![image-20210917094044247](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210917094044.png)

使用 aspectJ 进行功能增强的步骤（**以前置注解为例**）：

1. 加入依赖

   - spring 依赖
   - aspectJ 依赖
   - junit 依赖

2. 创建目标类，接口和实现类，给类中的方法增加功能

3. 创建**切面类**：普通类

   1. 在类上面加入 @AspectJ，这个注解是 aspectJ 框架中的类，用来表示此类属于切面类，用来给业务增加功能的类，里面定义的方法是实现切面的功能，方法要求：

      1. public
      2. 无返回值
      3. 方法名自定义
      4. 可有可无参数

   2. 在类中定义方法，方法就是切面要执行的功能代码，在方法上面加入 aspectJ 中的通知注解，例如 @Before，@Before 前置通知注解，用于目标方法的上面，其中有属性 value，value = "切入点表达式"，就是上面 execution 部分。

      ```java
      @Before(value = "execution(public void com.springdemo.pag1.SomeServiceImpl.doSomeService(..))")
      public void myBefore(JoinPoint joinPoint) {
          //方法签名：包名 + 类名 + 方法名
          System.out.println(joinPoint.getSignature());
          //方法名
          System.out.println(joinPoint.getSignature().getName());
          //参数，以数组形式返回
          Object[] args = joinPoint.getArgs();
          for (Object arg : args) {
              System.out.println(arg);
          }
          System.out.println("before... " + new Date());
      }
      ```
      
      以上注解告诉对象，在 com.springdemo.pag1 的 SomeServiceImpl 类的 doSomeService 方法的执行之前加入 myBefore 方法。这里的 public 和 void 指的是 doSomeService 方法。

4. 创建 spring 配置文件，交给容器统一管理，声明对象可使用注解或 xml 配置

   1. 声明目标对象

      ```xml
      <bean id="mySomeService" class="com.springdemo.pag1.SomeServiceImpl"></bean>
      ```

   2. 声明切片对象

      ```xml
      <bean id="myAspectJ" class="com.springdemo.pag1.MyAspectJ"></bean>
      ```

   3. 声明 aspectJ 中的自动代理器生成标签，自动代理生成器：用来完成代理对象的自动创建功能。使用 aspectJ 框架内部的功能，创建目标对象的代理对象，这个创建过程是在内存中实现的，修改了目标对象的内存中的结构，创建为目标对象，所以目标对象就是被修改后的代理对象。aspectj-autoproxy 能够一次性将所有的目标对象都生成代理。

      ```xml
      <aop:aspectj-autoproxy />
      ```

5. 创建测试类，从 spring 容器中获取目标对象（实际是代理对象），通过代理执行方法，实现 AOP 功能增强。在以下代码中，当目标对象执行 `doSomeService();` 方法时，先执行切面类中的 myBefore 方法，然后再执行目标类中的方法。

   ```java
   @Test
   public void test() {
       String config = "applicationContext.xml";
       ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
       //获取目标对象
       SomeService mySomeService = (SomeService) applicationContext.getBean("mySomeService");
       //通过代理的对象执行目标对象中的方法，实现功能增强
       mySomeService.doSomeService();
   }
   ```

   在以上中，加入 `System.out.println(mySomeService.getClass().getName());` 获取代理名称，如下 `com.sun.proxy.$Proxy6` ，发现底层使用的是 JDK 的动态代理

以上配置文件的执行分析：

```xml
<!--声明目标对象-->
<bean id="mySomeService" class="com.springdemo.pag1.SomeServiceImpl"></bean>
 <!--声明切面对象-->
<bean id="myAspectJ" class="com.springdemo.pag1.MyAspectJ"></bean>
<!--声明自动代理生成器-->
<aop:aspectj-autoproxy />
```

在上述配置文件中，执行第二行和第四行后，只是在容器内生成了两个对象，未实现任何目标代理，通过第六行代码，调用 aspectJ 框架，在切面类中找到 @AspectJ 注解和 @Before 注解，在 @Before 中找到功能增强的目标类，实现动态代理功能增强。

对以上的 `@Before(value = "execution(public void com.springdemo.pag1.SomeServiceImpl.doSomeService())")` 进行优化：

```java
@Before(value = "execution(void com.springdemo.pag1.SomeServiceImpl.doSomeService())")
```

```java
@Before(value = "execution(void *..pag1.SomeServiceImpl.doSomeService())")
```

```java
@Before(value = "execution(void *..pag1.SomeServiceImpl.doSomeService(..))") //若有参数
```

```java
@Before(value = "execution(void *..SomeServiceImpl.do*(..))")
```

其实就是采用通配符不断优化。

若以上注解中，@Before(value = "execution(void com.springdemo.pag1.SomeServiceImpl.doSomeService())") 中的目标方法出错，则不会报错，但是也不会按照动态代理实现功能增强。

什么时候使用 AOP：

1. 给系统中存在的一个类增加功能，但是没有源码，则可用 AOP 增加功能
2. 给多个类增加功能，可使用 AOP
3. 给业务方法增加功能，实现日志输出

**通知方法中的参数 JoinPoint**

作用：可在通知方法中获取方法执行的信息，例如方法名，方法参数

若 SomeService 中方法为 void doSomeService(String name, int age);，在切面类中定义的增强方法如下

```java
@Before(value = "execution(public void com.springdemo.pag1.SomeServiceImpl.doSomeService())")
public void myBefore(JoinPoint joinPoint) {
    //方法签名：包名 + 类名 + 方法名
    System.out.println(joinPoint.getSignature());
    //方法名
    System.out.println(joinPoint.getSignature().getName());
    //参数，以数组形式返回
    Object[] args = joinPoint.getArgs();
    for (Object arg : args) {
        System.out.println(arg);
    }
    System.out.println("before... " + new Date());
}
```

通过动态代理，测试 doSomeService 方法

```java
@Test
public void test() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //获取目标对象
    SomeService mySomeService = (SomeService) applicationContext.getBean("mySomeService");
    //得到代理名称
    System.out.println(mySomeService.getClass().getName());
    //通过代理的对象执行目标对象中的方法，实现功能增强
    mySomeService.doSomeService("Kurisu", 18);
}
```

结果如下，发现 aspectJ 使用的是 jdk 动态代理实现。

```html
com.sun.proxy.$Proxy6
void com.springdemo.pag1.SomeService.doSomeService(String,int)
doSomeService
Kurisu
18
before... Fri Sep 17 11:27:47 CST 2021
Kurisu18 doSomeService...
```

**后置注解的使用**

后置注解切面类中的方法要求如下：

1. public
2. 无返回值
3. 方法名自定义
4. 方法有参数，推荐 Object，参数名自定义

`@AfterReturning` 后置通知，属性 value，和 `@Before` 相同，为切入点表达式，还有一个属性为 returning，表示方法的返回值，这里方法的返回值是目标方法的返回值。

```java
@AfterReturning(value = "execution(* com.springdemo.pag2.*.doOther*(..))", returning = "res")
```

目标类中定义的方法如下：

```java
@Override
public String doOtherService(String name, int age) {
    System.out.println(name + age + " doOtherService...");
    return "abcd";
}
```

特点：

1. 目标方法之后执行

2. 能获取目标方法的返回值，可根据返回值的不同做不同处理

3. 可修改返回值

   ```java
   @AfterReturning(value = "execution(* com.springdemo.pag2.*.doOther*(..))", returning = "res")
   public void myAfterReturning(Object res) {
       System.out.println("AfterReturning.... " + res);
   }
   ```

在测试方法中，通过代理的对象执行目标对象中的方法实现功能增强，同时获得返回值

```java
@Test
public void test() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //获取目标对象
    SomeService mySomeService = (SomeService) applicationContext.getBean("mySomeService");
    //通过代理的对象执行目标对象中的方法，实现功能增强
    String s = mySomeService.doOtherService("Kurisu", 18);
    System.out.println("s = " + s);
}
```

结果如下：

```html
Kurisu18 doOtherService...
AfterReturning.... abcd
s = abcd
```

若在后置通知中将方法的返回值进行修改，在测试类中并不会得到修改后的值，仍然为目标方法中返回的内容。以下程序中，后置通知的执行顺序：`1、Object res = doOther*(..)；2、myAfterReturning(res)`，因此由于 doOther\* 方法返回的是 String 类型，而 java 中调用方法  myAfterReturning 为值传递，因此不会改变原本值 res 的结果。

```java
@AfterReturning(value = "execution(* com.springdemo.pag2.*.doOther*(..))", returning = "res")
public void myAfterReturning(Object res) {
    System.out.println("AfterReturning.... " + res);
    res = "Hello aspectj";
}
```

若 doOther\* 类型为引用类型（包括自己定义的类），那么 myAfterReturning 传入参数 res 为引用，则通过 myAfterReturning 改变 res 后会将原本值改变，即测试类中得到的结果为切面对象修改后的内容。

若要使用 JoinPoint，则必须将 JoinPoint 放在参数列表的第一个位置

```java
public void myAfterReturning(JoinPoint joinPoint, Object res)
```

**环绕通知**

环绕通知的方法定义格式：

1. public
2. 必须有返回值，推荐使用 Object，返回值即为目标方法的返回值
3. 方法名称自定义
4. 方法有参数，固定参数为 ProceedingJoinPoint

`@Around` 为环绕通知的注解，特点：

1. 功能最强的注解
2. 可在目标方法的前后增加功能
3. 控制目标方法是否执行
4. **修改原来目标方法执行的结果**

等同于 JDK 动态代理的 InvocationHandler，参数 ProceedingJoinPoint 等同于 InvocationHandler 接口 invoke 方法中的 Method 参数。

ProceedingJoinPoint 继承自 JoinPoint，因此 ProceedingJoinPoint 可以使用 JoinPoint 中的方法，比如获取执行的方法名称，获取传入的参数等。

切面类中的环绕通知如下：

```java
@Around(value = "execution(* com.springdemo.pag3.SomeServiceImpl.*First(..))")
public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
    Object result = null;
    System.out.println("before " + new Date());
    //目标方法执行
    result = pjp.proceed();
    System.out.println("after trans");
    return result;
}
```

测试类如下：

```java
@Test
public void test() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //获取目标对象
    SomeService mySomeService = (SomeService) applicationContext.getBean("mySomeService");
    //通过代理的对象执行目标对象中的方法，实现功能增强
    String s = mySomeService.doFirst("Kurisu", 18); //执行的是切面类中的方法
    System.out.println("s = " + s);
}
```

当调用` mySomeService.doFirst` 方法时，其实调用的是切面类中的 myAround 方法，因此在 myAround 方法中改变返回值 result，也会改变调用的返回结果。

通常环绕通知应用在业务方法的调用。

**异常通知**

`@AfterThrowing`  为异常通知注解，方法要求如下：

1. public
2. 无返回值
3. 方法名称自定义
4. 方法有一个参数 Exception，还可加 JoinPoint

`@AfterThrowing` 的两个属性，一个是 value 表示切入点表达式，另一个是 throwing 表示目标方法抛出异常的对象，变量名必须和方法的 Exception 参数名相同。

特点：

1. 目标方法抛出异常执行
2. 可做目标方法的异常监控程序

```java
@AfterThrowing(value = "execution(* com.springdemo.pag3.SomeServiceImpl.do*(..))", throwing = "e")
public void myAfterThrowing(Exception e) {
    System.out.println("myAfterThrowing ...");
}
```

执行过程是 

```java
try{
	com.springdemo.pag3.SomeServiceImpl.do*(..)
} catch(Exception e) {
	myAfterThrowing()
}
```

**最终通知**

`@After` 为最终通知的注解，最终通知的方法格式：

1. public
2. 无返回值
3. 方法名称自定义
4. 无参数，可加 JoinPoint

`@After` 注解属性，value 为切入点表达式，特点：

1. 总会执行
2. 目标方法之后执行

通常用在资源释放中，和 finally 类似

```java
@After(value = "execution(* com.springdemo.pag3.SomeServiceImpl.do*(..))")
public void myAfter() {
    System.out.println("myAfter");
}
```

> @PointCut

当有大量重复的切入点表达式时，可使用 @PointCut 对切入点表达式起别名。@PointCut 中属性有 value 为切入点表达式，在自定义的方法上面，当使用了 @PointCut 后，此时方法名便为切入点表达式的别名，可用方法名来代替切入点表达式。因为该方法只是起个别名，因此方法中无需任何代码。因为该方法无需被外界调用，因此可定义为私有。

```java
@Around(value = "mypt()")
public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
    Object result = null;
    System.out.println("before " + new Date());
    //目标方法执行
    result = pjp.proceed();
    System.out.println("after trans");
    return result;
}

@Pointcut(value = "execution(* com.springdemo.pag3.SomeServiceImpl.*First(..))")
private void mypt() {

}
```

### 3.2.2 cglib

当目标类中不使用接口，其余不变时，仍然可实现动态代理，原因是 Spring 框架自动使用了 cglib 动态代理。

目标类：

```java
public class SomeServiceImpl {
    public void doSomeService(String name, int age) {
        System.out.println(name + age + " doSomeService...");
    }
}
```

切面类：

```java
@Aspect
public class MyAspectJ {
    @Before(value = "execution(* com.springdemo.pag4.SomeServiceImpl.doSomeService(..))")
    public void myBefore() {
        System.out.println("myBefore ...");
    }
}
```

配置文件：

```xml
<!--声明目标对象-->
<bean id="mySomeService" class="com.springdemo.pag4.SomeServiceImpl"></bean>
 <!--声明切面对象-->
<bean id="myAspectJ" class="com.springdemo.pag4.MyAspectJ"></bean>
<!--声明自动代理生成器-->
<aop:aspectj-autoproxy />
```

测试方法：

```java
@Test
public void test() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    //获取目标对象
    SomeServiceImpl mySomeService = (SomeServiceImpl) applicationContext.getBean("mySomeService");
    //通过代理的对象执行目标对象中的方法，实现功能增强
    System.out.println(mySomeService.getClass().getName());
    mySomeService.doSomeService("kurisu", 18);
}
```

得到结果如下：

```html
com.springdemo.pag4.SomeServiceImpl$$EnhancerBySpringCGLIB$$b2915568
myBefore ...
kurisu18 doSomeService...
```

若有接口，还想使用 cglib 动态代理，则需要在自动代理生成器中，将 proxy-target-class 设为 true 来强制使用动态代理。

```xml
<!--声明自动代理生成器-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

# 4、Spring 集成 MyBatis

使用的核心技术：IoC，IoC 可创建对象，将 MyBatis 中的对象交给 IoC 统一创建。

MyBatis 的使用步骤：

1. 定义 dao 接口，StudentDao.java
2. 定义 mapper 文件，StudentDao.xml
3. 定义主配置文件，mybatis.xml
4. 定义 dao 代理对象，StudentDao dao = SqlSession.getMapper(studentDao.class)
5. 使用 dao 代理对象查询数据库

获得 SqlSession 对象的方法：读取主配置文件，通过 build 方法获得 SqlSessionFactory ，然后使用 SqlSessionFactory 对象的 openSession 方法获取 SqlSession 对象。

主配置文件的内容：1、连接数据库的信息；2、sql 映射文件的位置。

在大型项目中，dataSource 中不使用连接池 POOLED 来维护数据库，将连接池类交给 Spring 创建。

使用 Spring 的 IoC 技术创建如下对象：

1. 独立的连接池类对象，阿里的 druid 连接池
2. SqlSessionFactory 对象
3. dao 对象

> 使用 spring 集成 MyBatis 的步骤如下

1. 新建 maven 项目
2. 加入 maven 依赖
   1. spring 依赖
   2. mybatis 依赖
   3. mysql 驱动
   4. spring 事务依赖
   5. spring-jdbc 依赖
   6. mybatis 和 spring 集成依赖
3. 创建实体类
4. 创建 dao 接口和 mapper 文件
5. 创建 mybatis 主配置文件
6. 创建 service 接口和实现类
7. 创建 spring 配置文件，创建 mybatis 对象
   1. 数据源
   2. SqlSessionFactory
   3. Dao
   4. service

> 第一个例子

pom 文件的依赖和插件如下：

```xml
<dependencies>
  <!--单元测试-->
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
  </dependency>
  <!--spring 框架-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.9</version>
  </dependency>
  <!--spring 事务-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.3.10</version>
  </dependency>
  <!--mybatis 依赖-->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
  </dependency>
  <!--spring-jdbc 依赖-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.10</version>
  </dependency>
  <!--mybatis 和 spring 集成-->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
  </dependency>
  <!--mysql 驱动-->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.25</version>
  </dependency>
  <!--druid 连接池-->
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
  </dependency>
</dependencies>

<build>
  <resources>
    <resource>
      <directory>src/main/java</directory> <!--所在的目录-->
      <includes> <!--包括目录下的.properties,.xml 文件都会扫描到-->
        <include>**/*.properties</include>
        <include>**/*.xml</include>
      </includes>
      <filtering>false</filtering>
    </resource>
  </resources>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.0</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

dao 层：

Student.java 接口：

```java
public interface StudentDao {
    int insertStudent(Student student);
    List<Student> selectStudents();
}
```

sql 映射文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.spring.dao.StudentDao">

    <insert id="insertStudent">
        insert into student values (#{stuNo}, #{stuName}, #{stuPassword}, #{major}, #{phone})
    </insert>

    <select id="selectStudents" resultType="com.spring.domain.Student">
        select * from student;
    </select>
</mapper>
```

使用 service 接口中的方法调用 dao 层的接口方法来操作数据库：

```java
public interface StudentService {
    int addStudent(Student student);
    List<Student> queryStudents();
}
```

```java
public class StudentServiceImpl implements StudentService {

    private StudentDao studentDao;

    public void setStudentDao(StudentDao studentDao) {
        this.studentDao = studentDao;
    }

    @Override
    public int addStudent(Student student) {
        int nums = studentDao.insertStudent(student);
        return nums;
    }

    @Override
    public List<Student> queryStudents() {
        List<Student> students = studentDao.selectStudents();
        return students;
    }
}
```

Mybatis 主配置文件：

```xml
<configuration>
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <typeAliases>
        <package name="com.spring.domain"/>
    </typeAliases>

    <mappers>
        <package name="com.spring.dao"/>
    </mappers>
</configuration>
```

注意，由于使用的是外部 druid 数据池来操作数据库，因此在这里不配置连接数据库的信息，在 spring 配置文件中进行配置。

spring 主配置文件：

```xml
<!--声明数据源来连接数据库-->
<bean id="myDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <!--配置数据库连接信息，使用的是 set 注入-->
    <property name="url" value="jdbc:mysql://localhost:3306/school" />
    <property name="username" value="root" />
    <property name="password" value="xxx" />
    <property name="maxActive" value="20" />
</bean>

<!--声明 SqlSessionFactory 类来创建对象-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--数据源的注入，使用的是外部数据源-->
    <property name="dataSource" ref="myDataSource" />
    <!--mybatis 主配置文件位置-->
    <property name="configLocation" value="classpath:mybatis.xml" />
</bean>

<!--创建 dao 对象，使用 SqlSession 的 getMapper-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--指定 SqlSessionFactory-->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    <!--
    指定 dao 接口所在包名，MapperFactoryBean 会扫描这个包中所有接口
    每个都执行依次 getMapper 得到每个接口的对象，多个包采用',' 分隔
    得到的对象名为接口对象的首字母小写，例如 StudentDao -> studentDao
    -->
    <property name="basePackage" value="com.spring.dao" />
</bean>

<bean id="studentService" class="com.spring.service.impl.StudentServiceImpl">
    <!--这里由于 StudentServiceImpl 有一个构造方法
	 传入的参数为 studentDao，因此需要将对象 StudentDao 传入到 StudentServiceImpl 中-->
    <property name="studentDao" ref="studentDao" />
</bean>
```

测试方法：由于在配置文件中使用 MapperScannerConfigurer 创建了 dao 接口中的对象 studentDao，因此使用 ApplicationContext 的 getBean 方法可获得这个对象来操作数据库。

```java
@Test
public void test() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    StudentDao studentDao = (StudentDao) applicationContext.getBean("studentDao");
    Student student = new Student("001", "A", "123", "AA", "12558");
    int insertStudent = studentDao.insertStudent(student);
    System.out.println("insertStudent = " + insertStudent);
}
```

使用 service 调用 dao 接口中的方法来操作数据库：

```java
@Test
public void test1() {
    String config = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
    StudentService service = (StudentService) applicationContext.getBean("studentService");
    Student student = new Student("008", "D", "123", "AA", "12558");
    int insert = service.addStudent(student);
    System.out.println("insert = " + insert);
}
```

> 使用属性配置文件

将数据库的配置信息单独写到 .properties 中，首先使用 context:property-placeholder 指定数据库配置文件 .properties 位置：

```xml
<context:property-placeholder location="classpath:jdbc.properties" />
```

再使用 ${} 将对应位置替换为 .properties 文件中的内容

```xml
<!--声明数据源来连接数据库-->
<bean id="myDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <!--配置数据库连接信息，使用的是 set 注入-->
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
    <property name="maxActive" value="${jdbc.maxActive}" />
</bean>
```

# 5、Spring 事务处理

事务指的是一组不可分割的 sql 语句集合，当涉及到多个表或多个 insert/update/delete 时，需要使用事务。在业务方法中，可能会调用多个 dao 方法执行多条 sql 语句，例如

```java
public int addStudent(Student student) {
    int nums = studentDao.insertStudent(student);
    perDao.insertPer(per); //两条插入语句
    return nums;
}
```

多种数据库的访问技术中，有不同的事务处理机制，对象和方法。针对这种问题，Spring 提供一种处理事务的统一模型，使用统一步骤来完成不同数据库访问技术的事务处理。

**声明式事务**：把事务相关的资源和内容提供给 Spring，由 Spring 来处理事务提交、回滚。

1、事务提交、回滚由 Spring 内部的事务管理器对象来完成 commit、rollback。事务管理器接口 PlatformTransactionManager 和他的多个实现类完成事务操作，实现类 DataSourceTransactionManager 可操作 jdbc 和 mybatis。在 xml 配置文件中，使用声明即可。

```xml
<bean id="xx" class="...DataSourceTransactionManager"></bean>
```

2、TransactionDefinition 接口中定义了事务描述下相关的三个常量：事务隔离级别、事务传播行为、事务默认超时时限。

1. 事务隔离级别常量：
   - DEFAULT：采用 DB 默认的事务隔离级别。MySQL 的默认为 REPEATABLE_READ
   - READ_UNCOMMITTED：读未提交。未解决任何并发问题。
   - READ_COMMITTED：读已提交。解决脏读，存在不可重复读与幻读。 
   - REPEATABLE_READ：可重复读。解决脏读、不可重复读，存在幻读 
   - SERIALIZABLE：串行化。不存在并发问题。
   
2. 事务超时时间：若方法执行超过了事务超时时间，则自动回滚，默认为 -1

3. **事务传播行为**：处于不同事务中的方法在相互调用时，执行期间事务的维护情况。事务传播行为是加在方法上的。有如下七种类型：
   
   - **PROPAGATION_REQUIRED** （默认）：指定的**方法必须在事务内执行**。若当前存在事务，就加入到当前事务中；若当前没有事务，则创建一个新事务。
   - **PROPAGATION_REQUIRES_NEW** ：总是新建一个事务，若当前存在事务，就将当前事务挂起，直到新事务执行完毕。
   - **PROPAGATION_SUPPORTS** ：指定的方法支持事务，但若没有事务，也可以以非事务方式执行
   - PROPAGATION_MANDATORY  
   - PROPAGATION_NESTED 
   - PROPAGATION_NEVER 
   - PROPAGATION_NOT_SUPPORTED
   
   | 事务传播行为类型             | 说明                                                         |
   | ---------------------------- | ------------------------------------------------------------ |
   | PROPAGATION_**REQUIRED**     | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
   | PROPAGATION_**SUPPORTS**     | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
   | PROPAGATION_MANDATORY        | 使用当前的事务，如果当前没有事务，就抛出异常。               |
   | PROPAGATION_**REQUIRES_NEW** | 新建事务，如果当前存在事务，把当前事务挂起。                 |
   | PROPAGATION_NOT_SUPPORTED    | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
   | PROPAGATION_NEVER            | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
   | PROPAGATION_NESTED           | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

3、事务回滚时机：

1. 执行事务无异常，则提交事务执行完毕
2. 抛出运行时异常或错误，则调用事务管理器的 rollback 进行回滚
3. 若抛出非运行时异常，则需要进行异常处理，主要是受检查异常，例如 IO 异常、SQL 异常等

总结：管理实务主要是事务管理器和他的实现类；Spring 事务是一个统一模型：指定要使用事务管理器的实现类，指定哪些类的哪些方法需要加入事务功能，指定方法的隔离级别、传播行为、超时时限

Spring 提供的事务处理方案：

1、适合**中小项目**的注解方案

Spring 使用 AOP 给业务添加事务功能，使用 @Transactional 注解添加事务，是 Spring 框架自己的注解，放在 public 方法上面，来添加事务。有如下属性：

![image-20210927102003008](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210927102003.png)

![image-20210927102025292](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210927102025.png)

若 @Transactional 添加在类上，则该类上的所有方法均添加事务。

使用 @Transactional 的步骤：

1. 声明事务管理器对象 DataSourceTransactionManager

   ```xml
   <bean id="xx" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       
   </bean>
   ```

2. 开启事务注解驱动

   ```xml
   <tx:annotation-driven transaction-manager="自己声明的事务管理器" />
   ```

   在待添加的 public 业务方法添加事务，直接在方法上添加 @Transactional 即可。

   ```java
   /*    @Transactional (
               propagation = Propagation.REQUIRED,
               isolation = Isolation.DEFAULT,
               readOnly = false,
               rollbackFor = {
                       NullPointerException.class,
                       NotEnoughException.class
               }
       )*/
       //直接使用默认值，和以上相同
       @Transactional
   ```

   对于 rollbackFor 说明如下：若业务方法的异常为 rollbackFor 中声明的异常，则回滚；若异常不在 rollbackFor 中，则判断是否为 RuntimeException，若是，则回滚。通常使用默认值即可。

   spring 使用环绕通知为业务方法添加提交或回滚事务（由 spring 完成）

   ```java
   @Around("业务方法名称")
   Object myAround() {
       开启事务，由 spring 开启
       try {
           /业务方法
       	spring 事务管理 commit()
       } catch(Exception e) {
           spring 事务管理 rollback()
       }
   }
   ```

2、在**大型项目**中，使用 AspectJ 框架来实现事务，在 Spring 配置文件中声明需要添加事务的方法、类，来使业务方法和事务完全分离。

步骤如下：

1. 加入 AspectJ 依赖

   ```xml
   <!--AspectJ 依赖-->
   <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-aspects</artifactId>
     <version>5.3.9</version>
   </dependency>
   ```

2. 声明事务管理器对象（和第一种配置方式相同）

3. 声明需要添加的事务类型（传播行为、隔离级别等）

4. 在 Spring 主配置文件中配置需要添加事务的业务方法、类

      ```xml
      <tx:advice id="myAdvice" transaction-manager="transactionManager">
          <tx:attributes>
              <!--给具体方法配置事务属性-->
              <tx:method name="buy" propagation="REQUIRED" isolation="DEFAULT" read-only="false"
                         rollback-for="java.lang.NullPointerException, com.spring.exception.NotEnoughException"/>
          </tx:attributes>
      </tx:advice>
      ```

      `<tx:advice>` 表示添加事务类型，id 为自定义名称，transaction-manager 为事务管理器；tx:attributes 表示添加事务属性；tx:method 表示给具体方法配置事务属性，name 为添加事务的方法，不带包和类，可使用通配符，propagation、isolation、read-only、rollback-for 和之前相同。

5. 配置 AOP

   ```xml
   <!--配置 aop-->
   <aop:config>
       <!--
   	   配置切入点表达式
          id 为切入点表达式唯一名称
          expression 为切入点表达式
       -->
       <!--execution(* *..service..*.*(..)) 表示 service 包中的所有子包中的所有方法配置为切入点-->
       <aop:pointcut id="servicePt" expression="execution(* *..service..*.*(..))"/>
   
       <!--配置增强器：关联 advice 和 pointCut-->
       <aop:advisor advice-ref="myAdvice" pointcut-ref="servicePt" />
   </aop:config>
   ```


# 6、销售表和商品表小项目

有两个表格，goods 和 sale，模仿购物行为

1、创建 maven，添加依赖（和 4 中的依赖相同）

2、创建 domain 实体类，Goods.java 和 sale.java

Goods.java：

```java
public class Goods {
    private Integer id;
    private String name;
    private int amount;
    private Float prices;
    
    setter....
}
```

Sale.java：

```java
public class Sale {
    private Integer id;
    private Integer gid;
    private Integer nums;
    
    setter....
}
```

3、创建 dao 持久层接口

GoodsDao.java

```java
public interface GoodsDao {
    //更新库存
    int updateGoods(Goods goods);

    //根据商品 id 查询商品信息
    Goods selectGoods(Integer id);
}
```

SaleDao.java

```java
public interface SaleDao {
    //增加销售记录
    int insertSale(Sale sale);
}
```

通过观察 dao 文件，我们发现对数据库的插入、更新方法传入的参数都为实体类 Goods 或 Sale。

4、创建 mapper 映射文件

GoodsDao.xm

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.spring.dao.GoodsDao">

    <update id="updateGoods">
        update goods set amount = amount - #{amount} where id = #{id}
    </update>

    <select id="selectGoods" resultType="com.spring.domain.Goods">
        select id, name, amount, prices from goods where id = #{id}
    </select>
</mapper>
```

SaleDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.spring.dao.SaleDao">
    <!--仅添加 gid 和 nums-->
    <insert id="insertSale">
        insert into sale(gid, nums) values(#{gid}, #{nums})
    </insert>
</mapper>
```

5、创建 mybatis 主配置文件，使用 druid 外部数据源

mybatis.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <typeAliases>
        <package name="com.spring.domain"/>
    </typeAliases>

    <mappers>
        <package name="com.spring.dao"/>
    </mappers>
</configuration>
```

通过以上步骤，对数据库的操作基本完成，接下来写业务代码

6、创建业务接口

```java
public interface BuyGoodsService {
    void buy(Integer goodsId, Integer nums);
}
```

7、创建业务逻辑的实现类

```java
package com.spring.service.impl;

import com.spring.dao.GoodsDao;
import com.spring.dao.SaleDao;
import com.spring.domain.Goods;
import com.spring.domain.Sale;
import com.spring.exception.NotEnoughException;
import com.spring.service.BuyGoodsService;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

public class BuyGoodsServiceImpl implements BuyGoodsService {

    private GoodsDao goodsDao;
    private SaleDao saleDao;

    public void setGoodsDao(GoodsDao goodsDao) {
        this.goodsDao = goodsDao;
    }

    public void setSaleDao(SaleDao saleDao) {
        this.saleDao = saleDao;
    }

/*    @Transactional (
            propagation = Propagation.REQUIRED,
            isolation = Isolation.DEFAULT,
            readOnly = false,
            rollbackFor = {
                    NullPointerException.class,
                    NotEnoughException.class
            }
    )*/
    //直接使用默认值，和以上相同
    @Transactional
    @Override
    public void buy(Integer goodsId, Integer nums) {
        //记录消费，向 sale 表中添加记录
        Sale sale = new Sale();
        sale.setGid(goodsId);
        sale.setNums(nums);
        saleDao.insertSale(sale);
        //更新库存
        Goods goods = goodsDao.selectGoods(goodsId);
        if (goods == null) {
            throw new NullPointerException(goodsId + " 商品不存在");
        } else if (goods.getAmount() < nums) {
            throw new NotEnoughException(goodsId + " 商品不足");
        }
        Goods buyGoods = new Goods();
        buyGoods.setId(goodsId);
        buyGoods.setAmount(nums);
        goodsDao.updateGoods(buyGoods);

    }
}
```

注意在这里对业务方法 buy 添加了事务，使用的是 spring 中的注解添加。

8、完成了以上操作后，配置 spring 主配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--告诉 spring 配置文件的位置-->
    <context:property-placeholder location="classpath:jdbc.properties" />

    <!--声明数据源来连接数据库-->
    <bean id="myDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!--配置数据库连接信息，使用的是 set 注入-->
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="maxActive" value="${jdbc.maxActive}" />
    </bean>

    <!--声明 SqlSessionFactory 类来创建对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--数据源的注入，使用的是外部数据源-->
        <property name="dataSource" ref="myDataSource" />
        <!--mybatis 主配置文件位置-->
        <property name="configLocation" value="classpath:mybatis.xml" />
    </bean>

    <!--创建 dao 对象，使用 SqlSession 的 getMapper，没有 id 属性-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--指定 SqlSessionFactory-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
        <!--
        指定 dao 接口所在包名，MapperFactoryBean 会扫描这个包中所有接口
        每个都执行依次 getMapper 得到每个接口的对象，多个包采用',' 分隔
        得到的对象名为接口对象的首字母小写，例如 StudentDao -> studentDao
        -->
        <property name="basePackage" value="com.spring.dao" />
    </bean>

    <bean id="buyGoodsService" class="com.spring.service.impl.BuyGoodsServiceImpl">
        <!--这里传入的 goodsDao 和 saleDao 分别在上面那个 bean 中通过 getMapper 创建-->
        <property name="goodsDao" ref="goodsDao" />
        <property name="saleDao" ref="saleDao" />
    </bean>
	<!--声明事务管理器对象-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--连接的数据库信息，指定数据源-->
        <property name="dataSource" ref="myDataSource" />
    </bean>

    <!--开启事务注解驱动-->
    <tx:annotation-driven transaction-manager="transactionManager" />

</beans>
```

jdbc.properties 配置文件如下：

```properties
jdbc.url = jdbc:mysql://localhost:3306/springdemo
jdbc.username = root
jdbc.password = @Kurisu&0725
jdbc.maxActive = 20
```

在主配置文件中，配置的主要 `<bean>` 如下：

1. DruidDataSource：数据源，这里使用的是 druid
2. SqlSessionFactoryBean：用于之后使用 getMapper 方法来创建 dao 接口中的对象。其中 SqlSessionFactoryBean 中主要配置的属性为 dataSource 数据源和 configLocation mybatis 主配置文件
3. MapperScannerConfigurer：使用 getMapper 创建 dao 接口中的对象，对象名为 dao 接口中首字母小写，例如 StudentDao -> studentDao
4. 以上 `<bean>` 创建完成之后，再创建业务的 impl 实现方法，并对 impl 实现方法赋值，赋的值为 dao 接口中的接口
5. 根据需求配置事务，分别声明事务管理器对象 DataSourceTransactionManager（属性中传入数据源）和开启事务注册驱动 tx:annotation-driven
6. 对于配置事务的操作，也可采用 AspectJ 框架来实现，配置步骤如下：
   1. 声明事务管理器
   2. 使用 `<tx:advice>` 标签声明事务类型，声明需要添加事务的方法
   3. 使用 `<aop:config>` 配置 AOP，配置切入点表达式，并使用 `<aop:advisor>` 标签将 advice 和 pointCut 关联起来。

# 7、Web 项目中使用容器对象

以学生注册为例，web 前端 index.jsp 设置为

![image-20210928105736910](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20210928105737.png)

核心的 daomain、dao、service 和之前操作学生表相同，插入操作和查询操作。在 controller 控制包中，index.jsp 跳转到 RegisterServlet 进行控制，如下：

```java
public class RegisterServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String stuNo = req.getParameter("stuNo");
        String stuName = req.getParameter("stuName");
        String stuPassword = req.getParameter("stuPassword");
        String major = req.getParameter("major");
        String phone = req.getParameter("phone");

        //创建 spring 对象的容器信息
        String config = "applicationContext.xml";
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
        StudentService service = (StudentService) applicationContext.getBean("studentService");
        Student student = new Student();
        student.setStuNo(stuNo);
        student.setStuName(stuName);
        student.setStuPassword(stuPassword);
        student.setMajor(major);
        student.setPhone(phone);
        service.addStudent(student);

        //跳转成功页面
        req.getRequestDispatcher("/result.jsp").forward(req, resp);
    }
}
```

在练习中遇到如下几个问题：

- Invalid bound statement (not found)：mapper 文件的 namespace 配置错误导致，或者 pom 文件中未添加

  ```xml
  <resource>
    <directory>src/main/java</directory> <!--所在的目录-->
    <includes> <!--包括目录下的.properties,.xml 文件都会扫描到-->
      <include>**/*.properties</include>
      <include>**/*.xml</include>
    </includes>
    <filtering>false</filtering>
  </resource>
  ```

  没有将 .xml 文件添加到 class 目录下

- 在 RegisterServlet 中获取到某个值为 null，则可能是 getParameter 方法中参数和表单中 name 不同

如上代码中有一些问题，如每次请求时都会新建 ApplicationContext 对象，导致资源浪费，可以对容器对象只创建一次，然后将容器对象添加到全局作用域 ServletContext 中。

解决方法是使用监听器，创建 ApplicationContext 对象，然后将容器对象添加到 ServletContext 中。可自己创建监听器，也可以使用框架的 ContextLoaderListener。

步骤：

1、加入 spring-web 依赖

```xml
<!--监听器依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>5.3.10</version>
</dependency>
```

2、在 web.xml 中注册监听器

```xml
<!--注册监听器-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

若此时执行，则会抛出异常

```
IOException parsing XML document from ServletContext resource [/WEB-INF/applicationContext.xml];
```

监听器会默认在 WEB-INF 目录下读取文件名为 applicationContext.xml 的配置文件，然后创建容器对象，可自行对文件位置进行修改：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

3、自己创建监听器对象

```java
WebApplicationContext applicationContext = null;
//获取 ServletContext 中的容器对象
String key = WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE;
//getServletContext 是 HttpServlet 父类 GenericServlet 中的方法
Object attribute = getServletContext().getAttribute(key);
if (attribute != null) {
    applicationContext = (WebApplicationContext) attribute;
}
```

也可以使用框架中的方法创建监听器对象，框架创建 WebApplicationContext 和上述自己创建 WebApplicationContext 方法相同，都是使用 getServletContext().getAttribute() 方法获得 attribute 然后判断是否为空，再进行强行转换。

```java
ServletContext servletContext = getServletContext();
WebApplicationContext applicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(servletContext);
```









