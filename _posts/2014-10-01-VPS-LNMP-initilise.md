---
layout: post
title: "阿里云 Openresty+redis+PHP+Mysql环境搭建"
description: "Vagrant 本质上来说，是对 virtualbox，vmware，kvm 等镜像的管理操作，是一个中间层技术。使用它的前提是你本机必须有 virtualbox，vmware，kvm 等虚拟机。"
keywords: "ngx_openresty, 配置, 使用"
category: Linux
tags: [ngx_openresty, redis, LNMP]
---

#### 一.准备工作

##### 1.环境介绍

阿里云购买了入门级 ECS，配置如下:

    CPU: Intel(R) Xeon(R) CPU E5-2630 0 @ 2.30GHz * 2
    Memory: 4G
    Disk: 40G
    OS: Ubuntu 14.04 LTS


##### 2.设置 swap 分区

必须吐槽下阿里云坑爹的设置，默认安装好系统没有帮忙设置 swap 分区。

    $ sudo dd if=/dev/zero of=swapfile bs=4M count=1024 # 创建 4G 大小的文件（建议和内存一样）。
    $ sudo mkswap swapfile # 生成 swap 格式。
    $ sudo swapon swapfile # 挂载 swap 分区。
    $ sudo vim /etc/fstab  # 设置 fstab, 使 swap 分区开机就默认被挂载。
    => 添加
    /root/swap/swapfile swap swap defaults 0 0
    
##### 3.设置权限

nginx 和 PHP-fpm 均已 www 用户启动，便于权限管理控制。

如果 nginx 和 PHP-fpm 用户不一致，会导致 PHP-fpm 接收不到 Nginx proxy_pass 的 sock，导致 502 错误；

    $ useradd -M -s /sbin/nologin www

<!-- more -->

#### 二.安装步骤

##### 1.安装nginx_openresty(with luajit)

    $ sudo apt-get install libreadline-dev libpcre3-dev libssl-dev perl build-essential -y
    $ wget http://openresty.org/download/ngx_openresty-1.7.2.1.tar.gz
    $ tar xvf ngx_openresty-1.7.2.1.tar.gz
    $ cd ngx_openresty-1.7.2.1
    $ ./configure --with-luajit --prefix=/usr/local/openresty
    $ sudo make && sudo make install
    $ ln -s /usr/local/openresty/nginx/sbin/nginx /usr/sbin/nginx
    
openresty daemon sample 配置如下:
 
- <https://github.com/hilllinux/conf/blob/master/nginx>

将配置内容另存为 /etc/init.d/nginx
 
    $ sudo update-rc.d -f nginx defaults      # 设置开机启动
    $ sudo service nginx (start/stop/reload)  # nginx 命令
    
nginx conf 参考配置:

    server {  
        listen 80;  
        
        gzip on; 
        gzip_min_length 1k; 
        gzip_buffers 4 16k; 
        gzip_comp_level 2; 
        gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; 
        gzip_vary off; 
        gzip_disable "MSIE [1-6]\."; 
        root /var/www;
        index index.html index.htm index.php;    
        
        location / {     
            try_files $uri $uri/ /index.php?p=$uri&$args;  
        }   

        location ~ \.php$ {
            try_files $uri =404;
            include fastcgi.conf;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
        }
        
        location /lua {
            content_by_lua_file conf/lua/redis_lua_test.lua;
        }
        
    }
    
redis_lua _test.lua :

```lua
    local cmd = tostring(ngx.var.arg_cmd)
    local key = tostring(ngx.var.arg_key)
    local val = tostring(ngx.var.arg_val)
    local commands = {
            get="get",
            set="set"
    }
    cmd = commands[cmd]
    if not cmd then
            ngx.say("command not found!")
            ngx.exit(400)
    end

    local redis = require("resty.redis")
    local red = redis:new()

    red:set_timeout(1000) -- 1 second

    local ok,err = red:connect("127.0.0.1",6379)
    if not ok then
            ngx.say("failed to connect: ",err)
            return
    end
    if cmd == "get" then
            if not key then ngx.exit(400) end
            local res,err = red:get(key)
            if not res then
                    ngx.say("failed to get ",key,": ",err)
                    return
            end
            ngx.say(res)
    end

    if cmd == "set" then
            if not (key and val) then ngx.exit(400) end
            local ok,err = red:set(key,val)
            if not ok then
                    ngx.say("failed to set ",key,": ",err)
                    return
            end
            ngx.say(ok)
    end
```
    

##### 2.安装redis

    $ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
    $ tar xvf redis-2.8.17.tar.gz 
    $ cd redis-2.8.17/
    $ sudo make && sudo make install
    
修改redis.conf，配置按照需求修改；

    daemonize yes
    
redis daemon sample 配置如下:

- <https://github.com/hilllinux/conf/blob/master/redis>

将配置内容另存为 /etc/init.d/redis
 
    $ sudo update-rc.d -f redis defaults      # 设置开机启动
    $ sudo service nginx (start/stop)         # redis 命令

##### 3.安装PHP-fpm 和mysql

    $ sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql fcgiwrap php5-fpm php5-xcache php5-pgsql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl -y

修改php.ini

    $ sudo vim /etc/php5/fpm/php.ini
    ;cgi.fix_pathinfo=0 => cgi.fix_pathinfo=1
                                  
修改 www.conf                

    $ sudo vim /etc/php5/fpm/pool.d/www .conf
    listen = 127.0.0.1:9000 => listen = /var/run/php5- fpm.sock
    
mysql 导入sql文件                 

    $ mysql -h localhost -uroot -p      
    create database xxxx; 
    use xxxx;             
    source /home/wangsongqing/xxxx_backup.sql 
                                  
    
##### 4.重启服务

    $ sudo service nginx restart
    $ sudo service redis restart
    $ sudo service mysql restart
    
#### 三.压力测试
    
使用 apache_ab 工具进行并非和多连接的压力测试    

    $ sudo apt-get install apache2-utils
    
    $ ab -kc 1000 -n 1000 http://localhost/lua?cmd=set&key=a&val=b
    $ ab -kc 1000 -n 1000 http://localhost/lua?cmd=get&key=a
    
c 代表并发数量，n 代表连接次数。

