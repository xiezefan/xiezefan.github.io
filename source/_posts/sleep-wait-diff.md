---
layout: post
title: "sleep(), wait()的区别"
date: 2014-05-19
categories:
- Technical
tags:
- Java

---


### sleep(), wait()的区别

#### sleep(milliseconds)
接收一个参数，使当前线程休眠一段时间。用户线程控制。
特点：  

* 不释放同步锁。

<!-- more -->

#### wait()
调用wait()方法将会将调用者的线程挂起，直到其他线程调用同一个对象的notify()方法后，才会重新激活调用者。
特定：  

* 释放同步锁

#### 总结
其实两者都可以让线程暂停一段时间,但是本质的区别是一个线程的运行状态控制,一个是线程之间的通讯的问题


