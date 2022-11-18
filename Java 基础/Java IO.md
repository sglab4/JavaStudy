# 1、概览

Java IO 大致分为以下几类：

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

# 2、磁盘操作

File 类可以用于表示文件和目录的信息，但是它**不表示文件的内容**。

递归地列出一个目录下所有文件：

```java
import java.io.File;

public class Test {

    public static void main(String[] args) {
        new Test().listAllFiles(new File("E:\\JavaStudy"));
    }

    private void listAllFiles(File dir) {
        //递归结束条件
        if (dir == null || !dir.exists()) {
            return;
        }
        if (dir.isFile()) {
            System.out.println(dir.getName());
            return;
        }
        for (File file : dir.listFiles()) {
            listAllFiles(file);
        }
    }
}
```

从 Java7 开始，可以使用 Paths 和 Files 代替 File。

# 3、字节操作

## 文件复制

```java
import java.io.*;

public class Test {

    public static void main(String[] args) throws IOException {
        new Test().copy("E:\\JavaStudy\\算法\\我的第一本算法书.md", "E:\\JavaStudy\\算法\\其他\\copy.md");
    }

    private void copy(String in, String out) throws IOException {
        FileInputStream fileInputStream = new FileInputStream(in);
        FileOutputStream fileOutputStream = new FileOutputStream(out);

        byte[] buffer = new byte[5 * 1024];
        int temp = 0;
        while ((temp = fileInputStream.read(buffer, 0, buffer.length)) != -1) {
            fileOutputStream.write(buffer);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }

}
```

## 装饰者模式

Java I/O 使用了装饰者模式来实现。以 InputStream 为例，

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210703200347.png)

- InputStream 是抽象组件；FileInputStream、PipedInputStream 等均继承自 InputStream，并对 InputStream 中的抽象方法进行了实现
- FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
- FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

实例化一个具有缓存功能的字节流对象 BufferedInputStream 时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```java
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

# 4、字符操作

## 编码与解码

编码就是将字符转换为字节，解码就是将字节转换为字符。

若在编解码过程中使用不同的编码方式，就会出现乱码。

- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。

UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。

Java 的内存编码使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。（char 类型占 2 个字节）

## String 的编码方式

String 可以看成一个字符序列，可以指定一个编码方式将它编码为字节序列，同样，也可以指定一个编码方式将一个字节序列解码为 String。

```java
String str1 = "中文";
byte[] bytes = str1.getBytes("UTF-8");
String str2 = new String(bytes, "UTF-8");
System.out.println(str2);
```

在调用无参数 getBytes() 方法时，默认的编码方式不是 UTF-16be。双字节编码的好处是可以使用一个 char 存储中文和英文，而将 String 转为 bytes[] 字节数组就不再需要这个好处，因此也就不再需要双字节编码。getBytes() 的默认编码方式与平台有关，一般为 UTF-8。

## Reader 和 Writer

无论传输什么文件，最小的存储单位是字节而非字符。但程序中操作通常都是字符的形式，因此需要提供字符的操作方法。

- InputStreamReader：将字节流解码为字符流（字节流 - 字符流）
- OutputStreamWriter：将字符流编码为字节流（字符流 - 字节流）

## 逐行输出文本文件的内容

字节流和字符流的对应中，FileInputStream 对应 FileReader，FileOutputStream 对应 FileWriter，BufferedInputStream 对应 BufferedReader， BufferedOutputStream 对应 BufferedWriter。不同类的构造方法也依次对应。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        new Test().readFileContent("E:\\JavaStudy\\算法\\我的第一本算法书.md");
    }

    private void readFileContent(String filePath) throws IOException {
        FileReader reader = new FileReader(filePath);
        BufferedReader bufferedReader = new BufferedReader(reader);

        String line;
        while ((line = bufferedReader.readLine()) != null) {
            System.out.println(line);
        }
        //由于使用装饰着模式使得 bufferedReader 组合了一个 Reader 对象，因此在调用 bufferedReader.close() 时也会调用 Reader 类的 close() 方法，因此只需要写一个 bufferedReader.close() 即可
        bufferedReader.close();
        //reader.close();
    }
}
```

# 5、对象操作

## 序列化

序列化就是将一个对象转换为字节序列，反序列化正好相反。

- 序列化：ObjectOutputStream.writeObject()
- 反序列化：ObjectInputStream.readObject()

不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。

## Serializable

序列化 ObjectOutputStream 类和反序列化 ObjectInputStream 类的构造方法分别为 OutputStream 和 InputStream，因此可将 ObjectOutputStream 和 ObjectInputStream  的构造方法写为 FileOutputStream(fileDir) 和 FileInputStream(fileDir)。

```java
import java.io.*;

public class Test {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Student student = new Student("K", 18);
        String file = "E:\\JavaStudy\\test.txt";

        //序列化
        ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream(file));
        output.writeObject(student);
        output.close();

        //反序列化
        ObjectInputStream input = new ObjectInputStream(new FileInputStream(file));
        Student student1 = (Student) input.readObject();
        input.close();
        System.out.println(student1);
    }
}

class Student implements Serializable {
    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

## transient

transient 关键字可以使一些属性不会被序列化。

在 ArrayList 类中，elementData 用 transient 修饰，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    ....
}
```

```java
transient Object[] elementData; // non-private to simplify nested class access
```

# 6、网络操作

Java 中的网络支持：

- InetAddress：用于表示网络上的硬件资源，即 IP 地址；
- URL：统一资源定位符；
- Sockets：使用 TCP 协议实现网络通信；
- Datagram：使用 UDP 协议实现网络通信。

## InetAddress

InetAddress 类提供了操作 IP 地址的各种方法。该类本身没有构造方法，而是通过调用相关静态方法获取实例。InetAddress 类中的常用方法如下表所示。

|                   方法名称                    | 说明                                                         |
| :-------------------------------------------: | ------------------------------------------------------------ |
|          boolean equals(Object obj)           | 将此对象与指定对象比较                                       |
|              byte[] getAddress()              | 返回此 InetAddress 对象的原始 IP 地址                        |
| static InetAddress[] getAHByName(String host) | 在给定主机名的情况下，根据系统上配置的名称，服务器返 回其 IP 地址所组成的数组 |
| static InetAddress getByAddress(byte[] addr)  | 在给定原始 IP 地址的情况下，返回 InetAddress 对象            |
| static InetAddress getByAddress(String host)  | 在给定主机名的情况下确定主机的 IP 地址                       |
|         String getCanonicalHostName()         | 获取此 IP 地址的完全限定域名                                 |
|            String getHostAddress()            | 返回 IP 地址字符串（以文本表现形式）                         |
|             String getHostName()              | 返回此 IP 地址的主机名                                       |
|       static InetAdderss getLocalHost()       | 返回本地主机                                                 |
|        static InetAddress getByName()         | 给定域名或 IP 地址的情况下返回 InetAddress                   |

```java
public class Test {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        InetAddress ia1 = InetAddress.getByName("www.baidu.com");
        InetAddress ia2 = InetAddress.getByName("127.0.0.1");
        InetAddress ia3 = InetAddress.getLocalHost();

        System.out.println(ia1.getHostAddress());
        System.out.println(ia2.getHostAddress());
        System.out.println(ia3.getHostName());
    }
}
```

```html
36.152.44.96
127.0.0.1
ASUS
```

## URL

可以直接从 URL 中读取字节流数据。

```java
public class Test {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        URL url = new URL("https://www.baidu.com");
        //字节流
        InputStream is = url.openStream();
        //字节流转换为字符流
        InputStreamReader isr = new InputStreamReader(is);
        //缓存功能
        BufferedReader br = new BufferedReader(isr);
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
        br.close();
    }
}
```

## Sockets

- ServerSocket：服务器端类
- Socket：客户端类
- 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210704204721.jpeg)

## Datagram

- DatagramSocket：通信类
- DatagramPacket：数据包类

# 7、NIO

新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的，弥补了原来的 I/O 的不足，提供了高速的、面向块的 I/O。

## 流和块

I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流的 I/O 一次处理一个字节数据：一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。缺点是面向流的 I/O 通常相当慢。

面向块的 I/O 一次处理一个数据块，按块处理数据比按流处理数据要快得多。

对于 I/O 包和 NIO 包，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

## 通道与缓冲区

NIO中的两个核心对象：缓冲区和通道

### 1、通道

通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

通道包括以下类型：（以下均为抽象方法）

- FileChannel：从文件中读写数据；
- DatagramChannel：通过 UDP 读写网络中数据；
- SocketChannel：通过 TCP 读写网络中数据；
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

### 2、缓冲区

发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，**不会直接对通道进行读写数据，而是要先经过缓冲区**。

缓冲区实质上是一个数组，但它是一个特殊的数组，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

## 缓冲区状态变量

- capacity：最大容量；
- position：当前已经读写的字节数，position 总是指向将要访问的下一个元素
- limit：还有多少字节的数据需要取出 / 读到缓冲区中。

状态变量的改变过程举例：

① 新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705202854.png)

② 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 为 5，limit 保持不变。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705203011.png)

③ 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，flip() 方法有两个作用：

1. 将 limit 设置为当前 position
2. 将 position 设置为 0。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705203042.png)

position 为 0，可以保证在下一步输出时读取到的是缓冲区中的第一个字节；而 limit 被设置为当前 position，可保证读取的数据正好是之前写入到缓冲区中的数据。

④ 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。当调用 get() 方法从缓冲区中读取数据写入到输出通道，这会导致 position 的增加而 limit 保持不变，但 position 不会超过 limit 的值。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705203257.png)

⑤ 在从缓冲区中读取数据完毕后，最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705203345.png)



## 文件快速复制 NIO 实例

```java
public class Test {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        new Test().fastCopy("E:\\JavaStudy\\t1.txt", "E:\\JavaStudy\\t2.txt");
    }

    private void fastCopy(String src, String dist) throws IOException {
        //输入字节流
        FileInputStream fis = new FileInputStream(src);
        //输入字节流文件通道
        FileChannel fcIn = fis.getChannel();
        //输出字节流
        FileOutputStream fos = new FileOutputStream(dist);
        //输出字节流文件通道
        FileChannel fcOut = fos.getChannel();
        //缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while (true) {
            //将通道中的数据读取到缓冲区中
            int r = (int) fcIn.read(buffer);
            if (r == -1) {
                break;
            }
            //转换为输出
            buffer.flip();
            //输出通道
            fcOut.write(buffer);
            //清空缓冲区
            buffer.clear();
        }
    }
}
```

## 选择器

NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。

NIO 实现了 IO 多路复用中的 Reactor 模型，**一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件**。

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于 IO 密集型的应用具有很好地性能。

应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能配置为非阻塞。

![img](https://gitee.com/sgkurisu/pic-go/raw/master/picture2/20210705205838.png)

### 1、创建选择器

```java
Selector selector = Selector.open();
```

### 2、将通道注册到选择器上

```java
ServerSocketChannel ssChannel = ServerSocketChannel.open();
ssChannel.configureBlocking(false);
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了。在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类：

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

源码如下：

```java
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;
```

可以看出每个事件可以被当成一个位域，从而组成事件集整数。例如：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

### 3、监听事件

```java
int num = selector.select();
```

使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。

### 4、获取到达的事件

```java
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = keys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key.isAcceptable()) {
        // ...
    } else if (key.isReadable()) {
        // ...
    }
    keyIterator.remove();
}
```

### 5、事件循环

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端需要将处理事件的代码一般会放在一个死循环内。

```java
while (true) {
    int num = selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
            // ...
        } else if (key.isReadable()) {
            // ...
        }
        keyIterator.remove();
    }
}
```

### 6、实例

服务器端：

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

//服务器端
public class NIOServer {
    public static void main(String[] args) throws IOException {
        //创建选择器
        Selector selector = Selector.open();
        //将通道注册到选择器上
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        //服务器
        ServerSocket serverSocket = serverSocketChannel.socket();
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        serverSocket.bind(address);
        while (true) {
            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = keys.iterator();

            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();
                if (key.isAcceptable()) {
                    ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();
                    //服务器为每个新连接创建一个 SocketChannel
                    SocketChannel sChannel = ssChannel.accept();
                    sChannel.configureBlocking(false);
                    //这个新连接主要用于读取数据
                    sChannel.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) { //读取数据
                    SocketChannel sChannel = (SocketChannel) key.channel();
                    System.out.println(readDataFromSocketChannel(sChannel));
                    sChannel.close();
                }
                keyIterator.remove();
            }
        }
    }

    //从 SocketChannel 中读取数据
    private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        StringBuilder stringBuilder = new StringBuilder();

        while (true) {
            buffer.clear();
            int n = sChannel.read(buffer);
            if (n == -1) {
                break;
            }
            buffer.flip();
            int limit = buffer.limit();
            char[] dst = new char[limit];
            for (int i = 0; i < limit; i++) {
                dst[i] = (char) buffer.get();
            }
            stringBuilder.append(dst);
            buffer.clear();
        }
        return stringBuilder.toString();
    }
}
```

用户端：

```java
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

//客户端
public class NIOClient {
    public static void main(String[] args) throws IOException {
        //服务器端地址
        Socket socket = new Socket("127.0.0.1", 8888);
        OutputStream outputStream = socket.getOutputStream();
        String s = "Hello world";
        outputStream.write(s.getBytes(StandardCharsets.UTF_8));
        outputStream.close();
    }
}
```

## 内存映射文件

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。

向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

```java
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```

## 对比

NIO 与普通 I/O 的区别主要有以下两点：

- NIO 是非阻塞的；
- NIO 面向块，I/O 面向流。







