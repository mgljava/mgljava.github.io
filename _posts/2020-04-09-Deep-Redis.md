---
layout:     post
title:      深入学习Redis
subtitle:   Redis
date:       2020-04-09
author:     Monk
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Redis
---

### Redis内存划分
#### 1. 数据
- 作为数据库，数据是最主要的部分；这部分占用的内存会统计在used_memory中。  
- Redis使用键值对存储数据，其中的值（对象）包括5种类型，即字符串、哈希、列表、集合、有序集合。这5种类型是Redis对外提供的，实际上，在Redis内部，每种类型可能有2种或更多的内部编码实现；此外，Redis在存储对象时，并不是直接将数据扔进内存，而是会对对象进行各种包装：如redisObject、SDS等

#### 2. 进程本身运行需要的内存
Redis主进程本身运行肯定需要占用内存，如代码、常量池等等；这部分内存大约几兆，在大多数生产环境中与Redis数据占用的内存相比可以忽略。这部分内存不是由jemalloc分配，因此不会统计在used_memory中

#### 3. 缓冲内存
缓冲内存包括客户端缓冲区、复制积压缓冲区、AOF缓冲区等；其中，客户端缓冲存储客户端连接的输入输出缓冲；复制积压缓冲用于部分复制功能；AOF缓冲区用于在进行AOF重写时，保存最近的写入命令。在了解相应功能之前，不需要知道这些缓冲的细节；这部分内存由jemalloc分配，因此会统计在used_memory中

#### 4. 内存碎片
- 内存碎片是Redis在分配、回收物理内存过程中产生的。例如，如果对数据的更改频繁，而且数据之间的大小相差很大，可能导致redis释放的空间在物理内存中并没有释放，但redis又无法有效利用，这就形成了内存碎片。内存碎片不会统计在used_memory中。
- 内存碎片的产生与对数据进行的操作、数据的特点等都有关；此外，与使用的内存分配器也有关系：如果内存分配器设计合理，可以尽可能的减少内存碎片的产生。后面将要说到的jemalloc便在控制内存碎片方面做的很好
- 如果Redis服务器中的内存碎片已经很大，可以通过安全重启的方式减小内存碎片：因为重启之后，Redis重新从备份文件中读取数据，在内存中进行重排，为每个数据重新选择合适的内存单元，减小内存碎片

### Redis数据存储细节
- 概述：关于Redis数据存储的细节，涉及到内存分配器（如jemalloc）、简单动态字符串（SDS）、5种对象类型及内部编码、redisObject

### Redis的对象类型与内部编码
1. 字符串:字符串是最基础的类型，因为所有的键都是字符串类型,且字符串之外的其他几种复杂类型的元素也是字符串,字符串长度不能超过512MB,编码类型如下：
- **int**：8个字节的长整型。字符串值是整型时，这个值使用long整型表示
- **embstr**：<=39字节的字符串。embstr与raw都使用redisObject和sds保存数据，区别在于，embstr的使用只分配一次内存空间（因此redisObject和sds是连续的），而raw需要分配两次内存空间（分别为redisObject和sds分配空间）。因此与raw相比，embstr的好处在于创建时少分配一次空间，删除时少释放一次空间，以及对象的所有数据连在一起，寻找方便。而embstr的坏处也很明显，如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间，因此redis中的embstr实现为只读
- **raw**：大于39个字节的字符串  
==编码转换==：当int数据不再是整数，或大小超过了long的范围时，自动转化为raw。而对于embstr，由于其实现是只读的，因此在对embstr对象进行修改时，都会先转化为raw再进行修改，因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了39个字节

2. 列表：列表（list）用来存储多个有序的字符串，每个字符串称为元素；一个列表可以存储2^32-1个元素。Redis中的列表支持两端插入和弹出，并可以获得指定位置（或范围）的元素，可以充当数组、队列、栈等
- **ziplist**：由一个list结构和多个listNode结构组成，压缩列表可以节省内存空间，但是进行修改或增删操作时，复杂度较高；因此当节点数量较少时，可以使用压缩列表；压缩列表不仅用于实现列表，也用于实现哈希、有序列表；使用非常广泛。
- **linkedlist**：但是节点数量多时，还是使用双端链表划算。
- **quicklist**: 现在的list均采用quicklist实现，quicklist包含了ziplist和linkedlist的特性，使用linkedlist，然后每个linkedlist节点是一个ziplist。通过参数可以进行配置。  
==编码转换==：只有同时满足下面两个条件时，才会使用压缩列表：列表中元素数量小于512个；列表中所有字符串对象都不足64字节。如果有一个条件不满足，则使用双端列表；且编码只可能由压缩列表转化为双端链表，反方向则不可能。也可以通过参数配置
3. **哈希**
- **ziplist**：压缩列表用于元素个数少、元素长度小的场景
- **hashtable**：哈希表  
==编码转换==：只有同时满足下面两个条件时，才会使用压缩列表：哈希中元素数量小于512个；哈希中所有键值对的键和值字符串长度都小于64字节。如果有一个条件不满足，则使用哈希表；且编码只可能由压缩列表转化为哈希表，反方向则不可能
4. 集合：集合（set）与列表类似，都是用来保存多个字符串，但集合与列表有两点不同：集合中的元素是无序的，因此不能通过索引来操作元素；集合中的元素不能有重复，一个集合中最多可以存储2^32-1个元素；除了支持常规的增删改查，Redis还支持多个集合取交集、并集、差集
- **intset**：整数集合适用于集合所有元素都是整数且集合元素数量较小的时候，与哈希表相比，整数集合的优势在于集中存储，节省空间
- **hashtable**：集合在使用哈希表时，值全部被置为null   
==编码转换==:只有同时满足下面两个条件时，集合才会使用整数集合：集合中元素数量小于512个；集合中所有元素都是整数值。如果有一个条件不满足，则使用哈希表；且编码只可能由整数集合转化为哈希表，反方向则不可能

5. 有序集合:有序集合与集合一样，元素都不能重复；但与集合不同的是，有序集合中的元素是有顺序的。与列表使用索引下标作为排序依据不同，有序集合为每个元素设置一个分数（score）作为排序依据
- **ziplist**:同集合一样
- **skiplist**: 跳跃表是一种有序数据结构，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的  
==编码转换==：只有同时满足下面两个条件时，才会使用压缩列表：有序集合中元素数量小于128个；有序集合中所有成员长度都不足64字节。如果有一个条件不满足，则使用跳跃表；且编码只可能由压缩列表转化为跳跃表，反方向则不可能。

### Redis持久化
#### RDB

#### AOF
RDB持久化是将进程数据写入文件，而AOF持久化(即Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中（有点像MySQL的binlog）；当Redis重启时再次执行AOF文件中的命令来恢复数据。与RDB相比，AOF的实时性更好，因此已成为主流的持久化方案。

1. 开启AOF：appendonly yes
2. 执行流程
- 命令追加(append)：将Redis的写命令追加到缓冲区aof_buf
- 文件写入(write)和文件同步(sync)：根据不同的同步策略将aof_buf中的内容同步到硬盘
- 文件重写(rewrite)：定期重写AOF文件，达到压缩的目的，操作方式：**AOF重写是把Redis进程内的数据转化为写命令，同步到新的AOF文件；不会对旧的AOF文件进行任何读取、写入操作!**

### Redis主从复制
**主从复制的开启，完全是在从节点发起的；不需要我们在主节点做任何事情**，从节点开启主从复制，有3种方式：
1. 配置文件：在从服务器的配置文件中加入：slaveof <masterip> <masterport>
2. 启动命令：redis-server启动命令后加入 --slaveof <masterip> <masterport>
3. 客户端命令：Redis服务器启动后，直接通过客户端执行命令：slaveof <masterip> <masterport>，则该Redis实例成为从节点。

#### 断开复制：slaveof no one
从节点断开复制后，不会删除已有的数据，只是不再接受主节点新的数据变化，从节点又变回主节点

#### 主从复制实现原理
1. 连接建立阶段：该阶段的主要作用是在主从节点之间建立连接，为数据同步做好准备
- 保存主节点信息
- 建立socket连接
- 发送ping命令
- 身份验证：masterauth
- 发送从节点端口信息
2. 数据同步阶段：从节点向主节点发送psync命令（Redis2.8以前是sync命令），开始同步
3. 命令传播阶段：数据同步阶段完成后，主从节点进入命令传播阶段；在这个阶段主节点将自己执行的写命令发送给从节点，从节点接收命令并执行，从而保证主从节点数据的一致性。

#### 全量复制
用于初次复制或其他无法进行部分复制的情况，将主节点中的所有数据都发送给从节点，是一个非常重型的操作。

复制原理：
1. 从节点判断无法进行部分复制，向主节点发送全量复制的请求；或从节点发送部分复制的请求，但主节点判断无法进行全量复制；具体判断过程需要在讲述了部分复制原理后再介绍
2. 主节点收到全量复制的命令后，执行bgsave，在后台生成RDB文件，并使用一个缓冲区（称为复制缓冲区）记录从现在开始执行的所有写命令
3. 主节点的bgsave执行完成后，将RDB文件发送给从节点；从节点首先清除自己的旧数据，然后载入接收的RDB文件，将数据库状态更新至主节点执行bgsave时的数据库状态
4. 主节点将前述复制缓冲区中的所有写命令发送给从节点，从节点执行这些写命令，将数据库状态更新至主节点的最新状态
5. 如果从节点开启了AOF，则会触发bgrewriteaof的执行，从而保证AOF文件更新至主节点的最新状态

#### 部分复制
用于网络中断等情况后的复制，只将中断期间主节点执行的写命令发送给从节点，与全量复制相比更加高效。需要注意的是，如果网络中断时间过长，导致主节点没有能够完整地保存中断期间执行的写命令，则无法进行部分复制，仍使用全量复制
部分复制的3个前置条件：
1. 复制偏移量：主节点和从节点分别维护一个复制偏移量（offset），代表的是主节点向从节点传递的字节数；主节点每次向从节点传播N个字节数据时，主节点的offset增加N；从节点每次收到主节点传来的N个字节数据时，从节点的offset增加N。
2. 复制积压缓冲区（配置repl-backlog-size大小，默认1MB）：复制积压缓冲区是由==主节点维护==的、固定长度的、先进先出(FIFO)队列，默认大小1MB；当主节点开始有从节点时创建，其作用是备份主节点最近发送给从节点的数据。注意，无论主节点有一个还是多个从节点，都只需要一个复制积压缓冲区。
3. 服务器运行ID(runid):每个Redis节点(无论主从)，在启动时都会自动生成一个随机ID(每次启动都不一样)，由40个随机的十六进制字符组成；runid用来唯一识别一个Redis节点。通过info Server命令

复制原理：
1. 首先，从节点根据当前状态，决定如何调用psync命令；如果从节点之前未执行过slaveof或最近执行了slaveof no one，则从节点发送命令为psync ? -1，向主节点请求全量复制；如果从节点之前执行了slaveof，则发送命令为psync <runid> <offset>，其中runid为上次复制的主节点的runid，offset为上次复制截止时从节点保存的复制偏移量
2. 主节点根据收到的psync命令，及当前服务器状态，决定执行全量复制还是部分复制
- 如果主节点版本低于Redis2.8，则返回-ERR回复，此时从节点重新发送sync命令执行全量复制；
- 如果主节点版本够新，且runid与从节点发送的runid相同，且从节点发送的offset之后的数据在复制积压缓冲区中都存在，则回复+CONTINUE，表示将进行部分复制，从节点等待主节点发送其缺少的数据即可；
- 如果主节点版本够新，但是runid与从节点发送的runid不同，或从节点发送的offset之后的数据已不在复制积压缓冲区中(在队列中被挤出了)，则回复+FULLRESYNC <runid> <offset>，表示要进行全量复制，其中runid表示主节点当前的runid，offset表示主节点当前的offset，从节点保存这两个值，以备使用。

#### 心跳机制
主 -> 从：PING
从 -> 主：REPLCONF ACK：命令格式：REPLCONF ACK {offset}


### Redis哨兵

---   待完善

### Redis集群
---   待完善