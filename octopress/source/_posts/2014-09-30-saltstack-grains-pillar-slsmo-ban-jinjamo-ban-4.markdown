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
minion上操作   
```
vim /etc/salt/minion
default_include: /mnt/config/salt/minion.d/*.conf #增加这项
mkdir -p /mnt/config/salt/minion.d/
vim /mnt/config/salt/minion.d/test.conf
grains:
  leon and leon: 1
  leon and alex: print 2
  cpis:
    - a
    - b
  leon.com:
    -------------------------------------------------

/etc/init.d/salt-minion restart


[root@angel01 ~]# salt -E 'angel01' grains.item leon\ and\ leon
angel01.i.fdmdns.com:
  leon and leon: 1
[root@angel01 ~]# salt -E 'angel01' grains.item leon.com
angel01.i.fdmdns.com:
  leon.com: -------------------------------------------------
[root@angel01 ~]# salt -E 'angel01' grains.item cpis
angel01.i.fdmdns.com:
  cpis:
      a
      b

```

####pillar使用
pillar是用于给特定的minion定义任何你需要的数据,数据可以被salt其他组件使用,Pillar 数据是与特定 minion 关联的，也就是说每一个minion 都只能看到自己的数据， 所以 Pillar 可以用来传递敏感数据。    
pillar可以通过变量的方式再不同的操作系统或者特定的服务server上进行特别的设置。   
```
#首先定义pillar的位置
vim /mnt/config/salt/master
interface: 192.168.0.210
user: root
auto_accept: True
worker_threads: 5
file_roots:
  base:
    - /mnt/config/salt/
pillar_roots:                   #标识pillar目录
  base:
    - /mnt/config/salt/pillar
```

在pilla目录中新建一个top.sls,类似Puppet的入口文件   

```
vim /mnt/config/salt/pillar/top.sls
base:       #自定义名称
  '*':      #客户端过滤,可以是angel0[1-2].*,等等
   - test   #模板名称

vim /mnt/config/salt/pillar/test.sls
MY_NAME: leon
ID: 7758258
CONTENT:  lol

/etc/init.d/salt-master restart
/etc/init.d/salt-minion restart

[root@angel01 pillar]# salt  'angel01.i.fdmdns.com' pillar.data
angel01.i.fdmdns.com:
    ----------
    CONTENT:
        lol
    ID:
        7758258
    MY_NAME:
        leon
    #这边有master默认的一些信息省略了
```

下面结合state.sls和jinja做一个写文件的动作   

```bash
vim /mnt/config/salt/top.sls
base:
  '*':
   - test.tmp
mkdir -p /mnt/config/salt/test/
vim /mnt/config/salt/test/tmp.sls
/tmp/tmp.conf:                              #文件位置
  file.managed:                             #采用新增文件的方式
  - source:  salt://test/tmp.conf.jinja     #jinja资源位置
  - template: jinja                         #模板

vim /mnt/config/salt/test/tmp.conf.jinja
visible hostname  \{{ grains['fqdn'] }}      #通过grains获得的fqdn(hostname)
\{{ pillar['MY_NAME'] }}                     #前面我用pillar定义的MY_NAME
\{{ pillar['ID'] }}                          #注释的\号自己去掉,双括号情况下在makedown不显示= =！
server_os \{{ grains['os'] }}

1.salt  '*' state.highstate -v              #方式1
2.salt  'angel01.i.fdmdns.com' state.sls test.tmp  #方式2
angel01.i.fdmdns.com:
----------
          ID: /tmp/tmp.conf
    Function: file.managed
      Result: True
     Comment: File /tmp/tmp.conf updated
     Changes:   
              ----------
              diff:
                  New file
              mode:
                  0644

Summary
------------
Succeeded: 1
Failed:    0
------------
Total:     1
#表示完成

[root@angel01 salt]# cat /tmp/tmp.conf 
visible hostname  angel01
leon
7758258
server_os CentOS
```
