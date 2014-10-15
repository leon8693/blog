---
layout: post
title: "yum 私有源"
date: 2014-10-15 16:17
comments: true
categories: Other
description: "yum,rpm,私有源,本地源"
keywords: yum,rpm,私有源,本地源
---


##yum私有源
当自己制作RPM包后发现还需要一个私有的yum源   
下面就自己搭建一个   

<!-- more -->
```
#安装需要的包
yum -y install createrepo 

#创建需要存放RPM包的目录
mkdir -p /mnt/rpm/centos6

#放入自定义的包
cp -rp php-leon-5.3.10-0.x86_64.rpm /mnt/rpm/centos6/

#创建yum使用的createrepo索引
createrepo /mnt/rpm/centos6/

#这里使用nginx中的http模式,所以编辑nginx
vim /etc/nginx/conf.d/yum.conf
server  {
        listen 80;
        server_name yum.leon.com;
        index  idex.html;
        autoindex on;
        root /mnt/rpm/;
}

/etc/init.d/nginx reload

#增加repo
vim /etc/yum.repos.d/leon.repo
[leon-centos6]
name=Extra Packages for Enterprise Linux 6 - $basearch
baseurl=http://yum.leon.com/rpm/
enabled=1
gpgcheck=0
keepcache=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

#yum makecache内容
yum makecache

yum install php-leon -y
```

大功告成！  
