---
layout: post
title: "使用Git SubModule对Maven Module进行优化"
date: 2016-8-13
categories:
- maven
tags:
- maven
- git
description: 这篇文章主要讲，如何将Maven Module功能与Git SubModule功能配合使用的问题。
---


这篇文章主要讲，如何将Maven Module功能与Git SubModule功能配合使用的问题。

之所以有这篇文章，是因为在使用Maven管理Java项目的过程中，当项目逐渐发展到一点规模后，我们将项目进行模块划分，将不同业务功能拆分为不同的Maven Module，模块之间有依赖关系，但可以分开部署。

但紧随而来的问题是：

1. 当项目模块越来越多的时候，项目的编译时间越来越长
2. 我们使用Gitlab CI做持续集成与持续交付。 Gitlab CI的持续集成与交付的是由每次Commit与每次Release Tag触发的。也就是说，每次触发CI的时候，各个模块都需要跑一边集成测试，而更麻烦的在于，无法使用Gitlab CI直接对项目进行自动部署，因为你不能每次发布的时候，都将所有的子模块都部署一遍。

<!-- more -->

为了应对这个问题，显然我们需要将项目进行拆分，按部署单元将拆分为不同的Git Repository。常规的做法是将项目中公共的模块抽取成独立的Maven Dependency。 打包发布到私有的Maven仓库。但这显然太繁琐了，因为项目迭代开发过程中，即使是公共的模块，也会频繁的变更，每次变更其余依赖模块都需要升级版本。

自此终于引出文本的主题，如何使用Git SubModule与Maven Module来优雅的解决上述问题。


### Maven Module 拆分示例

我们以一个Blog后台的API项目作为示例

```
.
├── dashboard-api       // Blog管理后台API模块
│   └── pom.xml
├── blog-api            // Blog前端的API模块
│   └── pom.xml
├── blog-common         // Blog系统的公共代码，如Entity，Util，Helper等逻辑
│   └── pom.xml
└── pom.xml
```

dashboard-api与blog-api都依赖blog-common，但两个模块最终都会打成独立的包分开部署。

**PS: 显然一个Blog系统不需要做这么细致的模块划分，这里只是举例子方便大家理解。**

我们的目标是将该项目拆分为3个Git Repositories:

1. blog-common
2. blog-api
3. blog-dashboard-api

首先我们将blog-common项目拆分出来

```shell
cd blog-common
git init
git remote add origin git@your-repository.com:xiezefan/blog-common.git
git add .
git commit
git push -u origin master
```

接下来我们将为blog-api独立成一个新的项目，并将blog-common设置为blog-api的子模块

```shell
mkdir blog-api
cp -rf my-blog/blog-api blog-api/api
cp my-blog/pom.xml blog-api/

cd blog-api
git submodule add -b master git@your-repository.com:xiezefan/blog-common.git
```

这时候blog-api项目下将会生成`.gitmodule`文件，该文件对本项目的Git子模块进行声明

```
// .gitmodule
submodule ["blog-common"]
        path = blog-common
        url = git@your-repository.com:xiezefan/blog-common.git
        branch = master
```

接着我们编辑根目录下的pom.xml文件，声明`blog-common`, `api` 两个子模块

```xml
// pom.xml
...

<modules>
  <module>blog-common</module>
  <module>api</module>
</modules>

...
```

最后在`api`模块中，将`blog-common`进行引入即可。

```xml
// api/pom.xml

...

<dependency>
  <groupId>me.xiezefan.blog</groupId>
  <artifactId>blog-common</artifactId>
  <version>0.0.1</version>
</dependency>

...
```

在此大功告成，此时项目的目录结构应为

```
.
├── blog-api
    ├── .gitmodule
    ├── .gitignore
    ├── api
    │   └── pom.xml
    ├── blog-common
    │   └── pom.xml
    └── pom.xml
```

我们可以执行`mvn clean package`进行打包看看拆分是否成功。 依据同样的方法，我们可以对blog-dashboard-api进行拆分。 


### 如何使用Git SubModule

因为blog-commmon本身是一个独立的项目，我们每次对其进行修改后，只需要在这模块中执行常规的git提交命令即可对模块进行更新。

而对子模块的更新也有些许不同，当开发者第一次checkout带子模块的项目的时候，需要额外执行子模块检出命令。

```shell
git clone 
git@your-repository.com:xiezefan/blog-api.git

cd blog-api
git submodule init
git submodule update --remote
```

此后，每次需要对子模块进行更新的时候，只需要执行

```shell
git submodule update --remote
```


### 一些思考

之所以研究Git Module来对Maven的项目结构进行优化，主要是因为在使用Gitlab CI进行持续集成过程中，面对一个项目中有多个模块的情况下，持续集成无法对子模块进行按需部署。

为了解决这个问题，我一开始的思路是在CI脚本中编写逻辑判断，然后使用Gitlab Triggering API触发CI脚本并传入不同参数来选择部署不同的子模块。但这个解决方案还需要有一个定制的控制台来选择触发指定模块以及指定版本。

对于创业公司永远不足的人力面前，最忌讳就是盲目造轮子。所以后来我选择的Git Module模块来解决这个问题，虽然会照成多出大量的Git Repositories的负作用，但这显然比造一个轮子来的经济实惠。










### 参考资料

* [Using Git Submodules for Maven Artifacts Not in Central](http://alex.nederlof.com/blog/2013/07/08/using-git-submodules-for-maven-artifacts-not-in-central/)
* [Git 工具 - 子模块](https://git-scm.com/book/zh/v2/Git-工具-子模块)



