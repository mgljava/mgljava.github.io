---
layout:     post
title:      Grafana监控JVM进程
date:       2020-04-10
author:     Monk
header-img: img/JVM_Args.jpg
catalog: true
tags:
    - JVM
    - Grafana
---

##### 总体架构图
下图展示了各个组件在此过程中的作用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403222955302.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)
#### 组件1：jmx_prometheus，负责生成JVM的监控信息
1. 下载agent的jar包：jmx_prometheus_javaagent-0.12.0.jar，[下载地址](https://download.csdn.net/download/MGL_1/12300896)
2. 配置：jmx_prometheus.yaml，更多配置信息请参考 jmx_exporter
```yaml
wercaseOutputLabelNames: true
lowercaseOutputName: true
whitelistObjectNames: ["java.lang:type=OperatingSystem"]
blacklistObjectNames: []
rules:
  - pattern: 'java.lang<type=OperatingSystem><>(committed_virtual_memory|free_physical_memory|free_swap_space|total_physical_memory|total_swap_space)_size:'
    name: os_$1_bytes
    type: GAUGE
    attrNameSnakeCase: true
  - pattern: 'java.lang<type=OperatingSystem><>((?!process_cpu_time)\w+):'
    name: os_$1
    type: GAUGE
    attrNameSnakeCase: true
```
3. 启动：`java -javaagent:jmx_prometheus_javaagent-0.12.0.jar=6666:./jmx_prometheus.yaml -jar yourJar.jar`

#### 组件2：Prometheus，负责收集和存储JVM信息
1. 下载Prometheus，[下载地址](https://github.com/prometheus/prometheus/releases/download/v2.17.1/prometheus-2.17.1.linux-amd64.tar.gz)
2. 配置yaml文件

```yaml
scrape_configs:
  - job_name: 'java'
    scrape_interval: 30s
    static_configs:
    - targets:
      - '127.0.0.1:6666' // 代表先前启动的agent的地址和端口
```
3. 启动prometheus: `./prometheus --config.file=./prom-jmx.yml`
4. 访问 http://127.0.0.1:9090/ 有界面代表启动成功
5. 查看已有的JVM信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403223018298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)

#### 组件3：Grafana，负责展示JVM状态
1. 安装Grafana，按照官网操作，很简单，默认启动端口3000
2. 新建数据源(选择Prometheus)和新建 JVM dashboard
3. 结果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403223038769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403223056853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403223106414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403223116521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01HTF8x,size_16,color_FFFFFF,t_70)