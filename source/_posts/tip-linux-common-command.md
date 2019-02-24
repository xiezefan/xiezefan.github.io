---
layout: post
title: "Linux常用命令"
date: 2014-10-05
categories:
- tips
tags:
- linux 
---


**安装右键从终端启动**
> sudo apt-get install nautilus-open-terminal

**复制文件到远程目录**
> scp filename  xiezf@192.168.248.124:/home/push

<!-- more -->

**如果是复制文件夹，使用**
> scp -r filename  xiezf@192.168.248.124:/home/push

**清理dns cache**
> sudo /etc/init.d/dns-clean start 

**查看域名解析**
> nslookup  api.jpush.cn
