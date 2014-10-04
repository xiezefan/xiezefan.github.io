---
layout: post
title: "finally引起的异常丢失问题"
categories:
- Java
tags:
- Java

---

## finally引起的异常丢失问题

### 场景一

```Java

	public void loseException() throws Exception {
		try {
			throw new Exception("Exception A");
		} finally {
			throw new Exception("Exception B");
		}
	}

```

调用 ``` loseException() ``` 你会发现，Exception A 被 Exception B覆盖掉了。这是非常严重的设计缺陷，并且很难察觉这些错误。
目前Java还未修正这个错误。 其解决办法是将所有抛出异常的方法都打包同一个try-catch中。  

### 场景二

```Java

	public void loseException2() throws Exception {
		try {
			throw new Exception("Exception A");
		} finally {
			return;
		}
	}

```

这种方法让你更简单粗暴的丢失异常，并且不会产生任何输出。