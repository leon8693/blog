---
layout: post
title: "Fastdfs Nginx Lua GraphicsMagick 实时缩略图"
date: 2013-12-09 14:50
comments: true
categories: FastDFS
description: "Fastdfs Nginx Lua GraphicsMagick 实时缩略图"
tags: [linux,Fastdfs,lua,GraphicsMagick,缩略图，实时]
keywords: linux,Fastdfs,lua,GraphicsMagick,缩略图，实时
---




```bash
#信息版本
Centos 2.6.32-358.el6.x86_64
FastDFS 4.06
Nginx 1.4.1


#官网下载地址
https://fastdfs.googlecode.com/files/FastDFS_v4.06.tar.gz

```


###安装FastDFS
参考[FastDFS install](http://leon8693.github.io/blog/2013/11/03/FastDFS-install/)



###安装需要的包
```
yum install readline readline-devel perl-ExtUtils-Embed  #lua所需要的包以及后面编译需要的其他模块和库
yum install libjpeg libjpeg-devel libpng libpng-devel giflib giflib-devel freetype freetype-devel  openjpeg openjpeg-devel #gm需要的图库
```


<!-- more -->


###下载相关的包并安装
```
###graphicsmagick install
wget "http://sourceforge.net/projects/graphicsmagick/files/graphicsmagick/1.3.16/GraphicsMagick-1.3.16.tar.gz/download"
tar xf GraphicsMagick-1.3.16.tar.gz
./configure --prefix=/usr/local/GraphicsMagick-1.3.16
make && make install


###nginx 安装方式
###LuaJIT install
wget "http://luajit.org/download/LuaJIT-2.0.0-beta10.tar.gz"
tar xf LuaJIT-2.0.0-beta10.tar.gz
cd tar xf LuaJIT-2.0.0-beta10
make && make install PREFIX=/usr/local/lj2


###ngx_devel_kit install
wget "https://github.com/simpl/ngx_devel_kit/zipball/master"
unzip master
ls simpl-ngx_devel_kit-8dd0df5


###lua-nginx-module install
wget "https://github.com/chaoslawful/lua-nginx-module/tarball/master" -O master-lug-nginx-module.tar.gz
tar xf master-lug-nginx-module.tar.gz


###echo
wget "https://github.com/agentzh/echo-nginx-module/zipball/master" -O echo-nginx-module.zip
unzip echo-nginx-module.zip

###cache
wget "http://labs.frickle.com/files/ngx_cache_purge-1.6.tar.gz"
tar xf ngx_cache_purge-1.6.tar.gz

###vim /etc/profile
export LUAJIT_LIB=/usr/local/lj2/lib
export LUAJIT_INC=/usr/local/lj2/include/luajit-2.0
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH 
export  PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
GM_HOME=/usr/local/GraphicsMagick-1.3.16;
PATH=$GM_HOME/bin:$PATH;
export PATH
export GM_HOME
source /etc/profile
```

###nginx install
```
#安装前必须安装好Fastdfs
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-http_perl_module --with-mail --with-mail_ssl_module --add-module=/srv/fastdfs-nginx-module/src/  --add-module=/srv/fastdfs/simpl-ngx_devel_kit-8dd0df5/ --add-module=/srv/fastdfs/ngx_cache_purge-1.6/ --with-http_perl_module --add-module=/srv/fastdfs/agentzh-echo-nginx-module-9f811cf/ --with-ld-opt=-Wl,-rpath,/usr/local/lj2/lib --add-module=/srv/fastdfs/chaoslawful-lua-nginx-module-d11dda1/

make && make install
```

###Nginx 配置
#####这边Fastdfs的配置就不多说了，归档中有
```
server {

        listen  80;
        server_name     www.test.com;
        access_log      /data1/logs/nginx/test_access.log gzip;
        error_log       /data1/logs/nginx/test_error.log;

        location /lua1 {
                default_type 'text/plain';
                content_by_lua 'ngx.say("hello, lua")';
        }


        location  ~* /g[0-9]/M00/ {
                ngx_fastdfs_module;
                set $needCreateImg 0;
                if ( !-f $request_filename) {
                        set $needCreateImg "${needCreateImg}1";
                }


                if ($uri ~* "/g[0-9]/M00/(\w+)/(\w+)/([A-Za-z0-9_-]+).(gif|jpg|jpeg|png).(\d+x\d+).(gif|jpg|jpeg|png)") {
                        set $needCreateImg "${needCreateImg}2";
                        set $conUri     "/$1/$2/$3.$4.$5.$6";
                }

                if ($needCreateImg = "012") {
                        set $image_root "/data/images01/data/";
                        set $file "$image_root$conUri";
                                rewrite_by_lua '
                                        local index = string.find(ngx.var.conUri, "([0-9]+)x([0-9]+)");
                                local originalUri = string.sub(ngx.var.conUri, 0, index-2);
                                local area = string.sub(ngx.var.conUri, index);
                                index = string.find(area, "([.])");
                                area = string.sub(area, 0, index-1);

                                function table.contains(table, element)
                                        for _, value in pairs(table) do
                                                if value == element then
                                                        return true
                                                        end
                                                                end
                                                                return false
                                                                end

                                                                local c = "/usr/local/GraphicsMagick-1.3.16/bin/gm convert " .. ngx.var.image_root ..  originalUri  .. " -thumbnail " .. area .. " - ";
                                local f = assert(io.popen(c, "r"))
                                local s = assert(f:read("*a"))
                                f:close()
                                ngx.say(s)
                                ';
                        }
                        alias /data/images01/data/;
                }
        }
```


###性能测试
GraphicsMagick VS Ngx_image_thumb(gd)  
#####测试图片大小  
大小	名字  
12M	10M.jpg   
1.1M	1M.jpg   
4.5M	5M.jpg   
#####图片其他信息
![png_info](/images/blog_img/info.png)  
#####测试方式
Apache ab命令    
ab -n200 -c 10 url   
#####图片url
http://192.168.1.100/g1/M00/00/00/CgoDBFKoIMqAIippABCLd5b4wbs388.jpg (1.1M)  
http://192.168.1.100/g1/M00/00/00/CgoDBVKoJhuAJtQfAEaYyGoiMsc873.jpg (4.5M)   
http://192.168.1.100/g1/M00/00/00/CgoDBFKoJl-ALBVOALTeZd_Vsio409.jpg (12M)    
#####服务器信息
DELL R420   
Intel Xeon E5-2403*1 @1.80GHz   
MEM 32G   


###Ngx_image_thumb测试
```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBFKoIMqAIippABCLd5b4wbs388.jpg!t300x300.jpg'   
```
CPU load average: 2.35, 0.53, 0.18   
MEM 8706M    
ab详细信息   
![gd1](/images/blog_img/gd1.png)


```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBVKoJhuAJtQfAEaYyGoiMsc873.jpg!t300x300.jpg'   
```
CPU load average: 3.30, 0.88, 0.35   
MEM 8689M    
ab详细信息   
![gd5](/images/blog_img/gd5.png)   

```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBFKoJl-ALBVOALTeZd_Vsio409.jpg!t300x300.jpg'  
```
CPU load average: 3.61, 1.21, 0.54   
MEM 8743M   
ab详细信息   
![gd10](/images/blog_img/gd10.png)   



###lua graphicsmagick测试  
```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBFKoIMqAIippABCLd5b4wbs388.jpg.300x300.jpg'   
```
CPU load average: 1.54, 0.33, 0.11   
MEM 3M    
ab详细信息   
![gm1](/images/blog_img/gm1.png)   

```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBVKoJhuAJtQfAEaYyGoiMsc873.jpg.300x300.jpg'  
```
CPU load average: 3.66, 0.91, 0.35    
MEM 1M    
ab 详细信息    
![gm5](/images/blog_img/gm5.png)    

```
ab -n200 -c10 'http://192.168.1.100/g1/M00/00/00/CgoDBFKoJl-ALBVOALTeZd_Vsio409.jpg.300x300.jpg'   
```
CPU load average: 3.76, 1.35, 0.58    
MEM 1M    
ab 详细信息   
![gm10](/images/blog_img/gm10.png)



###性能测试总结
从上面的测试情况来看,lua+graphicsmagick完爆Ngx_image_thumb   
1.CPU压力相同。   
2.MEM Ngx_image_thumb不释放，只有重启nginx后才会释放，而Lua+graphicsmagick使用的时候连接一旦释放内存也会释放。   
3.吞吐量以及请求时间,lua+graphicsmagick完爆Ngx_image_thumb  



###问题注意
我在使用graphicsmagick时候发现一台机器是可以对jpeg进行缩略图，一台则不行。  
通过gm version 查看到原来里面的jpeg功能没有打开，这个需要在编译graphicsmagick前系统就得有libjpeg这个包。  
```
Feature Support:
  Thread Safe              yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     yes
  DPS                      no
  FlashPix                 no
  FreeType                 yes
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     yes
  Little CMS               no
  Loadable Modules         no
  OpenMP                   yes (200805)
  PNG                      yes
  TIFF                     no
  TRIO                     no
  UMEM                     no
  WMF                      no
  X11                      yes
  XML                      yes
  ZLIB                     yes

Host type: x86_64-unknown-linux-gnu

Configured using the command:
  ./configure  '--prefix=/usr/local/GraphicsMagick-1.3.16/' --enable-ltdl-convenience

Final Build Parameters:
  CC       = gcc -std=gnu99
  CFLAGS   = -fopenmp -g -O2 -Wall -pthread
  CPPFLAGS = -I/usr/include/freetype2 -I/usr/include/libxml2
  CXX      = g++
  CXXFLAGS = -pthread
  LDFLAGS  = -L/usr/lib -L/usr/lib
  LIBS     = -lfreetype -ljpeg -lpng12 -lXext -lSM -lICE -lX11 -lbz2 -lxml2 -lz -lm -lgomp -lpthread
```
