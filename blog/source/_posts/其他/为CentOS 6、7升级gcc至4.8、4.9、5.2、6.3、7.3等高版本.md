---
title: 为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本
date: 2019-05-6 10:01:56
tags: 
categories:
---

## 参考博客
https://www.vpser.net/manage/centos-6-upgrade-gcc.html
## 备注
为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本，为防止原博客被删除，特做笔记
## GCC版本升级
### 注意
需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本
如果要长期使用gcc 6.3的话：
需要添加环境变量，echo "source /opt/rh/devtoolset-6/enable" >>/etc/profile
然后重新打开终端
### 升级到gcc 4.8
```
wget http://people.centos.org/tru/devtools-2/devtools-2.repo -O /etc/yum.repos.d/devtoolset-2.repo
yum -y install devtoolset-2-gcc devtoolset-2-gcc-c++ devtoolset-2-binutils
scl enable devtoolset-2 bash
```
### 升级到gcc 4.9
```
wget https://copr.fedoraproject.org/coprs/rhscl/devtoolset-3/repo/epel-6/rhscl-devtoolset-3-epel-6.repo -O /etc/yum.repos.d/devtoolset-3.repo
yum -y install devtoolset-3-gcc devtoolset-3-gcc-c++ devtoolset-3-binutils
scl enable devtoolset-3 bash
```
### 升级到gcc 5.2
```
wget https://copr.fedoraproject.org/coprs/hhorak/devtoolset-4-rebuild-bootstrap/repo/epel-6/hhorak-devtoolset-4-rebuild-bootstrap-epel-6.repo -O /etc/yum.repos.d/devtoolset-4.repo
yum install devtoolset-4-gcc devtoolset-4-gcc-c++ devtoolset-4-binutils -y
scl enable devtoolset-4 bash
```
### 升级到gcc 6.3
```
yum -y install centos-release-scl
yum -y install devtoolset-6-gcc devtoolset-6-gcc-c++ devtoolset-6-binutils
scl enable devtoolset-6 bash
```
### 升级到gcc 7.3
```
yum -y install centos-release-scl
yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
scl enable devtoolset-7 bash
```
