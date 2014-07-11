---
layout: post
title: "Octopress第三方评论"
date: 2013-11-13 11:12
comments: true
categories: Octopress
description: "Octopress第三方评论"
keywords: Octopress
---


###Octopress第三方评论添加
初始的Octopress只有一些twitter之类的评论可以使用，对于国内普遍使用微博，人人，QQ的人来说极其不方便的。
还好Octopress也可以增加第三方评论，这样可以使我们的Octopress符合我们所需要的。   

####配置文件中添加需要的参数
octopress/_config.yml  
```
# Weibo Like
weibo_share: true
```


####增加代码
octopress/source/_includes/post/sharing.html  
```bash
  {% if site.weibo_share %}
     {% include post/weibo.html %}
  {% endif %}
```
<!-- more -->

####增加分享以及留言代码
octopress/source/_includes/post/weibo.html  
```bash
<!-- JiaThis Button BEGIN -->
<div class="jiathis_style_32x32">
        <a class="jiathis_button_qzone"></a>
        <a class="jiathis_button_tsina"></a>
        <a class="jiathis_button_tqq"></a>
        <a class="jiathis_button_weixin"></a>
        <a class="jiathis_button_renren"></a>
        <a href="http://www.jiathis.com/share" class="jiathis jiathis_txt jtico jtico_jiathis" target="_blank"></a>
        <a class="jiathis_counter_style"></a>
</div>
<script type="text/javascript" src="http://v3.jiathis.com/code/jia.js" charset="utf-8"></script>
<!-- JiaThis Button END -->

<!-- UY BEGIN -->
<div id="uyan_frame"></div>
<script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=1863487"></script>
<!-- UY END -->
```

####上面2段代码获取地点
http://www.jiathis.com/   
http://uyan.cc/   
