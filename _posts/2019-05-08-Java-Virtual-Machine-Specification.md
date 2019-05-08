---
layout:     post
title:      Java虚拟机规范
subtitle:   Java Virtual Machine Specification
date:       2019-05-08
author:     Monk
header-img: img/Java-Virtual-Machine-Specification.png
catalog: true
tags:
    - JVM
---

# Java虚拟机规范(Java8)

## Java虚拟机结构
#### 运行时数据区域
1. PC寄存器：如果该方法不是native的,那么保存Java虚拟机正在执行的字节码指令的地址，否则值是undefined
2. Java虚拟机栈：每一个线程都有一个自己的虚拟机栈，用于存储栈帧。
3. Java堆：提供各个线程共享的运行时的内存区域，也是供所有类实例和数组对象分配的区域。
4. 方法区：供各个线程共享的数据区域。存储一个类的结构信息，如：运行时常量池，字段，方法数据，构造函数和普通方法的字节码的内容
5. 运行时常量池
6. 本地方法栈