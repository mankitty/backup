---
title: 在Centos 6.5以及Ubuntu 16.04下对WEBDAV的配置
date: 2019-06-20 15:33:33
tags:
categories: webdav
---

## Centos 6.5对WEBDAV的配置
### 安装
yum install httpd -y
### 开启dav以及dav_fs模块
apache的主配置文件为/etc/httpd/conf/httpd.conf，建议不要随意更改，新建副配置文件webdav.con

```
确认主配置文件中dav以及dav_fs模块是否已经开始，若没有开启，则添加以下两句话，开启dav以及dav_fs模块
[...]
LoadModule dav_module modules/mod_dav.so
[...]
LoadModule dav_fs_module modules/mod_dav_fs.so
[...]
```
### 创建虚拟主机
#### 创建虚拟主机使用的目录以及更改该目录的权限
```
mkdir -p /var/webdav
chown apache:apache /var/webdav
```
#### 配置WebDAV上的虚拟主机
```
创建WebDAV密码文件
htpasswd -c /etc/webdav/passwd.dav mankitty
```
#### 更改passwd.dav文件权限
```
chown root:apache /etc/webdav/passwd.dav 
chmod 640 /etc/webdav/passwd.dav
```
##### 最终的配置文件
```
LoadModule dav_module modules/mod_dav.so
LoadModule dav_fs_module modules/mod_dav_fs.so

NameVirtualHost *:80

<IfModule mod_dav_fs.c>
        # Location of the WebDAV lock database.
        DAVLockDB /var/lib/dav/lockdb
</IfModule>

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/webdav
        <Directory /var/webdav>
                Options Indexes MultiViews
                AllowOverride None
                Order allow,deny
                allow from all
        </Directory>

        Alias /webdav /var/webdav

        <Location /webdav>
                DAV On
                AuthType Basic
                AuthName "webdav"
                AuthUserFile /etc/webdav/passwd.dav
                Require valid-user
        </Location>
</VirtualHost>
```
#### 配置问题
```
[Mon Jul 15 15:07:46.037943 2019] [dav:error] [pid 2738] [client 192.168.7.239:49491] The lock database could not be opened, preventing access to the various lock properties for the PROPFIND.  [500, #0]
[Mon Jul 15 15:07:46.037999 2019] [dav:error] [pid 2738] [client 192.168.7.239:49491] A lock database was not specified with the DAVLockDB directive. One must be specified to use the locking functionality.  [500, #401]

DavLockDB  /ect/httpd/var/DavLock

Provider encountered an error while streaming a multistatus PROPFIND response

chcon -R -u system_u -r object_r -t httpd_sys_content_t /var/webdav

不能创建文件或者目录

<IfModule mod_dav_fs.c>
        # Location of the WebDAV lock database.
        DAVLockDB /var/lib/dav/lockdb
</IfModule>

```

#### window下命令行链接
```
net use x:  \\IP\webdav "密码" /user:"用户名"
```

## Ubuntu 16.04下对WEBDAV的配置
### 配置
```
直接摘抄的，不过，测试过，没问题

sudo apt-get install apache2 -y

# 开启Apache2中对WebDav协议的支持 (记住最好在用户目录下执行否则报错)
cd ~
sudo a2enmod dav
sudo a2enmod dav_fs
 
# 创建共享目录并修改权限
sudo mkdir -p /var/www/webdav
sudo chown -R www-data:www-data  /var/www/webdav
 
# 创建WebDav的访问用户数据库，顺便创建用户`pi`
sudo htpasswd -c /etc/apache2/webdav.password pi
# 创建guest用户
#sudo htpasswd /etc/apache2/webdav.password guest
 
# 修改用户数据库访问权限
sudo chown root:www-data /etc/apache2/webdav.password
sudo chmod 640 /etc/apache2/webdav.password
 
# 打开默认配置文件
sudo vim /etc/apache2/sites-available/000-default.conf
 
# 全部替换为以下内容（记得先备份）：
 
Alias /webdav  /var/www/webdav
 
<Location /webdav>
    Options Indexes
    DAV On
    AuthType Basic
    AuthName "webdav"'
    AuthUserFile /etc/apache2/webdav.password
    Require valid-user
 </Location>
```
### 访问
#### 浏览器
任意浏览器输入：http://树莓派的IP地址/webdav。注意，webdav后面没有/斜杠。
#### image
这一步完成，我们就可以开始把这个共享文件夹映射到Mac、Windows上的本地文件夹了。

#### 磁盘映射
网页里只能像FTP一样显示文件目录和下载文件。如果要正常使用，我们需要把它映射为本地目录才行：

Mac上：
在Finder中用CMD+K打开连接服务器选项，输入http://树莓派IP地址/webdav，输入Webdav创建过的用户名密码来完成映射。

Windows上：
比较麻烦的是，Win7以上默认只支持HTTPS的网络驱动器，做为HTTP的WebDav是不能连的。所以要修改Windows注册表，让它支持HTTP。方法入下：
1. 开始菜单 -> 运行 -> 输入regedit 并按回车，就打开了注册表
注册表中找到HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters\BasicAuthLevel这个项目，把值改为2。
2. 开始菜单 -> 运行 -> 输入cmd 并按回车，打开命令行
3. 输入net stop webclient并按回车，停止网络客户端
4. 输入net start webclient并按回车，开启网络客户端
5. 然后在文件夹菜单中找到映射网络驱动器，输入网址http://树莓派IP地址/webdav或\\树莓派IP地址\webdav，然后输入用户名密码，就能映射成功了。

#### 挂载外部磁盘（移动硬盘、U盘）

和Samba一样，只要在/var/www/webdav/这个共享出来的文件夹中，创建个空目录，然后把移动硬盘用mount命令挂载到这个目录上。外部就可以访问了。

## 参考文档
1. [Ubuntu安装WebDav文件共享服务器（NAS）](https://blog.csdn.net/weixin_33836874/article/details/86943592)
2. [如何建立与Apache2 WebDAV在CentOS 5.5](https://zfm-06dk.iteye.com/blog/1989160)