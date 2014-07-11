---
layout: post
title: "Octopress最新评论及友情链接"
date: 2013-11-13 16:35
comments: true
categories: Octopress
description: "Octopress最新评论及友情链接"
keywords: Octopress
---


###最新评论，热评
这边我使用的是友言，友言提供了很多组件，最新评论，热评等等   
我们可以将他们 做成侧边栏进行展示，或在首页文章列表中，显示每篇文章的评论数目  
首先登入友言页面，并注册   
http://uyan.cc/  
后台管理-->进入-->安装设置-->嵌入组件  

####创建文件
source/_includes/asides/uyan_hotcmt.html   
```bash
<section>
<h1>评论热榜</h1>
<div id="uyan_hotcmt_unit"></div>
<script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=xxxxxx"></script> <!-- 如果已经过加载此行JS，即可不用重复加载 -->
</section>
```

<!-- more -->

source/_includes/asides/uyan_newcmt.html  
```bash
<section>
<h1>最新评论</h1>
<div id="uyan_newcmt_unit"></div>
<script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=xxxxxx"></script> <!-- 如果已经过加载此行JS，即可不用重复加载 -->
</section>
```


####修改主配置文件
octopress/_config.yml  
```
default_asides: [asides/category_list.html,asides/weibo.html,asides/earth.html,asides/uyan_hotcmt.html,asides/uyan_newcmt.html]
```



###友情链接
source/_includes/asides/blog_link.html   
```
<section>
<h1>友情链接</h1>
<ul>
        <li>
        <a href="http://plumj.com/">plumj的博客</a>
        </li>
</ul>
</section>
```



####修改主配置
octopress/_config.yml   
```
default_asides: [asides/category_list.html,asides/weibo.html,asides/earth.html,asides/uyan_hotcmt.html,asides/uyan_newcmt.html,asides/blog_link.html]
```

--------
参考文章
[Octopress添加统计与SEO](http://812lcl.github.io/blog/2013/10/29/octopresstian-jia-tong-ji-yu-seo/)   
[Octopress主题样式修改](http://812lcl.github.io/blog/2013/10/27/octopresszhu-ti-yang-shi-xiu-gai/)    
[Octopress侧边栏及评论系统定制](http://812lcl.github.io/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)   

