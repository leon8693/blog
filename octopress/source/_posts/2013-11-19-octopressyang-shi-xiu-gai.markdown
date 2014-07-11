---
layout: post
title: "Octopress样式修改"
date: 2013-11-19 14:39
comments: true
categories: Octopress
description: "Octopress样式修改"
tags: [octopress,样式,背景修改，]
keywords: octopress,样式,背景修改
---


###样式修改
添加或修改控制样式，需编辑`sass/custom/_styles.scss`，博客的所有颜色控制在`/sass/base/_theme.scss`中进行设置。定制自己的配色，编辑`sass/custom/_colors.scss`。查看[HSL COLOR PICKER](http://hslpicker.com/#e1ff00)，帮你挑选喜欢的颜色。   
修改布局，需要编辑`sass/base/_layout.scss`，可以修改各部分的宽度等。  


###添加背景图片
在`sass/custom/_styles.scss`中添加   
```bash
html {
        background: #555555 url("/images/lol.jpg");
        //background: #555555;
}

body > div {
        background-image: none;
        //background: #F5F5D5
} //侧边栏

body > div > div { //文章内容
        background-image: none;
        //background: #F5F5D5;
        //background: url("/images/bg.jpg");
}
```

将背景图片放入`source/images/`中，修改上述代码中的路径指向想要的图片，即可 更改博客、侧边栏或文章的背景图片。博客使用背景图片后，与Header区不太和谐，所以我在`/sass/base/_theme.scss`中将`header-bg`设置成透明色了。
```bash
$header-bg: hsla(0, 0%, 15%, 0)  !default;
```

<!-- more -->


###LOGO图片
LOGO图片我所说的logo图片有两种，一个是打开一个网页时，标签栏上显示的小图片。还有一个 是标题栏主标题旁的图片。   
首先针对于第一种可以选择你喜欢的图片（大小适中），替换source目录下的 favicon.png即可。  
或者将logo图片放入`source/images`中，然后修改`source/_includes/head.html`，找到favicon.png，修改其路径指向你的图片即可。  
对于主标题旁的图片需要在`sass/custom/_styles.scss`中填入如下语句:  

```bash
//Blog logo pic
@media only screen and (min-width: 550px) {

        body > header h1{
                background: url("/images/logo1.png") no-repeat 0 1px;
                padding-left: 65px;
        }

        body > header h2 { padding-left: 65px; }
}
```

根据自己情况进行修改即可。

###导航栏倒圆角
我设置的header区背景色透明，所以导航栏的直角有些尖锐，在`sass/custom/_styles.scss`中添加如下语句，将其修改为圆角:

```bash
//倒圆角
@media only screen and (min-width: 1040px) {
        body > nav {
                @include border-top-radius(.4em);
        }

        body > footer {
                @include border-bottom-radius(.4em);
        }
}

```

###滑动返回顶部按钮
当文章较长，通常希望有一个返回顶部的按钮，如下方法实现了在页面右下方添加一个 返回顶部的图片按钮，点击后可以滑动的返回顶部。
首先创建`source/javascripts/top.js`，实现滑动返回顶部效果，添加如下代码：
```bash
function goTop(acceleration, time)
{
        acceleration = acceleration || 0.1;
        time = time || 16;

        var x1 = 0;
        var y1 = 0;
        var x2 = 0;
        var y2 = 0;
        var x3 = 0;
        var y3 = 0;

        if (document.documentElement)
        {
                x1 = document.documentElement.scrollLeft || 0;
                y1 = document.documentElement.scrollTop || 0;
        }
        if (document.body)
        {
                x2 = document.body.scrollLeft || 0;
                y2 = document.body.scrollTop || 0;
        }
        var x3 = window.scrollX || 0;
        var y3 = window.scrollY || 0;

        var x = Math.max(x1, Math.max(x2, x3));
        var y = Math.max(y1, Math.max(y2, y3));

        var speed = 1 + acceleration;
        window.scrollTo(Math.floor(x / speed), Math.floor(y / speed));

        if(x > 0 || y > 0)
        {
                var invokeFunction = "goTop(" + acceleration + ", " + time + ")";
                window.setTimeout(invokeFunction, time);
        }
}
```

然后创建`source/_includes/custom/totop.html`,设置返回顶部按钮样式和位置,代码如下:
```bash
<!--返回顶部开始-->
<div id="full" style="width:0px; height:0px; position:fixed; right:180px; bottom:150px; z-index:100; text-align:center; background-color:transparent; cursor:pointer;">
        <a href="#" onclick="goTop();return false;"><img src="/images/top.png" border=0 alt="返回顶部"></a>
</div>
<script src="/javascripts/top.js" type="text/javascript"></script>
<!--返回顶部结束-->
```


最后，还需要将返回顶部的图片放入`source/images`，命名为top.png(或修改totop.html中图片的路径)。  
在`source/_layouts/default.html`添加html样式。        

```bash
<body>
  ...
  {% include custom/totop.html %}
  ...
</body>
```


###感谢812lcl
他的博客讲解的很详细，我部分需求也是看了他的博客才会做，有些问题也是通过他来帮忙解决的。


------ 
参考文章  
[Octopress添加统计与SEO](http://812lcl.github.io/blog/2013/10/29/octopresstian-jia-tong-ji-yu-seo/)  
[Octopress主题样式修改](http://812lcl.github.io/blog/2013/10/27/octopresszhu-ti-yang-shi-xiu-gai/)   
[Octopress侧边栏及评论系统定制](http://812lcl.github.io/blog/2013/10/26/octopressce-bian-lan-ji-ping-lun-xi-tong-ding-zhi/)   
