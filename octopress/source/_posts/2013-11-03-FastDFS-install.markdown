---
layout: post
title: "FastDFS install"
date: 2013-11-03 16:11
comments: true
categories: FastDFS
description: "FastDFS install"
tags: [linux,Fastdfs,install,安装]
keywords: linux,Fastdfs,install,安装 
---

```bash
#信息版本     
Centos 2.6.32-358.el6.x86_64
FastDFS 4.06
Nginx 1.4.3


#官网下载地址
https://fastdfs.googlecode.com/files/FastDFS_v4.06.tar.gz

#测试服务器
test1 172.16.3.212     (tracker,storage)
test2 172.16.3.213     (tracker,storage)
```


###安装FastDFS

```bash
#安装需要的包
yum groupinstall -y "Development Tools"
yum install -y libevent-devel pcre-devel zlib-devel
```

<!-- more -->

####安装FastDFS服务
####由于4.06版本里面默认将内置的web server取消，即直接执行make

```
cd FastDFS
./make.sh && ./make.sh install
```

make.sh 安装前可修改的一些参数  
WITH_LINUX_SERVICE=1     #注释取消表示使用内置web server  
TARGET_PREFIX=/usr/local    #安装的目录  
TARGET_CONF_PATH=/etc/fdfs #配置文件目录  



####DFS配置文件位置

```bash
/etc/fdfs/  
├── client.conf     #client连接的配置文件  
├── http.conf       #tracker,storage内置web server的配置文件  
├── mime.types   #文件类型配置文件  
├── storage.conf  #storage配置文件  
└── tracker.conf   #tracker配置文件  
```

####放置启动tracker和storage的脚本

```
/srv/FastDFS/init.d/
├── fdfs_storaged
└── fdfs_trackerd
cp -rp /srv/FastDFS/init.d/* /etc/init.d/
```


####tracker配置
#####创建base_path目录（会生成data,logs）

```
vim /etc/fdfs/tracker.conf 
base_path=/etc/fdfs/
```

####storage_ids.conf配置解释

######/etc/fdfs/tracker.conf这个参数打开storage_ids_filename = storage_ids.conf ，默认打开4.0后的新功能
#####storage_ids.conf配置如下

```
# <id>  <group_name>  <ip_or_hostname>
100001   g1  172.16.3.212
100002   g1  172.16.3.213
```



####启动tracker

```
/etc/init.d/fdfs_trackerd start
```


####检查是否开启

```
[root@test1 fdfs]# netstat -lntp |grep fdfs
tcp        0      0 0.0.0.0:22122               0.0.0.0:*                   LISTEN      5236/fdfs_trackerd
```


####storage配置
#####安装nginx取代内置web server
#####下载nginx,fastdfs-nginx-module

```
wget https://fastdfs.googlecode.com/files/fastdfs-nginx-module_v1.15.tar.gz
wget http://nginx.org/download/nginx-1.4.3.tar.gz
```
```
cd nginx-1.4.3
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-file-aio --with-ipv6 --with-cc-opt='-O2 -g -pipe -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' --add-module=/srv/fastdfs-nginx-module/src/

make && make install
```

#####/srv/fastdfs-nginx-module/src/config 中也可以更改读取mod_fastdfs.conf位置然后再编译


#####storage配置修改

```
vim /etc/fdfs/storage.conf
base_path=/etc/fdfs/             #放置data和log的位置
store_path0=/data/images01/     #放置数据的位置，如果新增盘，可以增加store_path1
tracker_server=172.16.3.212:22122   #告知tracker server 的位置
tracker_server=172.16.3.213:22122   #有多个tracker就写多行
group_name=g1 #此台storage1所属的服务器组名，同组内storage数据完全相同
mkdir -p /data/images01/ #创建存储数据的目录
```

####启动storage
```
/etc/init.d/fdfs_storaged start
```

#####下面会生成许多2级目录就此结束


####查看是否启动

```
[root@test1 /]# netstat -lntp|grep fdfs
tcp        0      0 0.0.0.0:23000               0.0.0.0:*                   LISTEN      8209/fdfs_storaged 
tcp        0      0 0.0.0.0:22122               0.0.0.0:*                   LISTEN      5236/fdfs_trackerd
```

####mod_fastdfs.conf配置
#####nginx使用了mod_fastdfs模块，这个模块配置直接影响nginx使用
```
cp -rp /srv/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
vim /etc/fdfs/mod_fastdfs.conf
base_path=/etc/fdfs/     #放置storage日志的位置
store_path0=/data/images01/ #放置storage数据的位置
tracker_server=172.16.3.212:22122
tracker_server=172.16.3.213:22122
group_name=g1 #此台storage1所属的服务器组名
url_have_group_name = true #在URL是否中包含group名称(访问路径不带group名,storage只有一个group的情况),为false时，group_count必须为0
```

####nginx配置
```bash
server {
        listen  80;
        server_name     172.16.3.212;     #域名自己可以定义
        access_log      /data1/logs/nginx/fastdfs_access.log gzip;
        error_log       /data1/logs/nginx/fastdfs_error.log;

        location /g1/M00        {
                alias /data/images01/data;
                ngx_fastdfs_module;
        }
}
```


####172.16.3.213 一样配置

####测试
#####测试时会调用client.conf
```
vim /etc/fdfs/client.conf
http.tracker_server_port=80 #访问80端口
base_path=/etc/fdfs/
tracker_server=172.16.3.212:22122
tracker_server=172.16.3.213:22122
```

1.测试是否可以上传文件，并访问
```
echo "111" > /tmp/test.txt
fdfs_test /etc/fdfs/client.conf upload /tmp/test.txt
http://172.16.3.212/g1/M00/00/00/rBAD1FJw0uyAWXOIAAAAB-kMomQ551.txt
```

2.测试其中tracker,storage异常退出
```
172.16.3.212
/etc/init.d/fdfs_storaged  stop
/etc/init.d/fdfs_trackerd stop
172.16.3.213 上传文件
echo "hahaha" > /tmp/test1.txt
fdfs_test /etc/fdfs/client.conf upload /tmp/test1.txt
http://172.16.3.213/g1/M00/00/00/rBAD1FJwzuaAAIXoAAAAB-kMomQ310.txt
```

#####然后在212上面访问看看
#####http://172.16.3.212/g1/M00/00/00/rBAD1FJwzuaAAIXoAAAAB-kMomQ310.txt
#####如果返回同样是hahaha那就表示成功
#####访问正常即可

![fastdfs test](/images/blog_img/fastdfs.png)

------------------------------------------

##问题集合
1. ERROR - file: ../common/connection_pool.c, line: 84, connect to 172.16.3.213:22122 fail, errno: 111, error info: Connection refused  
说明，当时另一边没有搭建就开始测试了，所以没找到tracker 213  
