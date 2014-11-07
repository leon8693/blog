---
layout: post
title: "octopress 加快博客访问速度"
date: 2014-11-07 11:08
comments: true
categories: Octopress
description: "Octopress 访问速度"
keywords: Octopress
---


##增加Octopress访问速度
大家会发现搭建的Octopress访问的非常慢,除非翻墙才能访问的快  
其中原因就是googleapis.com相关的东西造成我们访问缓慢(GFW懂的入)  
这边感谢大神张吉给的意见[传送门](https://github.com/jizhang/jizhang.github.com/commit/340817e45b1c3a0e6c0fcb09c5d153141358c59d)   
虽然大神写了,我这里还是补刀一下,自己记录一下  


```
cat source/_includes/head.html

  <script src="{{ root_url }}/javascripts/modernizr-2.0.js"></script>
  <!-- <script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>                                                                                                                           
  <script>!window.jQuery && document.write(unescape('%3Cscript src="./javascripts/lib/jquery.min.js"%3E%3C/script%3E'))</script> -->
  <script src="{{ root_url }}/javascripts/libs/jquery.min.js"></script>
  <script src="{{ root_url }}/javascripts/octopress.js" type="text/javascript"></script>
  <script>

```



```
cat source/_includes/custom/head.html

<!--Fonts from Google"s Web font directory at http://google.com/webfonts -->
<!-- <link href="http://fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">  -->
<!-- <link href="http://fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css"> -->      
```


好了就到这,再访问一下blog是不是很快: )  
