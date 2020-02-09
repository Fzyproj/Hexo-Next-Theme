---
title: 阿里云centos7.3下安装nextcloud 18.0.0最新版
tags:
  - 个人网盘
date: 2020-01-22 13:26:10
categories: 技术类
---
xx网盘容量不够大？
开会员又觉着不爽？
手头有个学生机服务器该怎么利用一下？
私人云盘nextcloud完全开源，阿里云oss TB级助力大容量满足你的欲望。
<!--more-->

> 写在前面：自己发现的一些小问题，首先是php版本如果nextcloud版本较高推荐使用最新版的php否则nextcloud服务器会判断php版本过低而报错，第二同一个服务器不要有两个业务同时占用443端口，我们都知道普通端口可以选用两个不一样的端口来分配两个业务，但443端口就一个，所以如果有其他业务也占用443端口只有两种办法，①移除多余业务释放443端口给nextcloud使用。②选取空配服务器。

## 安装并配置Nginx和php-fpm
1.查找相关的epel，nginx，pgp包
```
# rpm -qa|grep php
# rpm -qa|grep php-common
# rpm -qa|grep nginx
```
2.将自带的epel、nginx、php全部卸载，将上面查出来的每一个包名都卸载掉
```
# rpm -e 包名 --nodeps
```
3.CentOS默认的yum源中并不包含Nginx和php-fpm，首先要为CentOS添加epel源：
```
# yum -y install epel-release
```
4.安装nginx，php-fpm，php7-fpm和一些其它的必要的组件。(此处要注意18.0.0版本的nextcloud要用php7.2+版本以上的)
```
# yum -y install nginx
# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
# yum -y install php72w-fpm php72w-cli php72w-gd php72w-mcrypt php72w-mysql php72w-pear php72w-xml php72w-mbstring php72w-pdo php72w-json php72w-pecl-apcu php72w-pecl-apcu-devel
```
5.检查php是否安装成功。
```
# php -v
[info]
PHP 7.2.24 (cli) (built: Oct 26 2019 12:28:19) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
```
6.配置php-fpm
```
# vim /etc/php-fpm.d/www.conf
···
user = nginx                                   //将用户和组都改为nginx
group = nginx
···
listen = 127.0.0.1:9000                        //php-fpm所监听的端口为9000
···
env[HOSTNAME] = $HOSTNAME                     //去掉下面几行注释
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
···
```
7.在/var/lib目录下为session路径创建一个新的文件夹，并将用户名和组设为nginx
```
# mkdir -p /var/lib/php/session
# chown nginx:nginx -R /var/lib/php/session/
# ll -d /var/lib/php/session/
[info]
drwxr-xr-x. 2 nginx nginx 4096 1月  25 09:47 /var/lib/php/session/
```
8.启动Nginx和php-fpm服务，并添加开机启动
```
# systemctl start php-fpm
# systemctl start nginx
# systemctl enable php-fpm
# systemctl enable nginx
```
## 安装并配置MariaDB
1.使用MaraiDB作为Nextcloud数据库。yum安装MaraiDB服务
```
# yum -y install mariadb mariadb-server
```
2.启动MariaDB服务并添加开机启动
```
# systemctl start mariadb
# systemctl enable mariadb
```
3.设置MariaDB的root密码
```
# mysql_secure_installation //按照提示设置密码，首先会询问当前密码，密码默认为空，直接回车即可
[info]
Enter current password for root (enter for none):          //直接回车
Set root password? [Y/n] Y
New password:                                              //输入新密码
Re-enter new password:                                     //再次输入新密码
     
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
=================================================================
或者采用另一种修改密码的方式：跳过授权表
1）在/etc/my.cnf文件里添加"skip-grant-tables"
2）重启mariadb服务
3）无密码登陆mariadb，然后重置mysql密码
MariaDB [(none)]> update mysql.user set password=password("kevin@123") where user="root";
4）去掉/etc/my.cnf文件里的"skip-grant-tables"内容
5）重启mariadb服务
6）这样就可以使用上面重置的新密码kevin@123登陆mariadb了
==================================================================
设置完MariaDB的密码后，使用命令行登录MariaDB，并为Nextcloud创建相应的用户和数据库。
例如数据库为nextcloud_db，用户为nextclouduser，密码为nextcloudpasswd：
# mysql -p
......
MariaDB [(none)]> create database nextcloud_db;          
MariaDB [(none)]> create user nextclouduser@localhost identified by 'nextcloudpasswd';
MariaDB [(none)]> grant all privileges on nextcloud_db.* to nextclouduser@localhost identified by 'nextcloudpasswd';
MariaDB [(none)]> flush privileges;
```
## 为Nextcloud生成自签名SSL证书(如果自己的域名申请过ssl证书的话，可以跳过这个步骤)
```
# cd /etc/nginx/cert/
# openssl req -new -x509 -days 365 -nodes -out /etc/nginx/cert/nextcloud.crt -keyout /etc/nginx/cert/nextcloud.key
.....
Country Name (2 letter code) [XX]:cn                                           //国家
State or Province Name (full name) []:beijing                                  //省份
Locality Name (eg, city) [Default City]:beijing                                //地区名字
Organization Name (eg, company) [Default Company Ltd]:lucfzy                    //公司名
Organizational Unit Name (eg, section) []:IT                           //部门
Common Name (eg, your name or your server's hostname) []:lucfzy                 //CA主机名
Email Address []:lucfzy@163.com                                                
```
1.将证书文件的权限设置为660
```
# chmod 700 /etc/nginx/cert
# chmod 600 /etc/nginx/cert/*
```
## 下载并安装Nextcloud 18.0.0
```
# yum -y install wget unzip
# cd /usr/local/src/
# wget https://download.nextcloud.com/server/releases/nextcloud-18.0.0.zip
# unzip nextcloud-18.0.0.zip
# ls
[info]
nextcloud nextcloud-18.0.0.zip
# mv nextcloud /usr/share/nginx/html/
```
1.进入Nginx的root目录，并为Nextcloud创建data目录，将Nextcloud的用户和组修改为nginx
```
# cd /usr/share/nginx/html/
# mkdir -p nextcloud/data/
# chown nginx:nginx -R nextcloud/
# ll -d nextcloud
[info]
drwxr-xr-x. 15 nginx nginx 4096 1月  24 17:04 nextcloud
```
## 设置Nginx虚拟主机
1.进入Nginx的虚拟主机配置文件所在目录并创建一个新的虚拟主机配置（记得修改两个`server_name`为自己的域名）：
```
# cd /etc/nginx/conf.d/
# vim nextcloud.conf
···
upstream php-handler {
    server 127.0.0.1:9000;
    #server unix:/var/run/php5-fpm.sock;
}
     
server {
    listen 80;
    server_name aicloud.net.cn;
    # enforce https
    return 301 https://$server_name$request_uri;
}
     
server {
    listen 443 ssl;
    server_name aicloud.net.cn;
     
    ssl_certificate /etc/nginx/cert/nextcloud.crt;
    ssl_certificate_key /etc/nginx/cert/nextcloud.key;
     
    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    add_header Strict-Transport-Security "max-age=15768000;
    includeSubDomains; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
     
    # Path to the root of your installation
    root /usr/share/nginx/html/nextcloud/;
     
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
     
    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json
    # last;
     
    location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
     
    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;
     
    # Disable gzip to avoid the removal of the ETag header
    gzip off;
     
    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;
     
    error_page 403 /core/templates/403.php;
    error_page 404 /core/templates/404.php;
     
    location / {
        rewrite ^ /index.php$uri;
    }
     
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }
     
    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+|core/templates/40[34])\.php(?:$|/) {
        include fastcgi_params;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }
     
    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }
     
    # Adding the cache control header for js and css files
    # Make sure it is BELOW the PHP block
    location ~* \.(?:css|js)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=7200";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        add_header Strict-Transport-Security "max-age=15768000;includeSubDomains; preload;";
        add_header X-Content-Type-Options nosniff;
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
    }
     
    location ~* \.(?:svg|gif|png|html|ttf|woff|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```
2.接下来测试以下配置文件是否有错误，确保没有问题后重启Nginx服务。
```
# nginx -t
# systemctl restart nginx
```
3.关闭防火墙
```
# systemctl stop firewalld
# systemctl disable firewalld
```
## 安装Nextcloud
1.解析上面nginx中配置的域名`aicloud.net.cn`，访问访问`http://aicloud.net.cn`进行Nextcloud界面安装（访问http域名会自动跳转到https，安装提示安装即可！）
以下是推荐配置：
①创建管理员账号：账号：`admin` 密码：`admin@123`
②数据目录：`/usr/share/nginx/html/nextcloud/data`
③配置数据库：选择`mysql/mariaDB`数据库，下面一共四项分别填写：`nextclouduser`,`nextcloudpasswd`,`nextcloud_db`,`localhost`。
④点击完成安装即可。
