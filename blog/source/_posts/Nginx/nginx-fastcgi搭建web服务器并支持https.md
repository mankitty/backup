---
title: nginx+fastcgi搭建web服务器并支持https
date: 2018-06-21 10:37:22
tags:
categories: NGINX随笔
---
## 描述
CGI全称是“公共网关接口”(Common Gateway Interface)，FastCGI与CGI同样都具有语言无关性，可以使用各种语言进行编写。
## 工作方式
### CGI
每当客户请求CGI的时候，WEB服务器就请求操作系统生成一个新的CGI解释器进程(如php-cgi.exe)，CGI 的一个进程则处理完一个请求后退出，下一个请求来时再创建新进程。
### FastCGI
[1]. Web Server启动时载入FastCGI进程管理器。
[2]. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
[3]. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。 Web server将CGI环境变量和标准输入发送到FastCGI子进程。
[4]. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时， 请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。
## 遇到的问题
### spawn-fcgi: child exited with: 127
在使用spawn-fcgi绑定的时候，绑定命令如下：spawn-fcgi -a 127.0.0.1 -p 9001 -f /appfs/web/snmpManager.cgi，遇到child exited with: 127问题，试过各种方法，最终看到一个帖子上说，可能CGI程序有问题，我就单独的执行的了一下二进制，结果发现如下问题，**./snmpManager.cgi: error while loading shared libraries: libfcgi.so.0: cannot open shared object file: No such file or directory**
解决方案：将libfcgi.so.0连接到可以让你所有需要的进程，例如，fcgi以及CGI程序都可以找到的地方
### 端口问题
原本的端口设置的为87，导致页面显示一直有问题，最后通过开发者工具查看到显示为端口不安全，更改了端口显示才正确
## spawn-fcgi绑定命令
spawn-fcgi -a 127.0.0.1 -p 9001 -f /appfs/web/snmpManager.cgi
## 相关文件下载地址
[https://pan.baidu.com/s/1xCCj63WYqcGOHWBZmuHGrw](https://pan.baidu.com/s/1xCCj63WYqcGOHWBZmuHGrw)
## nginx 编译配置如下
nginx version: nginx/1.8.1
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/usr \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--http-client-body-temp-path=/etc/nginx/client_body_temp  \
--http-proxy-temp-path=/etc/nginx/proxy_temp \
--http-fastcgi-temp-path=/etc/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/etc/nginx/uwscgi_temp \
--http-scgi-temp-path=/etc/nginx/scgi_temp \
--user=nobody --group=nobody --with-http_sub_module --with-http_stub_status_module \
--with-http_gunzip_module --with-http_ssl_module \
--add-module=nginx/nginx.auth-1.8.1/src/self/ngx_devel_kit \
--add-module=nginx/nginx.auth-1.8.1/src/self/set-misc-nginx-module \
--with-pcre=nginx.auth/pcre-8.36
## nginx 配置文件如下
```
user root root;
worker_processes auto;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    keepalive_timeout 65;
    sendfile on;
    gzip  on;

    server {
        listen 24796;
        listen 443 default ssl;
        server_name  localhost;

        ssl on;
        ssl_certificate      /appfs/etc/nginx.auth/ssl/server.crt;
        ssl_certificate_key  /appfs/etc/nginx.auth/ssl/server.key;
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

		keepalive_timeout 65;

        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        fastcgi_buffer_size 64k;
        fastcgi_buffers 4 64k;
        fastcgi_busy_buffers_size 128k;
        fastcgi_temp_file_write_size 128k;

        # CGI以及html所在的目录
        root /appfs/web;

        location / {
            index index.php index.html index.htm;
        }

        location ~ \.cgi$ {
            include        fastcgi_params;
            fastcgi_pass   127.0.0.1:9001;
            #fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
            #fastcgi_pass  unix:/tmp/php-cgi.sock;
            #try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.cgi)(/.+)$;
            fastcgi_index  snmpManager.cgi;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        }
    }
}
```

## HTTPS加密连接
OpenSSL 1.0.0生成p12、jks、crt等格式证书的命令个过程
参考文章 [OpenSSL 1.0.0生成p12、jks、crt等格式证书的命令个过程(转)](https://www.cnblogs.com/bluestorm/archive/2013/06/26/3155945.html)

#### 创建根证私钥
openssl genrsa -out root-key.key 1024
#### 创建根证书请求文件
openssl req -new -out root-req.csr -key root-key.key -keyform PEM
#### 自签根证书
openssl x509 -req -in root-req.csr -out root-cert.cer -signkey root-key.key -CAcreateserial -days 3650
#### 导出p12格式根证书
openssl pkcs12 -export -clcerts -in root-cert.cer -inkey root-key.key -out root.p12
#### 生成root.jks文件
keytool -import -v -trustcacerts -storepass 123456 -alias root -file root-cert.cer -keystore
root.jks

生成客户端文件
#### 生成客户端key
openssl genrsa -out client-key.key 1024
#### 生成客户端请求文件
openssl req -new -out client-req.csr -key client-key.key
#### 生成客户端证书（root证书，rootkey，客户端key，客户端请求文件这4个生成客户端证书）
openssl x509 -req -in client-req.csr -out client-cert.cer -signkey client-key.key -CA root-cert.cer
-CAkey root-key.key -CAcreateserial -days 3650
#### 生成客户端p12格式根证书
openssl pkcs12 -export -clcerts -in client-cert.cer -inkey client-key.key -out client.p12
#### 客户端jks：
keytool -import -v -trustcacerts -storepass 123456 -alias client -file client-cert.cer -keystore
client.jks
 
生成服务端文件
#### 生成服务端key
openssl genrsa -out server-key.key 1024
#### 生成服务端请求文件
openssl req -new -out server-req.csr -key server-key.key
#### 生成服务端证书（root证书，rootkey，客户端key，客户端请求文件这4个生成客户端证书）
openssl x509 -req -in server-req.csr -out server-cert.cer -signkey server-key.key -CA root-cert.cer
-CAkey root-key.key -CAcreateserial -days 3650
#### 生成服务端p12格式根证书
openssl pkcs12 -export -clcerts -in server-cert.cer -inkey server-key.key -out server.p12
#### 服务端JKS
keytool -import -v -trustcacerts -storepass 123456 -alias server -file server-cert.cer -keystore
server.jks
#### 无密码key命令
openssl rsa -in client-key.key -out client-key.key.unsecure