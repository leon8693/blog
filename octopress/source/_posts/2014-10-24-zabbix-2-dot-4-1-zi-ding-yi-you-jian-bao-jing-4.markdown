---
layout: post
title: "zabbix 2.4.1 自定义邮件报警 (四)"
date: 2014-10-24 16:31
comments: true
categories: Monitor
description: "zabbix,2.4.1,监控,monitor,邮件报警"
---


##自定义邮件报警
前面几步完成后,发现报警机制没有,本章开始说报警,这边我取一个最简单的方式而且又方便  

####创建报警邮件脚本
下载[SendEmail](http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.55.tar.gz)
```
#将sendEmail复制到/usr/local/bin/下
#编写邮件脚本
vim /mnt/www/zabbix/share/zabbix/alertscripts/sendEmail.sh
#!/bin/bash
MAIL="/usr/local/bin/sendEmail"
USER_LIST="$1"          #收件人列表
SUBJECT="$2"            #主题
MESSAGE="$3"            #内容
SMTP="smtp.gmail.com"
USERNAME="aaa@gmail.com"
USERPASSWD="123456"
$MAIL -f aaa@gmail.com -t $USER_LIST -u $SUBJECT -m $MESSAGE -s $SMTP -xu $USERNAME -xp $USERPASSWD
```
<!-- more -->

####添加脚本邮件
选择管理-->示警媒介类型-->创建媒体类型-->如图将脚本名称加入类型选择脚本   
![zabbixmail01](/images/blog_img/zabbixmail01.png)
![zabbixmail02](/images/blog_img/zabbixmail02.png)


####添加用户邮件设置
设置用户关联,管理-->用户   
![zabbixmail03](/images/blog_img/zabbixmail03.png)
选择使用的用户并且编辑设置   
![zabbixmail04](/images/blog_img/zabbixmail04.png)
![zabbixmail05](/images/blog_img/zabbixmail05.png)
![zabbixmail06](/images/blog_img/zabbixmail06.png)

####报警动作
组态-->动作--创建动作(根据图片进行设置)   
![zabbixmail07](/images/blog_img/zabbixmail07.png)
![zabbixmail08](/images/blog_img/zabbixmail08.png)

PS:900指邮件发送的间隔,1-0指发送邮件的步骤,我这边设置的是无限次,并且自己添加需要发送的用户组或者用户   
![zabbixmail09](/images/blog_img/zabbixmail09.png)

####测试
PS:动作1黄色代表正在操作中,当变成绿色则代表发送成功   
![zabbixmail10](/images/blog_img/zabbixmail10.png)
![zabbixmail11](/images/blog_img/zabbixmail11.png)


