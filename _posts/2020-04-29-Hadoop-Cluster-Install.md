---
layout:     post
title:      搭建Hadoop集群
date:       2020-04-29
author:     Monk
header-img: img/blog_banner.jpg
catalog: true
tags:
    - 大数据
    - Hadoop
---

### Hadoop集群安装（非HA）
#### 环境准备
1. 软件版本  
3台服务器配置，系统：centos 7，master节点：4G 2V 50G，slave节点：1G 1V 50G  
jdk版本：1.8.0_221(系统提前安装好JDK8)  
Hadoop版本：2.10.0  

2. 角色分类  
192.168.0.101： master节点  
192.168.0.102/103： slave节点  

3. 修改hostname    
在101上执行：`echo hadoop-master > /etc/hostname`  
在102上执行：`echo hadoop-slave1 > /etc/hostname`   
在103上执行：`echo hadoop-slave2 > /etc/hostname`  

4. 配置 hosts  
hadoop-master 192.168.0.101  
hadoop-slave1 192.168.0.102  
hadoop-slave2 192.168.0.103  

5. 防火墙设置   
关闭防火墙：`systemctl stop firewalld.service`  
永久关闭：`systemctl disable firewalld.service `

##### [配置免密登录](https://blog.csdn.net/mmd0308/article/details/73825953)

#### 配置hadoop master节点
1. 在hadoop-master上 解压缩安装包及创建基本目录
```shell script
wget http://apache.claz.org/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz

tar -zxvf hadoop-2.10.0.tar.gz -C /usr/local

mv hadoop-2.10.0 hadoop
```

2. 配置Hadoop环境变量
```shell script
vim /etc/profile 加入以下两行

export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

使配置生效,执行下面命令
source /etc/profile
```

3. 配置hadoop的配置文件,配置文件的目录都在 $HADOOP_HOME/etc/hadoop下
* 配置 core-site： hadoop核心配置文件
```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-master:9000</value>
    </property>
</configuration>
```

* 配置hdfs-site.xml: hdfs的配置文件
```xml
<configuration>
    <property> // 副本数量，我们这里只有两个DataNode节点，所以置为2，默认是3
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>/usr/local/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/usr/local/hadoop/hdfs/data</value>
    </property>
</configuration>
```

* 配置 mapred-site.xml
```xml
cp $HADOOP_HOME/etc/hadoop/mapred-site.xml.template $HADOOP_HOME/etc/hadoop/mapred-site.xml
vim $HADOOP_HOME/etc/hadoop/mapred-site.xml

<configuration>
  <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
  </property>
   <property>
      <name>mapred.job.tracker</name>
      <value>http://hadoop-master:9001</value>
  </property>
</configuration>
```

* 配置yarn-site.xml
```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-master</value>
    </property>
</configuration>
```

* 配置masters和slave文件 
```shell script
配置masters文件：echo hadoop-master >masters

配置slaves文件
vi slaves，加入slave节点
hadoop-slave1
hadoop-slave2
```

* 更多配置文件的内容，可以打开目录 `$HADOOP_HOME/share/doc/hadoop` 下的index.html 离线帮助文档

#### 配置hadoop slave节点
1. 拷贝Hadoop安装包，可以删掉doc目录，因为doc目录下的文件太多，将先前在master节点的hadoop目录直接拷贝到各个slave节点的 /usr/local目录，删除 $HADOOP_HOME/etc/hadoop下的slaves文件
2. 配置Hadoop的环境变量,和master节点配置hadoop环境变量一致

#### 启动hadoop集群
```shell script
1. 格式化HDFS文件系统，会创建相应的目录，执行命令为
$HADOOP_HOME/bin/hadoop namenode -format

2. 启动hadoop集群
$HADOOP_HOME/sbin/start-all.sh

```

#### WebUI界面查看
在浏览器中输入 192.168.0.101:50070 可以看到Hadoop集群和浏览HDFS