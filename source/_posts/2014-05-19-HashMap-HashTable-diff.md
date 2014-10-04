---
layout: post
title: "HashMap,HashTable区别"
categories:
- Java
tags:
- Java

---

## HashMap,HashTable区别

### HashTable
特点：线程同步 

### HashMap
特定：

* 线程不同步，但可以通过 ``` Collections.synchronizedMap(HashMap map) ``` 实现线程同步。
* 允许Key，value值为空。
* 优于HashTable的Hash算法，使Hash值更广泛的分布到数组的不同位置。
* 更优的效率

实际应用中，我们并不经常需要保证HashMap这些底层代码的同步，交由上层逻辑去控制同步。所以，大多数时候建议使用HashMap
