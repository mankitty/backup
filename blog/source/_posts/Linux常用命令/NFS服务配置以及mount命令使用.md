---
title: NFS服务配置以及mount命令使用
date: 2018-10-10 10:58:14
tags:
---

## 配置NFS服务
### NFS默认配置文件
/etc/exports			/* 主配置文件,默认为空 */
### NFS配置文件语法
格式：[共享的目录] [主机名或IP(参数,参数)]
例子：/var/lib/tftpboot/Linux 192.168.9.25/24(ro,async,no_root_squash,no_subtree_check)
### NFS配置文件参数释义
#### 参数一
NFS服务想要共享的目录
#### 参数二
允许访问的主机，可以是单独的主机，或者地址段(192.168.9.25/24)，或者一定的域名集合(*.sin.com)，或者允许所有的主机访问(*)
#### 共享参数（权限,常用）
ro  只读访问
rw  读写访问
sync    所有数据在请求时写入共享内存（同步）
async   NFS在写入数据前可以相应请求（异步）
secure  NFS通过1024以下的安全TCP/IP端口发送
insecure    NFS通过1024以上的端口发送
wdelay      如果多个用户要写入NFS目录，则归组写入（默认）
no_wdelay   如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置
hide    在NFS共享目录中不共享其子目录
no_hide 共享NFS目录的子目录
subtree_check   如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认）
no_subtree_check    如果共享/usr/bin之类的子目录时，不检查父目录权限
all_squash  共享文件的UID和GID映射匿名用户anonymous，适合公用目录。
no_all_squash   保留共享文件的UID和GID（默认）
no_root_squash  登入到NFS主机的用户如果是root，该用户即拥有root权限；
root_squash 登入NFS主机的用户如果是root，该用户权限将被限定为匿名使用者nobody（默认）
anonuid 将登入NFS主机的用户都设定成指定的UID，此ID必须存在于/etc/passwd中
## mount命令使用
### 命令格式
mount [-t vfstype] [-o options] device dir
### 常用参数详解
#### -t vfstype
指定文件系统的类型，通常不必指定。mount 会自动选择正确的类型。常用类型有：
1. 光盘或光盘镜像：iso9660
2. DOS fat16文件系统：msdos
3. Windows 9x fat32文件系统：vfat
4. Windows NT ntfs文件系统：ntfs
5. Mount Windows文件网络共享：smbfs
6. UNIX(LINUX) 文件网络共享：nfs

#### -o options
主要用来描述设备或档案的挂接方式。常用的参数有：
1. loop：用来把一个文件当成硬盘分区挂接上系统
2. ro：采用只读方式挂接设备
3. rw：采用读写方式挂接设备

#### iocharset
指定访问文件系统所用字符集
#### device
准备挂接(mount)的设备
#### dir
设备在系统上的挂接点(mount point)

### mount 命令使用实例
mount -t nfs 10.0.2.2:/linuxos/rhel4.0_update3/ /mnt/nfs/

### Linux从光盘制作光盘镜像文件
命令：dd if=/dev/cdrom of=/home/sunky/mydisk.iso
其中：/dev/sr0是光驱的设备名，/dev/cdrom代表光驱,cdrom是sr0的软链接.

### linux制作ISO启动盘命令
mkisofs -o backup.iso -x /home/joeuser/junk/ -J -R -A -V -v /home/joeuser/
#### 参数详解
```
-a或者--all mkisofs通常不处理备份文件，使用此参数可以把备份文件加到映像文件中
-A<应用程序ID>或-appid<应用程序ID>   指定光盘的应用程序ID
-abstract<摘要文件>   指定摘要文件的文件名
-b<开机映像文件>或-eltorito-boot<开机映像文件>  指定在制作可开机光盘时所需的开机映像文件
-biblio<ISBN文件>   指定ISBN文件的文件名，ISBN文件位于光盘根目录下，记录光盘的ISBN。
-c<开机文件名称>   制作可开机光盘时，mkisofs会将开机映像文件中的全-eltorito-catalog<开机文件名称>全部内容作成一个文件。
-C<盘区编号，盘区编号>   将许多节区合成一个映像文件时，必须使用此参数。
-copyright<版权信息文件>   指定版权信息文件的文件名。
-f或-follow-links   忽略符号连接。
-hide<目录或文件名>   使指定的目录或文件在ISO 9660或Rock RidgeExtensions的系统中隐藏。
-hide-joliet<目录或文件名>   使指定的目录或文件在Joliet系统中隐藏。
-J或-joliet   使用Joliet格式的目录与文件名称。
-l或-full-iso9660-filenames   使用ISO 9660 32字符长度的文件名。
-L或-allow-leading-dots   允许文件名的第一个字符为句号。
-log-file<记录文件>   在执行过程中若有错误信息，预设会显示在屏幕上。
-m<目录或文件名>或-exclude<目录或文件名>   指定的目录或文件名将不会房入映像文件中。
-M<映像文件>或-prev-session<映像文件>   与指定的映像文件合并。
-N或-omit-version-number   省略ISO 9660文件中的版本信息。
-o<映像文件>或-output<映像文件>   指定映像文件的名称。
-p<数据处理人>或-preparer<数据处理人>   记录光盘的数据处理人。
-print-size   显示预估的文件系统大小。
-quiet   执行时不显示任何信息。
-r或-rational-rock   使用Rock Ridge Extensions，并开放全部文件的读取权限。
-R或-rock   使用Rock Ridge Extensions。
-sysid<系统ID>   指定光盘的系统ID。
-T或-translation-table   建立文件名的转换表，适用于不支持Rock Ridge Extensions的系统上。
-v或-verbose   执行时显示详细的信息。
-V<光盘ID>或-volid<光盘ID>   指定光盘的卷册集ID。
-volset-size<光盘总数>   指定卷册集所包含的光盘张数。
-volset-seqno<卷册序号>   指定光盘片在卷册集中的编号。
-x<目录>   指定的目录将不会放入映像文件中。
-z   建立通透性压缩文件的SUSP记录，此记录目前只在Alpha机器上的Linux有效
```
## 参考文章
1. [mkisofs命令磁盘管理](http://man.linuxde.net/mkisofs)
2. [linux的mount挂载命令详解](https://www.jb51.net/os/RedHat/1109.html)