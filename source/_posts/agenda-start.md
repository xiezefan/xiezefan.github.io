---
layout: post
title: "Agenda笔记"
date: 2014-12-29
categories:
- nodejs
tags:
- agenda

---

[Agenda][1]是类似Quarzt的轻量级持久化任务调度框架, 数据存储再mongodb上, 部署与学习十分简单.
第三方开发者还为Agenda提供了一个简易的UI界面 [Agenda UI][2]. 提供简单的可视化界面.

<!-- more -->

### Base
Agenda 将任务对象保存再mongo里面, 任务对象包含任务的name, 附带数据, 定时规则等.
我们可以动态对任务进行CRUD操作. Agenda提供对应API.

虽然Agenda本身提供的对Job的操作比较简单, 但因为Agenda的Job都是存储再mongo之中,所以我们可以通过直接操作monogo实现的Job的更新与删除.

### Start
简单的添加任务操作:
```
var job = agenda.create('testJob', {key: "value"});
job.repeatEvery('5 seconds');
job.save(function(err) {
    console.log("Job successfully saved");
});
```
程序开始时,我们需要创建agenda对象,指定事件响应函数,并开始执行任务调用.

```
var agenda = new Agenda({db: { address: 'localhost:27017/agenda-example'});
agenda.define('testJob',  function(job) {
    console.log("Job execute, now is " + new Date()  + " Data=" + JSON.stringify(job.attrs.data));
});
agenda.run();
```


#### 数据存储
Agenda使用mongodb。

```
// 指定使用的mongodb，默认使用agendaJobs集合
var agenda = new Agenda({db: { address: 'localhost:27017/agenda-example'}});

// 如果需要指定其他集合
var agenda = new Agenda({db: { address: 'localhost:27017/agenda-test', collection: 'agendaJobs' }});
```

#### 指定循环规制

```
// 直接构造的agenda对象的时候指定
var agenda = new Agenda({processEvery: '1 minute'});
// 1分钟循环一次
agenda.processEvery('1 minute');

```

#### 指定任务最大并发数

```
// 构造时指定
var agenda = new Agenda({maxConcurrency: 20});
// 直接指定
agenda.maxConcurrency(20);
```

#### 指定默认并发数

```
// 构造时指定
agenda.defaultConcurrency(5);
// 直接指定
var agenda = new Agenda({defaultConcurrency: 5});
```

#### 设置任务最大响应时间
指定任务从开始执行到调用finish的时间（指调用done函数），这能有效的解决任务奔溃或者超时


```
// 构造时指定
var agenda = new Agenda({defaultLockLifetime: 10000});
// 直接指定
agenda.defaultLockLifetime(10000);
```


### 定义任务处理器

#### 定义任务行为
当执行的任务为异步方法的时候，需要再方法最后调用done()
如果为同步方法，则省略声明done

影响任务行为的参数：

* concurrency 指定最大任务并发数
* lockLifetime 锁生存时间,指定异步调用时最长等待时间
* priority 优先级(lowest|low|normal|high|highest|number)


```
// 执行任务为异步方法
agenda.define('job_name', function(job, done) {
    doSomething(function() {
        done();
    });
});

// 执行任务为同步方法
agenda.define('say hello', function(job) {
  console.log("Hello!");
});
```

#### 定时任务

指定任务重复的规制,支持数字, cron表达式, 甚至如10 minutes等可读性较高的表达式

```
job.repeatEvery('10 minutes');
job.repeatEvery('*/10 * * * * * *');
```

指定特定的重复时间

```
// 每天15:30
job.repeatAt('3:30pm');
```
指定只执行一次的的任务

```
job.schedule('3:30pm');
```

now
马上执行某事件
```
agenda.now('do the hokey pokey');
```


  [1]: https://github.com/rschmukler/agenda
  [2]: https://github.com/moudy/agenda-ui
