# 1、概述

SpringBoot 官方文档 https://docs.spring.io/spring-boot/docs/current/reference/html/index.html

SpringBoot 特性 https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features

SpringBoot 可用于不同场景 https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters

SpringBoot 不同配置参数 https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties

SpringBoot 是基于 Spring Framework，整合 Spring 技术栈，简化配置文件。Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。Spring Boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包，Spring Boot 整合了所有的框架。

**SpringBoot 优点：**

- 创建独立 Spring 应用
- 内嵌 web 服务器
- 自动导入依赖，简化配置
- 自动化配置 Spring 及第三方功能
- 无代码生成，无需编写 xml

# 2、入门案例

步骤：

1. 创建 maven 项目
2. 导入依赖
3. 编写主程序
4. 编写业务程序
5. 测试
6. 简化配置
7. 简化部署

详细如下：

pom 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.springboot</groupId>
    <artifactId>springboot-test-1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--SpringBoot-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.5</version>
    </parent>

    <dependencies>
        <!--web 开发-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

parent 标签是 SpringBoot的父级依赖，用来声明此项目为 SpringBoot 项目，SpringBoot 中 parent 标签的作用：

1、统一管理 jar 包的版本，其依赖需要在子工程中定义才有效

2、统一的依赖管理（父工程的 <dependencies\>，子工程不必重新引入）

3、控制插件的版本

4、聚合工程

在 parent 标签中，定义了一个父标签 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。

使用默认编码格式为 UTF-8

定义了 Java 编译版本为 1.8

定义了针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml 执行打包操作的配置，自动化的资源过滤，自动化的插件配置。

在**主程序类**中，使用 @SpringBootApplication 来进行声明，主程序类写法几乎固定。SpringApplication.run 返回的是 IoC 容器 ConfigurableApplicationContext，可使用 getBean 方法获得不同对象。

```java
/**
 * @SpringBootApplication 用来声明这是一个 SpringBoot 应用
 * 主程序类
 */
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
```

对于**控制器方法** controller，编写如下。从 @RestController 类源码可以看出 @RestController 是 @Controller 和 @ResponseBody 两个注解的结合体。

```java
//@ResponseBody
//@Controller
//可用 @RestController 来代替 @ResponseBody 和 @Controller
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String myController1() {
        return "Hello SpringBoot!";
    }

}
```

完成以上步骤后运行即可，因为 SpringBoot 有内嵌 web 服务器 Tomcat，默认端口为 8080，因此不需要配置服务器。

若想**更改配置**，例如 Spring、SpringMVC 或服务器的一些配置，都可在配置文件中新建 application.properties 进行更改，例如更改端口号为 80，则可作如下更改

```properties
server.port=80
```

对于其他配置，参考 https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties 官方配置文档。

例如访问根路径的设置，访问路径为 localhost:8080/springboot-test-3，可设置

```properties
server.servlet.context-path=springboot-text-3
```

对于部署，可将项目打为可执行 jar 包，jar 包中包括了项目中所有环境和服务器，之后尽管 idea 中程序服务器关闭，也仍可在目标服务器通过 jar 包启动项目，在 pom 文件中新增的插件如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!--和 SpringBoot 版本号相同-->
            <version>2.5.5</version>
        </plugin>
    </plugins>
</build>
```

![image-20211007111154116](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007111154.png)

之后在目标目录下启动命令行，使用 java -jar springboot-test-1-1.0-SNAPSHOT.jar 便可完成部署，命令行关闭则对应服务器关闭。

![image-20211007111452316](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007111452.png)

# 3、自动配置原理

对于以上案例进行详细说明。

## 1、SpringBoot 特点

### 1.1 依赖管理

在 pom 文件中，首先引入了父项目 parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.5</version>
</parent>
```

进入源码可发现 parent 的父项目

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.5.5</version>
</parent>
```

再次进入可发现有大量依赖，因此在之后的依赖 dependencies 标签中不需要写版本号。称为 SpringBoot 的自动版本仲裁机制。

![image-20211007112152036](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007112152.png)

若想修改版本，即不使用 SpringBoot 中提供的版本，则依然需要使用底层的方法，在 pom 文件中，将不符合要求的重写即可。

```xml
<properties>
    <mysql.version>5.1.14</mysql.version>
</properties>
```

starter 称为场景启动器，对于某种场景，引入一个 starter 则该场景的所有依赖均会被引入，例如刚刚引入的 spring-boot-starter-web，Web 服务应用。

### 1.2 自动配置

- 自动配置 SpringMVC，引入 SpringMVC 常用组件和功能
- 自动配置常用问题，例如编解码问题
- 默认包结构，无需使用 context:component-scan 进行组件扫描，默认主程序所在包及其子包均会被扫描。
- 不同配置都有默认值，可在 resources 中创建 application.properties 中进行更改。不同配置的值最终绑定在不同类上，进行赋值。
- 按需加载自动配置，由于有大量的 starter 场景，只有在 pom 文件中配置了的 starter 才会生效，其余不会生效。

在 spring-boot-starter-web 依赖中，引入如下依赖

![image-20211007113432646](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007113432.png)

## 2、容器功能

### 2.1 组件添加

> @Configuration

放在类上，用来声明一个配置类，在类中可使用 @Bean 来进行注册组件，和 Spring 配置文件中使用 <bean\> 进行注册相同，其中，方法返回值类型表示组件类型，方法名表示 id，new User("Kurisu", 23) 使用 User 中的构造方法，来对 name 和 age 进行赋值。

```java
//当使用 @Bean("user") 时，创建的组件对象的 id 为 user，没有则默认为方法名
@Bean("user")
public User user01() {
	...
}
```

```java
//声明这是一个配置类
@Configuration
public class MyConfig {

    @Bean
    public User user01() {
        return new User("Kurisu", 23);
    }

}
```

```java
public class User {
    private String name;
    private Integer age;

    public User() {
    }

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

因为只使用了一个方法，相当于 <bean\> 中创建了一个对象，因此容器中只有一个 id 为 user01 的实例，当使用容器 getBean 方法获得 user01 时，获得的对象均相同。默认为单实例。

```java
ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
User user01 = run.getBean("user01", User.class);
User user02 = run.getBean("user01", User.class);
System.out.println(user01 == user02); //输出 true
```

此外，配置类 MyConfig 本身也是一个组件，也可使用容器的 getBean 获取到，获取到 MyConfig 后，可调用 MyConfig 中的 user01() 方法来获取组件。

```java
MyConfig bean = run.getBean(MyConfig.class);
User user = bean.user01();
User user1 = bean.user01();
System.out.println(user == user1); //true
```

注意，以上是在配置类上的注解 @Configuration 默认情况下，即 @Configuration 中的 proxyBeanMethods 属性默认为 true 的情况下的结果，proxyBeanMethods 用来设置代理 bean 的方法。在默认 true 的情况下，当使用 MyConfig 的方法获取到响应组件时，调用的是容器中的组件对象，而容器中的组件对象是单实例，因此相等为 true，而当 proxyBeanMethods 属性为 false 时，这时再次执行以上代码，通过方法获取组件，是通过新建了不同的组件，输出结果为 false。

```java
@Configuration(proxyBeanMethods = false)
public class MyConfig {
	....
}
```

以上对应了 Full 模式和 Lite 模式

- Full：proxyBeanMethods = true
- Lite：proxyBeanMethods = false

存在组件依赖时，应使用 Full 模式。组件依赖例子如下：

```java
@Bean
public User user01() {
    User user = new User("Kurisu", 23);
    user.setPet(pet());
    return user;
}

@Bean
public Pet pet() {
    return new Pet("Tom");
}
```

若 proxyBeanMethods = true，则使用容器获得 user01 后，它的 pet 属性为通过容器获得 pet 的组件；当 proxyBeanMethods = false，则不保证组件依赖。

除此之外，在之前学的 Spring 框架中的 @Component、@Repository、@Controller、@Service 等注解均可使用。接下来再说明下 @Import

> @Import

@Import 可用在类上，@Import 中的属性为 class 数组，可用于在容器中注册组件，默认组件的名字就是全类名。

```java
@Import({User.class, Pet.class})
```

>@Conditional

条件注册

![image.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211007145539.png)

```java
//当容器中有 pet 组件时，才会注册 user01 组件
@ConditionalOnBean(name="pet")
@Bean
public User user01() {
    User user = new User("Kurisu", 23);
    user.setPet(pet());
    return user;
}
```

### 2.2 原生配置文件引入

> @ImportResource

当使用 Spring 的 xml 配置文件注册组件时，可使用 @ImportResource 将其引入，不然 SpringBoot 扫描不到。

```java
@ImportResource("classpath:beans.xml")
```

### 2.3 配置绑定

>@Component 和 @ConfigurationProperties

对于要给类中属性输入的内容，在配置文件 application.xml 配置了之后，可使用 @ConfigurationProperties 注解自动配置。@ConfigurationProperties 放在需要赋值的类上，然后将 @ConfigurationProperties 中的属性 prefix 进行赋值后，便可对类中属性自动赋值。注意，除了在类上加 @ConfigurationProperties，还需要添加 @Component 将此类添加到容器中，因为只有在容器中才可使用 SpringBoot 提供的注解。

```properties
mycar.brand=abcd
mycar.prices=1000
```

```java
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer prices;
    
    构造方法
    getter and setter ....
}
```

> @EnableConfigurationProperties 和 @ConfigurationProperties

@EnableConfigurationProperties(Car.class) 需要加在配置类上，功能是

1. 将 Car 这个类添加到容器中
2. 开启 Car 的自动配置绑定功能

除了在配置类上加 @EnableConfigurationProperties 注解外，仍需要在对应 Car 类上加 @ConfigurationProperties(prefix = "mycar") 来自动配置。

```java
@Configuration(proxyBeanMethods = true)
@EnableConfigurationProperties(Car.class)
public class MyConfig {
	....
}
```

```java
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer prices;
    
    ...
}
```

```properties
mycar.brand=abcd
mycar.prices=1000
```

## 3、自动配置原理

@SpringBootApplication 源码：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

@SpringBootConfiguration：声明这是一个配置类

```java
@Configuration
@Indexed
public @interface SpringBootConfiguration {
    ...
}
```

@ComponentScan：指定包扫描

**@EnableAutoConfiguration**：

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

1. @AutoConfigurationPackage：给容器中导入一个组件 Registrar.class，而 Registrar 给容器中批量导入组件，通过 registerBeanDefinitions 方法中的 PackageImports 方法，将主程序类下的所有包（子包）下的组件导入到容器中。

   ```java
   @Import({Registrar.class})
   public @interface AutoConfigurationPackage {
       String[] basePackages() default {};
   
       Class<?>[] basePackageClasses() default {};
   }
   ```

   Registrar.class 部分源码

   ```java
   public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
       AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
   }
   
   public Set<Object> determineImports(AnnotationMetadata metadata) {
       return Collections.singleton(new AutoConfigurationPackages.PackageImports(metadata));
   }
   ```

2. @Import({AutoConfigurationImportSelector.class})：AutoConfigurationImportSelector.class 部分源码如下：

   1、调用 getAutoConfigurationEntry 方法来给容器中导入一些组件

   2、调用 List<String\> configurations = this.getCandidateConfigurations(annotationMetadata, attributes); 获得一些组件，获得的组件共有 127 个，已经在配置文件 

   3、利用工厂加载 Map<String, List<String\>> loadSpringFactories(ClassLoader classLoader) 获得所有组件

   4、根据指定，从 META-INF/spring.factories 中加载组件，核心的包为 spring-boot-autoconfiguer，在 META-INF/spring.factories 中已经写定了之前的 127 个组件，这 127 个组件在启动时全部加载，但是不会全部生效，根据条件装配注解，即 @Conditional 注解按需生效。

   ```java
   public String[] selectImports(AnnotationMetadata annotationMetadata) {
       if (!this.isEnabled(annotationMetadata)) {
           return NO_IMPORTS;
       } else {
           AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
           return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
       }
   }
   
   protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
       if (!this.isEnabled(annotationMetadata)) {
           return EMPTY_ENTRY;
       } else {
           AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
           List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
           configurations = this.removeDuplicates(configurations);
           Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
           this.checkExcludedClasses(configurations, exclusions);
           configurations.removeAll(exclusions);
           configurations = this.getConfigurationClassFilter().filter(configurations);
           this.fireAutoConfigurationImportEvents(configurations, exclusions);
           return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
       }
   }
   
   ```

   ![image-20211008111021986](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211008111022.png)

之前说过，SpringBoot 中不需要解决中文乱码问题，这是因为在 HttpEncodingAutoConfiguration 中，配置如下组件，并且注解 @ConditionalOnMissingBean，在未编 CharacterEncodingFilter 方法时，按照 SpringBoot 的默认方法，使用 shouldForce 默认编码为 UTF-8，若不满足 SpringBoot 中 shouldForce 默认的编码 UTF-8，可修改配置文件 application.xml 进行更改。当用户编写了 CharacterEncodingFilter 方法后，由于 @ConditionalOnMissingBean ，按照用户编写的优先。

```java
@Bean
@ConditionalOnMissingBean
public CharacterEncodingFilter characterEncodingFilter() {
    CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
    filter.setEncoding(this.properties.getCharset().name());
    filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.REQUEST));
    filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.RESPONSE));
    return filter;
}
```

# 4、开发小技巧

## lombok

不同类都需要加入 getter 和 setter 方法，有参无参构造，通过 lombok 方法可在编译期间自动生成这些方法。

步骤：

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
   </dependency>
   ```

2. 在 Plugins 中安装 Lombok 插件

使用 lombok 后，类可写为

```java
@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer prices;
}
```

- @Data：getter 和 setter

- @ToString：重写 toString 方法

- @AllArgsConstructor：全参构造器

- @NoArgsConstructor：无参构造器


还可在控制器类上加 @Slf4j 用来调试、日志输出等，加入 @Slf4j 后，便可使用 log 中的方法进行输出、调试等。

```java
@RestController
@Slf4j
public class HelloController {

    @RequestMapping("/hello")
    public String myController1() {
        log.info("Slf4j 方法");
        return "Hello SpringBoot!";
    }

}
```

## Dev-tools

在 pom 文件中加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

之后再修改代码后，只需要 ctrl + F9 重新编译即可

![image-20211009112418410](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211009112418.png)

## Spring Initializr

创建项目时

![image-20211009112921395](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211009112921.png)

选择需要的场景

![image-20211009112939537](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211009112939.png)

之后会自动搭建项目结构

![image-20211009113214291](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211009113214.png)

resources 中 static 包放的是 web 静态资源，templates 放的是页面

# 5、配置文件

## 1、properties

可使用 properties 对 SpringBoot 进行配置

## 2、yaml

也可使用 yaml，yaml 非常适合做以数据为中心的配置文件。

语法：

- key: value；kv之间有空格
- 大小写敏感

- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格（IDEA 中可用）

- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释

- 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义

对象：键值对的集合。map、hash、set、object 

```html
行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3
```

数组：一组按次序排列的值。array、list、queue

```html
行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

在写 yaml 文件时，IDEA 不会对类中属性进行提示，可在 pom 文件中加入如下依赖，IDEA 便可有提示。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

由于这个 jar 包依赖和业务无关，因此在打包时不必添加此 jar 包，在 pom 文件中配置如下插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 6、Web 开发

## 6.1 静态资源访问

- classpath：/META-INF/resources/
- classpath：/resources/
- classpath：/static/
- classpath：/public/
- /：当前项目的根路径

在以上路径下寻找静态资源，级别由高到低

由于静态资源映射的是 /**，因此对于请求，先在 Controller 中进行请求处理，若在 Controller 中不可处理，则在静态资源中进行寻找处理，若还是不可处理，则 404。

静态资源访问前缀：

```yaml
spring:
  mvc:
    static-path-pattern: /springboot-test-3/**
```

加上前缀后，之后访问所有静态资源时必须加上 springboot-test-3 前缀。因此加上静态资源前缀后，访问欢迎页也必须加上静态资源前缀。

## 6.2 欢迎页

在静态资源目录下，新建一个 index.html

注意，若 index 为 html 文件，需要引入 thymeleaf 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

对于访问时出现的小图标 ![image-20211010143832857](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211010143833.png) ，则生成 ico 的图片，更名为 favicon.ico，放到静态资源目录下即可。

在操作过程中，发现 ico 图标无法生效，参考 https://www.cnblogs.com/jusha/p/11979734.html，由于自己配置了项目名，由于谷歌浏览器的策略， ico 图标无法生效，有两种解决方法

```yaml
server:
  servlet:
    context-path: /springboot-test-3
```

方案一：不配项目名，http://域名(或者IP+端口)/favicon.ico能访问到图片，就没有问题

方案二：页面明确指定ico

```html
<link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
```

但是这样比较费劲，所有页面都要加，折中方案就是用母版页，统一添加，已有项目的话改动也比较大，

折中方案二：通过拦截器，在页面渲染完成后追加一段js，

可以继承 HandlerInterceptor 接口，重写 afterCompletion 方法，添加以下代码通过 js 写入

```javascript
String link = "<script>" +
                            "var link = document.createElement('link');" +
                            "link.type = 'image/x-icon';" +
                            "link.rel = 'shortcut icon';" +
                            "link.href = '/nascan/images/favicon.ico';" +
                            "document.getElementsByTagName('head')[0].appendChild(link);" +
                            "</script>";

                    response.getWriter().append(link);
```

## 6.3 静态资源配置原理

- SpringBoot 启动后，默认加载指定的 xxxAutoConfiguration 类
- SpringMVC 功能的自动配置类 **WebMvcAutoConfiguration**，对 WebMvcAutoConfigurationAdapter 进行了 @EnableConfigurationProperties 配置，其中 WebMvcProperties 和 spring.mvc 绑定，ResourceProperties 和 spring.resources 绑定，WebProperties 和 spring.web 绑定。

当配置类只有一个有参构造器时，有参构造器中所有参数的值都会从容器中确定。

静态资源可通过 add-mappings 设置是否可访问，设置为 false 后，所有静态资源不可访问。

```yaml
spring:
  web:
    resources:
      add-mappings: false
```

## 6.4 请求参数处理

> 请求映射

Rest 风格，用户提交的请求可能为 GET、POST、DELETE、PUT，但是表单 form 只能处理 GET 和 POST 请求，根据源码，可通过设置隐藏表单和在配置文件中开启隐藏模式，使用 DELET 和 PUT 提交。

```html
<!--get-->
<form action="hello" method="get">
    <input value="get" type="submit" />
</form>
<!--post-->
<form action="hello" method="post">
    <input value="post" type="submit" />
</form>
<!--delete-->
<form action="hello" method="post">
    <input name="_method" type="hidden" value="DELETE" />
    <input value="delete" type="submit" />
</form>
<!--put-->
<form action="hello" method="post">
    <input name="_method" type="hidden" value="PUT" />
    <input value="put" type="submit" />
</form>
```

```java
@RestController
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String get() {
        return "GET";
    }

    @RequestMapping(value = "/hello", method = RequestMethod.POST)
    public String post() {
        return "POST";
    }

    @RequestMapping(value = "/hello", method = RequestMethod.DELETE)
    public String del() {
        return "DELETE";
    }

    @RequestMapping(value = "/hello", method = RequestMethod.PUT)
    public String update() {
        return "PUT";
    }

}
```

注意在配置文件中开启隐藏表单，因为 SpringBoot 底层默认关闭 false

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
```

以上在表单中实现 DELETE/PUT 原理如下：

表单提交时会携带 _method，请求被 HiddenHttpMethodFilter 过滤器链拦截，获取到 _method 值，包装模式 requestWrapper 重写 getMethod 方法，返回 wrapper，之后控制器类便可根据包装后的结果调用相应方法进行处理。

@RequestMapping(value = "/hello", method = RequestMethod.GET) 可替换为 @GetMapping("hello")

@RequestMapping(value = "/hello", method = RequestMethod.POST) 可替换为 @PostMapping("hello")

@RequestMapping(value = "/hello", method = RequestMethod.DELETE) 可替换为 @DeleteMapping("hello")

@RequestMapping(value = "/hello", method = RequestMethod.PUT) 可替换为 @PutMapping("hello")

> 请求映射原理

在 SpringMVC 中，每个请求都会调用 DispatcherServlet 中的 doService 方法，然后 doService 方法又会调用 doDispatch 方法处理页面请求，在 doDispatch 方法中，调用 getHandler 方法获得当前页面请求需要使用哪个 Handler（Controller）方法进行处理。

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   
   ....
   
   try {
        try {
            ModelAndView mv = null;
            Object dispatchException = null;

            try {
                processedRequest = this.checkMultipart(request);
                multipartRequestParsed = processedRequest != request;
                mappedHandler = this.getHandler(processedRequest);
                if (mappedHandler == null) {
                    this.noHandlerFound(processedRequest, response);
                    return;
                }
         }

	....
}
```

在 handlerMappings 变量中，handlerMappings 维护了一个 List 集合，存储框架默认的映射，称为处理器执行链 HandlerExecutionChain

- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。
  - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping


![image.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211010171230.png)

**RequestMappingHandlerMapping** 类中保存了所有 @RequestMapping 和 handler 的映射规则，所有的请求映射都在HandlerMapping中。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211010171549.png)

## 6.5 普通参数与基本注解

在参数中使用到的注解有 @PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

```java
@RestController
public class ParameterTestController {


    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv,
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header,
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params,
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){


        Map<String,Object> map = new HashMap<>();

//        map.put("id",id);
//        map.put("name",name);
//        map.put("pv",pv);
//        map.put("userAgent",userAgent);
//        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }


    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }


    //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd
    //2、SpringBoot默认是禁用了矩阵变量的功能
    //      手动开启：原理。对于路径的处理。UrlPathHelper进行解析。
    //              removeSemicolonContent（移除分号内容）支持矩阵变量的
    //3、矩阵变量必须有url路径变量才能被解析
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10

    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;

    }

}
```

> @RequestAttribute

对于转发请求，RequestAttribute 可获得 request 中 setAttribute 中的内容

```java
@GetMapping("forward2")
public Map forward2(@RequestAttribute("msg") String msg,
                    HttpServletRequest request) {
    String str = (String) request.getAttribute("msg");
    System.out.println(str);
    Map<String, Object> map = new HashMap<>();
    map.put("str", str);
    map.put("msg", msg);
    System.out.println(msg);
    return map;
}
```

>@MatrixVariable

请求 url 路径可以为

1. /car/path?xxx=xxx&xxx=ccc&... 这种形式称为查询字符串
2. /car/sell;low=34;brand=abcd;... 这种形式称为矩阵变量

由于 SpringBoot 默认禁用了 url 分号后面的内容，因此需要手动开启矩阵变量。

```java
@Configuration()
public class UrlConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        //不移除 url 中分号后面的内容，从而使用矩阵变量
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

也可写成

```java
@Configuration(proxyBeanMethods = false)
public class UrlConfig {
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }
        };
    }
}
```

其中，`@Configuration`用于定义配置类，可替换`xml`配置文件，被注解的类内部包含有一个或多个被`@Bean`注解的方法，这些方法将会被`AnnotationConfigApplicationContext`或`AnnotationConfigWebApplicationContext`类进行扫描，并用于构建`bean`定义，初始化`Spring`容器。在 `@Configuration` 注解中，有 proxyBeanMethods 属性，表示每次调用 @Bean 从容器中获取还是重新创建对象，默认为 true，从容器中获取，可改为 false，每次创建新的对象。

```java
/**
 * 在 @MatrixVariable 注释中，默认在 UrlPathHelper 类中 removeSemicolonContent 变量为 true，关闭矩阵变量，因此需要手动开启
 * 矩阵变量是绑定于路径的
 * @return
 */
//cars/sell;low=34;brand=abcd, cdef
@RequestMapping("car/{path}")
public Map controller(@MatrixVariable("low") Integer low,
                      @MatrixVariable("brand") List<String> brand) {
    Map<String, Object> map = new HashMap<>();
    map.put("low", low);
    map.put("brand", brand);
    return map;
}
```

# 7、视图解析器 thymeleaf

在 pom 文件中加入 thymeleaf 依赖后，ThymeleafAutoConfiguration 会在 ThymeleafProperties 中自动配置好 SpringTemplateEngine 和 ThymeleafViewResolver。

在标签属性中写，标签中的内容将会发生改变

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 根据 id 获取数值                   |
| 链接       | @{...} | 生成链接 {..} 括号中的内容即为链接 |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |

在如下例子中，首先需要引入 thymeleaf 命名空间， xmlns:th="http://www.thymeleaf.org"，${msg} 和 \${link} 表示取出 msg 和 link 中的值，而 @{link} 直接将 link 作为访问路径，访问的是 http://localhost/项目名/link 

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>success</title>
</head>
<body>
hello thymeleaf <br/>
<a th:text="${msg}"></a>
<a href="www.cn.bing.com" th:href="${link}">百度</a>
<a href="www.cn.bing.com" th:href="@{link}">link</a>
</body>
</html>
```

```java
@Controller
public class ViewController {
    @RequestMapping("view")
    public String hello(Model model) {
        model.addAttribute("msg", "view");
        model.addAttribute("link", "http://www.baidu.com");
        return "success.html";
    }
}
```

# 8、项目中可能遇到的问题

> 刷新页面表单重复提交问题

使用重定向解决

```java
@Controller
public class IndexController {

    @PostMapping("login")
    public String main(User user, HttpSession session, Model model) {
        if (!user.getUsername().isEmpty() && !user.getPassword().isEmpty()) {
            session.setAttribute("username", user.getUsername());
            session.setAttribute("password", user.getPassword());
            //重定向防止重复提交
            return "redirect:main.html";
        } else {
            model.addAttribute("msg", "账号密码错误");
            //重定向
            return "redirect:main.html";
        }
    }

    @GetMapping("main.html")
    public String mainPage() {
        return "main";
    }
}
```

> thymeleaf 抽取公共页面代码

在定义公共代码时，有如下两种方式：

- 定义 id 名进行引入
- 使用 th:fragment 定义公共代码部分

```html
<span th:fragment="com" id="comId">
    公共部分
</span>
```

Thymeleaf提供了三种引入方式：

- th:insert
- th:replace
- th:include

```html
<div th:insert="~{success :: com}"></div>
<br/>

<div th:replace="~{success :: #comId}"></div>
<br/>

<div th:include="~{success :: #comId}"></div>
<br/>
```

网页上对应为

```html
<div><span id="comId">
    公共部分
</span></div>
<br/>

<span id="comId">
    公共部分
</span>
<br/>

<div>
    公共部分
</div>
<br/>
```

# 9、拦截器

之前在 SpringMVC 中学过，拦截器类需要实现 HandlerInterceptor 接口，然后根据需求重写 HandlerInterceptor 接口中的 preHandle、postHandle 和 afterCompletion 方法。在 SpringMVC 中，拦截器需要拦截的 url 路径，需要在 SpringMVC 配置文件中进行声明。

而在 SpringBoot 中，不需要使用 xml 配置文件声明，而是定义一个配置类实现 WebMvcConfigurer 接口中的 addInterceptors 方法，需要将拦截器添加到容器中，再配置拦截的 url 路径即可。

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

```java
//声明配置文件
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //将自定义拦截器添加到容器中
        //addPathPatterns 表示添加拦截路径，此处配置的是拦截此项目下的所有路径
        //excludePathPatterns 表示不拦截的路径
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/common")
                .excludePathPatterns("/success");
    }
}
```

# 10、文件上传功能

对于前端 <form\> 中 type 为 file，在 Controller 中，可采用  @RequestPart("img") MultipartFile file 获得，使用 MultipartFile 封装文件上传项。

在 SpringBoot 默认配置中，单次上传最大为 1MB，可在配置文件中修改单词上传文件大小和总共上传完文件大小

```yaml
spring:
  servlet:
    multipart:
      # 总共文件上传大小
      max-request-size: 100MB
      # 单个文件上传大小
      max-file-size: 1MB
```

index.html

```html
<form action="upload" method="post" enctype="multipart/form-data">
    <input type="file" name="img">
    <br/>
    <input type="file" name="photos" multiple>
    <br/>
    <input type="submit" value="提交">
</form>
```

```java
@Controller
public class FormController {

    @PostMapping("upload")
    public String upload(@RequestPart("img") MultipartFile file,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {
        System.out.println("===================");
        //单文件上传
        if (!file.isEmpty()) {
            String name = file.getOriginalFilename();
            file.transferTo(new File("E:\\JavaStudy\\" + name));
        }

        //多文件上传
        if (photos.length > 0) {
            for (MultipartFile photo : photos) {
                if (!photo.isEmpty()) {
                    String photoOriginalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("E:\\JavaStudy\\" + photoOriginalFilename));
                }
            }
        }
        return "success";
    }
}
```

在进行多文件上传时，即使没有输入任何文件，控制器类中得到的 photos 的长度为 1，因此需要加入判断 photo 是否为空。

# 11、错误处理机制

默认情况下， SpringBoot 使用 /error 处理所有的错误映射，若为浏览器，则返回错误页面，包含各种 json 信息；如果是非浏览器客户端，会返回一个 json 信息，返回的json数据有：

- status
- error
- exception
- message
- trace
- path

![image.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211012145148.png)

定制错误处理逻辑方法：

- 自定义错误页
- @ControllerAdvice + @ExceptionHandler 处理全局异常
- @ResponseStatus + 自定义异常
- Spring 底层的异常，如参数类型转换异常

## 自定义错误页

若想修改为自己的错误页面，则在 templates 中创建 error 包，创建 4xx.html 和 5xx.html 处理展示错误页面即可。

![image-20211012145910746](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211012145911.png)

在错误页面中，可通过 theymeleaf 获得不同的错误信息，例如 timestamp、status、error 等

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>4xx</title>
</head>
<body>
4xx 错误页面 <br/>
<a th:text="${message}"></a>
</body>
</html>
```

> SpringBoot 错误信息处理机制及原理

在 `ErrorMvcAutoConfiguration` 类中存放了所有关于错误信息的自动配置。

访问步骤：

- 首先客户端访问了错误界面。例：404 或者 500
- `SpringBoot`注册错误请求`/error`。通过`ErrorPageCustomizer`组件实现，`ErrorPageCustomizer`为 `ErrorMvcAutoConfiguration` 的一个静态内部类
- 通过`BasicErrorController` 类处理`/error`，对错误信息进行了自适应处理，在 BasicErrorController 中，分为 errorHtml 和 error 两个主要处理方法，分别是处理浏览器发送的请求和其它浏览器发送的请求的。浏览器会响应一个界面，其他端会响应一个`json`数据
- 如果响应一个界面，通过`DefaultErrorViewResolver`类来进行具体的解析。可以通过模板引擎解析也可以解析静态资源文件，如果两者都不存在则直接返回默认的错误`JSON` 或者错误`View`
- 通过`DefaultErrorAttributes`来添加具体的错误信息

源代码如下：

```java
//错误信息的自动配置
public class ErrorMvcAutoConfiguration {
    //包括响应具体的错误信息(errorAttributes)、处理错误请求(basicErrorController)、注册错误界面(errorPageCustomizer)
    ....
}
```

```java
//注册错误界面,错误界面的路径为/error
private static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {
	....
}
```

```java
//处理/error请求,从配置文件中取出请求的路径
@RequestMapping({"${server.error.path:${error.path:/error}}"})
public class BasicErrorController extends AbstractErrorController {
    //浏览器行为，通过请求头来判断，浏览器返回一个视图，ModelAndView
    @RequestMapping(produces = {"text/html"}) 
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response){
        ....
    }
    
    //其他客户端行为处理，返回一个JSON数据
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ....
    }
}
```

```java
public class DefaultErrorViewResolver implements ErrorViewResolver, Ordered {
    //使用 resolveErrorView 方法根据传入的状态码进行匹配
	....
}
```

```java
//添加错误信息
public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
   
	....
}
```

## @ControllerAdvice + @ExceptionHandler

自定义一个异常处理类

```java
/**
 * @ControllerAdvice 有 @Component 注解，因此会将类添加到容器中
 */
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * @ExceptionHandler 用来处理异常，参数为可处理的异常类型
     */
    @ExceptionHandler({ArithmeticException.class, NullPointerException.class})
    //页面跳转
    public String handleException() {
        return "exception";
    }
}
```

对于访问时，在 Controller 中出现的计算异常和空指针异常，都将由 handleException 方法来进行处理。

SpringBoot 使用 ExceptionHandlerExceptionResolver 来支持这种自定义异常处理机制，在 DispatcherServlet 初始化时，会将 ExceptionHandlerExceptionResolver 注册到 handlerExceptionResolvers 中，调用的关键方法是 ExceptionHandlerMethodResolver 类中的 ServletInvocableHandlerMethod 方法.

出现异常时，首先会从 ExceptionHandlerExceptionResolver  中的 exceptionHandlerCache 中去找参数 handlerMethod 所属 bean 的 class 对应的 ExceptionHandlerMethodResolver， 如果找不到则 new 一个 ExceptionHandlerMethodResolver 并缓存起来。 然后从ExceptionHandlerMethodResolver 去找该 exception 对应的异常处理方法。

```java
@Nullable
protected ServletInvocableHandlerMethod getExceptionHandlerMethod(@Nullable HandlerMethod handlerMethod, Exception exception) {
	....
}
```

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211012155117.png)

ExceptionHandlerMethodResolver 中存的是各个 Exception 到各个异常处理方法映射。

## @ResponseStatus + 自定义异常

在使用控制器 Controller 时，可手动 throw 抛出异常

```java
//可返回错误状态码，HttpStatus.FORBIDDEN 是 403 错误
@ResponseStatus(value = HttpStatus.FORBIDDEN, reason = "用户数量太多")
public class UserException extends RuntimeException {
    
}
```

```java
@Controller
public class CommonController {

    @RequestMapping("common")
    public String common() {
        throw new UserException();
        //return "common";
    }
}
```

底层仍然调用 ExceptionHandlerExceptionResolver 类的 ServletInvocableHandlerMethod 方法构造传入错误码和错误信息的 Model

# 12、Web原生组件（Servlet、Filter、Listener）注入

## 1、使用 Servlet API

> Servlet

在 Servlet 类上使用 @WebServlet(urlPatterns = "servlet") 指定该类为 Servlet，并指定 urlPatterns，然后需要在主类上添加注解 @ServletComponentScan(basePackages = "com.springboot.springboottest3") 并指定 Servlet 所在包，SpringBoot 会自动扫描 springboottest3 下所有包及其子包中的 Servlet。

```java
@WebServlet(urlPatterns = "/servlet")
public class ServletTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("abcdefg");
    }
}
```

```java
@ServletComponentScan(basePackages = "com.springboot.springboottest3")
@SpringBootApplication
public class SpringbootTest3Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTest3Application.class, args);
    }

}
```

但是 SpringMVC 中实现了 HandlerInterceptor  接口的拦截器并不会拦截 Servlet 请求

> Filter

在实现了 Filter 类上添加注解 @WebFilter(urlPatterns = "/servlet") 指定拦截的类，也可拦截静态资源。

```java
@WebFilter(urlPatterns = "/servlet")
public class FilterTest implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("WebFilter");
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

> Listener

使用 @WebListener 接口

```java
@WebListener
public class ListenerTest implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("contextInitialized");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("contextDestroyed");
    }
}
```

## 2、使用 Spring 提供的 RegistrationBean

使用 Spring 提供的 RegistBean 将 Servlet、Filter、Listener 添加到容器中，定义一个配置类，添加 Servlet、Filter、Listener。

```java
@Configuration
public class MyRegistConfig {
    @Bean
    public ServletRegistrationBean myServlet() {
        ServletTest servletTest = new ServletTest();
        //urlMappings 可为多个
        return new ServletRegistrationBean(servletTest, "/myservlet");
    }


    @Bean
    public FilterRegistrationBean myFilter() {
        FilterTest filterTest = new FilterTest();
        //下面这种方法拦截 myServlet 的所有路径
        //return new FilterRegistrationBean(filterTest, myServlet());
        FilterRegistrationBean bean = new FilterRegistrationBean(filterTest);
        //设置要拦截的路径
        bean.setUrlPatterns(Arrays.asList("/servlet", "/myservlet"));
        return bean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener() {
        ListenerTest listenerTest = new ListenerTest();
        return new ServletListenerRegistrationBean(listenerTest);
    }

}
```

# 13、定制化

当在配置类上加 @EnableWebMvc 后，表示全面接管 Web 的配置，这时原本 SpringBoot 所有配置将会失效，需要进行重新配置。

在 WebMvcAutoConfiguration 类中，配置了 SpringMVC 所需要的所有配置，而 @EnableWebMvc 中添加了类 DelegatingWebMvcConfiguration，这个类会将 WebMvcConfigurationSupport中的内容进行重新定制，WebMvcConfigurationSupport 中自动配置了一些非常底层的组件，例如 RequestMappingHandlerMapping，PathMatchConfigurer 等。而 WebMvcAutoConfiguration 配置了注解 @ConditionalOnMissingBean，因此当加上 @EnableWebMvc 后，WebMvcAutoConfiguration 会失效，所有内容需要自行配置。

在应用中，应该定义一个配置类实现 WebMvcConfigurer 接口，再结合 @Bean 为容器添加一些组件，由此来实现定制化和功能拓展，而不需要使用 @EnableWebMvc 将 SpringBoot 配置的所有内容均失效。

# 14、数据访问

## 使用步骤

1、导入场景驱动

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

在导入 spring-boot-starter-data-jdbc 后，并未导入 jdbc 数据库驱动，未导入驱动的原因是不知道用户需要使用什么驱动，可自行导入，并且 SpringBoot 已经自动进行了版本仲裁，默认版本为 `<mysql.version>8.0.26</mysql.version>`，但是在使用时注意驱动版本应该和安装的 MySQL 版本相同。

若想要修改驱动的版本，有如下两种方法：

- 在依赖文件中指定版本

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.6.17</version>
</dependency>
```

- 在 properties 中配置

```xml
<properties>
    <mysql.version>5.6.17</mysql.version>
</properties>
```

DataSourceAutoConfiguration  是数据库的自动配置类，默认使用的连接池是 HikariDataSource。

- DataSourceTransactionManagerAutoConfiguration： 事务管理器的自动配置
- JdbcTemplateAutoConfiguration： **JdbcTemplate的自动配置，可以来对数据库进行crud**

- - 可以修改这个配置项@ConfigurationProperties(prefix = **"spring.jdbc"**) 来修改JdbcTemplate
  - @Bean@Primary    JdbcTemplate；容器中有这个组件

- JndiDataSourceAutoConfiguration： jndi的自动配置
- XADataSourceAutoConfiguration： 分布式事务相关的 

2、修改数据库相关配置

可 spring.datasource 中修改数据库的相关属性

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: "000000"
    url: jdbc:mysql://localhost:3306/school
```

若未配置数据库的相关内容，则启动时将会失败，注意密码应该使用 "000000"

可在 spring.jdbc.template 中配置操作数据库的相关内容

```yaml
spring: 
    jdbc:
      template:
        query-timeout: 
```

3、操作数据库

对数据的操作封装在 JdbcTemplate 类中，由于 SpringBoot 添加了 @Bean 将 JdbcTemplate 添加到容器中，因此只需要 @Autowired 自动注入即可。

```java
@Autowired
JdbcTemplate jdbcTemplate;

@Test
void contextLoads() {
    //返回值为 Integer
    Integer query = jdbcTemplate.queryForObject("select count(*) from student", Integer.class);
    System.out.println(query);
}
```

## 配置 Druid 数据源

1、添加依赖，由于使用的是第三方数据源，因此需要指定版本。

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
</dependency>
```

2、为容器中添加数据源，当自定义 DataSource 后，SpringBoot 中的 HikariDataSource 数据源将会失效，转而使用自定义的数据源，@ConfigurationProperties 的作用是将第三方数据源和配置文件中的内容绑定

```java
@Configuration
public class MyDataConfig {
    @ConfigurationProperties("spring.datasource")
    @Bean
    public DataSource dataSource() {
        return new DruidDataSource();
    }
}
```

> Druid 的监控统计功能

首先在配置数据源的方法中添加 dataSource.setFilters("stat"); 然后需要定义 ServletRegistrationBean 新增页面监控功能，页面监控使用的是 Druid 的 StatViewServlet 类。

```java
public DataSource dataSource() throws SQLException {
    DruidDataSource dataSource = new DruidDataSource;
    dataSource.setFilters("stat");
    return dataSource;
}
```

```java
//配置 Druid 的监控页功能
@Bean
public ServletRegistrationBean statViewServlet() {
    StatViewServlet statViewFilter = new StatViewServlet();
    ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<StatViewServlet>(statViewFilter, "druid/**");
    return registrationBean;
}
```

## Druid 官方 starter 配置方式

pom 文件

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.17</version>
</dependency>
```

- DruidSpringAopConfiguration.**class**,   监控SpringBean的；配置项：**spring.datasource.druid.aop-patterns**
- DruidStatViewServletConfiguration.**class**, 监控页的配置：**spring.datasource.druid.stat-view-servlet；默认开启**

-  DruidWebStatFilterConfiguration.**class**, web监控配置；**spring.datasource.druid.web-stat-filter；默认开启**
- DruidFilterConfiguration.**class**}) 所有Druid自己filter的配置

配置示例：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

    druid:
      aop-patterns: com.admin.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false   #重置是否开启

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'


      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```

SpringBoot配置示例

https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

## MyBatis

### 入门案例

加入 MyBatis 依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

指定全局配置文件位置和 Mapper 文件位置

```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml  #sql映射文件位置
```

操作数据库的接口必须加上 @Mapper 接口。

```java
@Mapper
public interface CityMapper {
    
    public City getById(Long id);

    public void insert(City city);

}
```

对于全局配置文件有两种方式，第一种是上面的在 yaml 文件中指定全局配置文件的位置，第二种是如下直接在 yaml 文件 mybatis.configuration 中进行配置，这对应了如下 MyBatis 全局文件的配置。mapUnderscoreToCamelCase 的意思是开启驼峰命名规则，如果在数据库中列名为 user_name，而 bean 的属性为 userName则需要开启 mapUnderscoreToCamelCase 。

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

```yaml
# 配置mybatis规则
mybatis:
#  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

 注意，全局配置文件和 mybatis 在 yaml 中 mybatis.configuration 配置不能同时存在，二者只能选其一。

### MyBatis 注解模式

将 SQL 语句使用注解写在接口方法上。

```java
@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);

    public void insert(City city);

}
```

对于较为复杂的 SQL 语句，也可使用 Mapper 配置文件

```xml
<mapper namespace="com.springboot.springboottest3.CityMapper">
    <select id="insert" resultType="com.springboot.springboottest3.City">
        insert into city values(#{name}, #{state}, #{country})
    </select>
</mapper>
```

其中，MyBatis 中的 useGeneratedKeys="true" keyProperty="对应的主键的对象"，表示数据库表自增，插入时使用数据库自增的值。

### MyBatis-Plus

#### 入门案例

1、添加依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3.4</version>
</dependency>
```

引入 MyBatis-Plus 后，MyBatis 数据库驱动 mybatis-spring-boot-starter 和 JDBC 的依赖 spring-boot-starter-data-jdbc都可以不用加。

![image-20211020101802944](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020101803.png)

- MybatisPlusAutoConfiguration 配置类，MybatisPlusProperties 配置项绑定。在配置文件中，mybatis-plus 的配置路径为 **mybatis-plus：xxx 就是对** **mybatis-plus的定制**
- **SqlSessionFactory 自动配置好。如果没有配置数据源，则底层是容器中默认的数据源**
- **mapperLocations 自动配置好的。有默认值**。\*classpath\*:/mapper/\**/\*.xml；任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件，放在 mapper下

- **容器中也自动配置好了** **SqlSessionTemplate**
- **@Mapper 标注的接口也会被自动扫描；建议直接** @MapperScan(**"com.admin.mapper"**) 批量扫描就行，写在主类上面

2、在 application.yaml 中配置数据库

```yaml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    schema: classpath:db/schema-h2.sql
    data: classpath:db/data-h2.sql
    url: jdbc:h2:mem:test
    username: root
    password: test
```

在启动类上添加 @MapperScan 注解

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

3、编写实体类

```java
@Data
public class User {
    @TableField(exist="false") //表示当前属性在表中不存在，对数据库操作时将会忽略此属性
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

4、编写 Mapper 

```java
public interface UserMapper extends BaseMapper<User> {

}
```

在 BaseMapper 中已经定义了一些操作数据库的方法，因此在继承 BaseMapper 后不需要在写操作数据库的方法，也不需要写 Mapper 配置文件，除非是 BaseMapper 中不存在的 SQL 语句，对于这些自行定义即可。

相当于是将 MyBatis 中的 xml 配置文件转换成了 java 类的形式，更便于操作。

![image-20211020103247530](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020103247.png)

在测试操作数据库时，对于待操作的表，MyBatis-Plus 访问的默认是实体类 User 对应的 user 表，若表名发生了变化，则需要在实体类上添加注解 @TableName 指定表名。

```java
@Data
@TableName("user_test")
public class User {
    ...
}
```

#### CRUD操作

CRUD 指的是 create 添加数据 read 读取数据 update 修改数据 delete 删除数据

在实际开发中，使用 Service 操作数据库，具体如下

文件结构：

![image-20211020110108770](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020110109.png)

```java
@Data
//@TableName("user_table") 当表明不为 user 指定表名
public class User {
    private Integer userId;
    private String userName;
    private Integer userAge;
}
```

```java
public interface UserMapper extends BaseMapper<User> {
    
}
```

```java
public interface UserService extends IService<User> {
}
```

```java
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

在 IService 中定义了大量操作数据库的方法，在 ServiceImpl 中进行了实现，我们要做的只是继承和实现这两个类就可以拥有大量操作数据库的方法

![image-20211020110351082](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020110351.png)

若想要对查询的数据进行分列，则可使用 Page 类，默认传参可为当前页和最大页

![image-20211020111332305](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020111332.png)

#### 分页

PaginationInnerInterceptor

## NoSQL

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- RedisAutoConfiguration 自动配置类。RedisProperties 属性类，在 yaml 中修改**spring.redis.xxx是对redis的配置**
- 连接工厂是准备好的。**Lettuce**ConnectionConfiguration、**Jedis**ConnectionConfiguration

- **自动注入了RedisTemplate**<**Object**, **Object**> （k 和 v 的类型均为 Object）： xxxTemplate；
- **自动注入了StringRedisTemplate；k：v都是String**

- **key：value**
- **底层只要我们使用** **StringRedisTemplate、RedisTemplate就可以操作redis**

以上默认使用的为 netty，若想使用 jedis，则导入依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

然后将 yaml 的 spring.redis.client-type 改为 jedis。

> Filter 和 Interceptor

1、Filter 是 Servlet 的原生组件，脱离了 Spring 依旧可以使用

2、Inteceptor 是 Spring 定义的接口，可使用 Spring 的自动装配等功能

# 15、单元测试 JUnit 5

## 概述

**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库**

作为最新版本的JUnit框架，JUnit5与之前版本的Junit框架有很大的不同。由三个不同子项目的几个不同模块组成。

**JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

**JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

**JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部 包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x,Junit3.x的测试引擎。

**SpringBoot 2.4 以上版本移除了默认对** **Vintage 的依赖。如果需要兼容junit4需要自行引入（不能使用junit4的功能 @Test****）**

**JUnit 5’s Vintage Engine Removed from** `spring-boot-starter-test`，如果需要继续兼容junit4需要自行引入vintage

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 注解

- **@Test :**表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest :**表示方法是有参测试，必须至少声明一个源 @ValueSource，为每个调用提供参数

- **@RepeatedTest :**表示方法可重复执行，@RepeatedTest(5) 表示重复执行五次 
- **@DisplayName :**为测试类或者测试方法设置展示名称

- **@BeforeEach :**表示在每个单元测试之前执行
- **@AfterEach :**表示在每个单元测试之后执行

- **@BeforeAll :**表示在所有单元测试之前执行，由于只运行一次，因此为 static
- **@AfterAll :**表示在所有单元测试之后执行，使用 static 修饰

- **@Tag :**表示单元测试类别，类似于JUnit4中的@Categories
- **@Disabled :**表示测试类或测试方法不执行，类似于JUnit4中的@Ignore

- **@Timeout :**表示测试方法运行如果超过了指定时间将会返回超时异常
- **@ExtendWith :**为测试类或测试方法提供扩展类引用

对于 @ParameterizedTest 和 @ValueSource 方法使用有参测试时，例子如下

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
public void test2(int a) {
    System.out.println("a = " + a);
}
```

该方法 a 分别取三次不同的值，执行了 3 次。

@ValueSource 中不同类型取名不同，必须和源码中定义的相符，详细源码如下：

```java
public @interface ValueSource {
    short[] shorts() default {};

    byte[] bytes() default {};

    int[] ints() default {};

    long[] longs() default {};

    float[] floats() default {};

    double[] doubles() default {};

    char[] chars() default {};

    boolean[] booleans() default {};

    String[] strings() default {};

    Class<?>[] classes() default {};
}
```

## 断言

> 简单断言

| 方法            | 说明                                 |
| --------------- | ------------------------------------ |
| assertEquals    | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象   |
| assertNotSame   | 判断两个对象引用是否指向不同的对象   |
| assertTrue      | 判断给定的布尔值是否为 true          |
| assertFalse     | 判断给定的布尔值是否为 false         |
| assertNull      | 判断给定的对象引用是否为 null        |
| assertNotNull   | 判断给定的对象引用是否不为 null      |

```java
@Test
public void testAssertions() {
    int sum = doSome(1, 2);
    //第一个为期望值，第二个为实际值，第三个为错误信息
    Assertions.assertEquals(5, sum, "计算错误");
}
```

![image-20211020150035873](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020150036.png)

在测试中，若前面代码断言失败，则后续代码将不会执行。

> 数组断言

`Assertions.assertArrayEquals(new int[]{1, 2}, new int[]{1, 2})`

> 组合断言

`Assertions.assertAll()` 结合 Lamda 表达式

```java
@Test
@DisplayName("assert all")
public void all() {
 assertAll("Math",
    () -> assertEquals(2, 1 + 1),
    () -> assertTrue(1 > 0)
 );
}
```

> 异常断言

**Assertions.assertThrows()**

```java
@Test
@DisplayName("异常测试")
public void exceptionTest() {
    ArithmeticException exception = Assertions.assertThrows(
           //扔出断言异常
            ArithmeticException.class, () -> System.out.println(1 % 0));

}
```

> 超时断言

**Assertions.assertTimeout()**

```java
@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
```

> 快速失败

**Assertions.fail()**

```java
@Test
@DisplayName("fail")
public void shouldFail() {
 	fail("This should fail");
}
```

## 前置条件

前置条件 Assumptions 和断言 Assertions 类似，当失败时本方法下面的内容将不会执行，不同之处在于前置条件失败后，在最终的测试报告中报的为 ignore 忽略的错误，而断言失败后，报的是错误。

![image-20211020152427021](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211020152427.png)

## 嵌套测试

```java
public class JUnit5Test {
    @Test
    public void test3() {
        Assumptions.assumeTrue(false);
    }

    @Nested
    class Nest {
        @BeforeAll
        public void before() {
            System.out.println(1);
        }
    }
}
```

在嵌套测试中，对于 @BeforeAll，外侧不能驱动（启动）内层的 @BeforeAll/@AfterAll，但是内层可以驱动内层的 @BeforeAll/@AfterAll

# 16、SpringBoot Actuator

Spring Boot Actuator 模块提供了生产级别的功能，比如健康检查，审计，指标收集，HTTP 跟踪等，帮助我们监控和管理Spring Boot 应用。

## Endpoints

未来每一个微服务在云上部署以后，我们都需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

SpringBoot Actuator 1 和 SpringBoot Actuator 2 的不同：

![image.png](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture2/20211021091705.png)

添加 actuator 场景依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

添加依赖后，启动项目后可访问 http://localhost:8080/actuator 得到监控信息。使用 web 监控时，端点并非所有都是可见的，需要通过 management 中的属性进行设置。

```yaml
management:
  endpoints:  # 配置所有断点的信息
    enabled-by-default: true # 暴露所有端点
    web:
      exposure:
        include: '*' # 以 web 方式暴露
```

> 常用端点

| ID                 | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。                    |
| `caches`           | 暴露可用的缓存。                                             |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。         |
| `configprops`      | 显示所有`@ConfigurationProperties`。                         |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`                    |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。 需要一个或多个`Flyway`组件。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                           |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                             |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                               |
| `mappings`         | 显示所有`@RequestMapping`路径列表。                          |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                               |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                               |

最常用的 Endpoint

- **Health：监控状况**
- **Metrics：运行时指标**

- **Loggers：日志记录**

可设置开启端点的详细信息

```yaml
management:
  endpoint:
    health:
      show-details: always # 总是显示 health 端点的详细信息
```

在以上设置中，由于设置了 management.endpoints.exposure.include = '*'，可在网页显示所有端点信息，但实际中可能需要隐藏一些敏感内容，因此可手动指定开启

```yaml
management:
  endpoints:
    enabled-by-default: false # 是否暴露所有端点
    web:
      exposure:
        include: '*' # 以 web 方式暴露
  endpoint:
    health:
      show-details: always # 总是显示 health 端点的详细信息
      enabled: true
    metrics:
      enabled: true
    loggers:
      enabled: true
```

## 定制 Endpoint

### 定制 Health 信息

```java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }
        
        builder.withDetail("code",100)
                .withDetails(map);

    }
}
```

### 定制 Info 信息

有两种方法：

1、编写配置文件

```yaml
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
```

2、编写类实现 InfoContributor 接口

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
```

### 定制 Metris 信息

```java
@Service
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }

    public void hello() {
        counter.increment();
    }
}
```

也可使用

```java
//也可以使用下面的方式
@Bean
MeterBinder queueSize(Queue queue) {
    return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
}
```

### 定制 Endpoint

```java
@Component
@Endpoint(id = "container")
public class DockerEndpoint {


    @ReadOperation
    public Map getDockerInfo(){
        return Collections.singletonMap("info","docker started...");
    }

    @WriteOperation
    private void restartDocker(){
        System.out.println("docker restarted....");
    }
}
```

## spring boot admin 实现可视化

https://github.com/codecentric/spring-boot-admin

# 17、原理解析

## Profile功能

创建 application-prod.yaml 生产和 application-test.yaml 测试的配置文件，默认主配置文件 application.yaml 总会加载，通过在 application.yaml 中指定激活 **spring.profiles.active=prod/test** 来指定激活环境配置文件，并且指定环境配置文件会覆盖默认配置文件的内容。

> Profile 条件装配功能

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

> Profile 分组

```properties
spring.profiles.active=production  # 激活
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

# 18、SpringBoot 原理

Spring 原理、SpringMVC 原理、自动配置原理、SpringBoot 原理

## 1、SpringBoot 启动过程

参考 https://www.cnblogs.com/Narule/p/14253754.html

主要流程如下

0.启动main方法开始

1.初始化配置：通过类加载器，（loadFactories）读取classpath下所有的spring.factories配置文件，创建一些初始配置对象；通知监听者应用程序启动开始，创建环境对象environment，用于读取环境配置 如 application.yml

2.创建应用程序上下文-createApplicationContext，创建 bean工厂对象

3.刷新上下文（启动核心）

3.1 配置工厂对象，包括上下文类加载器，对象发布处理器，beanFactoryPostProcessor

3.2 注册并实例化bean工厂发布处理器，并且调用这些处理器，对包扫描解析(主要是class文件)

3.3 注册并实例化bean发布处理器 beanPostProcessor

3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat服务器）

3.5 实例化所有bean工厂缓存的bean对象（剩下的）

3.6 发布通知-通知上下文刷新完成（启动tomcat服务器）

4.通知监听者-启动程序完成

启动中，大部分对象都是BeanFactory对象通过反射创建

详细的源码解析可参考原文。

## 2、并发处理多个请求

web应用在部署时通常会部署多个实例，但是这多个实例能并发处理成百上千的请求，那么每个实例肯定都能并发处理多个请求。我们知道SpringBoot支持并发处理多个请求，如果采用默认的tomcat servlet容器，springboot可以并发处理200个请求。

通过将server.tomcat.max-threads添加到application.properties或application.yml，可以限制并发请求的数量。server.tomcat.max-threads 默认为 200，可在配置文件中修改

