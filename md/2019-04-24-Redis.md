### redis 基本数据类型
### redis 内存淘汰机制：redis 提供 6种数据淘汰策略
1. volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）.
5. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
6. no-eviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

### Redis持久化
1. RDB(snapshotting，RDB)
2. AOF(append-only file)
  - appendfsync always     #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
  - appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
  - appendfsync no      #让操作系统决定何时进行同步