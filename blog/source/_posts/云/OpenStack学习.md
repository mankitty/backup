---
title: OpenStack-Neutron
date: 2018-07-25 16:46:58
tags:
categories: cloud
---

## OpenStack-Neutron是什么
Neutron是OpenStack核心项目之一，提供云计算环境下的虚拟网络功能。Neutron网络目的是（为OpenStack云更灵活地）划分物理网络，在多租户环境下提供给每个租户独立的网络环境。Neutron可通过提供API来实现这种目标。

## Neutron主要组成部分
### Neutron Server
这一部分包含守护进程neutron-server和各种插件neutron-*-plugin，它们既可以安装在控制节点也可以安装在网络节点。neutron-server提供API接口，并把对API的调用请求传给已经配置好的插件进行后续处理。插件需要访问数据库来维护各种配置数据和对应关系，例如路由器、网络、子网、端口、浮动IP、安全组等等。

### 插件代理 （Plugin Agent）
虚拟网络上的数据包的处理则是由这些插件代理来完成的。名字为neutron-*-agent。在每个计算节点和网络节点上运行。一般来说你选择了什么插件，就需要选择相应的代理。代理与Neutron Server及其插件的交互就通过消息队列来支持。

### DHCP代理（DHCP Agent）

名字为neutron-dhcp-agent，为各个租户网络提供DHCP服务，部署在网络节点上，各个插件也是使用这一个代理。

### 3层代理 （L3 Agent）
名字为neutron-l3-agent， 为客户机访问外部网络提供3层转发服务。也部署在网络节点上。
下面这张图取自官网，很好的反映了Neutron内部各部分之间的关系。（SDN服务在这里是额外的外部功能，可以暂时略过。）

