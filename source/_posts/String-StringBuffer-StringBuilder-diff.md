---
layout: post
title: "String，StringBuffer，StringBuilder的区别"
date: 2014-05-19
categories:
- java
tags:
- java
desciption: String，StringBuffer，StringBuilder的区别

---

### String
String值是不可变的，每次对String的操作都会生出一个新的String对象。如果频繁改动的话，效率会很低，产生太多的垃圾会触发JVM的垃圾回收，影响系统性能。  
另外 `String s = new String("abc") ` 会生出两个对象, 因为括号里面的"abc"算一。

<!-- more -->

### StringBuffer
可变长度的字符串缓存区，特定是在append与insert操作的时候，速度会比String快很多。并且在多线程下是安全的。如果大量频繁的字符串操作，考虑使用该类。  


### StringBuilder
5.0后新增的方法，在多线程下不保证同步（即线程不安全），但速度会比StringBuffer快。所以，在单线程的环境下，建议使用StringBuilder。 

### 总结
一般情况下，对字符串的操作速度 StringBuilder > StringBuffer > String。  
* 字符串操作少的情况，使用 String
* 单线程，字符串操作多的情况， 使用StringBuilder
* 多线程，字符串操作多的情况， 使用StringBuffer

### 额外的注意
`String s = a + b ` 的情况 ，实际上JVM的处理是这样` String c = (new StringBuilder(String.valueOf(a))).append(b).toString() `  
如此一来每次字符串操作都会生出一个StringBuffer对象。效率多少会有点影响的。
