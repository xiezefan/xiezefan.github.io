---
layout: post
title: "maven入门"
date: 2014-05-13
categories:
- Technical
tags:
- Maven

---

Maven简单入门， 快速入手

<!-- more -->

## 下载
    http://maven.apache.org/download.html

## 配置
    MAVEN_HOME : D:\apache-maven-3.0.2  
    MAVEN : %MAVEN_HOME%\bin   
    (可选） MAVEN_OPTS : -Xms256m -Xmx512m
    PATH: 添加 %MAVEN%

## 开始
### 验证安装成功
    mvn -version

正常应该显示  

    Apache Maven 3.2.1 (ea8b2b07643dbb1b84b6d16e1f08391b666bc1e9;2014-02-15T01:37:52+08:00)
    Maven home: D:\apache-maven-3.2.1\bin\..
    Java version: 1.7.0_09, vendor: Oracle Corporation
    Java home: C:\Program Files\Java\jdk1.7.0_09\jre
    Default locale: zh_CN, platform encoding: GBK
    OS name: "windows 8", version: "6.2", arch: "amd64", family: "windows"

### 创建项目

    #此命令创建一个默认项目
    mvn archetype:create -DgroupId=com.xiezefan.app -DartifactId=my-app
    #此命令创建一个web项目
    mvn archetype:create -DgroupId=com.xiezefan.app -DartifactId=webappp -DarchetypeArtifactId=maven-archetype-webapp

创建一个默认项目，项目名为my-app，项目包结构为com.xiezefan.app  

* DgroupId 项目包结构
* DartifactId 项目名
* DarchetypeArtifactId 项目类型（maven-archetype-webapp是web项目，打包后生成war包）

### 常用命令

    mvn package  #打包项目，感觉pox.xml的packaging确定打包成jar or war
