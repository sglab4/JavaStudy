# 1、概述

Spring 是一个轻量级的开源的 J2EE 框架，目的是为了简化开发，降低企业开发复杂度。轻量级指的是引入的 jar 包比较少、小。

Spring 的核心两部分：IOC 和 AOP

1. IOC：控制反转，把创建对象交给 Spring 进行管理，不必再采用 new
2. AOP：面向切面编程，不修改源代码的情况下增强功能

# 2、体系结构

![Spring 体系结构](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210911163733.png)

# 3、Spring 开发步骤

![image-20210911164245771](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210911164246.png)

![image-20210911164419335](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210911164419.png)

# 4、Spring 入门案例

1、首先在 dom.xml 中导入依赖坐标

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.9</version>
</dependency>
```

2、编写 Dao 和相应的实现类

```java
package com.dao;

public interface UserDao {
    public void save();
}
```

```java
package com.dao.impl;

import com.dao.UserDao;

public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("save...");
    }
}
```

3、编写 Spring 配置文件，并配置 `<bean>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.dao.impl.UserDaoImpl"></bean>
</beans>
```

4、测试

```java
package com;

import com.dao.UserDao;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringTest {

    @Test
    public void test() {
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao) app.getBean("userDao");
        userDao.save();
    }
}
```

# 5、Spring 配置文件

## 1、基本配置

id 和 class，id 来表示 Dao 对象，class 表示对象路径

## 2、scope 范围配置

`<bean>` 中的标签可以配置 scope 范围，如下

![image-20210911171744207](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210911171756.png)

```xml
<!-- 单例 -->
<bean id="userDao" class="com.dao.impl.UserDaoImpl" scope="singleton"></bean>
```

```xml
<!-- 多例 -->
<bean id="userDao" class="com.dao.impl.UserDaoImpl" scope="prototype"></bean>
```

当 scope 配置为 singleton 时，bean 的实例化个数有一个

- 对象创建：加载配置文件时，创建对象实例
- 对象运行：容器在，对象便存在
- 对象销毁：应用卸载，销毁容器时，对象便被销毁

当 scope 配置为 prototype，bean 的实例化个数有多个

- 对象创建：在获得 bean 时 getBean，创建新的对象实例。
- 对象运行：使用中时，对象存在
- 对象销毁：长时间不使用，被 java 回收机制回收

```java
UserDao userDao = (UserDao) app.getBean("userDao");
```

## 3、生命周期配置

```xml
<bean id="userDao" class="com.dao.impl.UserDaoImpl" init-method="init" destroy-method="destory"></bean>
```

init-method 表示初始化方法，在加载配置文件后，先调用无参构造方法，在调用初始化方法 ，destroy-method 表示销毁时调用的方法。

## 4、Bean 实例化的三种方式

- 无参构造方法实例化

  以上配置为无参构造方法配置

- 工厂静态方法实例化

  工厂静态方法类：

  ```java
  package com.factory;
  
  import com.dao.impl.UserDaoImpl;
  
  public class StaticFactory {
      public static UserDao getUserDao() {
          System.out.println("factory");
          return new UserDaoImpl();
      }
  }
  ```

  在 applicationContext.xml 配置为

  ```xml
  <bean id="userDao" class="com.factory.StaticFactory" factory-method="getUserDao"></bean>
  ```

- 工厂动态方法实例化

  工厂动态方法类：

  ```java
  package com.factory;
  
  import com.dao.UserDao;
  import com.dao.impl.UserDaoImpl;
  
  public class DynamicFactory {
      public UserDao getUserDao() {
          return new UserDaoImpl();
      }
  }
  ```

  在 applicationContext.xml 配置为

  ```xml
  <bean id="factory" class="com.factory.DynamicFactory"></bean>
  <bean id="userDao" factory-bean="factory" factory-method="getUserDao"></bean>
  ```

  



















