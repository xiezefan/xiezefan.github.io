---
layout: post
title: "final, finally, finalize的区别"
date: 2014-05-19
categories:
- java
tags:
- java
description:  final, finally, finalize的区别

---


## final, finally, finalize的区别
### final
* 如果一个类被声明为final，此类被能被重载。因此final和abstract不能同时修饰一个类
* 如果一个方法被声明为final，此方法只能被使用，不能被重载
* 如果一个变量被声明为final，此变量只能被使用，不能被修改，并且在声明的时候一定要初始化

<!-- more -->

### finally
异常处理的程序块，使用finally来进行必要的清理工作（如关闭数据库联系，文件流之类的）。  
值得一提的是  

```
public int testFinally1() {
    int i = 0;
    try {
        i++;
        return i++;
    } finally {
        System.out.println("In finally");
        i++;
    }
}
public int testFinally2() {
    int i = 0;
    try {
        i++;
        return i++;
    } finally {
        System.out.println("In finally");
        i++;
    }
}
```
testFinally1()返回的值是1，testFinally2()返回的值是2。 也就是说，finally会在return后执行。并且，return后，变量的改变并不会对返回值照成影响

### finalize
类似于C++的析构函数，但实际上有很大区别，垃圾回收器准备释放对象占用的存储空间的时候，就会调用finalize()方法，做一些清理工作。但是必须等到下一次垃圾回收回收动作才会回收内存。  
所以，并不是finalize调用后内存就会被回收。因为垃圾回收是需要系统开销的。不到内存濒临用光或者程序退出的时候，垃圾回收动作很可能不会发送。  
另外，，执行finalize的线程优先级一般比较低，所以即使垃圾回收器工作，finalize也不一定得到及时的执行  
finalize使用场景：
  
* 调用本地方法（如C，C++），其申请的内存是不会被垃圾回收器回收的。finalize可用来回收这部分内存。
* 用于发现如文件是否被关闭，连接是否被关闭这种情景。