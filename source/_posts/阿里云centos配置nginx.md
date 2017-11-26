title: 阿里云Centos配置nginx
categories: 
  - 运维
tags:
  - Centos
  - nginx
date: 2017-01-24 00:17:23
author: mzeht
avatar: /images/favicon.png
---

最近写了一个自已用的微服务，放在阿里云主机上，但是带ip的接口看着还是很别扭的，于是安装nginx设置一下域名
<!-- more -->

采用的是下载源码包，编译安装的方式
# 安装依赖
make

```
yum -y install gcc automake autoconf libtool make
```

g++

```
yum install gcc gcc-c++
```

PCRE

```
yum install pcre pcre-devel
```
zlib

```
yum install zlib zlib-devel
```

OpenSSL

```
yum install openssl openssl-devel
```

# 下载源码包

```
wget https://nginx.org/download/nginx-1.10.1.tar.gz
tar zxf nginx-1.10.1.tar.gz
cd nginx-1.10.1/
```

# 安装
./configure
make
make install

# 配置
安装好后出现以下目录
```
/usr/local/nginx
ls
client_body_temp  fastcgi_temp  logs        sbin       uwsgi_temp
conf              html          proxy_temp  scgi_temp
```

conf文件夹下nginx.conf文件就是我们要动刀的配置文件

```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
##并发连接数量
    worker_connections  100;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  ******.com;##域名

        #charset koi8-r;
        
         #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://127.0.0.1:****;##服务器端口号
        }

        location / {
            proxy_pass http://127.0.0.1:10101;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


```


好了配置完成，效果是访问域名，请求被转发到服务器本地的对应端口号

然而就在我成功一次之后，阿里云就报警了，访问网页提示我没有备案，会杭州了再捣鼓备案






