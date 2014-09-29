---
layout: post
title: "python OptionParser"
date: 2014-06-06 15:28
comments: true
categories: Python
description: "python OptionParser"
tags: [python,OptionParser]
keywords: linux,python,OptionParser
---

##脚本原型
```python
#!/usr/bin/python
#-*- coding:utf-8 -*
#django添加数据库用户密码
import sys
import os
from optparse import OptionParser

os.environ.setdefault( "DJANGO_SETTINGS_MODULE", "mysite.settings" )
from accounts.models import User
import hashlib



###编码转移,可以输出中文####
reload(sys)
sys.setdefaultencoding('utf-8')


###%prog表示脚本本身
MSG_USAGE = "%prog [ -u <username>][-p <password>][-t <user_type>][-h <help>]"
MSG_VERSION = "%prog 20140529"

###usage打印用法,version输出版本信息
optParser = OptionParser(usage=MSG_USAGE,version=MSG_VERSION)

###对optParser增加选项
optParser.add_option("-u","--username",action = "store",type = "string",dest = "UserName",help="添加数据库用户名(必填)")
optParser.add_option("-p","--password",action = "store",type = "string",dest = "Password",help="添加数据库密码(必填)")
optParser.add_option("-t","--user_type",action = "store",type = "string",dest = "User_Type",default="1",help="添加用户权限,0为管理员,1为普通,默认为1")

#fakeArgs = ['-u','leon','-p','123456','-t','a']
options,args = optParser.parse_args()

TYPE = options.User_Type
USERNAME = options.UserName
PASSWORD = options.Password

#User_Type返回不是1或者0则退出
if TYPE not in ('1','0'):
    optParser.print_help()
    sys.exit(1)

#判断参数-u,-p是否为空,返回帮助
args_list = [USERNAME,PASSWORD]
for i in args_list:
    if i is None:
        print '请使用(-h,--help)来获得帮助'
        sys.exit(0)


#User是在accounts中models.py的类,定义了数据库的一些信息,插入用户
#判断用户是否存在
def init():
    CHECK_NAME=User.objects.filter(name=USERNAME)
    if len(CHECK_NAME) > 0:
        print "%s 用户已经存在" % USERNAME
    else:
        u = User(name=USERNAME, password=hashlib.md5(PASSWORD).hexdigest(), user_type=TYPE)
        u.save()

if '__main__' == __name__:
    init()
```
<!-- more -->

中间我碰到过一个错误  
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 204: ordinal not in range(128)  
字符集问题，导致超出长度,所以需要在最前面的地方添加  
```
reload(sys)
sys.setdefaultencoding('utf-8')
```




下面这个语句就是产生一个OptionParser的物件，传入2个值，并且通过usage和version定义，这个在使用-h|--help 会产生效果.  
```
optParser = OptionParser(usage=MSG_USAGE,version=MSG_VERSION)
```

对optParser增加选项
```
optParser.add_option("-u","--username",action = "store",type = "string",dest = "UserName",help="添加数据库用户名(必填)")
```

add_option()参数说明:前面2个""里面的参数是，使用脚本时需要指定的选项  
1.action----是存储方式，分为3种`store`,`store_false`,`store_true`，一般情况下是store,store_false,store_true,都代表参数存在则为true或者false,当false,dest后面的变量将为None  


2.type----type代表该option存储dest的形态,支持的形态有string, int,long, choice, float 和 complex,这一块不全部讲，需要用到的时候查看，简单说一个int，如果type使用的是int,那么参数后面（如-u aaa)时，由于aaa不是数字所以type这边就会报错，返回报错信息  


