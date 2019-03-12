---
title: 基于SR-IOV的IO虚拟化技术
date: 2019-03-08 13:06:32
tags:
---

## 服务器配置要求
1. x86服务器内存不能低于32GB
2. 服务器CPU需要支持虚拟化和设备虚拟化 VT-x VT-d,SR-IOV 功能，并且在BIOS中能启用了SR-IOV
3. 网卡配置最起码为千兆配置
4. 支持 SR-IOV 的网卡必须插在总线带宽 X8 以上的扩展槽中

## 基本定义
SR-IOV 技术标准允许在虚拟机之间高效共享 PCIe（Peripheral Component Interconnect Express，快速外设组件互连）设备，并且它是在硬件中实现的，可以获得能够与本机性能媲美的 I/O 性能。SR-IOV 规范定义了新的标准，根据该标准创建的新设备可允许将虚拟机直接连接到 I/O 设备。

## SRIOV的功能类型
### 物理功能 (Physical Function, PF)
用于支持 SR-IOV 功能的 PCI 功能，如 SR-IOV 规范中定义。PF 包含 SR-IOV 功能结构，用于管理 SR-IOV 功能。PF 是全功能的 PCIe 功能，可以像其他任何 PCIe 设备一样进行发现、管理和处理。PF 拥有完全配置资源，可以用于配置或控制 PCIe 设备。
### 虚拟功能 (Virtual Function, VF)
与物理功能关联的一种功能。VF是一种轻量级 PCIe功能，可以与物理功能以及与同一物理功能关联的其他 VF共享一个或多个物理资源。VF仅允许拥有用于其自身行为的配置资源。
### 功能简介  
每个 SR-IOV 设备都可有一个物理功能 (Physical Function, PF)，并且每个 PF 最多可有 64,000 个与其关联的虚拟功能 (Virtual Function, VF)。PF 可以通过寄存器创建 VF，这些寄存器设计有专用于此目的的属性。
如果在 PF 中启用了 SR-IOV，就可以通过 PF 的总线、设备和功能编号（路由 ID）访问各个 VF 的 PCI 配置空间。每个 VF 都具有一个PCI内存空间，用于映射其寄存器集。VF设备驱动程序对寄存器集进行操作以启用其功能，并且显示为实际存在的PCI设备。创建VF后，可以直接将其指定给IO来宾域或各个应用程序。此功能使得虚拟功能可以共享物理设备，并在没有 CPU 和虚拟机管理程序软件开销的情况下执行 I/O。

## GPU SRIOV
### System Requirements
#### Operating System Requirements
Ubuntu 16.04. has been fully validated as host, other Linux operating system like Centos 7 are also OK.
#### Hardware Requirements
The device supports VT-d, and the physical graphics card supports SRIOV technology.
### Modify the BIOS
we need reset bios to defaults,and enable VT-d and SRIOV function
### Kernel preparation 
In order to use the graphics cards and all its cores, we'll have to recompile the Linux Kernel, modifying some values and applying some paches.
#### Download kernel source
```
vi /etc/apt/sources.list

# deb cdrom:[Ubuntu-Server 16.04.6 LTS _Xenial Xerus_ - Release amd64 (20190226)]/ xenial main restricted

#deb cdrom:[Ubuntu-Server 16.04.6 LTS _Xenial Xerus_ - Release amd64 (20190226)]/ xenial main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ xenial main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://us.archive.ubuntu.com/ubuntu/ xenial universe
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial universe
deb http://us.archive.ubuntu.com/ubuntu/ xenial-updates universe
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu 
## team, and may not be under a free licence. Please satisfy yourself as to 
## your rights to use the software. Also, please note that software in 
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://us.archive.ubuntu.com/ubuntu/ xenial multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial multiverse
deb http://us.archive.ubuntu.com/ubuntu/ xenial-updates multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://us.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://security.ubuntu.com/ubuntu xenial-security main restricted
deb-src http://security.ubuntu.com/ubuntu xenial-security main restricted
deb http://security.ubuntu.com/ubuntu xenial-security universe
deb-src http://security.ubuntu.com/ubuntu xenial-security universe
deb http://security.ubuntu.com/ubuntu xenial-security multiverse
deb-src http://security.ubuntu.com/ubuntu xenial-security multiverse

apt update && apt upgrade -y
```
apt install -y dpkg-dev
apt source linux-image-$(uname -r)
#### Download gim modules
git clone https://github.com/GPUOpen-LibrariesAndSDKs/MxGPU-Virtualization
#### Patch the kernel
The patch is located in the SRC_DIR/MxGPU-Virtualization/patch directory.
```
cd linux-4.4.0 && make oldconfig && make menuconfig

patch < ../MxGPU-Virtualization/patch/0001-Added-legacy-endpoint-type-to-sriov-for-ubuntu-4.4.0-75-generic.diff
File to patch: ./drivers/pci/iov.c
patch < ../MxGPU-Virtualization/patch/0002-add-pci-io-access-cap-for-ubuntu-4.4.0-75-generic.diff
File to patch: ./drivers/vfio/pci/vfio_pci_config.c
```
#### Kernel compilation
make -j <number of threads that the CPU> deb-pkg LOCALVERSION=-vm-firepro
#### Kernel installation
```
cd ..
dpkg -i *.deb
```
### GRUB and BLACKLIST preparation
#### GRUB
```
vim /etc/default/grub
---
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on"
---
Check if IOMMU is supported after rebooting the system.
Command is as follows:
dmesg | grep -e DMAR -e IOMMU
You should be able to see something like the following:DMAR: IOMMU enabled
```
#### Blacklist amdgpu and amdkfd
adding the following line to the end of file
```
vi /etc/modprobe.d/blacklist.conf
---
blacklist amdgpu
blacklist amdkfd
---
```
### gim module compilation
#### Compile gim module
cd MxGPU-Virtualization && make && make install
#### Use gim module
```
modprobe gim
modprobe -r gim
vi /etc/gim_config
----
...
vf_num=16 # This is going to activate the 16 cores of the GPU
...
----
modprobe gim
dmesg # Finally, we need to check that the gim module has successfully been loaded, without errors
lspci -vvv # We can now see that the FirePro S7150 has all the 16 cores activated
```