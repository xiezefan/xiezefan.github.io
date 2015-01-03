---
layout: post
title: "Ubuntu快速安装Sublime-Text"
date: 2014-10-05
categories:
- tips
tags:
- sublime-text
---


### 通过添加PPA安装
```
sudo add-apt-repository ppa:webupd8team/sublime-text-2
sudo apt-get update
sudo apt-get install sublime-text-2
```

<!-- more -->

### 安装Soda主题
安装 Package Control, 按 `Ctrl+~`进入控制台，粘贴以下代码
> import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'

preferences -> package control -> 输入install package
在此打开控制台，输入Soda，选择 Theme-Soda
Preferences -> Settings – User中修改参数
```
{
    "ignored_packages":
    [
        "Vintage"
    ],
    "theme": "Soda Dark.sublime-theme",
    "font_size": 12,
    "font_face": "YaHei Consolas Hybrid"
}
```
重启Sublime Text


