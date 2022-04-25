---
layout: post
title: "ELK实战 - 利用Nginx日志分析API耗时"
date: 2017-04-09
categories:
- elk
tags:
- elk
- gork

---


本篇主要介绍如何利用ELK分析Nginx日志，统计出API的耗时数据

ELK是ElasticSearch，Logstash，Kibana的简称，但我觉得这个名字已经过时了，现在叫ELKB更合适，因为Elastic家族近期迎来了一位新成员Beats，专职做数据采集工作。

我们先来介绍下ELK的整体架构吧。

<!-- more -->

早些时候，ELK的流水线是这样

```
Raw Logs(原始日志数据) -> Logstash(数据采集与解析) -> ElasticSearch(数据存储) -> Kibana(数据展示)
```

但是该模式有一个小缺陷是Logstash，主要因为Logstash是使用Java编写的，本身启动需要JVM资源，并且在宿主机上进行日志采集并分析必然会消耗资源。

后来，Elastic使用Go写了一个专门用于数据采集的工具集[Beats](https://www.elastic.co/products/beats)，其包括:

现在，ELKB的流水线编程这样

```
Raw Logs(原始日志数据) -> Beats(数据采集) -> Logstash(数据解析) -> ElasticSearch(数据存储) -> Kibana(数据展示)
```

在此之前，我们先搭建好ElasticSearch与Kibana，ELK家族的有一个优良传统，就是开箱即用。网上也有很多文章详细的讲述了ELK的安装细节，所以，本文就不在安装细节方面赘述，具体的安装细节，可以参考官方文档: 

* [ElasticSearch 安装](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)
* [Kibana 安装](https://www.elastic.co/guide/en/kibana/current/targz.html)



### Configure Filebeat 

我们直接进入正题，如何使用ELK来分析Nginx日志，并绘制API响应耗时的图表。

首先我们看一下Nginx的日志打印设置

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$request_time" "$host" '
                      '"$upstream_status" "$upstream_addr" "$upstream_response_time"' ;
```

默认情况下，我们将不同业务的Nginx访问日志写入到不同的文件里面，在我的系统里，Nginx日志被保存在`/usr/local/nginx/logs/api_acess.log`中。

我们在这里，使用Beats工具中的FileBeats来收集日志Nginx日志

```
filebeat:
  prospectors:
    -
      paths:
        - /usr/local/nginx/logs/api_acess.log
      input_type: log

output:

  logstash:
    hosts: ["192.168.248.216:5045", "192.168.248.217:5045"]
    loadbalance: true
    index: <your service index>

logging:
  files:
    rotateeverybytes: 10485760 # = 10MB
```

以上配置文件主要是读取`/usr/local/nginx/logs/api_acess.log`的日志，并写入对应LogStash提供的TCP接口中。具体Filebeat的详细使用文档，可以参考（//TODO 文档链接）


### Configure Logstash

接下来，我们需要启动对应的LogStash服务，它主要提供以下两个功能：

1. 提供5044接口给Filebeat做文件写入
2. 解析Nginx日志，提取需要的数据结构并写入ElasticSearch

```
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    patterns_dir => ["/opt/push/logstash/patterns"]
    match => { 'message' => '%{IPORHOST:clientip} - %{APPKEY:appkey} \[%{HTTPDATE:timestamp}\] "%{HTTPMETHOD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:http_status} %{NUMBER:response_size} %{QS:http_referer} %{QS:user_agent} %{QS:x_forwarded_for} "%{BASE10NUM:request_time}" "%{HOST:request_host}"  %{BASE10NUM:upstream_header_time} %{NUMBER:upstream_response_time:float} %{HOST:upstream_addr}:%{NUMBER:upstream_port}' }
  }
}

output {
    elasticsearch {
        hosts => ["172.16.99.xxx:1xxxx"]
        manage_template => false
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
        flush_size => 20000
        idle_flush_time => 10
    }
}
```

这里我们利用到了gork表达式。 他是Elastic公司设计专门用来解析日志的工具，具体[Gork简易入门](http://xiezefan.me/2017/04/09/elk_in_action_grok_start/)。

简单说，gork表达式可以方便的组合正则表达式，提取出日志的相关数据，最终转变成JSON格式输出。以下是我专门为以上Nginx日志结构写的Gork表达式，我们可以使用Gork Debuger工具来帮我们校验Gork表达式的正确性。

Gork表达式需要一定的学习成本，但有了它，我们可以更方便快捷地分析日志。

接下来，依次启动LagStash与FileBeat。 我们先认为的制造一点日志输出，可以看到在ElasicSearch中已经出现了响应的索引。

### Configure Kibana

我们在Kibana中建立该索引，可以看到，日志的关键数据已经成功被提取并存入到ElasticSearch。接下来，我们将利用Kibana来统计报表。

![](http://pics.xiezefan.me/blog/elk_in_action_kibana_chart)


在Kibana绘制图标本身什么技术难度，关键一点是在将数据写入ElasticSearch中的时候，需要将一些关键指标转换成数值类型，我们可以在Grok转换的时候，指定生成Number类型，或者之间预定义相关业务的ElasticSearch Mapping Template。

有一些绘图小技巧接下来准备独立整理一篇文章做单独分享。



