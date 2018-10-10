---
title: OpenWrt 之setfilehandler函数详解
date: 2018-06-20 10:20:28
tags:
categories: OpenWrt
---
## setfilehandler原型

```lua
local fp,fsname
luci.http.setfilehandler(
    function(meta, chunk, eof)
        -- 防止每一次进入回调函数的时候都执行打开文件的操作
        if not fp and not fsname then
            if meta and meta.name == "image" then
                fp = io.open(image_tmp, "w")
            end
        end

        fsname = true

        -- 如果文件流存在，那么将文件流写进文件中
        if chunk and fp then fp:write(chunk) end
        -- 如果文件已经读取完毕，关闭文件描述符
        if eof and fp then fp:close() end
    end
)
```
## setfilehandler注意事项
[1]. 在使用该接口之前不能有任何非声明语句的存在
[2]. 在回调函数中有三个参数，分别为meta（包含上传文件的各种信息），chunk（文件流），eof（文件结束标志）
[3]. meta参数
其中，meta具有以下几个参数
```
["file"] = "xxx-180611.f3.e1f7.bin"	--上传文件的文件名
["name"] = "version"	--html控件中的name属性
 ["headers"] = {	--headers属性
     ["Content-Disposition"] = "form-data; name=\"version\"; filename=\"xxx-180611.f3.e1f7.bin\""
     ["Content-Type"] = "application/octet-stream"
}
```