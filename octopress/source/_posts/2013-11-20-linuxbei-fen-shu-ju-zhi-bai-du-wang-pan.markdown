---
layout: post
title: "Linux备份数据至百度网盘"
date: 2013-11-20 14:59
comments: true
categories: Other
description: "Linux备份数据至百度网盘"
tags: [linux,百度,网盘,备份,自动]
keywords: linux,百度,网盘,备份,自动
---

##前言
前阵同事说百度网盘可以升级到2TB级的空间，井底之蛙的我，才发现自己用的百度网盘才6G，弱爆了，果断升级，但是怎么大的空间如何利用呢？所以想定期对自己linux系统下的文件进行备份放入百度网盘，这时候谷哥，度娘发挥作用了，我将搜咯到的资料进行整合，写下这个篇文章。

###下载安装bpcs_uploader百度pcs上传脚本
####下载安装
bpcs_uploader作者官网  
http://oott123.github.io/bpcs_uploader/   
```bash
wget https://github.com/oott123/bpcs_uploader/zipball/master #下载zip文件
unzip master	#解压
mv oott123-bpcs_uploader-ecb07c3 bpcs_uploader #重命名
cd bpcs_uploader 
chmod +x bpcs_uploader.php #赋予执行权限
yum install php curl -y	#安装需要的软件包
```

<!-- more -->

####百度网盘申请应用
地址:http://developer.baidu.com/dev#/create  
![申请百度应用](/images/blog_img/api3.png)  

这个链接可以看到你有多少应用  
http://developer.baidu.com/console#app
![查看应用](/images/blog_img/api4.png)

点击API管理  
![API管理](/images/blog_img/api5.png)

打开PCS API  
![API PCS](/images/blog_img/api6.png)

设置目录    
![API FILE](/images/blog_img/api7.png)

记录KEY   
![API KEY](/images/blog_img/api8.png)

运行bpcs_uploader.php   
```bash
./bpcs_uploader.php init #初始化
```
![bpcs init](/images/blog_img/api.png)
1. 初始化选择Y    
2. 输入你前面API KEY值   
3. 输入你前面API SECRET值   
4. 输入开启API接口时创建的目录名称  
5. 登入链接输入KEY激活  
6. 成功则显示你所使用的容量和空余容量   


###使用bpcs_uploader上传测试
```bash
/root/scripts/bpcs_uploader/bpcs_uploader.php upload  /tmp/test.html /test/test.html
```
![bpcs_uploader upload](/images/blog_img/api10.png)

####查看网盘
登入自己的网盘发现居然有那个文件了！以此类推，就可以在linux下备份数据到百度网盘了。NICE！  
![bpcs_uploader upload](/images/blog_img/api11.png)

###bpcs_uploader基本命令
```php
./bpcs_uploader.php quota #查看配额
./bpcs_uploader.php upload [path_local] [path_remote] #上传文件
./bpcs_uploader.php download [path_local] [path_remote]  #下载文件
./bpcs_uploader.php delete [path_remote]  #删除文件
./bpcs_uploader.php fetch [path_remote] [path_to_fetch] #离线下载
./bpcs_uploader.php init #初始化
```

###bpcs_uploader配置解释
在`bpcs_uploader/_bpcs_files_/config`下注意几个文件  
```php
bpcs_uploader/_bpcs_files_/config
├── access_token
├── appkey 	#前面填入的appkey
├── appname	#百度网盘应用的目录
├── appsec	#前面天如的appsec
├── config.lock
├── readme.txt
└── refresh_token
```


###linux下载百度网盘
进入自己的网盘找到需要下载的文件  
创建分享链接  
![linke](/images/blog_img/wget1.png)  
![linke](/images/blog_img/wget2.png)   
查看链接，firebug等工具查看  
![find](/images/blog_img/wget3.png)   

```bash
wget -c -O leon.png "http://d.pcs.baidu.com/thumbnail/be04f0e3e6e96e2102ff9c376778915b?fid=1191379893-250528-3966810088&time=1385030781&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-6tGvvJKsVQd%2FKXr5mscLmcGSFyQ%3D&rt=sh&expires=8h&sharesign=unknown&r=984765212&size=c710_u500&quality=100"
-c #断点续传
-O #指定下载下来的文件名称
```

就怎么简单！  

###多线程下载百度网盘
由于wget只能单线程,可是有些大文件单线程实在太慢了,所以需要找个能够多线程的下载工具  
Axel下载
```
#Centos6.4 i686 i386
wget http://pkgs.repoforge.org/axel/axel-2.4-1.el6.rf.i686.rpm
rpm -ivh axel-2.4-1.el6.rf.i686.rpm

#Ubuntu 12.04
apt-get install axel  
```
根据系统不一样下载相应的包,并安装    

使用axel下载百度网盘里的大文件,这里我下一个视频大小为188M   
```
#用9个线程下载
axel -n 9 "http://d.pcs.baidu.com/file/2eab76d2c3fee9335378275313f18d40?fid=1191379893-250528-2944587601&time=1385032735&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-WLdPSqPZBPH2ogqHIEfBTfYbWKk%3D&rt=sh&expires=8h&sharesign=zOo3WfKU2OHg3ebqvI82xwvc2lHPOnXMCe3I8C9LQ8dveBIuprdikBzxxswmQru+GIgsZD9TtLI=&r=553543714&sh=1&cflg=65535"

#axel常用参数
-n #线程数
-s #限速,如-s 10240即每秒10K
-o #指定另存目录
-q #静默模式

```
![axel](/images/blog_img/axel1.png)

再看看单线程的wget   
![wget](/images/blog_img/wget4.png)

还是很大的差距的，但是建议不要用axel开太多的进程，控制在100以内。  

