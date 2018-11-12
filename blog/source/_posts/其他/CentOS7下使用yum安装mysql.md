# CentOS7下使用yum安装mysql
---
## 下载mysql的repo源
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
## 安装mysql-community-release-el7-5.noarch.rpm包
rpm -ivh mysql-community-release-el7-5.noarch.rpm
## 安装mysql
yum install mysql-server
## 重置mysql密码
**mysql -u root**
**登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘**
原因:
1. /var/lib/mysql的访问权限问题
    ```
    下面的命令把/var/lib/mysql的拥有者改为当前用户：
    chown -R andrew:andrew /var/lib/mysql
    ```
2. 初次安装好是没有启动服务的，启动服务
重启服务：service mysqld restart
## 重置登录密码
1. mysql -u root
2. use mysql;
3. update user set password=password(pass) where user='root';
4. exit;