根据 [thread dump线程状态以及线程的定义 转](https://blog.csdn.net/wangen2010/article/details/100064702) 整理

# 1、在IDEA中使用命令行操作

![image-20210414205044137](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191559.png)

# 2、输入jps查看线程的pid

![image-20210414205124806](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191602.png)

# 3、jstack

jstack pid 获得线程状态，jstack能得到运行java程序的java stack和native stack的信息

![image-20210414205230752](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191604.png)

![image-20210414205250665](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191608.png)

# 4、jmap

jmap pid得到运行java程序的内存分配的详细情况

![image-20210414205509564](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191611.png)

5、jstat

可以观察到classloader，compiler，gc相关信息。可以时时监控资源和性能 

命令格式 
-class：统计class loader行为信息 
-compile：统计编译行为信息 
-gc：统计jdk gc时heap信息 
-gccapacity：统计不同的generations（不知道怎么翻译好，包括新生区，老年区，permanent区）相应的heap容量情况 
-gccause：统计gc的情况，（同-gcutil）和引起gc的事件 
-gcnew：统计gc时，新生代的情况 
-gcnewcapacity：统计gc时，新生代heap容量 
-gcold：统计gc时，老年区的情况 
-gcoldcapacity：统计gc时，老年区heap容量 
-gcpermcapacity：统计gc时，permanent区heap容量 
-gcutil：统计gc时，heap情况 

![image-20210414205752506](https://gitee.com/sgkurisu/pic-go/raw/master/picture/20210415191613.png)