---
layout: post
title: "django 基础"
date: 2014-05-15 09:33
comments: true
categories: Django 
description: "django 基础"
tags: [django,base]
keywords: linux,django,python
---


###django重点
` django-admin.py startproject mysite` 命令在当前目录创建一个`mysite`目录,这个作为一个项目目录
```python
mysite/
├── __init__.py         #django 初始化模块
├── __init__.pyc
├── settings.py         #django 主要配置,全局环境变量
├── settings.pyc
├── urls.py             #django 主要配置,url跳转
├── urls.pyc
├── views.py            #django 主要配置,视图功能
├── views.pyc
├── wsgi.py             #django 默认加载的模块
└── wsgi.pyc
```
django请求流程，用户请求-->django-urls.py-->django-views.py

<!-- more -->

###settings.py
```python
#是数据库设置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'test',                      # Or path to database file if using sqlite3.
        'USER': 'root',                      # Not used with sqlite3.
        'PASSWORD': '123456',                  # Not used with sqlite3.
        'HOST': '127.0.0.1',                      # Set to empty string for localhost. Not used with sqlite3.
        'PORT': '3306',                      # Set to empty string for default. Not used with sqlite3.
    }
}

#需要查看python是否有Mysql模块
python
>>> import MySQLdb
>>> test=MySQLdb.connect(db='test',host='localhost',user='root',passwd='123456')
>>> cur=test.cursor()
>>> cur.execute('show databases;')
即可




#运行命令：python manage.py collectstatic 之后静态文件将要复制到的目录，这个目录只有在运行collectstatic时候才会用到，不能想当然的以为这个目录和MEDIA_ROOT的作用是相同的，否则在开发环境的时候可能一直无法找到静态文件。
STATIC_ROOT = '/var/www/html/'


#设置的static file的起始url，这个只是在template里边引用到，这个参数和MEDIA_URL的含义相同。
STATIC_URL = '/static/'


#STATICFILES_DIRS和TEMPLATE_DIRS的含义差不多，就是除了各个app的static目录以外还需要管理的静态文件设置，比如项目的公共文件差不多。
STATICFILES_DIRS = (
    "/var/www/html/devops",
    "/var/www/html/static",
)


#INSTALLED_APPS 告诉 Django 项目哪些 app 处于激活状态
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'mysite.books',
    'django.contrib.admin',
)
```


###django日志设置
```
vim settings.py
import logging
#添加报错日志，注意/var/log下的权限和uwsgi启动进程的权限是否有
logging.basicConfig(
          level = logging.ERROR,
            format = '%(asctime)s %(levelname)s %(module)s.%(funcName)s Line:%(lineno)d %(message)s',
              filename = '/var/log/django/django.log',
              )
```
