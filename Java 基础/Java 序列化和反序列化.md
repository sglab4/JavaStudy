# 什么是序列化和反序列化

序列化是将 Java 对象转换成与平台无关的二进制流，而反序列化则是将二进制流恢复成原来的 Java 对象，二进制流便于保存到磁盘上或者在网络上传输。

如果实现 Serializable 接口，由于该接口只是个 “标记接口”，接口中不含任何方法，序列化是使用 ObjectOutputStream（处理流）中的 writeObject(obj) 方法将 Java 对象输出到输出流中，反序列化是使用 ObjectInputStream 中的 readObject(in) 方法将输入流中的 Java 对象还原出来。

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209222037889.png)

序列化步骤：

1. 创建一个对象输出流，它可以包装一个其它类型的目标输出流，如文件输出流：

   ```java
   ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(“目标地址路径”));

2. 通过对象输出流的 writeObject() 方法写对象：

   ```java
   out.writeObject(new Date());

反序列化步骤：

1. 创建一个对象输入流，它可以包装一个其它类型输入流，如文件输入流：

   ```java
   ObjectInputStream in = new ObjectInputStream(new fileInputStream(“目标地址路径”));
   ```

2. 通过对象输出流的readObject()方法读取对象：

   ```java
   String obj1 = (String)in.readObject();
   Date obj2 =  (Date)in.readObject();
   ```

## 序列化和反序列化的版本问题

在 Java 的序列化机制中，允许给类提供一个 private static final 修饰的 SerialVersionUID 类常量，来作为类版本的代号。这样即使类被修改了（如修改了方法），也会把修改前的类和修改后的类当成同一版本的类，序列化和反序列化照样可以正常使用。如果我们不显式的定义这个 SerialVersionUID，Java 虚拟机会根据类的信息帮我们自动生成，修改前和修改后的计算结果往往不同，造成版本不兼容而发生反序列化失败，另外由于平台的差异性，在程序移植中也可能出现无法反序列化。强大的 IDE 工具，也都有自动生成 SeriaVersionUID 的方法，这里就不多说了。JDK 中自带的也有生成 SeriaVersionUID 值的工具 serialver.exe，使用 `serialver 类名（编译后）` 命令就能生成该类的 SeriaVersionUID 值啦！