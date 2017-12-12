---
title: ELK搭建日志分析系统
date: 2017-12-09 15:02:19
tags:
---

## ELK搭建日志分析系统

> 目标：1）完成ELK环境搭建，配置
        2）logstash 日志过滤处理
        3）多行日志处理


### 环境搭建与日志准备

ElasticSearch，Logstash，Kibana都是elastic.co的产品。可以在官网上下载到。下载后，需要注意的是logstash需要自己配置一个conf文件。

![2017-12-09-15-31-19](/images/qiniu/2017-12-09-15-31-19.png)。

其它的我们先不动，接下来我们准备一份日志文件，我主要是java开发，就准备了一份java打印的日志文件:[java日志文件]()


kibana
![2017-12-09-15-46-07](/images/qiniu/2017-12-09-15-46-07.png)