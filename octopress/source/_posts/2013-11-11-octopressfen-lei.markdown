---
layout: post
title: "Octopress分类"
date: 2013-11-11 08:09
comments: true
categories: Octopress
description: "Octopress 添加分类"
keywords: Octopress
---

###添加侧边栏文章分类
octopress/plugins/category_list_tag.rb    
```
module Jekyll 
  class CategoryListTag < Liquid::Tag 
    def render(context) 
      html = "" 
      categories = context.registers[:site].categories.keys 
      categories.sort.each do |category| 
        posts_in_category = context.registers[:site].categories[category].size 
        category_dir = context.registers[:site].config['category_dir'] 
        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase) 
        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n" 
      end 
      html 
    end 
  end 
end
Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```
<!-- more -->

####增加侧栏标注
octopress/source/_includes/asides/category_list.html  
```bash
<section> 
  <h1>文章分类</h1> 
  <ul id="categories"> 
    {% category_list %} 
  </ul> 
</section>
```



###导航栏设置
octopress/source/_includes/custom/navigation.html  
```
<ul class="main-navigation">
  <li><a href="{{ root_url }}/">首页</a></li>
  <li><a href="{{ root_url }}/blog/archives">归档</a></li>
  <li><a href="{{ root_url }}/huangli">程序员老黄历</a></li>
  <li><a href="{{ root_url }}/about">关于</a></li>
</ul>
```
使用命令rake new_page['about']创建一个页面，页面路径为source/about/index.markdown;



###设置网站的description和keywords
在网页的head部分设置name为description和keywords的meta元素可以让网页更容易被搜索引擎搜索到，下面说说怎样在Octopress中设置首页和每篇博文的description和keywords  

vim octopress/source/index.html  
```
layout: default
description: "世界唯一的真理"
keywords: Asp.Net,C#
```

```
---
layout: post
title: "这个是test"
date: 2013-10-02 06:11
comments: true                          #是否允许评论
categories: test                          #文章分类
description: "test"                         #页面描述
keywords: test
---
```
