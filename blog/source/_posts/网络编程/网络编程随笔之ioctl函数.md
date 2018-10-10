---
title: 网络编程随笔之ioctl函数
date: 2018-05-26 16:20:55
tags:
categories: 网络编程随笔
---
## ioctl函数
```
本函数影响由fd(socket)参数引用的一个打开的文件
#include <sys/ioctl.h>
int ioctl(int d, int request, ...);
其中，第三个参数总是一个指针，但是指针的类型依赖于request参数。
```
总体来说，与网络相关的请求可以划分为6类
1.套接字操作 2.文件操作 3.接口操作 4.ARP高速缓存操作 5.路由表操作 6.流系统
下图列出了网络相关的ioctl请求的request参数及arg地址必须指向的数据类型

|类别|Request|说明|数据类型|
|:---:|:---:|:---:|:---:|
|套接字|SIOCATMARK|是否位于带外标记|int|
|套接字|SIOCSPGRP|设置套接口的进程ID或进程组ID|int|
|套接字|SIOCGPGRP|获取套接口的进程ID或进程组ID|int|
|文件|FIONBIN|设置/清除非阻塞I/O标志|int|
|文件|FIOASYNC|设置/清除信号驱动异步I/O标志|int|
|文件|FIONREAD|获取接收缓存区中的字节数|int|
|文件|FIOSETOWN|设置文件的进程ID或进程组ID|int|
|文件|FIOGETOWN|获取文件的进程ID或进程组ID|int|
|接口|SIOCGIFCONF|获取所有接口的清单|struct ifconf|
|接口|SIOCSIFADDR|设置接口地址|struct ifreq|
|接口|SIOCGIFADDR|获取接口地址|struct ifreq|
|接口|SIOCSIFFLAGS|设置接口标志|struct ifreq|
|接口|SIOCGIFFLAGS|获取接口标志|struct ifreq|
|接口|SIOCSIFDSTADDR|设置点到点地址|struct ifreq|
|接口|SIOCGIFDSTADDR|获取点到点地址|struct ifreq|
|接口|SIOCGIFBRDADDR|获取广播地址|struct ifreq|
|接口|SIOCSIFBRDADDR|设置广播地址|struct ifreq|
|接口|SIOCGIFNETMASK|获取子网掩码|struct ifreq|
|接口|SIOCSIFNETMASK|设置子网掩码|struct ifreq|
|接口|SIOCGIFMETRIC|获取接口的测度|struct ifreq|
|接口|SIOCSIFMETRIC|设置接口的测度|struct ifreq|
|接口|SIOCGIFMTU|获取接口MTU|struct ifreq|
|接口|SIOCxxx|还有很多取决于系统的实现|struct ifreq|
|ARP|SIOCSARP|创建/修改ARP表项|struct arpreq|
|ARP|SIOCGARP|获取ARP表项|struct arpreq|
|ARP|SIOCDARP|删除ARP表项|struct arpreq|
|路由|SIOCADDRT|增加路径|struct rtentry|
|路由|SIOCDELRT|删除路径|struct rtentry|
|流|I_xxx|||