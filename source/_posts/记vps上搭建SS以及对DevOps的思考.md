---
title: 记vps上搭建SS以及对DevOps的思考
date: 2016-07-19 17:58:56
tags: 运维 博客
---

![jj](http://img4.imgtn.bdimg.com/it/u=4067514976,2212248928&fm=15&gp=0.jpg)


由于国内的阿里云ECS服务器到期，续费价格又过于高昂，一个月花几百块在一台国内的流量很有限的服务器上怎么算都划不来。同样的价格买国外的VPS划算的多，乘着搬瓦工20刀年费的vps又开放购买了就入了一个。本文主要记录一下，在[bandwagonhost](https://bandwagonhost.com)上搭建SS的注意事项。

主要针对这家kiwiVM构架的VPS来说，需要注意这三点

1. [远程SSH登陆是肯定需要的，但搬瓦工的VPS默认是禁止root用户登录的](http://www.itbulu.com/linux-root-ssh.html)

2. [配置nginx和反向代理](http://www.jianshu.com/p/605c3d32cab9)

3. [开启防火墙http和https的通过许可](https://www.zybuluo.com/phper/note/90310)

> 各种安装shadowsocks服务端代理的教程网上都能找到，但是遇到这些问题的时候还是需要自己去解决的。如果对Linux的命令和常见的网站构架熟悉的话，通过过curl本地端口，查看ssh和nginx的配置文件都能知道。
> 对`systemctl`和`firewall-cmd`等命令不熟悉也很正常，必须google一下。


### 感想

`systemctl`和`firewall-cmd`这些命令还是运维工程师更加了解，但这不意味着Dev就可以对此不闻不问了。所以Dev和OP们如果要实践敏捷开发或者敏捷运维，就应该往DevOps这个概念上靠。运维工程师也需要转变思路，主动寻求解决办法。
开发也要了解运维的关注点、知识点和需求，所谓的DevOps是为这个而产生的，两者相辅相成。两者的界限随着云计算的普及更加模糊了。在云的平台上，服务资源和网络资源、甚至负载的分配都有相应的服务化的接口提供，这实际上是自动化运维的一个很好的基础。
归根结底，这个领域的各种变革发展都是为了`效率`两字。


[最后分享一篇漫画](http://blog.xiqiao.info/blogimg/programmers/56_Almighty_language.gif)

题图：Docker的可爱鲸鱼。
