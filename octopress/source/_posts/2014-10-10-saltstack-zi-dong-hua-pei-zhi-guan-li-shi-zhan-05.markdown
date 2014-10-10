---
layout: post
title: "saltstack 自动化配置管理实战 (五)"
date: 2014-10-10 10:17
comments: true
categories: Automation
description: "saltstack grains,pillar,sls模板,jinja模板,自动化部署"
keywords: saltstack,salt,自动化运维,自动化部署,grains,pillar,sls模板,jinja模板
---

##自动化部署配置文件
####/etc/resolv.conf
对所有/etc/resolv.conf进行部署,指定nameserver和search    
angel01是DNS SERVER,angel02是客户端,我们做一个让angel01的resolv.conf中nameserver为127.0.0.1,其他nameserver ip是192.168.0.210   
<!-- more -->
```
#首先设置pillar需要区分的数据
vim /mnt/config/salt/pillar/top.sls
base:
  '*':
   - base.resolv
mkdir /mnt/config/salt/pillar/base/
vim /mnt/config/salt/pillar/base/resolv.sls
{% if grains['fqdn'] == 'angel01' %}            #匹配Hostname如果是angel01则
nameserver: ['127.0.0.1','202.96.209.133']
{% else %}
nameserver: ['192.168.0.210','202.96.209.133']
{% endif %}

vim /mnt/config/salt/top.sls
base:
  '*':
  - base.resolv
mkdir -p /mnt/config/salt/base/
vim /mnt/config/salt/base/resolv.sls
/etc/resolv.conf:                                   #文件路径
  file.managed:                                     #sls函数,修改添加文件
  - source:  salt://base/jinja/resolv.conf.jinja    #jinja的文件位置
  - user: root                                      #用户属性
  - group: root                                     #用户组属性
  - mode: 644                                       #用户权限
  - backup: minion                                  #minion客户端本地备份/var/cach/salt/
  - template: jinja                                 #使用jinja模板

vim /mnt/config/salt/base/jinja/resolv.conf.jinja
search i.fdmdns.com
nameserver {{ pillar['nameserver'][0] }}
nameserver {{ pillar['nameserver'][1] }}

#使用命令测试
[root@angel01 salt]# salt '*' pillar.item nameserver
angel01.i.fdmdns.com:
    ----------
    nameserver:
        - 127.0.0.1
        - 202.96.209.133
angel02.i.fdmdns.com:
    ----------
    nameserver:
        - 192.168.0.210
        - 202.96.209.133
#pillar已经有我们属性了,然后同步写入文件内容
[root@angel01 salt]# salt '*' state.highstate -v
Executing job with jid 20141010150603492444
-------------------------------------------

angel01.i.fdmdns.com:
----------
          ID: /etc/resolv.conf
    Function: file.managed
      Result: True
     Comment: File /etc/resolv.conf updated
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
angel02.i.fdmdns.com:
----------
          ID: /etc/resolv.conf
    Function: file.managed
      Result: True
     Comment: File /etc/resolv.conf updated
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
#查看文件，如果符合则完成
```
未完待续。。。。。
