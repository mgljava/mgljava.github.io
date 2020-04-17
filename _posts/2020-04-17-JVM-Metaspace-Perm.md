---
layout:     post
title:      JVM永久代和元空间
date:       2020-04-17
author:     Monk
header-img: img/JVM_Args.jpg
catalog: true
tags:
    - JVM
    - Java
---

#### 1. 介绍
在本教程中，我们将研究Java环境中永久代（PermGen）和元空间（Metaspace）内存区域之间的差异。
重要的是要记住，从Java 8开始，Metaspace取代了PermGen，带来了一些实质性的变化。

#### 2. 永久代（PermGen）
**PermGen（永久代）是与主内存堆分开的特殊堆空间。**
JVM跟踪PermGen中已加载的类的元数据。此外，JVM将所有静态内容存储在此内存部分中。这包括所有静态方法，原始变量和对静态对象的引用。

此外，它包含有关字节码，名称和JIT信息的数据。在Java 7之前，字符串池也是该内存的一部分。

32位JVM的默认最大内存大小为64 MB，而64位版本的默认最大内存大小为82 MB

然而，我们可以改变默认的JVM的参数
- -XX:PermSize=[Size] 设置永久代空间的初始化或最小值
- -XX:MaxPerm=[Size] 设置永久代最大值

最重要的是，Oracle在JDK 8版本中完全删除了此内存空间。

由于其有限的内存大小，PermGen参与了产生著名的OutOfMemoryError的工作。简而言之，没有正确地收集类加载器，结果导致了内存泄漏。

因此，我们收到一个内存空间错误；在创建新的类加载器时，这主要发生在开发环境中。

#### 3. 元空间（Metaspace）
简单的说，元空间是一种新的内存空间–从Java 8版本开始；**它已替换了较旧的永久代（PermGen）内存空间**。最重要的区别是它如何处理内存分配。

因此，默认情况下，此本机内存区域会自动增长。在这里，我们还有新的参数来调整内存：
- MetaspaceSize 和 MaxMetaspaceSize - 设置元空间的上限大小
- MinMetaspaceFreeRatio - 是垃圾收集后可用的类元数据容量的最小百分比
- MaxMetaspaceFreeRatio - 是垃圾收集后为避免减少空间量而释放的类元数据容量的最大百分比

此外，垃圾收集过程也从这一更改中获得一些好处。一旦类元数据使用量达到其最大元空间大小，垃圾收集器现在会自动触发对类的清理（回收由已经不存活的类加载器加载的类）。

因此，通过这种改进，JVM减少了出现OutOfMemory的错误的机会。

尽管有这些改进，我们仍然需要监视和调整元空间，以避免内存泄漏。

#### 4. 总结
本文简要介绍了PermGen和元空间内存区域。此外，我们还解释了它们之间的关键区别。

PermGen仍然支持JDK 7和旧版本，但是Metaspace为我们的应用程序提供了更灵活和可靠的内存使用。

文章来源：https://www.baeldung.com/java-permgen-metaspace