---
title: OpenWrt 页面登录认证
date: 2018-06-19 15:44:33
tags:
categories: OpenWrt
---
## 页面登录简介
Openwrt在登录的时候默认会寻找相应的index.lua文件，寻找到节点，执行方法
在用户登录的时候，默认会寻找到/usr/lib/lua/luci/controller/admin/index.lua文件，内容如下
```lua
module("luci.controller.admin.index", package.seeall)

function index()
    local root = node()
    if not root.target then
    	root.target = alias("admin")
    	root.index = true
    end

    local page   = node("admin")
    page.target  = firstchild()
    page.title   = _("Administration")
    page.order   = 10
    page.sysauth = luci.sys.user.getloginusers()	--获取登录的用户名
    page.sysauth_authenticator = "htmlauth"		--使用dispatcher.lua文件中定义的htmlauth方法验证用户的合法性
    page.ucidata = true
    page.index = true

    -- Empty services menu to be populated by addons
    entry({"admin", "services"}, firstchild(), _("Services"), 40).index = true
end
```