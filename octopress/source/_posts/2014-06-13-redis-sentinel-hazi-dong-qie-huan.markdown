---
layout: post
title: "Redis Sentinel HA自动切换"
date: 2014-06-13 17:11
comments: true
categories: database
description: "Redis Sentinel HA自动切换"
tags: [Redis Sentinel HA自动切换]
keywords: linux,python,OptionParser
---


##Redis Sentinel  HA自动切换

###redis 2.8.11 下载
http://redis.io/download    
http://download.redis.io/releases/redis-2.8.11.tar.gz     

```
master 192.168.0.210
slave 192.168.0.211
slave 192.168.0.213

#安装redis 2.8.11
cd /srv/
tar xf redis-2.8.11.tar.gz
cd redis-2.8.11
make && make install 

#复制启动文件
cp -rp /srv/redis-2.8.11/utils/redis_init_script  /etc/init.d/redis
chmod +x /etc/init.d/redis


#master 配置文件
/etc/redis/redis.conf
####
daemonize yes
pidfile /var/run/redis/redis.pid
port 6379
bind 0.0.0.0
timeout 0
loglevel notice
logfile /var/log/redis/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /var/lib/redis/
slave-serve-stale-data yes
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
slowlog-log-slower-than 10000
slowlog-max-len 1024
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
activerehashing yes
requirepass angel01
slave-read-only yes
maxmemory 1G
########


#slave配置
########
daemonize yes
pidfile /var/run/redis/redis.pid
port 6379
bind 0.0.0.0
timeout 0
loglevel notice
logfile /var/log/redis/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
rdbcompression yes
dbfilename dump.rdb
dir /var/lib/redis/
slave-serve-stale-data yes
appendonly yes
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
slowlog-log-slower-than 10000
slowlog-max-len 1024
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
activerehashing yes
slave-serve-stale-data yes
slaveof 192.168.0.210 6379
masterauth angel01
slave-read-only yes
maxmemory 1G
#############

```
<!-- more -->


可以通过以下2种方式查看是否主从  
方案1  
```
redis-cli -a angel01 -h 192.168.0.210 info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.0.211,port=6379,state=online,offset=21327,lag=1
slave1:ip=192.168.0.213,port=6379,state=online,offset=21327,lag=1
master_repl_offset:21327
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:21326
```

方案2  
```
master 上操作
redis-cli -a angel01
set leon good
slave 上操作
redis-cli -a angel01
get leon
"good"
即可成功
```

`slave-read-only yes` 从只有只读权限  
`maxmemory 1G`        最大使用内存1G  
`appendonly yes`      slave 上，开启aof备份，路径在dir /var/lib/redis/下面  

这边需要注意的是，AOF和RDB的备份，在主上只开启RDB，在SLAVE上2个都开启，但在故障转移的时候slave升成master时，压力会比原来的master大，那是因为RDB和AOF是不会自动转移的。
到此说明redis正常工作了，接下来开始使用哨兵sentinel进行监控以及主从切换

如下操作  
```
cp -rp /srv/redis-2.8.11/sentinel.conf /etc/redis/
vim /etc/redis/sentinel.conf
port 26379
dir /var/lib/redis
sentinel monitor mymaster 192.168.0.210 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel auth-pass mymaster angel01
/etc/init.d/redis-sentinel /etc/sentinel.conf --sentinel  #启动命令
```


###故障演示
3台机器都启动查看  
redis-cli -h 192.168.0.210 -p 26379 info Sentinel 可以查看信息  
关闭slave  
没有任何响应  

关闭master  
![stop_master](/images/blog_img/redis1.png)
可以看到213接替了210成了master  
到此完成切换  

