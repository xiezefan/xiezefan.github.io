---
layout: post
title: "MQTT 基础知识"
date: 2014-12-29
categories:
- mqtt
tags:
- mqtt 
- im

---

### 基础内容

MQTT的固定头部包含以下信息

#### MessageType
消息类型，使用4位二进制标示，共16种消息类型，其中0和15位做保留待用，实际使用共14种消息事件类型

#### DUP flag 
默认为0，标示该消息为第一次发送，当该值为一时，标示消息先前已经被传输过了，该位前置条件为Qos > 0，标示消息需要回复确认

#### QoS level
服务质量，由两个二进制位标示
*　0：至多一次
*　1：至少一次
*　2：只有一次
*　3：保留

#### RETAIN
是否对PUBLISH消息进行持久化
* 1：标示需要持久化， 当新订阅者出现时，会收到最新一个持久化消息
* 2：标示不需要持久化，推送仅对当前订阅者
**当RETAIN=1，Payload=NULL时标示删除该Topic的持久化PUBLISH消息**

### Topic通配符
> /：用来表示层次，比如a/b，a/b/c。
> \#：表示匹配>=0个层次，比如a/#就匹配a/，a/b，a/b/c。
> 单独的一个#表示匹配所有。
> 不允许 a#和a/#/c。
> +：表示匹配一个层次，例如a/+匹配a/b，a/c，不匹配a/b/c。
> 单独的一个+是允许的，a+不允许，a/+/b不允许

### 心跳 PINGREQ/PINGRES
Client告知Server其心跳间隔KeepAliveTime，Client需要在该时长内发送PINGREQ，Server收到后返回PINGRES确认以保持Client与Server的长链接。
Server在1.5个时长内未收到PINGREQ，就断开连接
Client在1个时长内未收到Server的PINGRES，就断开连接
时间最长为18hours，0标示不断开

### Clean Session
服务端是否保存Client的订阅信息
* true:保存
* false:不保持

### 字段建议长度
* clientId 客户端->服务端, 服务端->客户端的单向唯一标示,length<=23
* username 用户名,用于身份验证, length<=12
* password 用户密码,用户身份验证, length<=12


### 遗嘱消息 WillMessage
遗嘱消息标示客户端网络异常导致连接中断后, 服务器将发布该遗嘱消息
遗嘱消息包含以下信息:
* Will Flag:是否定义遗嘱消息，Will Flag=1是标示指定遗嘱消息，否则将直接忽略Will Qos，Will RETAIN的值
* Will Qos:遗嘱消息的通讯质量
* Will RETAIN:遗嘱消息是否持久化
* Will Topic:遗嘱消息主题
* Will Message:遗嘱消息Payload

### 建立连接CONNECT的响应机制
* 客户端绕过CONNECT消息直接发送其它类型消息，服务器应关闭此非法连接
* 客户端发送CONNECT之后未收到CONNACT，需要关闭当前连接，然后重新连接
* 相同Client ID客户端已连接到服务器，先前客户端必须断开连接后，服务器才能完成新的客户端CONNECT连接
* 客户端发送无效非法CONNECT消息，服务器需要关闭



### 参考资料

* [MQTT协议简记][1]
* [MQTT协议笔记之头部信息][2]
* [MQTT协议笔记之连接和心跳][3]


  [1]: http://www.cnblogs.com/caca/p/mqtt.html
  [2]: http://www.blogjava.net/yongboy/archive/2014/02/07/409587.html
  [3]: http://www.blogjava.net/yongboy/archive/2014/02/09/409630.html