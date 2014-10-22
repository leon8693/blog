---
layout: post
title: "zabbix 2.4.1 自动发现 (三)"
date: 2014-10-22 16:22
comments: true
categories: Monitor
description: "zabbix,2.4.1,监控,monitor,自动发现"
keywords: zabbix,2.4.1,监控,monitor,自动发现
---

zabbix有个很有用的特性,就是自动发现,可以自动找已经启用zabbix agent的服务器并且对这些服务器进行添加监控以及分组等工作   
##自动发现
首先添加发现规则   
点击--组态--探索  
![zabbixat01](/images/blog_img/zabbixat01.png)


<!-- more -->
点击右侧的创建发现规则  
![zabbixat02](/images/blog_img/zabbixat02.png)

根据图片进行匹配IP范围,并且设定通过zabbix代理进行验证  
![zabbixat03](/images/blog_img/zabbixat03.png)

增加发现的主机后的操作   
![zabbixat04](/images/blog_img/zabbixat04.png)

事件源选择探索-->点击创建动作-->取名称-->选择匹配范围-->选择动作操作   
![zabbixat05](/images/blog_img/zabbixat05.png)
![zabbixat06](/images/blog_img/zabbixat06.png)
![zabbixat07](/images/blog_img/zabbixat07.png)

点击检测中查看最下面探索状态  
![zabbixat07](/images/blog_img/zabbixat07.png)

看到如图所示说明已经成功！  
PS:网卡流量和磁盘空间需要等一段时间agent会推送给server  
![zabbixat08](/images/blog_img/zabbixat08.png)
![zabbixat09](/images/blog_img/zabbixat09.png)
![zabbixat10](/images/blog_img/zabbixat10.png)
