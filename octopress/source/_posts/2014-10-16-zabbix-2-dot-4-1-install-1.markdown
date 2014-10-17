---
layout: post
title: "zabbix 2.4.1 安装 （一）"
date: 2014-10-16 15:09
comments: true
categories: Monitor
description: "zabbix,2.4.1,监控,monitor"
keywords: zabbix,2.4.1,监控,monitor
---


##下载zabbix 2.4.1
```
wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/2.4.1/zabbix-2.4.1.tar.gz/download -O zabbix-2.4.1.tar.gz
yum install libc-client.x86_64 curl-devel.x86_64 openssl-devel.x86_64 libpng-devel.x86_64 libjpeg-devel.x86_64 mysql.x86_64 mysql-devel.x86_64 libc-client-devel bzip2-devel.x86_64 gmp-devel.x86_64 libmcrypt-devel.x86_64 libxslt-devel.x86_64 libtool-ltdl-devel.x86_64 freetype-devel freetype -y
yum install php-leon -y    #这个包可以参考rpm制作篇
```

##配置PHP
```
#修改几个配置
vim /usr/local/php-fpm/lib/php.ini
max_execution_time = 300
max_input_time=300
date.timezone = Asia/Shanghai
post_max_size = 32M
memory_limit = 128M
error_reporting = E_ALL & ~E_NOTICE
short_open_tag = On


#修改php-fpm配置
[global]
pid = /usr/local/php-fpm/var/run/php-fpm.pid
error_log = /mnt/data1/logs/php-fpm/php-fpm.log
log_level = notice
emergency_restart_threshold = 20
emergency_restart_interval = 10m
process_control_timeout = 10s
events.mechanism = epoll
[php-1]
user = nobody
group = nobody
listen = /usr/local/php-fpm/php-fpm.sock
listen.backlog = -1
pm = dynamic
pm.max_children = 80
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500
pm.status_path = /status
access.log = /mnt/data1/logs/php-fpm/php-access.log
access.format = %m - [%u %t] " "%n"   ""%r%Q%q""    "%s"   "%{mili}d"   "%{kilobytes}M" "%C%%
slowlog = /mnt/data1/logs/php-fpm/php_slow.log
request_slowlog_timeout = 1s
request_terminate_timeout = 120s
rlimit_files = 65535
rlimit_core = unlimited
catch_workers_output = yes
env[HOSTNAME] = $HOSTNAME
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp


#php添加gettext模块,jpeg模块,FreeType模块
yum install autoconf m4 -y
cd php-5.3.10/ext/gettext/
/usr/local/php-fpm/bin/phpize
./configure && make && make install

cd php-5.3.10/ext/gd/   #我打的php包中没有使用默认gd,这边可以方便一起添加jpeg和freetype,不然需要重新编译
./configure --with-jpeg-dir=/usr/lib64/ --with-freetype-dir=/usr/lib64/
make && make install 


#在php.ini配置文件中增加
vim /usr/local/php-fpm/lib/php.ini
[getext]
extension=gettext.so
[gd]
extension=gd.so
/usr/local/php-fpm/bin/php -m |grep "gettext" #存在说明完成
/etc/init.d/php-fpm start
```
<!-- more -->

##配置nginx
```
vim /etc/nginx/conf.d/zabbix.conf
server {
    listen 80;
    server_name  _;
    root  /var/www/html/;
    error_log /mnt/data1/logs/nginx/localhost_error.log debug;
    index index.php;
        location  / {
        root /var/www/html/;
        index index.php index.html index.htm;
  }
        location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/usr/local/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_temp_path /tmp/temp;
    }
    access_log /mnt/data1/logs/nginx/localhost.log gzip;
}

vim /var/www/html/index.php
<?php
phpinfo();
?>

/etc/init.d/nginx restart
```
如出现下图说明nginx+php部署完成   
![phpinfo](/images/blog_img/phpinfo.png)


##install zabbix
```
#创建zabbix用户
groupadd zabbix
useradd -g zabbix -s /sbin/nologin zabbix

yum install net-snmp net-snmp-devel -y
cd zabbix-2.4.1
./configure --prefix=/mnt/www/zabbix/ --enable-server --enable-agent --with-mysql=/usr/lib64/mysql/mysql_config --with-net-snmp  --with-curl
make && make install

#添加services中zabbix端口规划
vim /etc/services
zabbix-agent    10050/tcp               #Zabbix Agent
zabbix-agent    10050/udp               #Zabbix Agent
zabbix-trapper  10051/tcp               #Zabbix Trapper
zabbix-trapper  10051/udp               #Zabbix Trapper

#修改配置
mkdir -p  /mnt/data1/logs/zabbix/
mkdir -p /var/run/zabbix/
mkdir -p /mnt/www/zabbix/web
cp -rp /srv/zabbix-2.4.1/frontends/php/* /mnt/www/zabbix/web/
chown zabbix.zabbix -R /mnt/www/zabbix/
chown zabbix.zabbix /var/run/zabbix/
chown zabbix.zabbix -R /mnt/data1/logs/zabbix/


vim /mnt/www/zabbix/etc/zabbix_server.conf
LogFile=/mnt/data1/logs/zabbix/zabbix_server.log
PidFile=/var/run/zabbix/zabbix.pid
DBHost=192.168.0.210
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix


#增加zabbix库
mysql -uroot -p
mysql> create database zabbix;
mysql> GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@'192.168.0.%' IDENTIFIED BY 'zabbix';
mysql> flush privileges;

#将安装包里的库文件导入到zabbix库
mysql -uroot -p zabbix -h192.168.0.210 < /srv/zabbix-2.4.1/database/mysql/schema.sql
mysql -uroot -p zabbix -h192.168.0.210 < /srv/zabbix-2.4.1/database/mysql/images.sql
mysql -uroot -p zabbix -h192.168.0.210 < /srv/zabbix-2.4.1/database/mysql/data.sql


#复制修改zabbix启动脚本
\cp -rp /srv/zabbix-2.4.1/misc/init.d/tru64/zabbix_* /etc/init.d/
chmod +x /etc/init.d/zabbix*
sed -i "s/DAEMON=.*/DAEMON=\/mnt\/www\/zabbix\/sbin\/zabbix_server/g" /etc/init.d/zabbix_server
sed -i "s/PIDFILE=.*/PIDFILE=\/var\/run\/zabbix\/zabbix.pid/g" /etc/init.d/zabbix_server


#启动server
/etc/init.d/zabbix_server start
```

##web配置
查看需要的依赖是否解决   
http://192.168.0.212/   
![zabbix01](/images/blog_img/zabbix01.png)

设置zabbix数据库配置   
![zabbix02](/images/blog_img/zabbix02.png)

设置web配置   
![zabbix03](/images/blog_img/zabbix03.png)

如果设置出现问题也可以通过配置文件直接修改如下   
```php
cp -rp  /mnt/www/zabbix/web/conf/zabbix.conf.php.example /mnt/www/zabbix/web/conf/zabbix.conf.php
vim /mnt/www/zabbix/web/conf/zabbix.conf.php
<?php
// Zabbix GUI configuration file
global $DB;

$DB["TYPE"]                     = 'MYSQL';
$DB["SERVER"]                   = '192.168.0.210';
$DB["PORT"]                     = '3306';
$DB["DATABASE"]                 = 'zabbix';
$DB["USER"]                     = 'zabbix';
$DB["PASSWORD"]                 = 'zabbix';
// SCHEMA is relevant only for IBM_DB2 database
$DB["SCHEMA"]                   = '';

$ZBX_SERVER                     = '192.168.0.212';
$ZBX_SERVER_PORT                = '10051';
$ZBX_SERVER_NAME                = 'zabbix';

$IMAGE_FORMAT_DEFAULT   = IMAGE_FORMAT_PNG;
?>
```

可以访问后输入默认用户名密码进入,就大功告成了  
username: Admin  
password: zabbix  
