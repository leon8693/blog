---
layout: post
title: "zabbix 2.4.1 汉化界面和图 (二)"
date: 2014-10-17 16:52
comments: true
categories: Monitor
description: "zabbix,2.4.1,监控,monitor,汉化,中文,图"
keywords: zabbix,2.4.1,监控,monitor,汉化,中文,图
---

##界面修改中文
```
vim /mnt/www/zabbix/web/include/locales.inc.php 
'zh_CN' => array('name' => _('Chinese (zh_CN)'),        'display' => false),
修改成
'zh_CN' => array('name' => _('Chinese (zh_CN)'),        'display' => true),
```


##图画中文显示
在没有处理的情况下,图片中的文字是乱码,如下   
![zabbixcn01](/images/blog_img/zabbixcn01.png)

<!-- more -->

随后我们需要改变他,首先复制windows中的字库   
这个是我虚拟机中的XP字库,其他版本windows路径都一样如下  
控制面板->字体->楷体(我复制的这个)   
![zabbixcn02](/images/blog_img/zabbixcn02.png)

复制出来的名字如图  
![zabbixcn03](/images/blog_img/zabbixcn03.png)

我们将他复制到下面这个目录,并且修改配置  
```
cp -rp simkai.ttf /mnt/www/zabbix/web/fonts/

vim /mnt/www/zabbix/web/include/defines.inc.php
//define('ZBX_GRAPH_FONT_NAME',         'DejaVuSans'); // font file name
define('ZBX_GRAPH_FONT_NAME',           'simkai'); // font file name
#保存退出

```
刷新页面查看  
![zabbixcn04](/images/blog_img/zabbixcn04.png)


OK,大功告成!   
