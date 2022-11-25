---
title: 云主机部署并同步更新二级域名
author: mykk
top: false
cover: false
toc: true
mathjax: false
summary: 建站初期是托管在github二级域名底下，现在独立域名，采用的是nginx反向代理+hexo+shell
tags:
  - blog
  - 部署
  - 小姿势
categories:
  - 博客
abbrlink: 829b453d
reprintPolicy: cc_by
date: 2022-03-20 16:17:13
coverImg:
img:
password:
---



## 一、本地拉取配置

### 1、创建并启动

#### 1、拉取

```docker
docker pull nginx
```



#### 2、启动

```docker
docker run --name nginx-test -p 80:80 -d nginx
```

-- name 容器命名

-v 映射目录

-d 设置容器后台运行

-p 本机端口映射 将容器的80端口映射到本机的80端口







### 2、映射到本地



#### 1、创建

首先在本机创建nginx的一些文件存储目录

```linux
mkdir -p /root/nginx/www /root/nginx/logs /root/nginx/conf
```



**www**: nginx存储网站网页的目录

**logs**: nginx日志目录

**conf**: nginx配置文件目录





#### 2、映射

（1）先查看容器

```docker
docker ps -a
```



（2）映射

```docker
docker cp 481e121fb29f:/etc/nginx/nginx.conf /root/nginx/conf
```



### 3、启动容器

需要说明下，ngxin-test 容器是为了获得容器的配置文件，最终使用的是 nginx-web

目前已经启动 nginx-test 80端口，若是 nginx-web指定的也是 80，就需要关闭 nginx-test了

```docker
docker stop nginx-test
```



#### 1、新容器映射

创建新nginx容器nginx-web,并将**www,logs,conf**目录映射到本地

```docker
docker run -d -p 80:80 --name nginx-web -v /root/nginx/www:/usr/share/nginx/html -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/logs:/var/log/nginx nginx
```



#### 2、启动

```docker 
docker start nginx-web
```



### 



## 二、下载git

### 1、下载

```bash
yum install git
```



### 2、配置github 代理

```git
git config --global url."https://ghproxy.com/https://github.com".insteadOf "https://github.com"
```



### 3、拉取

```git
git clone  https://github.com/mykkTo/mykkTo.github.io.git
```



### 4、剪切文件

```bash
mv /root/nginx/www/mykkTo.github.io/* /root/nginx/www
```







## 三、定时任务

### 1、编写shell脚本

```shell
#!/bin/bash
#删除原始静态页面数据以及拉取的文件夹
rm -rf /root/nginx/www/*
rm -rf /root/nginx/mykkTo.github.iopwd

#拉取，剪切到80映射下
cd  /root/nginx/
git clone https://github.com/mykkTo/mykkTo.github.io.git
mv /root/nginx/mykkTo.github.io/* /root/nginx/www
#复制令牌用户百度站长验证使用
cp /root/nginx/baidu_verify_code-Os7hLX61vV.html /root/nginx/www/baidu_verify_code-Os7hLX61vV.html
#替换文本，用于百度站长seo映射
sed 's/github.io/cn/g' /root/nginx/www/baidu_urls.txt>/root/nginx/www/baidu_urls1.txt
sed 's/https/http/g' /root/nginx/www/baidu_urls1.txt>/root/nginx/www/baidu_urls2.txt
echo "============成功========="
```



### 2、创建定时任务



```bash
crontab -e
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220321202335.png)



**补充：**

linux 黑洞，防止资源占用

`/dev/null 2>&1`

```bash

#后缀加上
/root/nginx/synblog.sh /dev/null 2>&1
```



**启动：**

这边设置没一个小时更新一次（需要注意centos7.6写法启动）

```bash
service crond start
```



## 四、SSL证书

### 1、是什么

数据加密：开启 HTTPS 绿色加密通道，网站数据的加密传输，防止网站核心数据被窃取或篡改。

简单来说，就是把 http 访问的变成 https，并且浏览器显示安全，不在是不安全了



### 2、获取证书

#### 1、下载

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331205350.png)



![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331205411.png)



#### 2、上传到服务器并解压

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331205527.png)



### 3、重挂载

#### 1、删掉之前的容器

```docker
docker rm -f nginx-web
```



#### 2、重新挂载

1、新增了 443端口映射，目录挂载



2、容器外，在nginx底下，创建新目录

```bash
mkdir   lls
```

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331204305.png)



3、创建容器

多加了两个配置

```docker
docker run -d -p 443:443 -p 80:80 --name nginx-web -v /root/nginx/www:/usr/share/nginx/html -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/logs:/var/log/nginx -v /root/nginx/lls/:/etc/nginx/ssl nginx
```



### 4、修改配置

nginx.conf

```nginx

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

   include /etc/nginx/conf.d/*.conf;

  server{
        listen 443 ssl;
        #对应的域名，把mykkto.cn改成你们自己的域名就可以了
        server_name mykkto.cn;
        #证书的两个配置文件
        ssl_certificate /etc/nginx/ssl/7526194_www.mykkto.cn.pem;
        ssl_certificate_key /etc/nginx/ssl/7526194_www.mykkto.cn.key;
		#以下都是一些加密规则
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        #这是我的主页访问地址，因为使用的是静态的html网页，所以直接使用location就可以完成了。
        location / {
                #文件夹（这个其实挂载的就是外部的www目录下的静态资源）
                root //usr/share/nginx/html;
                #主页文件
                index index.html;
        }
    }

server {
   listen 80;
        #这边空格隔开，配置了两个，因为加了www也要配置
       server_name mykkto.cn www.mykkto.cn;
       rewrite ^/(.*)$ https://mykkto.cn:443/$1 permanent;
  # location / {
  #     limit_req zone=mylimit;
  #}
}
}
```





### 5、重启并测试

#### 1、重启

```docker
docker restart nginx-web
```



#### 2、测试

1、访问，www.mykkto.cnm，自动跳转

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331204859.png)



2、访问，mykkto.cn，自动跳转

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220331204919.png)





## 五、IP黑名单限制

### 1、前言

#### 1、用到什么技术栈

首先，基本架构是  docker+nginx+lua+mysql这是最初的想法，但是作者用了docker搭建的nginx，lua模块集成不是很方便，所以替换成了，OpenResty。

#### 2、什么是 OpenResty

简单来说就是 lua + nginx，当然还有更多功能，自己百度吧



#### 3、遇到的问题

简单描述下，上面用到的 `nginx.conf`写法，前缀 `user  nginx;` 会导致无法运行，因为openresty没有这个用户，可以自己新建，我采用的是改写配置。



### 2、搭建 OpenResty

#### 1、拉取

```docker
docker pull openresty/openresty
```



#### 2、挂载并启动

**说明下：**

- 完全基于上面的nginx配置的三个部分，唯一修改的是 `nginx.conf`配置文件，这边挂载改成了 `nginx2.conf`，用于保留之前的（自己懒而已）
- 新增了，lua文件挂载
- 修改了容器内的挂载位置，因为 openresty容器位置不一样了(外部还是不变，容器内的位置变了)

```docker
docker run -d -p 443:443 -p 80:80 --name openresty -v /root/nginx/www:/usr/local/openresty/nginx/html -v /root/nginx/conf/nginx2.conf:/usr/local/openresty/nginx/conf/nginx.conf -v /root/nginx/logs:/usr/local/openresty/nginx/logs -v  /root/nginx/lls/:/usr/local/openresty/nginx/ssl -v  /root/nginx/lua/:/usr/local/openresty/nginx/lua  openresty/openresty
```



### 3、配置文件修改

#### 1、初始化

这是自带的，可以自己  DIY

```nginx.conf
# nginx.conf  --  docker-openresty
#
# This file is installed to:
#   `/usr/local/openresty/nginx/conf/nginx.conf`
# and is the file loaded by nginx at startup,
# unless the user specifies otherwise.
#
# It tracks the upstream OpenResty's `nginx.conf`, but removes the `server`
# section and adds this directive:
#     `include /etc/nginx/conf.d/*.conf;`
#
# The `docker-openresty` file `nginx.vh.default.conf` is copied to
# `/etc/nginx/conf.d/default.conf`.  It contains the `server section
# of the upstream `nginx.conf`.
#
# See https://github.com/openresty/docker-openresty/blob/master/README.md#nginx-config-files
#

#user  nobody;
#worker_processes 1;

# Enables the use of JIT for regular expressions to speed-up their processing.
pcre_jit on;



#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    # Enables or disables the use of underscores in client request header fields.
    # When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the ignore_invalid_headers directive.
    # underscores_in_headers off;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

        # Log in JSON Format
        # log_format nginxlog_json escape=json '{ "timestamp": "$time_iso8601", '
        # '"remote_addr": "$remote_addr", '
        #  '"body_bytes_sent": $body_bytes_sent, '
        #  '"request_time": $request_time, '
        #  '"response_status": $status, '
        #  '"request": "$request", '
        #  '"request_method": "$request_method", '
        #  '"host": "$host",'
        #  '"upstream_addr": "$upstream_addr",'
        #  '"http_x_forwarded_for": "$http_x_forwarded_for",'
        #  '"http_referrer": "$http_referer", '
        #  '"http_user_agent": "$http_user_agent", '
        #  '"http_version": "$server_protocol", '
        #  '"nginx_access": true }';
        # access_log /dev/stdout nginxlog_json;

    # See Move default writable paths to a dedicated directory (#119)
    # https://github.com/openresty/docker-openresty/issues/119
    client_body_temp_path /var/run/openresty/nginx-client-body;
    proxy_temp_path       /var/run/openresty/nginx-proxy;
    fastcgi_temp_path     /var/run/openresty/nginx-fastcgi;
    uwsgi_temp_path       /var/run/openresty/nginx-uwsgi;
    scgi_temp_path        /var/run/openresty/nginx-scgi;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    # Don't reveal OpenResty version to clients.
    # server_tokens off;
}
```



#### 2、引入之前的配置

在初始的基础上加上，两个之前写好的 server 块，以及lua脚本用于测试

```lua
# nginx.conf  --  docker-openresty
#
# This file is installed to:
#   `/usr/local/openresty/nginx/conf/nginx.conf`
# and is the file loaded by nginx at startup,
# unless the user specifies otherwise.
#
# It tracks the upstream OpenResty's `nginx.conf`, but removes the `server`
# section and adds this directive:
#     `include /etc/nginx/conf.d/*.conf;`
#
# The `docker-openresty` file `nginx.vh.default.conf` is copied to
# `/etc/nginx/conf.d/default.conf`.  It contains the `server section
# of the upstream `nginx.conf`.
#
# See https://github.com/openresty/docker-openresty/blob/master/README.md#nginx-config-files
#

#user  nobody;
#worker_processes 1;

# Enables the use of JIT for regular expressions to speed-up their processing.
pcre_jit on;



#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    # Enables or disables the use of underscores in client request header fields.
    # When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the ignore_invalid_headers directive.
    # underscores_in_headers off;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

        # Log in JSON Format
        # log_format nginxlog_json escape=json '{ "timestamp": "$time_iso8601", '
        # '"remote_addr": "$remote_addr", '
        #  '"body_bytes_sent": $body_bytes_sent, '
        #  '"request_time": $request_time, '
        #  '"response_status": $status, '
        #  '"request": "$request", '
        #  '"request_method": "$request_method", '
        #  '"host": "$host",'
        #  '"upstream_addr": "$upstream_addr",'
        #  '"http_x_forwarded_for": "$http_x_forwarded_for",'
        #  '"http_referrer": "$http_referer", '
        #  '"http_user_agent": "$http_user_agent", '
        #  '"http_version": "$server_protocol", '
        #  '"nginx_access": true }';
        # access_log /dev/stdout nginxlog_json;

    # See Move default writable paths to a dedicated directory (#119)
    # https://github.com/openresty/docker-openresty/issues/119
    client_body_temp_path /var/run/openresty/nginx-client-body;
    proxy_temp_path       /var/run/openresty/nginx-proxy;
    fastcgi_temp_path     /var/run/openresty/nginx-fastcgi;
    uwsgi_temp_path       /var/run/openresty/nginx-uwsgi;
    scgi_temp_path        /var/run/openresty/nginx-scgi;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
	
  server{
        listen 443 ssl;
        #对应的域名，把mykkto.cn改成你们自己的域名就可以了
        server_name mykkto.cn;
        #证书的两个配置文件
        ssl_certificate /usr/local/openresty/nginx/ssl/7526194_www.mykkto.cn.pem;
        ssl_certificate_key /usr/local/openresty/nginx/ssl/7526194_www.mykkto.cn.key;
		#以下都是一些加密规则
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        #这是我的主页访问地址，因为使用的是静态的html网页，所以直接使用location就可以完成了。
        location / {
                #文件夹（这个其实挂载的就是外部的www目录下的静态资源）
                root /usr/local/openresty/nginx/html;
                #主页文件
                index index.html;
        }
		
				#lua脚本用于测试
		location /lua {
   default_type 'text/html';
   content_by_lua 'ngx.say("<h1> hello,openrestry</h1>")';
   }	
 }

server {
   listen 80;
        #这边空格隔开，配置了两个，因为加了www也要配置
       server_name mykkto.cn www.mykkto.cn;
       rewrite ^/(.*)$ https://mykkto.cn:443/$1 permanent;
  # location / {
  #     limit_req zone=mylimit;
  #}
}

    # Don't reveal OpenResty version to clients.
    # server_tokens off;
}

```



#### 3、测试lua脚本

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220403120349.png)



## 六、限流

### 1、配置

```bash
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=5r/s;
    server {
        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
}
```

**limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;**

- 第一个参数：`$binary_remote_addr` 表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址。
- 第二个参数：`zone=one:10m`表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息。
- 第三个参数：`rate=5r/s`表示允许相同标识的客户端的访问频次，这里限制的是每秒 5 次，还可以有比如30r/m的。



**limit_req zone=one burst=5 nodelay;**

- 第一个参数：`zone=one` 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应。
- 第二个参数：`burst=5`，重点说明一下这个配置，burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
- 第三个参数：`nodelay`，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。



### 2、网站配置源码

```bash
#user  nobody;
#worker_processes 1;

# Enables the use of JIT for regular expressions to speed-up their processing.
pcre_jit on;



#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
        server_tokens off;
        #引入lib包
    lua_package_path "/usr/local/openresty/lualib/?.lua;;";
        #开辟一块内存区域
        lua_shared_dict ip_blacklist 4m;

    # Enables or disables the use of underscores in client request header fields.
    # When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the ignore_invalid_headers directive.
    # underscores_in_headers off;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

        # Log in JSON Format
        # log_format nginxlog_json escape=json '{ "timestamp": "$time_iso8601", '
        # '"remote_addr": "$remote_addr", '
        #  '"body_bytes_sent": $body_bytes_sent, '
        #  '"request_time": $request_time, '
        #  '"response_status": $status, '
        #  '"request": "$request", '
        #  '"request_method": "$request_method", '
        #  '"host": "$host",'
        #  '"upstream_addr": "$upstream_addr",'
        #  '"http_x_forwarded_for": "$http_x_forwarded_for",'
        #  '"http_referrer": "$http_referer", '
        #  '"http_user_agent": "$http_user_agent", '
        #  '"http_version": "$server_protocol", '
        #  '"nginx_access": true }';
        # access_log /dev/stdout nginxlog_json;

    # See Move default writable paths to a dedicated directory (#119)
    # https://github.com/openresty/docker-openresty/issues/119
    client_body_temp_path /var/run/openresty/nginx-client-body;
    proxy_temp_path       /var/run/openresty/nginx-proxy;
    fastcgi_temp_path     /var/run/openresty/nginx-fastcgi;
    uwsgi_temp_path       /var/run/openresty/nginx-uwsgi;
    scgi_temp_path        /var/run/openresty/nginx-scgi;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

#限流设置
limit_req_zone $binary_remote_addr zone=one:30m rate=10r/s;

         server{
        listen 443 ssl;
        #对应的域名，把mykkto.cn改成你们自己的域名就可以了
        server_name mykkto.cn;
        #证书的两个配置文件
        ssl_certificate /usr/local/openresty/nginx/ssl/7526194_www.mykkto.cn.pem;
        ssl_certificate_key /usr/local/openresty/nginx/ssl/7526194_www.mykkto.cn.key;
                #以下都是一些加密规则
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        #这是我的主页访问地址，因为使用的是静态的html网页，所以直接使用location就可以完成了。

                set $real_ip $remote_addr;
  if ( $http_x_forwarded_for ~ "^(\d+\.\d+\.\d+\.\d+)" ) {
    set $real_ip $1;
  }

          # 管理信息，访问该URL可以查看nginx中的IP黑名单信息
  location /get-ipblacklist-info {
    access_by_lua_file /usr/local/openresty/nginx/lua/get_ipblacklist_info.lua;
  }


        # 同步URL，通过定时任务调用该URL,实现IP黑名单从mysql到nginx的定时刷新
  location /sync-ipblacklist {
   access_by_lua_file /usr/local/openresty/nginx/lua/sync_ipblacklist.lua;
  }

        location / {
#限流
limit_req zone=one burst=10 nodelay;
                                # 所有IP进来都要校验
                                access_by_lua_file /usr/local/openresty/nginx/lua/check_realip.lua;
#                               proxy_read_timeout  60s;
#                               proxy_set_header    Host $http_host;
#                               proxy_set_header    X-Real_IP $remote_addr;
#                               proxy_set_header    X-Forwarded-for $remote_addr;
#                               proxy_http_version  1.1;
                #文件夹（这个其实挂载的就是外部的www目录下的静态资源）
                root /usr/local/openresty/nginx/html;
                #主页文件
                index index.html;
        }
 }
 
 
 server {
   listen 80;
        #这边空格隔开，配置了两个，因为加了www也要配置
       server_name mykkto.cn www.mykkto.cn;
       rewrite ^/(.*)$ https://mykkto.cn:443/$1 permanent;
  # location / {
  #     limit_req zone=mylimit;
  #}

}

    # Don't reveal OpenResty version to clients.
    # server_tokens off;
}
 

```



### 3、测试



QPS：3000压测 5次

![](https://v1.mykkto.cn/image/blog/2022/springcloud/20220410235912.png)





压测的IP为  A，通过，同网段的机器访问（手机模拟），503异常，结论被限流成功

![](https://v1.mykkto.cn/image/blog/2022/springcloud/testfsjhdgjdhfjk1.jpg)





切换网段B ，非压测IP 不限流，测试成功

![](https://v1.mykkto.cn/image/blog/2022/springcloud/fkdgjgdhjf56jh7j6h.jpg)





## 七、DNS解析+Github Pages（无服务器化）

前提：先买个DNS解析，博主目前域名和DNS解析都是买阿里云的

### 1、DNS解析域名

这个是初版1.0，错误的，因为 主机记录解析  * 可能覆盖后面床图指向的 v1 配置

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211051308830.png)



正确配置：最新版！！！

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222248798.png)



### 2、Github Pages

进入自己二级静态网站仓库，配置（如果有配SSL证书的打勾，没有不需要打勾 HTTPS）

关于证书：后面搭建床图也需要指向，不然无法加载图片（小坑）

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211051309517.png)



## 八、Nginx搭建床图

基于dns解析  +  Github pages 映射适配

传统搭建方式（http）,会导致在  img 上渲染图片的时候无法显示（谷歌浏览器，IE浏览器会，360极速不会--具体不清楚），

因为 pages 开启了 https 所以，自己搭建的床图也要是  https （我是这么想的，然后实现，确实可以了）



分析：一开始看到 console 控制台， 一堆图片报错 ，于是拿连接去请求，发现不可以访问（带http，无 s）



### 1、配置域名解析

初版1.0，废弃，最好不要用  _  为前缀，因为后面要是有配置SSL证书，就无法通过

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211121142179.png)

最新2.0配置，完美解决了所有问题

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222248798.png)



### 2、配置SSL证书

购买证书，这边作者用的是免费的

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222259557.png)



然后下载部署，具体配置信息 4  nginx.conf

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222301426.png)





### 3、搭建具体参考 一、四

用了docker



### 4、补充

#### 1、nginx 配置

```conf
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

   include /etc/nginx/conf.d/*.conf;

  server{
        listen 443 ssl;
        #对应的域名，把mykkto.cn改成你们自己的域名就可以了
        server_name mykkto.cn;
        #证书的两个配置文件
        ssl_certificate /etc/nginx/ssl/8799443_v1.mykkto.cn.pem;
        ssl_certificate_key /etc/nginx/ssl/8799443_v1.mykkto.cn.key;
		#以下都是一些加密规则
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        #这是我的主页访问地址，因为使用的是静态的html网页，所以直接使用location就可以完成了。
        location / {
                #文件夹（这个其实挂载的就是外部的www目录下的静态资源）
                root //usr/share/nginx/html;
                #主页文件
                index index.html;
        }
    }

server {
   listen 80;
        #这边空格隔开，配置了两个，因为加了www也要配置
       server_name mykkto.cn;
       rewrite ^/(.*)$ https://mykkto.cn:443/$1 permanent;
  # location / {
  #     limit_req zone=mylimit;
  #}
}
}

```



#### 2、更新床图脚本

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222303115.png)



#### 3、PICGO 配置

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222304590.png)

![](https://v1.mykkto.cn/image/blog/2022/Interview/202211222303839.png)



## 参考 ↓

------ 【一到三】参考-----------

https://www.cnblogs.com/zltech/p/13517231.html

https://www.cnblogs.com/thepoy/p/14848080.html

https://www.cnblogs.com/jianqingwang/p/6726589.html

------ 【一到三】参考-----------



SSL证书参考>>>>>>>>>>>>>>>>>>>>>>>

https://www.cnblogs.com/zeussbook/p/11231820.html

https://www.cnblogs.com/yuyeblog/p/13582127.html

https://www.cnblogs.com/makalochen/p/14241052.html#%E5%A4%9A%E7%9B%AE%E5%BD%95%E6%8C%82%E8%BD%BD



IP黑名单参考>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

https://www.jb51.net/article/168907.htm

https://blog.csdn.net/weixin_33971205/article/details/89861486

https://www.csdn.net/tags/MtTaIgzsNDYzNDUtYmxvZwO0O0OO0O0O.html



限流>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

https://www.cnblogs.com/biglittleant/p/8979915.html



DNS解析+Github Pages（无服务器）

https://cloud.tencent.com/developer/article/2019284