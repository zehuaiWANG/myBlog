---
layout:     post
title:      "sakai照片爬取"
subtitle:   " \"python+selenium\""
date:       2017-04-20 12:00:00
author:     "Mike"
header-img: "img/2.png"
catalog: true
tags:
    - python爬虫
---

> “Yeah It's on. ”

## 背景

最近想做一个南科大平均脸的算法，就是opencv实现性别分类、人脸识别和人脸融合，所以首先需要照片资源，目光也就放在了sakai。
首先我想用模拟登陆的方法，
![](http://i.imgur.com/kyzyMGz.png)
可以看到post datas除了username、password还有一些其他信息
这些信息随机生成，可以在`[https://cas.sustc.edu.cn/cas/login?service=http%3A%2F%2Fsakai.sustc.edu.cn%2Fportal%2Flogin](https://cas.sustc.edu.cn/cas/login?service=http%3A%2F%2Fsakai.sustc.edu.cn%2Fportal%2Flogin "登陆界面")`中找到
（chorme浏览器按F12查看Element，然后ctrl+F可以搜索）
![](http://i.imgur.com/hVVOKnz.png)
所以我写了以下的代码（其实还有其他几个版本）
![](http://i.imgur.com/9y2Oqx8.png)
![](http://i.imgur.com/YKoCLeP.png)
但是！但是！并不成功，状态码是200，却没有返回相应的登陆后的界面（不知名aoe）
之后我想跳过登陆，直接用cookies去登陆，但是！但是！还是没有成功，orz。在我怀疑人生的时候，学长跟我说，可以用selenium去做。


## selenium
Selenium是一个用于Web应用程序测试的工具。Selenium 测试直接运行在浏览器中，就像真正的用户在操作一样。支持的浏览器包括IE,Mozilla和Firefox等。其实有点像按键精灵（有点low）。
其中Firefox有支持的插件Selenium IDE
可以在菜单中附加组件里面添加
![](http://i.imgur.com/EG3YEGW.png)

![](http://i.imgur.com/q0h5rw2.png)
安装后，你可以在工具中找到它

![](http://i.imgur.com/5ADCpsY.png)

然后按正常的顺序去实现一遍
![](http://i.imgur.com/LjxISjP.png)

记住有时候会找不到xpath，很有可能是因为浏览器还没加载出来就执行命令了，你可以右键，然后wait for it
![](http://i.imgur.com/KaBLzsH.png)
等所有成功后，你可以点击options，勾选Enable experiemental features
![](http://i.imgur.com/yrx8AH9.png)

然后你就可以在Options，Format把他转化成各种形式了，我用的是python2/.../webDriver

然后打开cmd：pip install -U selenium安装selenium


## python3+selenium代码

![](http://i.imgur.com/wAGSPkv.png)
最后的代码大概是这样

这里面有两个要注意的地方，一个是frame的切换
![](http://i.imgur.com/ojixr69.png)
如果没有
![](http://i.imgur.com/UW6zhH8.png)
就会找不到xpath
另外一个注意的地方（可能是我对python的语法不熟）

![](http://i.imgur.com/8Nnzz8t.png)

跟java，c++不同，这样的写法并不能改变python中n的值（卧槽，我好天真）。正确的姿势参考我上面的代码。其中的time.sleep时间可以自己测试找下最佳时间。


如果你恰好逛到了这里，希望你也能喜欢这个博客主题。

如果你也喜欢我的项目，欢迎star和follow我，感激不尽。

—— Mike 后记于 2017.4.20
