---
title: OpenWrt 页面添加相关介绍
date: 2018-06-19 13:36:28
tags:
categories: OpenWrt
---
## MVC框架
模型（Model） 用于封装与应用程序的业务逻辑相关的数据以及对数据的处理方法。
视图（View） 能够实现数据有目的的显示。
控制器（Controller） 可以起到不同层面间的组织作用，用于控制应用程序的流程。主要是可以将Model以及View层紧密的连接起来
这三层是相互独立而又紧密联系的，MVC在我看来是可以更好的实现数据与逻辑等的分离。
## OpenWrt 创建页面
OpenWrt也是采用上述的MVC框架，即Model-View-Controller
### 模块注册
module("luci.controller.admin.network", package.seeall) -–模块入口
这行标识了模块的入口，其中**luci.controller.admin.network**定义了模块注册所在的文件路径，上述的路径为/usr/lib/lua/luci/controller/admin/network.lua
### 节点注册
节点的注册我了解的有两种方法，其实这两种方法我在使用的时候并没有发现什么大的区别
```lua
[1]. entry 方法
	local page
	page = entry({"admin", "network", "hosts"}, cbi("admin_network/hosts"), _("Static Domain"),40)
	page.leaf = true
[2]. node 方法
	local page
	page = node("admin", "network", "hosts")
	page.target = cbi("admin_network/hosts")
	page.title  = _("Static Domain")
	page.order  = 40
```
#### 参数解释
##### entry的原型为：entry(path, target, title=nil, order=nil)
##### path：定义了节点的路径，即页面访问时的url路径
##### target：调用的方法，大致上可以分为以下几种
	[1]. call,直接调用指定的函数,比如写为"call("function_name")"，然后在对该函数做具体的定义就可以调用了。
	[2]. cbi,调用Model层的一些方法，比如写为"cbi("admin_network/hosts")"就可以调用/usr/lib/lua/luci/model/cbi/admin_network/hosts.lua文件,这里通常是对业务逻辑的相关处理。
	[3]. template,调用指定的页面，比如写为“template("admin_network/login")”就可以调用/usr/lib/lua/luci/view/admin_network/login.htm文件了
    [4]. alias,使用别名,即当你点击了某个菜单，相应的动作是另外一个菜单的入口，那么就使用alias来指明另外路口的路径，如：entry({"admin", "advanced"}, alias("admin", "advanced", "ddns"), _("Advanced"), 5)),表示节点("admin", "advanced", "ddns")是节点{"sixing", "advanced"}的别名
    [5]. arcombine,暂时没弄的太明白，文档上介绍 Create a combined dispatching target for non argv and argv requests

##### title：菜单对外显示的名称
##### order：菜单栏显示的顺序

### 数据库文件
默认情况下，数据存储都是放在/etc/config目录下，存储格式以及调用接口等可参考 [UCI-System-学习之路](http://localhost:4000/2018/05/13/OpenWrt/UCI-System-%E5%AD%A6%E4%B9%A0%E4%B9%8B%E8%B7%AF/)
