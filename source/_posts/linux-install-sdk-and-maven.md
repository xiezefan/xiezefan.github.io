---
layout: post
title: "Linux系统安装配置JDK与Maven"
date: 2015-01-03
categories: 
- linux
tags: 
- linux
- maven
- jdk
description: Linux 系统下快速按安装配置JDK与Maven的多种方法及利弊。

---

Linux 系统下快速按安装配置JDK与Maven的多种方法及利弊。

<!-- more -->

### 下载

> Java  
> http://www.oracle.com/technetwork/java/javase/downloads/index.html  
> Maven  
> http://maven.apache.org/download.cgi

### 配置

修改 `/etc/profile` 文件
#### Java
```
JAVA_HOME=/usr/share/jdk1.5.0_05 
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
export JAVA_HOME 
export PATH 
export CLASSPATH
```
#### Maven

```
MAVEN_HOME=~/apache-maven-3.2.3
export MAVEN_HOME
```

#### 即刻生效修改
```
source /etc/profile
```


### 其他配置方法

上述方法是通过修改/etc/profile文件，但这个修改是全局的，所以，基于安全考虑，当只需要给某个用户权限使用这个环境变量的时候，只需修改该用户目录下的.bashrc文件并重启系统即可。
当然也可以直接在sheel中设置，不过此方法设置后，在关闭了sheel后就会失效， 看需求使用

```
# 在sheel中导入JDK配置
export JAVA_HOME=/usr/share/jdk1.5.0_05
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

### 参考文章

* [百度知道-linux下如何设置JDK环境变量][1]


  [1]: http://zhidao.baidu.com/link?url=0XeoCXTgx-QLIMZVfWQlsak206gNr_7dkmdYHenFEB25gyt35Ctqzq5W0Kp9WmYaJT2LhSBsacETKP5Iizefm_
