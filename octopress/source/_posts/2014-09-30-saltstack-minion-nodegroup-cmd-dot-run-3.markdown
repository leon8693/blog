---
layout: post
title: "saltstack minion,nodegroup,cmd.run (三) "
date: 2014-09-30 10:31
comments: true
categories: Automation
description: "saltstack minion,nodegroup,cmd.run"
keywords: saltstack,salt,自动化运维,自动化部署,minion,nodegroup,cmd.run
---


##minion
客户端标识默认采用主机名,也可以修改配置中的id来指定minion的名字      
vim /mnt/config/salt/minion   
```
master: angel01.i.fdmdns.com
user: root
id: angel01.i.fdmdns.com #指定了这个客户端的名字
```

查看minion状态,并且可以使用正则   
PS:必须在MASTER上操作   
```
[root@angel01 ~]# salt  'angel01.i.fdmdns.com'  test.ping
angel01.i.fdmdns.com:
    True
[root@angel01 ~]# salt  '*'  test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
[root@angel01 ~]# salt 'angel0[1-2].i.fdmdns.com'  test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
[root@angel01 ~]# salt 'angel0*'  test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
```

<!-- more -->

上面展示了匹配的方式  
test.ping 是salt默认命令,主要测试客户端存活情况   

另2个参数常用   
-E来使用正则表达式   
-L以列表的方式匹配   
```
[root@angel01 ~]# salt -E 'angel0\w'  test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
[root@angel01 ~]# salt -L 'angel01.i.fdmdns.com,angel02.i.fdmdns.com'  test.ping
angel01.i.fdmdns.com:
    True
angel02.i.fdmdns.com:
    True
```
主要的一些匹配演示如上   

##cmd.run
这个是可以批量执行shell命令,可以代替我们以前复杂的for的  
```
[root@angel01 ~]# salt -L 'angel01.i.fdmdns.com,angel02.i.fdmdns.com'  cmd.run "free -m|awk '/Mem:/{print \$2}'"
angel01.i.fdmdns.com:
    3702
angel02.i.fdmdns.com:
    1869
```

##nodegroup
nodegroup可以对我们现有的服务器进行分组,默认配置文件放在/etc/salt/master   
这边为了更好的修改配置和区分,使用了default_include: /mnt/config/salt/master.d/*.conf   
```
cat /etc/salt/master
include: /mnt/config/salt/master
default_include: /mnt/config/salt/master.d/*.conf
mkdir -p /mnt/config/salt/master.d/
vim /mnt/config/salt/master.d/nodegroups.conf 
#group#
nodegroups:
  web1:  'angel01.i.fdmdns.com' #测试机1为单独一组
  web2:  'angel02.i.fdmdns.com' #测试机2为单独一租
/etc/init.d/salt-master restart

[root@angel01 ~]# salt -N 'web1' test.ping
angel01.i.fdmdns.com:
    True
[root@angel01 ~]# salt -N 'web2' test.ping
angel02.i.fdmdns.com:
    True
```
