---
layout: post
title: "Octopress访客地图"
date: 2013-11-13 16:10
comments: true
categories: Octopress
description: "Octopress访客地图"
keywords: Octopress
---


###Octopress访客地图
效果如我的博客右侧那个精美的3D旋转地球所示，它可以显示访客数量，访客来自的 地域，既有装饰作用，又有统计作用。它也有2D效果版，可以根据自己喜欢进行设置，地址在这里，然后获得代码。  
http://www.revolvermaps.com/?target=setup  


source/_includes/asides/earth.html
```bash
<section>
  <h1>访客地图</h1>
        <script type="text/javascript" src="http://jg.revolvermaps.com/2/1.js?i=6mb2spkrv1j&amp;s=220&amp;m=0&amp;v=false&amp;r=false&amp;b=000000&amp;n=false&amp;c=ff0000" async="async"></script>
</section>
```
<!-- more -->

####修改主配置增加参数
octopress/_config.yml
```bash
default_asides: [asides/category_list.html,asides/weibo.html,asides/earth.html]
```

--------
参考文章   
[Octopress添加统计与SEO](http://812lcl.github.io/blog/2013/10/29/octopresstian-jia-tong-ji-yu-seo/)    
[Octopress主题样式修改](http://812lcl.github.io/blog/2013/10/27/octopresszhu-ti-yang-shi-xiu-gai/)   
[Octopress侧边栏及评论系统定制](http://812lcl.github.io/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)   
