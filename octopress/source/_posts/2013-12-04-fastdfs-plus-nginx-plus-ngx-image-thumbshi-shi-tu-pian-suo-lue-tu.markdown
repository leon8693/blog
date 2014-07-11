---
layout: post
title: "Fastdfs Nginx ngx_image_thumb 实时图片缩略图"
date: 2013-12-04 14:40
comments: true
categories: FastDFS
description: "Fastdfs Nginx ngx_image_thumb 实时图片缩略图"
tags: [linux,Fastdfs,ngx_image_thumb,缩略图，实时]
keywords: linux,Fastdfs,ngx_image_thumb,缩略图，实时
---




```bash
#信息版本
Centos 2.6.32-358.el6.x86_64
FastDFS 4.06
Tengine 1.5.2


#官网下载地址
https://fastdfs.googlecode.com/files/FastDFS_v4.06.tar.gz

#oopul写的nginx模块ngx_image_thumb
wget https://github.com/3078825/nginx-image/archive/master.zip

#Tengine 1.5.2 下载
wget http://tengine.taobao.org/download/tengine-1.5.2.tar.gz
```

###安装FastDFS
参考[FastDFS install](http://leon8693.github.io/blog/2013/11/03/FastDFS-install/)




###安装需要的包
```
yum install gd-devel pcre-devel libcurl-devel gcc automake autoconf m4
```

###安装Tengine并加载ngx_image_thumb模块
```
tar xf tengine-1.5.2.tar.gz
cd tengine-1.5.2
./configure --add-module=/srv/fastdfs-nginx-module/src/ --add-module=/srv/ngx_image_thumb-master/
make && make install
```
<!-- more -->

###Nginx配置
```
server {
        listen  80;
        server_name     172.16.3.213;
        access_log      /data1/logs/nginx/fastdfs_access.log gzip;
        error_log       /data1/logs/nginx/fastdfs_error.log;

        location /g1/M00        {
                alias /data/images01/data;
                ngx_fastdfs_module;
                image on;
                image_output on;
        }


}
```



###测试
输入地址http://172.16.3.213/g1/M00/00/00/rBAD1FKgMQSAOJYaAAT4ktP-fyM783.jpg   
![huonan test](/images/blog_img/huonan.jpg)


输入地址http://172.16.3.213/g1/M00/00/00/rBAD1FKgMQSAOJYaAAT4ktP-fyM783.jpg!w300x300.jpg   
![huonan2 test](/images/blog_img/huonan2.jpg)


###参数说明

```
image on/off 是否开启缩略图功能,默认关闭

image_backend on/off 是否开启镜像服务，当开启该功能时，请求目录不存在的图片（判断原图），将自动从镜像服务器地址下载原图

image_backend_server 镜像服务器地址

image_output on/off 是否不生成图片而直接处理后输出 默认off

image_jpeg_quality 75 生成JPEG图片的质量 默认值75

image_water on/off 是否开启水印功能

image_water_type 0/1 水印类型 0:图片水印 1:文字水印

image_water_min 300 300 图片宽度 300 高度 300 的情况才添加水印

image_water_pos 0-9 水印位置 默认值9 0为随机位置,1为顶端居左,2为顶端居中,3为顶端居右,4为中部居左,5为中部居中,6为中部居右,7为底端居左,8为底端居中,9为底端居右

image_water_file 水印文件(jpg/png/gif),绝对路径或者相对路径的水印图片

image_water_transparent 水印透明度,默认20

image_water_text 水印文字 "Power By Vampire"

image_water_font_size 水印大小 默认 5

image_water_font 文字水印字体文件路径

image_water_color 水印文字颜色,默认 #000000
```

###调用说明
######这里假设你的nginx 访问地址为http://172.16.3.213/g1/M00/00/00/rBAD1FKgMQSAOJYaAAT4ktP-fyM783.jpg    
######通过访问http://172.16.3.213/g1/M00/00/00/rBAD1FKgMQSAOJYaAAT4ktP-fyM783.jpg!t300x300.jpg    
######其中t是生成图片缩略图的参数,300是生成缩略图的宽度,300是生成缩略图的高度   
######可以生成四种不同类型的缩略图  
######支持jpeg/png/gif(Gif生成后变成静态图片)   
######C 参数按请求宽高比例从图片高度 10% 处开始截取图片，然后缩放/放大到指定尺寸（ 图片缩略图大小等于请求的宽高 ）  
######M 参数按请求宽高比例居中截图图片，然后缩放/放大到指定尺寸（ 图片缩略图大小等于请求的宽高 ）  
######T 参数按请求宽高比例按比例缩放/放大到指定尺寸（ 图片缩略图大小可能小于请求的宽高 )  
######W 参数按请求宽高比例缩放/放大到指定尺寸，空白处填充白色背景颜色（ 图片缩略图大小等于请求的宽高 ）    


