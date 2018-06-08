---
title: nginx简介
category: linux
tags:
- linux
---

## 介绍

nginx作为一个web服务器，同时具有强大的反向代理和负载均衡能力，本文将对nginx的反向代理和负载均衡进行介绍。

<!--more-->

## 反向代理

### 概念理解

正向代理隐藏了客户端，反向代理隐藏了服务端:
- 正向代理: 允许客户端通过它访问任意网站并且隐藏客户端自身

```
Client ------\
              \
               \
Client ------  Proxy ------ Server
                /
               /
Client ------ /
```

- 反向代理: 对外都是透明的，访问者并不知道自己访问的是一个代理

```
Client ------\         /------ Server
              \       /
               \     /
Client ------   Proxy ------ Server
                /   \
               /     \
Client ------ /       \ ------ Server
```

### 文件配置

在http里面添加：

```
upstream node-server{
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name test.com;
    access_log /var/log/nginx/node-server;

    # Gzip Compression
    gzip on;
    gzip_comp_level 6;
    gzip_vary on;
    gzip_min_length  1000;
    gzip_proxied any;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_buffers 16 8k;

    # 反向代理 node-server
    location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy true;
      # 代理的地址
      proxy_pass http://node-server;
      proxy_redirect off;
    }
 }
```

在/etc/hosts文件中增加"127.0.0.1 test.com"，然后访问test.com:80端口，会自动调转到127.0.0.1:3000


## 负载均衡

负载均衡有多种策略，默认最简单的示例：

```
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}

```


## 实际配置

实际使用的测试配置文件：

```
# cat /etc/nginx/nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
}

stream {
    upstream apiserver {
        server 192.168.60.13:6443 weight=5 max_fails=3 fail_timeout=30s;
        server 192.168.60.14:6443 weight=5 max_fails=3 fail_timeout=30s;
        server 192.168.60.15:6443 weight=5 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 16443;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass apiserver;
    }
}
```


## 参考
- [https://nginx.org/en/docs/http/load_balancing.html](https://nginx.org/en/docs/http/load_balancing.html)
- [https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)
- [https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing)
- [https://blog.csdn.net/v1t1p9hvbd/article/details/72152482](https://blog.csdn.net/v1t1p9hvbd/article/details/72152482)
