---
layout: post
title: "基类构造函数，子类构造函数，成员类构造函数的调用顺序"
date: 2014-05-19
categories:
- java
tags:
- java

---

这是Java 笔试经常遇到的一个问题，所有特定写代码研究下。

<!-- more -->



```
	class Father {
	    public Father() {
	        System.out.println("In Father");
	    }
	}

	class Children extends Father {
	    private Friend friend = new Friend();
	    public Children() {
	        System.out.println("In Children");
	    }

	}

	class Friend {
	    public Friend() {
	        System.out.println("In Friend");
	    }
	}
```


以上三个对象，运行 ``` new Children(); ``` ，执行的结果是

	In Father
	In Friend
	In Children


### 结论
先执行基类构造函数，再执行成员类构造函数，最后执行子类构造函数。
