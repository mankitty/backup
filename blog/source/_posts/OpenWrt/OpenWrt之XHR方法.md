---
title: OpenWrt之XHR方法
date: 2018-06-22 19:14:46
tags:
---
## 描述
XHR方法定义在xhr.js文件中，其中定义了get,post,poll,run等方法，其中比较常用的是poll以及get,post方法
## 方法简介
### poll方法
```
原型：XHR.poll = function(interval, callPath, data, callback)
作用：XHR.poll可以通过callPath定时的从后台获取到数据
参数解释：
interval，定时刷新时间(单位为秒)
callPath，获取数据的路径
data，在获取数据的时候好像没什么用，JS不是太懂，通常都是填null
callback，回调函数,处理获取到的数据
Examples:
[1]. 获取源数据，使用json格式发送出去
function dev_info()

    local date = {}
	......

	luci.http.prepare_content("application/json")
	luci.http.write('[')
	luci.http.write_json(date)
	luci.http.write(']')
end
[2]. 使用XHR.poll获取解析数据
var callPath='<%=luci.dispatcher.build_url("admin", "system", "devinfo",parameter)%>';
XHR.poll(interval,callPath, null,function(x,date){
        ......
    }
);
```
### get方法
XHR.get方法与XHR.poll方法类似，不过，XHR.get方法是一次性调用