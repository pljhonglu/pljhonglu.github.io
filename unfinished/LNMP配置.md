# mac 安装 nginx+mysql+php

## nginx

```
brew install nginx
```

安装目录

```
/usr/local/etc/nginx
```

配置文件：`/usr/local/etc/nginx/nginx.conf`

nginx 默认使用8080端口，由于没有 root 权限，所以无法使用80端口，可以通过下面命令给 nginx 开启 root 权限

```
sudo chown root:wheel /usr/local/opt/nginx/bin/nginx
sudo chmod u+s /usr/local/opt/nginx/bin/nginx
```

启动 

```
sudo nginx
```

## mysql

```
brew install mysql
```

开启关闭

```
mysql.server start
mysql.server stop
```

开启 MySQL 安全机制

```
/usr/local/opt/mysql/bin/mysql_secure_installation
```
根据终端提示，输入 root 密码，依次确认一些安全选项，可以[参考这里](http://blog.frd.mn/install-nginx-php-fpm-mysql-and-phpmyadmin-on-os-x-mavericks-using-homebrew/)

## php

先添加 php 扩展库

```
brew update
brew tap homebrew/dupes
brew tap josegonzalez/homebrew-php
```

安装

```
brew install php56
```

修改环境变量

```
echo 'export PATH="$(brew --prefix php56)/bin:$PATH"' >> ~/.bash_profile  #for php
echo 'export PATH="$(brew --prefix php56)/sbin:$PATH"' >> ~/.bash_profile  #for php-fpm
echo 'export PATH="/usr/local/bin:/usr/local/sbib:$PATH"' >> ~/.bash_profile #for other brew install soft
source ~/.bash_profile
```

查看 php 和 php-fpm 版本

```
# 系统默认 php 和 php-fpm 版本
php -v 
php-fpm -v

# mac 自带 php 和 php-fpm 版本
/usr/bin/php -v 
/usr/sbin/php-fpm -v
```

修改 php-fpm 配置文件，`vim /usr/local/etc/php/5.5/php-fpm.conf`，找到pid相关大概在25行，去掉注释 `pid = run/php-fpm.pid`, 那么 php-fpm 的 pid 文件就会自动产生在 `/usr/local/var/run/php-fpm.pid`，下面要安装的 Nginx pid 文件也放在这里。

```
#测试php-fpm配置
php-fpm -t
php-fpm -c /usr/local/etc/php/5.5/php.ini -y /usr/local/etc/php/5.5/php-fpm.conf -t

#启动php-fpm
php-fpm -D
php-fpm -c /usr/local/etc/php/5.5/php.ini -y /usr/local/etc/php/5.5/php-fpm.conf -D
```

如果提示权限问题，可以使用 root 权限启动

```
sudo php-fpm -D
```

查看 php-fpm 进程

```
ps aux | grep php
```

## 修改 nginx 配置

`vi /usr/local/etc/nginx/nginx.conf`

修改下面部分

```
location ~ \.php$ {
    try_files                   $uri = 404;
    fastcgi_pass                127.0.0.1:9000;
    fastcgi_index               index.php;
    fastcgi_intercept_errors    on;
    include /usr/local/etc/nginx/fastcgi.conf;
}
```

打开 nginx 的 log 功能

```
worker_processes  1;

error_log   /usr/local/var/logs/nginx/error.log debug;


pid        /usr/local/var/run/nginx.pid;


events {
    worker_connections  256;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /usr/local/var/logs/access.log  main;

    sendfile        on;
    keepalive_timeout  65;
    port_in_redirect off;

    include /usr/local/etc/nginx/sites-enabled/*;
}
```

## 使用

开启 php-fpm : `sudo php-fpm -D`
开启 nginx : `nginx`
开启 MySQL : `mysql.server start`

## 参考

[Mac OS使用brew安装Nginx、MySQL、PHP-FPM的LAMP开发环境](http://tabalt.net/blog/install-nginx-mysql-php-fpm-by-brew-on-mac/)
[全新安装Mac OSX 开发者环境 同时使用homebrew搭建 PHP，Nginx ，MySQL，Redis，Memcache ... ... (LNMP开发环境)](https://segmentfault.com/a/1190000000606752#articleHeader6)
[Mac下安装LNMP(Nginx+PHP5.6)环境](http://avnpc.com/pages/install-lnmp-on-osx)

