---
layout: post
title: "生成力工具【瀑布IM】正确使用手册"
date: 2015-10-24
categories:
- tools
tags:
- tools

---


去年的这个时候，国外的企业协作工具[Slack](https://slack.com/)的报告铺天盖地，如今一年过去，Slack已经估值30亿美金，成了新的明星独角兽公司。  
    
不过Slack毕竟作为一款境外的软件，网络访问以及语言障碍在团队内推广还是挺成问题的。而近一年来，国内也涌现出一批类Slack的国产应用。如[瀑布IM](https://beta.pubu.im/)，[纷云](https://lesschat.com/)，[Bearychat](https://bearychat.com)等。


<!-- more -->


先简单介绍下我们团队的现状，目前我们使用Gitlab + Tower.im + 瀑布IM，团队成员提交Pull Request后，需要其他成员Review才能合并进Master分支。而项目正处于前期开发阶段，对BUG的收集整理并不需要像生产环境的项目一样走一个复杂的流程，所以我们使用Tower.im来进行项目任务管理与BUG追踪。 

![Tower.im](http://res.xiezefan.me/images/blog/tower_screen.png)  

无意中在Tower.im的主页上看到瀑布IM的介绍，试用了下效果非常不错，推广到整个开发小组，组内成员普遍好评，简单地设置了WebHooks就接入了Gitlab与Tower.im， 小组成员提交了代码，创建Pull Request，在Tower上创建/讨论/完成任务，第一时间在瀑布IM上就可以收到消息，这在团队密切协作的时候特别有用，你能快速地了解到，你的队友在做什么。所有地碎片化消息都统一地在一个平台内展示。

![Pubu.im Screen](http://res.xiezefan.me/images/blog/pubu_im_screen.png?t=1)

在以往，我们用QQ群，QQ讨论组做团队沟通，有时候很无奈，QQ上掺杂着很多私人的关系在上面，工作时候的确会有一些干扰，瀑布IM的组内讨论，体验上虽然有些许比不上QQ，但作为工作讨论可以屏蔽掉很多干扰。并且我很喜欢其中的一个功能，瀑布IM会收集团队聊天中的图片，文件，代码片段，特别是链接片段，这无疑是一个需求痛点，工作中，我经常翻好几页聊天记录去找同事N天前发给我的测试页面的链接。

![Pubu.im Files Collection](http://res.xiezefan.me/images/blog/pubu_im_files_collection.jpg)


### 自定义扩展

瀑布IM也提供了自定义扩展，方便有开发能力的用户接入内部的管理系统，其主要有

输出类：

* Command命令
* Outgoing


接入类：

* 代收邮件
* Incomming


输出类定义一个聊天事件，当聊天事件触发的时候，会往你指定的URL POST一些数据，接入类则通过设置Webhooks，当事件触发时，开发者主动调用Webhoods发送数据向指定渠道推送消息。 

我试验了一下[Incoming服务](https://blog.pubu.im/how-to-add-incoming/)，背景是这样，当一个用户支付了订单时候，服务端会进行一些Webhookds回调，正常情况下，我需要编写一个程序以查看回调通知的内容，在此之前，我使用纷云的开源Webhooks调试框架 - [Request](http://request.lesschat.com/)，为了试验瀑布IM的Incoming服务，我用Nodejs编写了一个简单的转发Webhooks的程序，当收到回调后，程序格式化文本并调用瀑布IM的回调接口。

```javascript
request({
    uri: 'https://hooks.pubu.im/services/xxxxxxx',
    method: 'POST',
    json: {text:text, attachments:attachments, displayUser:displayUser},
    headers : {'Content-Type' : 'application/json'}
}, function(err, rep) {
    if (err) {
        return console.log(err);
    }
    console.log(JSON.stringify(rep.body));
});
```

![Pubu.im Custom Service](http://res.xiezefan.me/images/blog/pubu_im_custom_service.png)  


有一点需要吐槽的是，瀑布IM提供的API有一个BUG，当未指定`Content-Type=application/json`的时候，API会抛500异常，回复邮件反馈也不是很迅速，最后还是我自己找到原因的。不过考虑到目前该产品仍然处于Beta版本，还是可以理解的。另外，在JSON中使用驼峰式命名是很不专业的行为呀。


### 额外的话

昨天和一位跳槽某大型互联网公司的前同事聊天，他同我吐槽说公司对第三方生成力工具很克制，禁止员工使用其他第三方平台提供管理工具，理由不外乎信息安全，而一般大型互联网公司的内部工具难免跟不上时代，效率上总有些瑕疵。像Tower.im，Teambition，瀑布IM这些工具想在内部推广阻力重重，而反倒乎创业团队，无须考虑太多这些事情，执行力也高，能第一时间体验这些提供工作效率的生产力工具。只不过在国内的大环境下，如何探索出一套可行的商业模式当下还是路漫漫。





