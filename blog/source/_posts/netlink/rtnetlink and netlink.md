---
title: rtnetlink and netlink
date: 2018-06-01 21:09:58
tags:
categories: netlink
---
## 概要
```c
#include <asm/types.h>
#include <sys/socket.h>
#include <linux/netlink.h>

netlink_socket = socket(AF_NETLINK, socket_type, netlink_family);
```
## 描述
Netlink用主要用于内核和用户空间进程之间传输信息。
由一个标准的套接字组成用户空间进程接口和内核模块的内部内核API。Netlink是一种面向数据报的服务,SOCK_RAW和SOCK_DGRAM都是socket_type的有效值。但是，netlink协议不区分数据报和原始套接字。netlink_family选择内核模块或netlink组与当前分配的netlink系列进行通信
## 路由属性
一些rtnetlink消息在初始头后有可选的属性，这些属性只能使用**RTA_ *宏**或libnetlink进行操作
```
struct rtattr {
    unsigned short rta_len;    /* Length of option */
    unsigned short rta_type;   /* Type of option */
    /* Data follows */
};
```
**RTA_OK(rta, attrlen)** returns true if rta points to a valid routing attribute; attrlen is the running length of the attribute buffer. When not true then you must assume there are no more attributes in the message, even if attrlen is nonzero.

**RTA_DATA(rta)** returns a pointer to the start of this attribute's data.

**RTA_PAYLOAD(rta)** returns the length of this attribute's data.

**RTA_NEXT(rta, attrlen)** gets the next attribute after rta. Calling this macro will update attrlen. You should use RTA_OK to check the validity of the returned pointer.

**RTA_LENGTH(len)** returns the length which is required for len bytes of data plus the header.

**RTA_SPACE(len)** returns the amount of space which will be needed in a message with len bytes of data.

### EXAMPLE
```
Creating a rtnetlink message to set the MTU of a device:
    #include <linux/rtnetlink.h>
    ...
    struct {
       struct nlmsghdr  nh;
       struct ifinfomsg if;
       char             attrbuf[512];
    } req;

    struct rtattr *rta;
    unsigned int mtu = 1000;

    int rtnetlink_sk = socket(AF_NETLINK, SOCK_DGRAM, NETLINK_ROUTE);

    memset(&req, 0, sizeof(req));
    req.nh.nlmsg_len = NLMSG_LENGTH(sizeof(struct ifinfomsg));
    req.nh.nlmsg_flags = NLM_F_REQUEST;
    req.nh.nlmsg_type = RTM_NEWLINK;
    req.if.ifi_family = AF_UNSPEC;
    req.if.ifi_index = INTERFACE_INDEX;
    req.if.ifi_change = 0xffffffff; /* ??? */
    rta = (struct rtattr *)(((char *) &req) +
                            NLMSG_ALIGN(req.nh.nlmsg_len));
    rta->rta_type = IFLA_MTU;
    rta->rta_len = RTA_LENGTH(sizeof(unsigned int));
    req.n.nlmsg_len = NLMSG_ALIGN(req.nh.nlmsg_len) +
                                 RTA_LENGTH(sizeof(mtu));
    memcpy(RTA_DATA(rta), &mtu, sizeof(mtu));
    send(rtnetlink_sk, &req, req.nh.nlmsg_len);
```

## Messages
Rtnetlink包含这些消息类型（除了标准的netlink消息）：
**RTM_NEWLINK, RTM_DELLINK, RTM_GETLINK**
创建，移除或者获取一个特定网络接口的信息，这些消息包含一个ifinfomsg结构，后跟一系列rtattr结构。
```c
struct ifinfomsg {
    unsigned char  ifi_family; /* AF_UNSPEC */
    unsigned short ifi_type;   /* Device type */
    int            ifi_index;  /* Interface index */
    unsigned int   ifi_flags;  /* Device flags  */
    unsigned int   ifi_change; /* change mask */
};
```

**RTM_NEWADDR, RTM_DELADDR, RTM_GETADDR**
添加，删除或接收关于与接口相关的IP地址的信息。Linux 2.2以后，一个接口可以携带多个IP地址，并且支持IPv4 and IPv6 地址,它们包含一个ifaddrmsg结构，可选地跟着rtattr路由属性。
```
struct ifaddrmsg {
    unsigned char ifa_family;    /* Address type */
    unsigned char ifa_prefixlen; /* Prefixlength of address */
    unsigned char ifa_flags;     /* Address flags */
    unsigned char ifa_scope;     /* Address scope */
    int           ifa_index;     /* Interface index */
};
```
**ifa_family** is the address family type (currently AF_INET or AF_INET6)
**ifa_prefixlen** is the length of the address mask of the address if defined for the family (like for IPv4)
**ifa_scope** is the address scope, ifa_index is the interface index of the interface the address is associated with. 
**ifa_flags** is a flag word of IFA_F_SECONDARY for secondary address (old alias interface), IFA_F_PERMANENT for a permanent address set by the user and other undocumented flags.

**RTM_NEWROUTE, RTM_DELROUTE, RTM_GETROUTE**
创建，删除或接收有关网络路由的信息。 这些消息包含一个rtmsg结构，其中包含一个可选的rtattr结构序列。 对于RTM_GETROUTE，将rtm_dst_len和rtm_src_len设置为0意味着您将获取指定路由表的所有条目。 对于其他字段，除rtm_table和rtm_protocol外，0可以作为通配符。
```
struct rtmsg {
	unsigned char rtm_family;   /* Address family of route */
	unsigned char rtm_dst_len;  /* Length of destination */
	unsigned char rtm_src_len;  /* Length of source */
	unsigned char rtm_tos;      /* TOS filter */

	unsigned char rtm_table;    /* Routing table ID */
	unsigned char rtm_protocol; /* Routing protocol; see below */
	unsigned char rtm_scope;    /* See below */
	unsigned char rtm_type;     /* See below */

	unsigned int  rtm_flags;
};
```
内核不解释大于RTPROT_STATIC的值，它们仅用于用户信息。 它们可能用于标记路由信息的来源或区分多个路由守护进程。
rtm_scope is the distance to the destination:

The values between RT_SCOPE_UNIVERSE and RT_SCOPE_SITE are available to the user.

The rtm_flags have the following meanings:

rtm_table specifies the routing table

The user may assign arbitrary values between RT_TABLE_UNSPEC and RT_TABLE_DEFAULT.

**RTM_NEWNEIGH, RTM_DELNEIGH, RTM_GETNEIGH**
Add, remove or receive information about a neighbor table entry (e.g., an ARP entry). The message contains an ndmsg structure.
```
struct ndmsg {
    unsigned char ndm_family;
    int           ndm_ifindex;  /* Interface index */
    __u16         ndm_state;    /* State */
    __u8          ndm_flags;    /* Flags */
    __u8          ndm_type;
};

struct nda_cacheinfo {
    __u32         ndm_confirmed;
    __u32         ndm_used;
    __u32         ndm_updated;
    __u32         ndm_refcnt;
};
```
ndm_state is a bit mask of the following states:
Valid ndm_flags are:

The rtattr struct has the following meanings for the rta_type field:

If the rta_type field is NDA_CACHEINFO then a struct nda_cacheinfo header follows

**RTM_NEWRULE, RTM_DELRULE, RTM_GETRULE**
Add, delete or retrieve a routing rule. Carries a struct rtmsg
**RTM_NEWQDISC, RTM_DELQDISC, RTM_GETQDISC**
Add, remove or get a queueing discipline. The message contains a struct tcmsg and may be followed by a series of attributes.
```
struct tcmsg {
    unsigned char    tcm_family;
    int              tcm_ifindex;   /* interface index */
    __u32            tcm_handle;    /* Qdisc handle */
    __u32            tcm_parent;    /* Parent qdisc */
    __u32            tcm_info;
};
```
In addition various other qdisc module specific attributes are allowed. For more information see the appropriate include files.

**RTM_NEWTCLASS, RTM_DELTCLASS, RTM_GETTCLASS**
Add, remove or get a traffic class. These messages contain a struct tcmsg as described above.

**RTM_NEWTFILTER, RTM_DELTFILTER, RTM_GETTFILTER**
Add, remove or receive information about a traffic filter. These messages contain a struct tcmsg as described above.

## netlink消息格式
![image][nlmsghdr]

[nlmsghdr]:data:image/png;base64,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
## netlink消息报头：struct nlmsghdr
```c
与正常的消息体一样，都是由header+payload组成，下面是nlmsghdr的构成
struct nlmsghdr {
	__u32 nlmsg_len;    /* Length of message including header. */
	__u16 nlmsg_type;   /* Type of message content. */
	__u16 nlmsg_flags;  /* Additional flags. */
	__u32 nlmsg_seq;    /* Sequence number. */
	__u32 nlmsg_pid;    /* Sender port ID. */
};
其中，nlmsg_len整个消息的长度，包括消息头，nlmsg_type表示消息内容的类型，
```
## nlmsg_type 消息类型
NLMSG_NOOP - 无需任何操作，消息必须被丢弃
NLMSG_ERROR - 错误消息
NLMSG_DONE - 分段序列的结束
NLMSG_OVERRUN - 通知信息越界（错误），即缓冲区溢出

```
NLMSG_ERROR:错误处理
struct nlmsgerr {
    int error;        /* Negative errno or 0 for acknowledgements */
    struct nlmsghdr msg;  /* Message header that caused the error */
};
```

A netlink family usually specifies more  message  types
**Standard flag bits in nlmsg_flags**
**NLM_F_REQUEST**   Must be set on all request messages.
**NLM_F_MULTI**     The  message is part of a multipart message terminated by NLMSG_DONE.
**NLM_F_ACK**       Request for an acknowledgment on success.
**NLM_F_ECHO**      Echo this request.


Additional flag bits for GET requests
**NLM_F_ROOT**     Return the complete table instead of a single entry.
**NLM_F_MATCH**    Return all entries matching criteria passed in mes-sage content.  Not implemented yet.
**NLM_F_ATOMIC**   Return an atomic snapshot of the table.
**NLM_F_DUMP**     Convenience macro; equivalent to (NLM_F_ROOT|NLM_F_MATCH).

Note that NLM_F_ATOMIC requires the CAP_NET_ADMIN capability or an effective UID of 0.

Additional flag bits for NEW requests
**NLM_F_REPLACE**   Replace existing matching object.
**NLM_F_EXCL**      Don't replace if the object already exists.
**NLM_F_CREATE**    Create object if it doesn't already exist.
**NLM_F_APPEND**    Add to the end of the object list.

nlmsg_seq 以及 nlmsg_pid被用来标记消息.
nlmsg_pid 显示消息的来源。请注意，如果消息来自netlink套接字，则nlmsg_pid和进程的PID之间不存在1：1关系。
nlmsg_seq 以及 nlmsg_pid 对于netlink核心来说都是不透明的.
Netlink 不是一个可靠的协议。 它尽最大努力将消息传递到其目的地，但可能会在内存不足，套接字缓冲区已满或其他错误发生时丢弃消息。为了进行可靠的传输，发送者可以通过设置NLM_F_ACK标志来请求来自接收者的确认。 确认是错误字段设置为0的NLMSG_ERROR数据包。应用程序必须自己为收到的消息生成确认。 内核尝试为每个失败的数据包发送NLMSG_ERROR消息。 用户进程也应该遵循这个惯例。但是，无论如何，都不可能实现从内核到用户的可靠传输。

地址格式
sockaddr_nl结构描述用户空间或内核中的netlink客户端。其中sockaddr_nl可以是单播的（仅发送给一个客户）或发送给netlink多播组（nl_groups不等于0）。
```
struct sockaddr_nl {
    sa_family_t     nl_family;  /* AF_NETLINK */
    unsigned short  nl_pad;     /* Zero. */
    pid_t           nl_pid;     /* Port ID. */
    __u32           nl_groups;  /* Multicast groups mask. */
};
```
Nl_pid是netlink套接字的单播地址。如果目标位于内核中，则始终为0.对于用户空间进程，nl_pid通常是拥有目标套接字的进程的PID。但是，nl_pid标识了一个netlink套接字，而不是一个进程。如果一个进程有多个netlink套接字，那么nl_pid只能等于最多一个套接字的进程ID。
Nl_groups是一个位掩码，每个位表示一个网络连接组号。
CAP_NET_ADMIN函数可以发送或监听网络链路多播组

EXAMPLE
       The following example creates a NETLINK_ROUTE netlink socket which will listen to the RTMGRP_LINK (network interface cre-ate/delete/up/down events) and RTMGRP_IPV4_IFADDR (IPv4 addresses add/delete events) multicast groups.
```
struct sockaddr_nl sa;

memset(&sa, 0, sizeof(sa));
sa.nl_family = AF_NETLINK;
sa.nl_groups = RTMGRP_LINK | RTMGRP_IPV4_IFADDR;

fd = socket(AF_NETLINK, SOCK_RAW, NETLINK_ROUTE);
bind(fd, (struct sockaddr *) &sa, sizeof(sa));

The  next example demonstrates how to send a netlink message to the kernel (pid 0).  Note that application must take care
of message sequence numbers in order to reliably track acknowledgements.

struct nlmsghdr *nh;    /* The nlmsghdr with payload to send. */
struct sockaddr_nl sa;
struct iovec iov = { nh, nh->nlmsg_len };
struct msghdr msg;

msg = { &sa, sizeof(sa), &iov, 1, NULL, 0, 0 };
memset(&sa, 0, sizeof(sa));
sa.nl_family = AF_NETLINK;
nh->nlmsg_pid = 0;
nh->nlmsg_seq = ++sequence_number;
/* Request an ack from kernel by setting NLM_F_ACK. */
nh->nlmsg_flags |= NLM_F_ACK;

sendmsg(fd, &msg, 0);

And the last example is about reading netlink message.

int len;
char buf[4096];
struct iovec iov = { buf, sizeof(buf) };
struct sockaddr_nl sa;
struct msghdr msg;
struct nlmsghdr *nh;

msg = { &sa, sizeof(sa), &iov, 1, NULL, 0, 0 };
len = recvmsg(fd, &msg, 0);

for (nh = (struct nlmsghdr *) buf; NLMSG_OK (nh, len);
    nh = NLMSG_NEXT (nh, len)) {
   /* The end of multipart message. */
   if (nh->nlmsg_type == NLMSG_DONE)
       return;

   if (nh->nlmsg_type == NLMSG_ERROR)
       /* Do some error handling. */
   ...

   /* Continue with parsing payload. */
   ...
}
```

## 参考文章
[1]. http://www.infradead.org/~tgr/libnl/doc/core.html#core_msg_format