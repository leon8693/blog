---
layout: post
title: "saltstack install (一)"
date: 2014-09-29 16:14
comments: true
categories: Automation
description: "saltstack install"
keywords: saltstack,install,salt,自动化运维,自动化部署
---


#下载安装
#####Centos  
epel源或者centos源都有   
```
yum install salt-master -y #服务端
yum install salt-minion -y #客户端
```

#####Ubuntu  
```
sudo apt-get install python-software-properties
echo deb http://ppa.launchpad.net/saltstack/salt/ubuntu `lsb_release -sc` main | sudo tee   
/etc/apt/sources.list.d/saltstack.list
wget -q -O- "http://keyserver.ubuntu.com:11371/pks/lookup?op=get&amp;search=0x4759FA960E27C0A6" |sudo apt-key add -
sudo apt-get update
sudo apt-get install salt-master
sudo apt-get install salt-minion
```


由于我配置文件目录更改,所以有些增加的选项都是非必要的,参考即可  


###master配置
vim /etc/salt/master  配置文件
```
include: /mnt/config/salt/master
default_include: /mnt/config/salt/master.d/*.conf
mkdir -p /mnt/config/salt/ /mnt/config/salt/master.d/
```
<!-- more -->

vim /mnt/config/salt/master #主配置文件
```
interface: 192.168.0.210
user: root
auto_accept: True     #自动认证
worker_threads: 5     #工作线程，默认5，根据cpu调整
```
PS: /mnt/config/salt/master.d/ 这个目录会在下篇里面说到

/etc/init.d/salt-master start #启动 

```
#表示启动成功
[root@angel01 ~]# netstat -lntp |grep -E "450[5|6]"
tcp        0      0 192.168.0.210:4505          0.0.0.0:*                   LISTEN      20004/python2.6    
tcp        0      0 192.168.0.210:4506          0.0.0.0:*                   LISTEN      19996/python2.6   
```

###minion配置
vim /etc/salt/minion 配置文件
```
include: /mnt/config/salt/minion
```

vim /mnt/config/salt/minion
```
master: angel01.i.fdmdns.com    #master地址
user: root                      #启动用户
id: angel01.i.fdmdns.com        #这个就是服务器用来标示客户端的东西
```
/etc/init.d/salt-minion start #启动

第一次启动minion,minion会生成钥匙   
```
[root@angel01 /]# tree /etc/salt/pki/minion/
/etc/salt/pki/minion/
├── minion_master.pub
├── minion.pem
└── minion.pub
```

###手动认证
```
[root@angel01 ~]# salt-key
Accepted Keys:             #验证通过
Unaccepted Keys:          #等待验证的
angel01.i.fdmdns.com
Rejected Keys:               #拒绝

[root@angel01 ~]# salt-key -A
接受所有key
```

###salt-key的基本命令
```
salt-key -L #检测当前server端所有minion端key的情况，三种：接收、等待接收和拒绝
salt-key -a hostname #指定接收某台minion的key
salt-key -A #接收Unaccepted Keys下所有的minion
salt-key -d hostname #删除已经接收的机器中指定机器minion key (Accepted Keys:)
salt-key -D #删除已经接收的所有机器(Accepted Keys:)
```

###测试
```
[root@angel01 ~]# salt '*' test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
```

表示成功了！   



###问题集合
问题1   
ImportError: No module named salt.scripts   
yum 的情况下salt的模块都安装在python2.6 进入/etc/init.d/salte-master进行修改   
```
批量修改命令
sed -i "s/^#\!\/usr\/bin\/python$/#\!\/usr\/bin\/python2.6/g" /usr/bin/salt-*
批量修改启动命令
sed -i "s/^PYTHON=\/usr\/bin\/python$/PYTHON=\/usr\/bin\/python2.6/g" /etc/init.d/salt-*
```
问题2   
[root@angel01 ~]# salt  'angel01.i.fdmdns.com' test.ping    
[ERROR   ] Salt request timed out. If this error persists, worker_threads may need to be increased.   
Failed to authenticate, is this user permitted to execute commands?    
由于修改过启动配置文件直接读取/mnt/config/salt/导致,最后改成/etc/salt/master 添加include解决   
