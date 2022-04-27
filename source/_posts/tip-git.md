---
layout: post
title: "Git常用命令集"
date: 2014-10-05
categories:
- Technical
tags:
- Git
---


<!-- more -->

**生成SSH Key**
> ssh-keygen -t rsa -C "committer_email@committermail.com"  

**查看自己拥有的权限**
> ssh -lgit <git host>
> exp: ssh -lgit git.github.com

<!-- more -->

**添加并提交到本地库**
> git commit -m 'your comment'

**将本地仓库添加到远程库**
> git remote add origin <your git.git>

**分支**
> git branch -r    #查看所有分支
> git branch [branch_name] #创建新分支
> git checkout [branch_name] #切换到分支
> git push origin branch_name #上传分支到远程服务器
> git branch --set-upstream master origin/master #将本地分支链接到远程分支

**Tag**
> git tag #显示标签
> git tag -a v3.1.1 -m 'version 3.1.1'    ＃添加标签
> git push origin v3.1.1 ＃推送到云端
> git tag -d v3.1.1 # 删除标签
> git push origin :refs/tags/v3.1.1 # 将删除操作更新到远程git库
