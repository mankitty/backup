---
title: nginx规则匹配
date: 2018-05-18 10:19:25
tags:
categories: NGINX随笔
---

## location 匹配规则
### 具体规则
~	#波浪线表示执行一个正则匹配，区分大小写
~*	#表示执行一个正则匹配，不区分大小写
^~	#^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
=	#进行普通字符精确匹配
@	#"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
### 规则优先级
等号类型（=）的优先级最高，如果匹配成功，则不再查找其他匹配项
^~类型表达式，如果匹配成功，则不再查找其他匹配项
正则表达式类型（~ ~*）的优先级次之，如果有多个location的正则能匹配的话，则使用正则表达式最长的那个
常规字符串匹配类型，按前缀匹配

## rewrite语法
last : 相当于Apache的[L]标记，表示完成rewrite
break : 停止执行当前虚拟主机的后续rewrite指令集
redirect : 返回302临时重定向，地址栏会显示跳转后的地址
permanent : 返回301永久重定向，地址栏会显示跳转后的地址
### last和break区别
last一般写在server和if中，而break一般使用在location中
last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
break和last都能阻止继续执行后面的rewrite指令
    
## 常用正则
'.' ： 匹配除换行符以外的任意字符
'?' ： 重复0次或1次
'+' ： 重复1次或更多次
'*' ： 重复0次或更多次
'\d' ：匹配数字
'^' ： 匹配字符串的开始
'$' ： 匹配字符串的介绍
{n}' ： 重复n次
{n,} ： 重复n次或更多次
[c] ： 匹配单个字符c
[a-z] ： 匹配a-z小写字母的任意一个
注意：小括号()之间匹配的内容，可以在后面通过$1来引用，$2表示的是前面第二个()里的内容

##全局变量
下面是可以用作if判断的全局变量
$args ： #这个变量等于请求行中的参数，同$query_string
$content_length ： 请求头中的Content-length字段。
$content_type ： 请求头中的Content-Type字段。
$document_root ： 当前请求在root指令中指定的值。
$host ： 请求主机头字段，否则为服务器名称。
$http_user_agent ： 客户端agent信息
$http_cookie ： 客户端cookie信息
$limit_rate ： 这个变量可以限制连接速率。
$request_method ： 客户端请求的动作，通常为GET或POST。
$remote_addr ： 客户端的IP地址。
$remote_port ： 客户端的端口。
$remote_user ： 已经经过Auth Basic Module验证的用户名。
$request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
$scheme ： HTTP方法（如http，https）。
$server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
$server_name ： 服务器名称。
$server_port ： 请求到达服务器的端口号。
$request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri ： 与$uri相同。
### 全局变量使用举例
```nginx
http://localhost:88/test1/test2/test.php
$host：localhost
$server_port：88
$request_uri：http://localhost:88/test1/test2/test.php
$document_uri：/test1/test2/test.php
$document_root：/var/www/html
$request_filename：/var/www/html/test1/test2/test.php
```
##Rewrite使用举例
```nginx
http://newhouse.bb.house365.com/tool/tiqian/跳转到http://bb.house365.com/tool/tiqian/
nginx正则匹配$host进行跳转的方法
if ($host ~* ^newhouse\.(.+)?\.house365\.com) {
	set $host_city $1;
    rewrite ^(.*)$ http://$host_city.house365.com$1 permanent;
}
rewrite ^/images/(.*)_(\d+)x(\d+)\.(png|jpg|gif)$ /resizer/$1.$4?width=$2&height=$3? last;
```
##Location匹配规则
###规则一
```nginx
仅仅匹配"/"，若是匹配到，停止匹配
location  = / {
	[configuration A]
}
```
	
###规则二
```nginx
匹配任何请求，因为所有请求都是以"/"开始，所以所有的请求最终都会匹配到，但是更长字符匹配或者正则表达式匹配会优先匹配
location  / {
  [configuration B]
}
```
###规则三
```nginx
匹配任何以/images/开始的请求，并停止匹配其它location
location ^~ /images/ {
  [configuration C]
}
```
###规则四
```nginx
匹配以 gif, jpg, or jpeg结尾的请求，但是所有/images/目录的请求将由[Configuration C]处理，所以规则四要比规则三的优先级要低
location ~* .(gif|jpg|jpeg)$ {
  [configuration D]
}
```