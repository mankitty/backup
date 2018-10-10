---
layout: nginx
title: Nginx代理模块
date: 2018-05-28 18:34:55
tags:
categories: NGINX随笔
---
## proxy基本设置
```nginx
server {
	listen       24742;

    resolver 127.0.0.1;
	server_name  localhost;

        location = / {
            proxy_pass http://xxx/yy/index.htm;
            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }

        location ~* (.*)\.apk {
            proxy_pass http://xxx;
            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
    }
参数解释：
proxy_pass
```
## 常用参数
### proxy_buffer_size
语法: proxy_buffer_size the_size
默认值: proxy_buffer_size 4k/8k
上下文: http, server, location
该指令设置缓冲区大小,从被代理的后端服务器取得的响应内容,会先读取放置到这里
### proxy_set_header    X-real-ip $remote_add
含义：通过X-real-ip可以获取到客户端真正的IP地址
### proxy_add_x_forwarded_for
NGINX even provides a $proxy_add_x_forwarded_for variable to automatically append $remote_addr to any incoming X-Forwarded-For headers.
Traditionally, an HTTP reverse proxy uses non-standard headers to inform the upstream server about the user’s IP address and other request properties:

X-Forwarded-For: 12.34.56.78, 23.45.67.89
X-Real-IP: 12.34.56.78
X-Forwarded-Host: example.com
X-Forwarded-Proto: https
## Examples
sub_filter_once on;
sub_filter <body 'www.baidu.com<body';