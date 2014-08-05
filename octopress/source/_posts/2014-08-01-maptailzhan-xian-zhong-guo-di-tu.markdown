---
layout: post
title: "maptail展现中国地图"
date: 2014-08-01 16:29
comments: true
categories: Other
description: "maptail展现中国地图"
tags: [maptail 中国地图]
keywords: maptail,chinamap,中国地图
---

###声明
本篇文章转自王强blog  
blog地址:[maptail展现中国地图](http://qiangwang.github.io/2013/01/27/maptail-china-map.html)   
github地址:[maptail-github](https://github.com/qiangwang/maptail)   



# maptail

<img src="http://qiangwang.github.io/img/maptail-china-map.png" border="0" />

maptail is a realtime map view of GeoIP data. Attach it to your server to track visitors, tail a log, pipe to its stdin or use it as a library to build your own implementation. Just emit IP addresses to it from any source and you'll automagically get a cool map with yellow dots and stuff like that streamed in with websockets or whatever transport you'd like to use.

## Installing

`npm install maptail -g`

Omit the `-g` to install as a module.

## How to use

### The command line tool:

`$ maptail -f nohup.out`

`$ tail -f nohup.out | maptail -h my.host.com -p 3000`

<!-- more -->

### In your server:

```javascript
var maptail = require('maptail')
var express = require('express')
var app = express.createServer()

app.use(maptail.track())
app.use('/map', maptail.static())

maptail.attach(app)

app.listen(8080, 'localhost')
```

Let me explain what these are doing here a bit. `maptail.track()` tracks visitors' IPs and emits them to maptail. `maptail.static()` is an `express.static()` middleware that points to our static data (maptail.html, css, etc.)

`maptail.attach(app)` attaches a [simpl](https://github.com/stagas/simpl) WebSocket server which makes it possible for our frontend app to easily subscribe to the GeoIP data events sent by maptail and display them on the map.

If for example you don't want to track visitors of the http server but instead you want to send IPs from another source, you can easily remove `maptail.track()` from the middleware and use `maptail.emit('ip', ipAddress[, logMessage])` to feed our map. It will take care the rest for you.

## Credits

This is based on [mape](https://github.com/mape)'s [wargames](https://github.com/mape/node-wargames).

[geoip-lite](https://github.com/bluesmoon/node-geoip) by [Philip Tellis](https://github.com/bluesmoon).

Earlier versions used [kuno](https://github.com/kuno)'s [GeoIP](https://github.com/kuno/GeoIP) module but since it now uses a C library, I couldn't use it.

[MaxMind](http://www.maxmind.com/) for their free to use GeoIP data.

## Licence

maptail is MIT/X11. The rest of the components are of their respective licences.



##Nginx配置
本人使用后非常好,这得感谢王强的付出和共享,随后我将这个通过nginx转发,发现通过nginx代理后页面变成静态了,这个是怎么回事?  
原本想研究一下,但迫于工作原因,没时间仔细看,不过和同事分享的时候发现了是websocket的问题,这得感谢@朱总的提议,随后查看页面如下  
![maptail001](/images/blog_img/maptail001.png)

确确实实就是websocket问题  
接下来我们来解决它,嘿嘿  
官方参考:http://nginx.org/en/docs/http/websocket.html  
使用这些配置即可使nginx支持websocket  
```
server {
        listen 80; 
        server_name _;
        access_log  /data1/logs/nginx/access.log  gzip;
        error_log   /data1/logs/nginx/error.log error;

        location / { 
                proxy_pass http://test;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header Host $host; 
        }   
}
```


OK 完成！  
