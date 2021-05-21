[TOC]

# Java诞生

## c语言

诞生于1972年

贴近硬件、运行速度快，效率高

## c++

诞生于1982年

面向对象，应用于图形、游戏领域

## Java

诞生于1995年，继承了c++优点，摒弃c++中多继承、指针等概念

Java 2 标准版（J2SE）：桌面程序

Java 2 企业版（J2EE）：服务器、web网页端

*Java 2 移动版（J2ME）：手机、小家电*

构建工具：Ant，Maven，Jekins

应用服务器：Tomcat，Jetty，Jboss，Webshpere，weblogic

Web开发：Struts，Spring，Hibernate，myBatis

开发工具：Eclipse，NetBeans，intellij idea，Jbuilder

# Java优势

面向对象

可移植性 write once,run anywhere

分布式

动态性：反射机制

多线程

安全性：异常机制

# JDK、JRE、JVM

作者“**冰湖一角**”的博客[JDK、JRE、JVM三者间的联系和区别](https://www.cnblogs.com/bingyimeiling/p/10266949.html)

正是由于JVM虚拟机，java才具有跨平台的特性

# HelloWorld

```java
public class Hello{
    public static void main(String[] args){
        System.out.println("Hello, World");
    }
}
```

在cmd命令窗口运行时，

- 在java文件的目录下打开cmd
- 使用javac命令，先对java文件编译，生成class文件，例：javac hello.java
- 再使用java命令运行字节码文件，不需要加后缀，例：java hello

注意：文件名和公共类名必须保持一致

# IDEA安装（2020.3.2）

1. 登录[idea官网](https://www.jetbrains.com/)下载，旗舰版收费，社区版免费

   ![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/o_210206095542IDEA%E4%B8%8B%E8%BD%BD.JPG)

2. 打开下载的文件

   ![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/t_210206100157%E5%9B%BE%E6%A0%87.JPG?a=1612606517806)



一步一步安装即可

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/o_210206101352%E9%80%89%E6%8B%A9%E7%9B%AE%E5%BD%95.JPG)

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/o_210206101321%E5%AE%89%E8%A3%85%E8%BF%87%E7%A8%8B.JPG)

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/o_210206101310%E5%AE%89%E8%A3%85.JPG)

![](https://images.cnblogs.com/cnblogs_com/sgKurisu/1928850/o_210206101335%E8%BF%87%E7%A8%8B%E4%B8%AD.JPG)

IDEA的缩写输入

psvm

```java
 public static void main(String[] args) {
     ...
 }
```

sout

```java
System.out.println("");
```

psf 静态变量方法

```java
public static final 
```

ctrl + d：复制当前行到下一行

