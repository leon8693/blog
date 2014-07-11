---
layout: post
title: "Octopress install"
date: 2013-11-11 00:46
comments: true
categories: Octopress
description: "Octopress install"
keywords: Octopress
---

#信息版本
CentOS release 6.2  
Ruby 1.9.3  
Nginx 1.4.3   


###安装需要的包
```bash
yum install -y git gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel
```


###安装ruby
```bash
curl -L https://get.rvm.io | bash -s stable --ruby
#curl -L 表示允许跳转
#bash -s 模拟标准输入
#安装位置/usr/local/rvm/scripts/rvm

vim /etc/bashrc #增加环境变量
source /usr/local/rvm/scripts/rvm
source /usr/local/rvm/scripts/completion
```
<!-- more -->

####测试
```bash
type rvm | head -n1
rvm is a function #表示成功
```


####更换源为淘宝源
如果服务器在国外则不需要更换源了,但是在国内的话还是换taobao的吧，不然速度实在慢的惨不忍睹,⊙﹏⊙b
```bash
sed -i.backup -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' /usr/local/rvm/config/db
```


####使用rvm安装ruby
```bash
rvm list known #查看源里面有那写软件
rvm install 1.9.3 #安装1.9.3的ruby
rvm use 1.9.3 --default #使用1.9.3的ruby
#rvm 其他命令
rvm list
#查看安装的rubies
 
rvm [-v] [--version]
#查看版本
 
rvm remove <ruby_version>
#卸载一个版本
```

###安装Octopress
```bash
git clone git://github.com/imathis/octopress.git octopress
cd octopress
#如果使用的是RVM，这一步会被讯网你是否相信.rvmrc，选择yes
gem install bundler
bundle install
#安装依赖
 
rake install['Theme Name']
#安装主题，无参数则是默认主题
#如果要更换主题，先看完主题的文章后再更换

octopress/_config.yml #octopress配置文件
#可以在octopress/_config.yml修改博客设置，诸如博客标题，插件，以及url格式等

#_config.yml基本配置
vim /var/www/html/blog/octopress/_config.yml
url: http://leon8693.github.io     #网站url
title: 唯一的真理                         #标签
subtitle: True or False                    #副标题
author: leon                                   #作者
simple_search: http://google.com/search     #搜索引擎
description:                              #网站的描述，出现在HTML页面中的 meta 中的 description
```

####测试
rake preview      #在本机4000端口生成访问内容

####rake基本命令
```bash
rake -T

rake clean                 # Clean out caches: .pygments-cache, .gist-cache, .sass-cache
rake copydot[source,dest]  # copy dot files for deployment
rake deploy                # Default deploy task
rake gen_deploy            # Generate website and deploy
rake generate              # Generate jekyll site
rake install[theme]        # Initial setup for Octopress: copies the default theme into the path of Jekyll's generator.
rake integrate             # Move all stashed posts back into the posts directory, ready for site generation.
rake isolate[filename]     # Move all other posts than the one currently being worked on to a temporary stash location (stash) so regener...
rake list                  # list tasks
rake new_page[filename]    # Create a new page in source/(filename)/index.markdown
rake new_post[title]       # Begin a new post in source/_posts
rake preview               # preview the site in a web browser
rake push                  # deploy public directory to github pages
rake rsync                 # Deploy website via rsync
rake set_root_dir[dir]     # Update configurations to support publishing to root or sub directory
rake setup_github_pages    # Set up _deploy folder and deploy branch for Github Pages deployment
rake update_source[theme]  # Move source to source.old, install source theme updates, replace source/_includes/navigation.html with sourc...
rake update_style[theme]   # Move sass to sass.old, install sass theme updates, replace sass/custom with sass.old/custom
rake watch                 # Watch the site and regenerate when it changes
```
