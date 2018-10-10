---
title: nginx sub_filter 模块
date: 2018-05-24 17:22:51
tags:
categories: NGINX随笔
---
## sub_filter简介
官方文档上有明确的介绍
The ngx_http_sub_module module is a filter that modifies a response by replacing one specified string by another.
This module is not built by default, it should be enabled with the --with-http_sub_module configuration parameter.
## 主要指令
### sub_filter_once
```nginx
Syntax:	sub_filter_once on | off;
Default:
sub_filter_once on;
Context:http, server, location
```
Indicates whether to look for each string to replace once or repeatedly.
### sub_filter_types
```nginx
Syntax:	sub_filter_types mime-type ...;
Default:
sub_filter_types text/html;
Context:http, server, location
```
Enables string replacement in responses with the specified MIME types in addition to “text/html”. The special value “*” matches any MIME type (0.8.29)
### sub_filter
Syntax:	sub_filter string replacement;
Default:	—
Context:	http, server, location
Sets a string to replace and a replacement string. The string to replace is matched ignoring the case. The string to replace (1.9.4) and replacement string can contain variables. Several sub_filter directives can be specified on one configuration level (1.9.4). These directives are inherited from the previous level if and only if there are no sub_filter directives defined on the current level.
#### Example Configuration
```nginx
location / {
    sub_filter '<a href="http://127.0.0.1:8080/'  '<a href="https://$host/';
    sub_filter '<img src="http://127.0.0.1:8080/' '<img src="https://$host/';
    sub_filter_once on;
}
```
### sub_filter_last_modified
Syntax:	sub_filter_last_modified on | off;
Default:
sub_filter_last_modified off;
Context:	http, server, location
This directive appeared in version 1.5.1.

Allows preserving the “Last-Modified” header field from the original response during replacement to facilitate response caching.

By default, the header field is removed as contents of the response are modified during processing.