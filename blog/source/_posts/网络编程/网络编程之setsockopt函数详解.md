---
title: 网络编程之setsockopt函数详解
date: 2018-06-09 13:44:06
tags:
categories: 网络编程随笔
---

## 参考
[1]. man 7 socket
[2]. man 2 setsockopt
## 概要
```
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int optname,void *optval, socklen_t *optlen);
int setsockopt(int sockfd, int level, int optname,const void *optval, socklen_t optlen);
```
## 返回值
成功时，返回零。 出错时，返回-1，并且适当地设置errno。
## 描述
getsockopt以及setsockopt可以获取以及设置与某个套接字关联的选项，选项可能存在于多个协议级别，选项影响套接口的操作，诸如加急数据是否在普通数据流中接收，广播数据是否可以从套接口发送，设置接收缓冲区的大小等等。

## 错误返回值
**EBADF**
相关描述：文件描述符无效
**EFAULT**
相关描述：optval指向的地址不在进程地址空间的有效部分。 对于getsockopt（），如果optlen不在进程地址空间的有效部分，也可能返回此错误
**EINVAL**
相关描述：在setsockopt（）中optlen无效。 在某些情况下，对于optval中的无效值也会发生此错误（例如，对于ip（7）中所述的IP_ADD_MEMBERSHIP选项）
**ENOTSOCK**
相关描述：文件描述符sockfd不引用套接字
**ENOPROTOOPT**
相关描述：该选项在所示级别是未知的

## 参数说明

1. level 指定控制套接字的层次.可以取三种值:
[1]. SOL_SOCKET:通用套接字选项.
[2]. IPPROTO_IP:IP选项.
[3]. IPPROTO_TCP:TCP选项.
2. optname 指定控制的方式(选项的名称),具体信息如下.
3. optval获得或者是设置套接字选项.

### SOL_SOCKET
**选项名称：SO_BINDTODEVICE**
解释说明：将此套接字绑定到特定设备，如"eth0"。 如果名称是空字符串或选项长度为零，则将删除套接字设备绑定。需要的参数是一个可变长度的以Null结尾的接口名称字符串，其最大大小为IFNAMSIZ。**注意：**如果套接字绑定到接口，则只有从该特定接口收到的数据包由套接字处理。 请注意，这仅适用于某些套接字类型，特别是AF_INET套接字，它不支持数据包套接字。
数据类型：string

**选项名称：SO_REUSEPORT**
解释说明：允许将多个AF_INET或AF_INET6套接字绑定到相同的套接字地址。 该选项必须设置在每个套接字（包括第一个套接字）上调用套接字上的bind之前。为了防止劫持端口，所有绑定到相同地址的进程必须具有相同的有效UID。这个选项可以使用与TCP和UDP套接字
数据类型：int

**选项名称：SO_REUSERADDR**
解释说明：
[1]. 当有一个有相同本地地址和端口的socket1处于TIME_WAIT状态时，而你启动的程序的socket2要占用该地址和端口，你的程序就要用到该选项。
[2]. SO_REUSEADDR允许同一port上启动同一服务器的多个实例(多个进程)。但每个实例绑定的IP地址是不能相同的。在有多块网卡或用IP Alias技术的机器可以测试这种情况。
[3]. SO_REUSEADDR允许单个进程绑定相同的端口到多个socket上，但每个socket绑定的ip地址不同。这和2很相似，区别请看UNPv1。
[4]. SO_REUSEADDR允许完全相同的地址和端口的重复绑定。但这只用于UDP的多播，不用于TCP。
**注意:一般来说这是不被支持，我觉得最主要的作用应该是防止服务器意外挂掉却没有将相应的端口释放掉，导致再次启用的时候会出现端口被占用的错误，因为你已经启用了SO_REUSERADDR选项，那么就可以重新启用**
数据类型：int
参考文章地址：[SO_REUSEADDR到底什么意思](https://www.cnblogs.com/my_life/articles/4397672.html)

**选项名称：SO_RCVBUF**
解释说明：以字节为单位设置或获取最大套接字接收缓冲区,当使用setsockopt设置时，内核将此值加倍,默认值由/proc/sys/net/core/rmem_default文件设置，最大允许值由/proc/sys/net/core/rmem_max文件设置。此选项的最小值（加倍后）为256
数据类型：int

**选项名称：SO_SNDBUF**
解释说明：以字节为单位设置或获取最大套接字发送缓冲区。当使用setsockopt设置内核时，内核将此值加倍，并且这个加倍的值由getsockopt返回。缺省值由/proc/sys/net/core/wmem_default文件设置，最大允许值由/proc/sys/net/core/wmem_max文件设置。该选项的最小值（加倍）是2048
数据类型：int

**选项名称：SO_BROADCAST**
解释说明：设置或获取广播标志。 启用时，允许数据报套接字将数据包发送到广播地址。 该选项对流式套接字没有影响
数据类型：int

**选项名称：SO_OOBINLINE**
解释说明：如果启用此选项，带外数据将直接放入接收数据流中。 否则只有在接收期间设置MSG_OOB标志时才会传递带外数据。

### IPPROTO_IP
**选项名称：IP_HDRINCL**
解释说明：在数据包中包含IP首部
数据类型：int
**选项名称：IP_OPTINOS**
解释说明：IP首部选项
数据类型：int
**选项名称：IP_TOS**
解释说明：服务类型
**选项名称：IP_TTL**
解释说明：生存时间
数据类型：int
### IPPRO_TCP
**选项名称：TCP_MAXSEG**
解释说明：TCP最大数据段的大小
数据类型：int
**选项名称：TCP_NODELAY**
解释说明：不使用Nagle算法
数据类型：int