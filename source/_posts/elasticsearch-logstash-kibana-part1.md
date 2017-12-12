---
title: 基于ELK+Filebeat搭建日志中心
date: 2017-12-12 16:32:02
tags:
	- ELK
    - Java
categories: ELK
---

## 基于ELK+Filebeat搭建日志中心

### 实验环境

1. ubuntu 16
2. jdk 1.8

> 我也在windows10下安装过，win10下只需要修改filebeat的文件路径配置就可以了。
### 概念介绍

elastic提供了非常多的工具，官方称为Elastic Stack。它提供了一些[解决方案](https://www.elastic.co/cn/solutions)。我们用到的就是其中Stack中的部分工具

1. [Elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

Elasticsearch 是一个分布式、可扩展开源全文搜索和分析引擎。它能够进行存储，并且能以很快的速度(接近实时)来进行搜索和分析大量的数据。它使用Java编写，底层基于Apache Lucene。

2. [Logstash](https://www.elastic.co/cn/downloads/logstash)

Logstash 是一个开源的，服务端数据处理管道。 它可同时从多个源来搜集数据，对数据进行过滤等操作，并且将数据传送到你想要传送的地方进行存储。

3. [Kibana](https://www.elastic.co/cn/downloads/kibana)

Kibana 被设计成一个和Elasticsearch共同使用的开源分析和可视化平台，它支持将ES中存储的数据生成多种维度的图等。

4. [Filebeat](https://www.elastic.co/downloads/beats/filebeat)

Filebeat 是一个轻量级的日志采集器，用于将源数据采集后发送给Logstash，并且Filebeat使用背压敏感协议，以考虑更多的数据量。如果Logstash正在忙于处理数据，则可以让Filebeat知道减慢读取速度。还有很多Beat可以提供选择，有监控网络的Packbeat，指标Metricbeat。各种beat详情：[https://www.elastic.co/cn/products/beats](https://www.elastic.co/cn/products/beats)

### 环境搭建

将ELK和filebeat下载后解压到一个目录下，四个产品都是开箱即用的。

#### Logstash安装与配置

Logstash的作用主要是用来处理数据，在Logstash的**根目录**，我们需要建立一个新文件`logstash.conf`:

```conf
# 配置logstash输入源
input {
  beats {
    host => "localhost"
    port => "5043" #注意要和filebeat的输出端口一致
  }
}

# 配置输出的地方
output {
  # 控制台
  stdout { codec => rubydebug }
  # es
  elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```

#### Filebeat安装与配置

Filebeat作为日志搜集器，肯定是需要指定输入源和输出地，所以我们需要先配置它。在Filebeat的根目录下我们需要在`filebeat.yml`中:

```yaml

# 输入源
filebeat.prospectors:
- type: log
  paths:
    - /var/log/*.log
    # 如果是windows,如下
    # - C:\Users\chen\Desktop\elastic\elkjava\log\*.log
# 输出地
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5043"]
```

#### Elasticsearch和Kibana安装与配置

这次演示我们对es和kibana不做特别的配置，就默认就好了。

#### 启动顺序

其实没有特别的顺序，我一般的启动顺序是elasticsearch->logsatsh->kibana->filebeat。其实差别都不大，特别注意一个就是，如果你想要测试的话，logstash支持配置自动更新，如果是日志文件更新，想让filebeat重新再搜索一次，删除掉filebeat根目录下`data/registry`文件。其实类推，如果要删除其它的软件(elk)的的数据，删掉`data`目录,很直接很暴力，但不建议在正式场景直接这样弄。

启动es:

```shell
./bin/elasticsearch
```

启动logstash：

```shell
# 测试conf文件是否正确配置
./bin/logstash -f logstash.conf --config.test_and_exit

# 启动logstash，如果有修改conf会自动加载
./bin/logstash -f logstash.conf --config.reload.automatic
```

启动kibana：

```shell
./bin/kibana
```

启动filebeat:

```shell
./filebeat -e -c filebeat.yml -d "publish"
```

现在在浏览器中输入：[http://localhost:5601](http://localhost:5601)打开kibana，

看到下面的图：
![2017-12-12-20-38-18](/images/qiniu/2017-12-12-20-38-18.png)

点击create，之后再点击左侧导航栏Discover：

![2017-12-12-20-39-51](/images/qiniu/2017-12-12-20-39-51.png)

就能看到值了。
