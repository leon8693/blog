---
layout: post
title: "rpmbuild 打包rpm实战"
date: 2014-10-14 15:49
comments: true
categories: Other
description: "rpmbuild,rpm,打包,Centos,Redhat"
keywords: rpmbuild,rpm,打包,Centos,Redhat,实战
---

##基本信息
CentOS release 6.4 (Final)   
2.6.32-358.el6.x86_64  


##rpmbuild目录设置
```bash
#自定义编译的目录
vim ~/.rpmmacros
%_topdir /root/rpmbuild
%_tmppath /root/rpmbuild/tmp
%buildroot /root/rpmbuild/BUILDROOT
%_prefix  /

#创建自定义目录
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}

#存放目录状态
[root@angel02 rpmbuild]# tree
.
├── BUILD
├── RPMS
├── SOURCES
│   └── php-5.3.10.tar.gz
├── SPECS
│   └── php5.spec
├── SRPMS
└── tmp

vim /usr/lib/rpm/macros
%_topdir                %{getenv:HOME}/rpmbuild #注释掉
%_topdir                %{_usrsrc}/

```

<!-- more -->

##编辑spec
```bash
%define version 5.3.10
%define so_version 5
%define release 0

Name: php-fdm
Summary: PHP: Hypertext Preprocessor
Group: Development/Languages
Version: %{version}
Release: %{release}
Source: php-5.3.10.tar.gz
URL: http://www.php.net/
Packager: bolatuleon@hotmail.com
License: GPL
BuildRoot: /var/tmp/php-%{version}

%description
Leon php5 package

%prep
%setup -q
./configure  --prefix=/usr/local/php-fpm --with-libdir=lib64 --enable-cgi --enable-fpm --with-imap=/usr --with-imap-ssl --enable-bcmath --with-kerberos --with-bz2 --enable-calendar --enable-ctype --with-curl --enable-dom --enable-exif --enable-fileinfo --enable-filter --enable-ftp --with-gd --with-gmp --enable-hash --with-iconv --enable-json --enable-libxml --enable-mbstring --with-mcrypt --with-mysql --with-mysqli --with-openssl --enable-pcntl --enable-PDO --with-pdo_mysql --with-pdo_sqlite --enable-Phar --enable-posix --enable-session --enable-shmop --enable-SimpleXML --enable-sockets --with-sqlite3 --enable-sysvmsg --enable-sysvsem --enable-tokenizer --enable-wddx --enable-xml --enable-xmlreader --enable-xmlwriter --with-xsl --enable-zip --with-zlib

make

%install
#make DESTDIR=$RPM_BUILD_ROOT install
#DESTDIR是Makefile文件中定义的一个安装路径的变量，根据实际情况修改，比如mysql和nginx的是DESTDIR，而php的是INSTALL_ROOT。
make INSTALL_ROOT=$RPM_BUILD_ROOT install
mkdir -p $RPM_BUILD_ROOT/usr/local/php-fpm/init/
cp -rp $RPM_BUILD_DIR/php-fdm-5.3.10/sapi/fpm/init.d.php-fpm $RPM_BUILD_ROOT/usr/local/php-fpm/init/php-fpm
cp -rp $RPM_BUILD_DIR/php-fdm-5.3.10/php.ini-production $RPM_BUILD_ROOT/usr/local/php-fpm/lib/php.ini


%clean
[ "$RPM_BUILD_ROOT" != "/" ] && rm -rf "$RPM_BUILD_ROOT"
make clean

%post
ln -s /usr/local/php-fpm/bin/* /usr/local/bin/
mkdir -p /mnt/data1/logs/php-fpm/
cat << EOF > /etc/logrotate.d/php-fpm
/mnt/data1/logs/php-fpm/*.log {
        daily
        missingok
        rotate 7
        compress
        notifempty
        sharedscripts
        dateext
        postrotate
                [ -f /var/run/php-fpm.pid ] && kill -USR1 \`cat /var/run/php-fpm.pid\`
        endscript
}
EOF

cp -rp /usr/local/php-fpm/init/php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
sed -i "s/php_opts=\"--fpm-config \$php_fpm_CONF\"/php_opts=\"--fpm-config \$php_fpm_CONF -g \$php_fpm_PID\"/g" /etc/init.d/php-fpm


%postun
rm -rf /usr/local/bin/pear /usr/local/bin/peardev /usr/local/bin/pecl /usr/local/bin/phar /usr/local/bin/phar.phar /usr/local/bin/php /usr/local/bin/php-config /usr/local/bin/phpize
rm -rf /etc/logrotate.d/php-fpm
rm -rf /etc/init.d/php-fpm







%files
%defattr (-,root,root)
/usr/local/php-fpm/
/usr/local/php-fpm/init/php-fpm
/usr/local/php-fpm/lib/php.ini
/.channels/.alias/pear.txt
/.channels/.alias/pecl.txt
/.channels/.alias/phpdocs.txt
/.channels/__uri.reg
/.channels/doc.php.net.reg
/.channels/pear.php.net.reg
/.channels/pecl.php.net.reg
/.depdb
/.depdblock
/.filemap
/.lock
```
##开始制作包
```
#安装需要的依赖包
yum install libc-client.x86_64 curl-devel openssl-devel.x86_64 libpng-devel.x86_64 libjpeg-devel mysql.x86_64 mysql-devel.x86_64 libc-client-devel bzip2-devel.x86_64 gmp-devel.x86_64 libmcrypt-devel.x86_64 libxslt-devel.x86_64 libtool-ltdl-devel.x86_64 -y

#制作src和rpm包
rpmbuild -ba ~/rpmbuild/SPECS/php5.spec

#经过一连串的编译过程之后如下
[root@angel02 php-fpm]# ls /root/rpmbuild/RPMS/x86_64/
php-fdm-5.3.10-0.x86_64.rpm
[root@angel02 php-fpm]# ls /root/rpmbuild/SRPMS/
php-fdm-5.3.10-0.src.rpm

#分别获得了这2个包,安装一下rpm包是否有问题,自行调试
```


##问题合集
1.error: Installed (but unpackaged) file(s) found:    
解决如下   
```
在%file中将文件全部加入
%files
%defattr (-,root,root)
/usr/local/php/
/.channels/.alias/pear.txt
/.channels/.alias/pecl.txt
/.channels/.alias/phpdocs.txt
/.channels/__uri.reg
/.channels/doc.php.net.reg
/.channels/pear.php.net.reg
/.channels/pecl.php.net.reg
/.depdb
/.depdblock
/.filemap
/.lock
或者
进入文件/usr/lib/rpm/macros,找到
%__check_files         %{_rpmconfigdir}/check-files %{buildroot}
这一行，把这一行注释掉，然后重新编译
```

2.error: License field must be present in package: (main package)   
解决如下
```
vim php5.spec
添加
License: GPL #一般都是GPL这个自行判断
```




