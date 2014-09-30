---
layout: post
title: "saltstack grains,pillar,sls模板,jinja模板 (四)"
date: 2014-09-30 16:16
comments: true
categories: Automation
description: "saltstack grains,pillar,sls模板,jinja模板"
keywords: saltstack,salt,自动化运维,自动化部署,grains,pillar,sls模板,jinja模板
---

##grains
grains类似puppet的facter一样 负责采集客户端一些基本信息,也可以自定义   

####grains使用
```
salt -E 'angel01' grains.ls   #查看有多少定义好的属性可以用
salt -E 'angel01' grains.items #查看所有属性返回的值
salt -E 'angel01' grains.item fqdn os #查看fqdn os 返回的值,以此类推
```

####grains自定义
#####服务端主动推送   
master上操作   
```
vim /mnt/config/salt/master
增加默认目录
file_roots:
  base:
      - /mnt/config/salt/

mkdir -p /mnt/config/salt/_grains/
vim /mnt/config/salt/_grains/test.py
# -*- coding: utf-8 -*-

'''
leon test
'''
import commands
import os

def l_test():
    '''
    leon test
    '''
    grains={}
    a = commands.getoutput("hostname")
    grains['leon_test']=a

    return grains
```
使用命令`saltutil.sync_all`进行推送    
<!-- more -->
再通过salt '*' grains.item leon_test就可以查看了    
```
[root@angel01 salt]# salt '*' saltutil.sync_all
angel01.i.fdmdns.com:
    ----------
    grains:
        - grains.test
    modules:
    outputters:
    renderers:
    returners:
    states:
angel02.i.fdmdns.com:
    ----------
    grains:
        - grains.test
    modules:
    outputters:
    renderers:
    returners:
    states:

[root@angel01 salt]# salt '*' grains.item leon_test
angel01.i.fdmdns.com:
  leon_test: angel01
angel02.i.fdmdns.com:
  leon_test: angel02
```
模块进行更改后需要使用`sys.reload_modules`
```
[root@angel01 salt]# salt '*' sys.reload_modules
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
[root@angel01 salt]# salt '*' saltutil.sync_all 
angel01.i.fdmdns.com:
    ----------
    grains:
        - grains.test
    modules:
    outputters:
    renderers:
    returners:
    states:
angel02.i.fdmdns.com:
    ----------
    grains:
        - grains.test
    modules:
    outputters:
    renderers:
    returners:
    states:
[root@angel01 salt]# salt '*' grains.item leon_test
angel01.i.fdmdns.com:
  leon_test: this is test
angel02.i.fdmdns.com:
  leon_test: this is test
```

####客户端自动汇报





