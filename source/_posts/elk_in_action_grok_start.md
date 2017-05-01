---
layout: post
title: "ELK实战 - Grok简易入门"
date: 2017-04-09
categories:
- elk
tags:
- elk
- gork

---



Grok是ELK栈中，用来快速解析日志的一个脚本工具，运用得好的话，可以极大程度的降低日志解析的工作，最好的Grok表达式的学习方式是去阅读[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)。

<!--more-->

### Example 示例

一个简单的Grok表达式:

```
%{IP:source_ip}
```

它表示使用名为IP的正则表达式提取出内容，并命名为`source_ip`。

我们以一个Nginx日志的例子来了解Grok表达式的工作模式:

```
// IP HTTP方法 请求资源 数据字节数 响应时间
55.3.244.1 - GET /index.html 15824 0.043
```
解析该日志的Grok表达式为:

```
%{IP:client} - %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}
```

最终，Grok表达式将会从日志中提取出以下数据:

```javascript
{
  "client": "55.3.244.1",
  "method": "GET",
  "request": "/index.html",
  "bytes": "15824",
  "duration": "0.043"
}
```

细心可以注意到，bytes, duration这两个字段，虽然我们定义为NUMBER类型，但是提取出来的数据仍然是String。 我们以通过强制指定数据类型的方式，告知Gork解析器将指定字段保存为对应数据类型:

```
%{NUMBER:bytes:int}
%{NUMBER:duration:float}
```

目前强制类型指定只支持float与int类型。

#### Gork表达式

官方默认提供了几组[Grok表达式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)

我介绍几个我用的比较顺手的Gork表达式

* QS : 匹配双引号之间的字符串
* NUMBER : 匹配数字，支持字符串与浮点型
* HTTPDATE : 匹配Nginx默认保存的日期格式
* IPORHOST : 同时匹配IP或者域名，试用与分析请求来源地址

如果现有的Gork表达式不满足需求，我们以可以直接编写Gork表达式, 格式为

```
# 格式
[名称] [正则表达式]

# 示例(APPKEY:长度为24的随机字符串)
APPKEY [0-9a-zA-Z]{24}
```

文件名并无强制要求，我们可以将所有的Gork表达式保存在patterns文件夹中，这样可以方便让logstash引入所有的正则表达式， 上述提到官方提供的通用[Grok表达式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)，也可以下载后，存放在patterns文件夹中。

```
...
filter {
  grok {
    patterns_dir => ["/opt/push/logstash/patterns"]
    match => { "message" => "<YOUT GROK PATTERN>" }
  }
}
...
```


最后，介绍一个方便的Grok测试工具 --- [Grok Debugger](http://grokdebug.herokuapp.com/) (PS: 需要注意的一点，该工具并不能支持Grok最新的指定数据类型的特性)


### 总结

Grok是节省解析日志时间与精力的利器，本身也不难，学习方法一是阅读[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)，二是看官方提供的[Grok表达式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)示例。稍微花点时间即可以基本掌握。



