---
layout:     post
title:      JVM内存划分
date:       2020-04-13
author:     Monk
header-img: img/JVM_Args.jpg
catalog: true
tags:
    - JVM
    - Java
---

### 内存划分
1. 虚拟机栈（Stack Frame）
2. 程序计数器（Program Counter）
3. 本地方法栈：调用本地方法时使用本地方法栈
4. 堆（Heap）：所有线程的共享区域，存放对象实例，JVM管理最大的一块空间。与堆相关的概念是垃圾收集器。现代几乎所有的垃圾收集器都是采用分代收集算法，所以，堆空间也基于这一点进行了划分：新生代和老年代。Eden空间，From Survivor空间与To Survivor空间。
5. 方法区（Method Area）：存储元信息。永久代（Permaent Generation），从JDK1.8开始，已经彻底放弃了永久代，使用元空间（Meta Space），元空间使用本地内存
6. 运行时常量池：方法区的一部分，编译期生成的数据结构
7. 直接内存（Direct Memory）：不归JVM管理，堆外内存，回收和释放取决与OS，与Java的NIO相关，JVM通过堆上DirectByteBuffer来操作直接内存。

### Java虚拟机构成
![image](https://raw.githubusercontent.com/mgljava/mgljava.github.io/master/img/Java虚拟机主要构成.png)

### Java对象创建的过程
new关键字创建对象的3个步骤
1. 在堆内存中创建类的实例
2. 为对象的实例成员变量赋初值（静态变量的初值在类加载期间完成）
3. 将对象的引用返回  
**指针碰撞**（前提是堆中的空间通过一个指针进行分割，一侧是已经被占用的空间，另一侧是未被占用的空间）  
**空闲列表**（前提是堆内存空间中已被使用和未被使用的空间是交织在一起的，这时，虚拟机就需要通过一个列表来记录哪些空间是可以使用的，哪些空间是已被使用的，接下来找出可以容纳下新创建对象的且未被使用的空间，在此空间存放该对象，同时修改表上的记录）

### 对象在内存中的布局
1. 对象头（存储对象运行时的数据信息）
2. 实例数据（在一个类中所声明的各项信息）
3. 对齐填充（可选）

### 引用访问对象的方式
1. 使用句柄的方式
2. 使用直接指针的方式

### Java自带工具的使用
1. jmap
2. jstat
3. jcmd
4. jmc
5. jhat
6. jstack

### jstat:用于统计内存分配速率、GC 次数，GC 耗时
jstat -gc <pid> <统计间隔时间>  <统计次数>
![image](https://raw.githubusercontent.com/mgljava/mgljava.github.io/master/img/jstat命令执行结果.png)

### jmap命令
1. jmap -clstats <pid> 打印类加载器
2. jmap -heap <pid> 打印堆信息

### jcmd命令
1. jcmd <pid> VM.flags 打印JVM的启动参数
2. jcmd <pid> help 列出当前Java进程可以执行的操作
3. jcmd <pid> help VM.flags 查看VM.flags参数的使用
4. jcmd <pid> PerfCounter 查看JVM性能相关的参数
5. jcmd <pid> VM.uptime 启动时间，运行了多久
6. jcmd <pid> GC.class_histogram 查看JVM中类的统计信息
7. jcmd <pid> Thread.print 查看JVM的线程信息
8. jcmd <pid> GC.heap_dump /Users/monk/Desktop/test.hprof 导出Heap Dump文件信息到桌面
9. jcmd <pid> VM.system_properties 查看JVM的系统属性信息
10. jcmd <pid> VM.version 查看JVM进程的版本信息
11. jcmd <pid> VM.command_line 查看JVM启动的命令行参数信息

### jmc(Java Mission Control)图形化界面工具

### jhat
进行堆转储的工具 

jhat test.hprof 输出如下内容：
```
➜  Desktop jhat test.hprof

Reading from test.hprof...
Dump file created Fri Mar 06 13:42:04 CST 2020
Snapshot read, resolving...
Resolving 73731 objects...
Chasing references, expect 14 dots..............
Eliminating duplicate references..............
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

### jstack
查看或者导出JVM进程中的堆栈信息

### 使用JMC或者JvisualVM远程连接Linux服务
java -jar -Djava.rmi.server.hostname=${远程host} -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.ssl=false  -Dcom.sun.management.jmxremote.authenticate=false -XX:+PrintGCDetails -XX:+UseConcMarkSweepGC rocketmq-console-ng-1.0.1.jar