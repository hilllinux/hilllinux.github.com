---
layout: post
title: "阿里云 Openresty+redis+PHP+Mysql环境搭建"
description: "Vagrant 本质上来说，是对 virtualbox，vmware，kvm 等镜像的管理操作，是一个中间层技术。使用它的前提是你本机必须有 virtualbox，vmware，kvm 等虚拟机。"
keywords: "ngx_openresty, 配置, 使用"
category: Linux
tags: [ngx_openresty, redis, LNMP]
---

nginx 和 PHP-fpm 均已www 用户启动，便于权限控制
!notice 如果nginx 和 PHP-fpm 用户不一致，会有PHP-fpm接收不到Nginx
的Sock，导致 502 错误；
useradd -M -s /sbin/nologin www

1. 安装nginx_openresty(with luajit)
sudo apt-get install libreadline-dev libpcre3-dev libssl-dev perl
build-essential -y
wget http://openresty.org/download/ngx_openresty-1.7.2.1.tar.gz
 ./configure --with-luajit --prefix=/usr/local/openresty
 sudo make && sudo make install
 ln -s /usr/local/openresty/nginx/sbin/nginx /usr/sbin/nginx
 openresty deamon 配置地址:
 https://gist.github.com/41bdd69f0da37da24ff7.git
 将配置内容另存为 /etc/init.d/nginx
 sudo update-rc.d -f nginx defaults
 sudo service nginx (start/stop/reload)

 2. 安装redis
 wget http://download.redis.io/releases/redis-2.8.17.tar.gz
 tar xvf redis-2.8.17.tar.gz 
 cd redis-2.8.17/
 sudo make && sudo make install
 修改redis.conf    daemonize yes

 3. 安装PHP-fpm 和mysql
 sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql
 fcgiwrap php5-fpm php5-xcache php5-pgsql php5-curl php5-gd php5-intl
 php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming
 php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy
 php5-xmlrpc php5-xsl -y

 修改php.ini
 sudo vim /etc/php5/fpm/php.ini
 找到類似下面的資料，把前面的 “;" 取消，並將數值改為 0
 cgi.fix_pathinfo=0
 
 修改 www.conf 資料
 sudo vim /etc/php5/fpm/pool.d/www.conf
 找到  listen = 127.0.0.1:9000 的字樣，改成下列資料，使用 php5-fpm 服務
 listen = /var/run/php5-fpm.sock
 
 
 安装 svn
 sudo apt-get install subversion
 
 mysql 导出sql文件
 
 mysql 导入sql文件
 mysql -h localhost -uroot -p
 
 create database yurtree_20140421;
 use yurtree_20140421;
 source /home/wangsongqing/yurtree_20141001.sql
  "
