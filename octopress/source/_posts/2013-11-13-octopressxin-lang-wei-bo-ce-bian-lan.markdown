---
layout: post
title: "Octopress新浪微博侧边栏"
date: 2013-11-13 15:26
comments: true
categories: Octopress
description: "Octopress新浪微博侧边栏"
tags: [octopress,新浪微博,侧边栏]
keywords: Octopress,新浪微博,侧边栏
---



###增加新浪微博侧边栏

####先通过微博秀
http://app.weibo.com/tool/weiboshow   
找到自己的代码
```bash
<iframe 
width="100%" 
height="550"
 class="share_self" 
 frameborder="0" 
scrolling="no" 
src="http://widget.weibo.com/weiboshow/index.php?language=&width=0&height=550&fansRow=2&ptype=1&speed=0&skin=1&isTitle=1&noborder=1&isWeibo=1&isFans=1&uid=xxxxxxxx&verifier=xxxxxxxx">
</iframe>
```

<!-- more -->

####将上面的代码转换成HTML
octopress/source/_includes/asides/weibo.html
```bash
{% if site.weibo_uid %}
<section>
  <h1>新浪微博</h1>
  <ul id="weibo">
    <li>
      <iframe
        width="100%"
        height="550"
        class="share_self"
        frameborder="0"
        scrolling="no"
        src=""http://widget.weibo.com/weiboshow/index.php?width=0&height=550&ptype={% if site.weibo_pic %}1{% else %}0{% endif %}&speed=0&skin={{weibo_skin}}&isTitle=0&noborder=1&isWeibo={% if site.weibo_show %}1{% else %}0{% endif %}&isFans={{weibo_fansline}}&uid={{site.weibo_uid}}&verifier={{site.weibo_verifier}}">
      </iframe>
    </li>
  </ul>
</section>
{% endif %}
```

####配置文件中增加参数
octopress/source/_config.yml  

```bash
default_asides: [asides/category_list.html,asides/weibo.html]

weibo_uid: xxxx     #微博中的uid，从微博秀中获得
weibo_verifier: xxxx     #微博中的verifier 从微博秀中获得
weibo_fansline: 1     #显示粉丝的数量，1行
weibo_show: true     #是否显示内容
weibo_pic: true         #是否显示图片
weibo_skin: 10          #使用哪种风格的10为序号10的风格
```
