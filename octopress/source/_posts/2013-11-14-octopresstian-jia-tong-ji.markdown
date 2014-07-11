---
layout: post
title: "Octopress添加统计"
date: 2013-11-14 14:06
comments: true
categories: Octopress
description: "Octopress添加统计"
tags: [octopress,博客,统计]
keywords: octopress,统计,关键字
---

搭建了Octopress发现自己不会js,css以及各类代码，要美化Octopress真的很难，所以只有谷哥，度娘，东找找，西找找,看一些大牛的文章,照搬过来，统一放到自己的博客里面，方便自己看，也方便其他人不需要频繁的在网上各种搜索。  

首先可以通过各类搜索引擎将自己的网站放入进去,这样大家在使用这些搜索引擎的时候方便搜索到自己的博客，网站。    
http://tool.lusongsong.com/addurl.html   
http://urlc.cn/tool/addurl.html   



###添加关键字
```bash
---
layout: post
title: "Octopress添加统计"
date: 2013-11-14 14:06
comments: true
categories: Octopress
description: "Octopress添加统计"
tags: [octopress,博客,统计]
keywords: octopress,统计,关键字
---
```

<!-- more -->


###统计工具
octopress模板里面默认带了`Google Analytics`工具,需要注册[Google Analytics](http://www.google.com/analytics/)获得一个`google_analytics_tracking_id`， 添加到`_config.yml`中对应位置,网站之后进行验证即可，然后可以通过`Google Analytics`分析网站的流量了。而且可以使用[Google站长工具](https://www.google.com/webmasters/tools/home?hl=zh-CN)，对网站进行更全面的分析，进行SEO。
对自己的网站进行验证，只需将网站提供的用于验证的代码添加到`source/_includes/head.html`的`<head>`标签之间，网站部署到网上后，过几分钟即可验证通过，其他 需要验证的也同样操作。
```bash
  <script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
  ga('create', 'xxxxxxxx', 'leonbb.com');
  ga('send', 'pageview');
  </script>
```


###文章部分显示
如果想让文章在首页只显示一部分，需要在文章相应的位置添加`<!-- more -->`，在`_config.yml`中修改`excerpt_link: "继续阅读 &rarr;"`来修改继续阅读 按钮的显示内容。


--------
参考文章    
[Octopress添加统计与SEO](http://812lcl.github.io/blog/2013/10/29/octopresstian-jia-tong-ji-yu-seo/)   
[Octopress主题样式修改](http://812lcl.github.io/blog/2013/10/27/octopresszhu-ti-yang-shi-xiu-gai/)    
[Octopress侧边栏及评论系统定制](http://812lcl.github.io/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)    
