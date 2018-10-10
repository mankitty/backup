---
title: Linux网卡唤醒
date: 2018-09-26 16:18:41
tags:
categories: cloud
---
## BIOS设置
目前一般的机器网卡和主板都支持远程唤醒开机，需要在BIOS里设置将网络唤醒开机开启。开机时进入BIOS，查看CMOS中的"Power Management Setup"，通常里面会有Power On by Onborad Lan，将其设置为"Enable"
## 查看网卡是否支持Wake On Lan
使用ethtool命令确认网卡是否支持，命令如下
ethtool 网卡名称
```
root@root:~$ sudo ethtool enp1s0 | grep "Wake-on"
Supports Wake-on: pumbg
Wake-on: g
```
若Wake-on为d，表示禁用Wake On LAN，需要启用,启用命令为ethtool -s eth0 wol g
若Wake-on为g，则说明目标机器的网卡已经支持Wake On LAN

## 网卡唤醒
使用wol命令进行唤醒操作，具体操作为:wol 网卡MAC地址