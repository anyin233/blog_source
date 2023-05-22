---
title: 记录一次宿舍NAS的搭建
date: 2023-02-21 10:35:09
tags:
---

# 前期准备

## 起因与需求

为了不被叔叔恶心，同时简化自己的追番流程，另外还考虑到可以整点homelab，故选择了在宿舍搭建自己的第一台nas，主要目的当然是方便自己进行高画质追番，次要的目的就是随时可以搞点奇奇怪怪的东西来玩。

既然是宿舍装机，那需求和限制都非常明确

- 7 * 24h 运行
- 静音
- 小体积
- 功率低于3200w
- 可用容量至少有个8T

<!--more-->

## 硬件选择

既然考虑到静音，同时还需要一定的性能拿来搞homelab，那么N5105就被喜闻乐见地排除掉了，毕竟这玩意儿的虚拟化存在一系列的问题，经过反复纠结（高性能AIO、低功耗妖板）后，选定如下配件

- CPU: i3-12100
- 主板: 华南 B660M-plus
- 内存: 金百达 8G✖️2 3200Mhz intel专用条
- 启动盘: 金百达 k230 Pro 500G
- 散热器: 利民 AXP90-X36
- 电源: 安钛克 HCG 550
- 机箱: 御夫座
- 存储盘: 希捷VX015 4T✖️4

上面的配置里面华南的板子只有两个风扇位置能够调速，其中一个是CPU风扇，另一个为机箱风扇，考虑到御夫座机箱的分区设计，可调速的风扇位连接机箱上半层，不可调速的则使用减速线降低转速以降低噪音。

## 系统选择

既然准备未来的扩展性，同时参考了网络上大量的现成案例，最终PVE作为基础系统（原计划unraid，但是不知道为啥无法从我的u盘启动），在其中安装TrueNAS作为文件存储（原计划选择OMV，奈何不支持btrfs的大多数特性，ZFS更别说了），剩余服务均在PVE中使用LXC容器实现。

目前已经配置完成的服务结构如下

- PVE
  - TrueNAS
  - LXC
    - Jellyfin
  - LXC
    - qbittorrent
    - Auto_Bangumi
  - LXC
    - ChineseSubFinder

所有LXC均使用smb挂载TrueNAS创建的存储池以实现媒体库共享。

# 系统安装

> 硬件的安装暂时给省了，直接进行系统安装

## PVE

PVE的安装过程最为简单，从官网下载安装镜像并使用rufus或者Etcher刷入U盘中，使用U盘启动后按照安装程序的指示继续即可，在该过程中设置的密码便是你的PVE管理员密码，邮箱随意，看起来没啥用的样子。

安装完毕后重启，在命令行中可以看到你的PVE的后台管理地址，使用另外一台在同一网络下的电脑访问这个地址，便可以进入PVE的后台管理界面。

安装完毕后需要在PVE中进行几个配置以提升使用体验

> 此处我将PVE安装在SSD上，后续SATA控制器将会被直通给TrueNAS

### IOMMU

为了实现硬件直通，IOMMU必须被开启，开启方法较为简单，只需修改`/etc/default/grub`，将

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

修改为

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on i915.enable_guc=7"
```

然后执行

```bash
update-grub
```

### SRIOV

SRIOV是来自Intel的虚拟化技术，通过这个技术可以把网卡或者其他支持的设备虚拟化成多个设备使用。此处使用该技术将核显虚拟化成8个核显以直通给更多的设备以便后续使用。

> 一般的主板均开启了该功能，但是还是请检查一下你的主板或者平台是否支持，赛扬和奔腾平台暂时别考虑了，不支持的。

由于此处我使用的是12代Intel CPU，故需要将Linux 内核更新到较高版本以确保支持，对于我的PVE，我将其内核更新至6.1.0。使用如下命令进行更新

```
apt update && apt install pve-kernel-6.1.0-1-pve
```

然后安装对应的headers

```
apt install pve-headers-6.1.0-1-pve
```

安装完毕后先进行重启，此时可以使用`uname -a`确认内核是否更新成功。

#### 安装i915-sriov-dkms

使用sriov特性需要特定的内核模块，首先安装dkms以便于安装模块。

```
apt install dkms -y
```

接下来下载`i915-sriov-dkms`并想办法上传到PVE中，~我把我手上的那个文件弄丢了，丢人~

其中一个比较方便的方法是先将下载得到的`i915-sriov-dkms`后缀名改为iso，并从PVE上传iso的位置进行上传，此时文件将会出现在`/var/lib/vz/template/iso/`下，将其移动到主目录下后解压即可进行安装。

```
mv /var/lib/vz/template/iso/i915-sriov-dkms.tar.iso  i915-sriov-dkms.tar
tar -xvf i915-sriov-dkms.tar
mv i915-sriov-dkms /usr/src
dkms install -m i915-sriov -v dkms
```

安装完成后记得检查是否安装成功

```
dkms status
```

若成功则会有如下输出

```
i915-sriov, dkms, 6.1.0-1-pve, x86_64: installed
```

#### 修改内核参数

上面在开启IOMMU的时候添加了一个`i915.enable_guc=7`，这个便是此处开启sriov所需要的内核参数。同时在`/etc/kernel/cmdline`下面添加如下的内容

```
intel_iommu=on i915.enable_guc=7 
```

如果没这个文件创建一个就好。

执行如下指令更新内核initramfs

```
update-initramfs -u -k all
pve-efiboot-tool refresh
```

然后重启设备。

重启完毕后建议检查是否有对应的i915固件

```
ls /lib/firmware/i915/tgl_guc_70.1.1.bin
```

如果没有的话可以在[这里](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/i915/)下载，下载后记得再重启一次。

#### 添加启动参数

完成上面的配置后，可以准备添加启动参数使得开机的时候自动进行虚拟化

首先需要安装sysfsutils

```
apt install sysfsutils
```

然后添加启动参数

首先使用`lspci`确认你的显卡pcie地址

找到其中含有VGA和Intel字样的行，例如

```
00:02.0 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
```

前面的`00:02.0`即为该设备的地址，

然后执行，注意其中的<PCI_ADDR>替换为该设备的地址

```
echo "devices/pci0000:00/0000:<PCI_ADDR>/sriov_numvfs = 7" > /etc/sysfs.conf
```

重启电脑

最后可以检查你的sriov是否开启成功，

```
lspci
```

若能看到你冒出了一堆的核显，sriov开启成功

```
00:00.0 Host bridge: Intel Corporation Device 4630 (rev 05)
00:02.0 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.1 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.2 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.3 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.4 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.5 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.6 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:02.7 VGA compatible controller: Intel Corporation Device 4692 (rev 0c)
00:06.0 PCI bridge: Intel Corporation Device 464d (rev 05)
00:0a.0 Signal processing controller: Intel Corporation Device 467d (rev 01)
00:0e.0 RAID bus controller: Intel Corporation Volume Management Device NVMe RAID Controller
00:14.0 USB controller: Intel Corporation Device 7ae0 (rev 11)
00:14.2 RAM memory: Intel Corporation Device 7aa7 (rev 11)
00:16.0 Communication controller: Intel Corporation Device 7ae8 (rev 11)
00:17.0 SATA controller: Intel Corporation Device 7ae2 (rev 11)
00:1c.0 PCI bridge: Intel Corporation Device 7ab8 (rev 11)
00:1f.0 ISA bridge: Intel Corporation Device 7a86 (rev 11)
00:1f.3 Audio device: Intel Corporation Device 7ad0 (rev 11)
00:1f.4 SMBus: Intel Corporation Device 7aa3 (rev 11)
00:1f.5 Serial bus controller [0c80]: Intel Corporation Device 7aa4 (rev 11)
01:00.0 Non-Volatile memory controller: MAXIO Technology (Hangzhou) Ltd. NVMe SSD Controller MAP1202 (rev 01)
02:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
```

至此所有的准备工作完成，准备开始安装所有的应用。

## TrueNAS

在PVE中安装系统的过程较为简单，对应安装镜像可以在[官网](https://www.truenas.com/download-truenas-scale/)获取，我选择的是TrueNAS Scale，但是实际上由于我并不需要TrueNAS负责除文件存储以外的服务，实际也可以选择基于BSD开发的TrueNAS Core。

将下载好的iso文件上传到PVE中，创建一个虚拟机，

<img src="https://i.imgur.com/MiQQ4fT.png" alt="image-20230221113151020" style="zoom:50%;" />

其中`VM ID`和`Node`保持默认即可，`Name`选择一个便于自己识别的名字，

<img src="https://i.imgur.com/6Ojdmos.png" alt="image-20230221113240119" style="zoom:50%;" />

ISO image选择刚刚上传的iso文件，剩余保持默认，System部分也可以保持默认，到Disk页面设备类型选择SATA，大小32GB左右即可

<img src="https://i.imgur.com/wwL06B7.png" alt="image-20230221113351589" style="zoom:50%;" />

核心数量根据你的设备性能分配1～4个核心均可，在这里我选择2个核心

<img src="https://i.imgur.com/tuGHwL5.png" alt="image-20230221113443271" style="zoom:50%;" />

内存建议多给TrueNAS一点，毕竟是吃内存大户，我分配了8GB

<img src="https://i.imgur.com/NDog3Wo.png" alt="image-20230221113524408" style="zoom:50%;" />

网络部分保持默认，创建即可。

创建完毕后不要急着启动，定位到你的虚拟机的Hardware页面，点击Add

<img src="https://i.imgur.com/73S5kvp.png" alt="image-20230221113625096" style="zoom:50%;" />

选择PCI Device，在其中选中你的核显，并勾选`All Functions`。

如果你的方案是直通SATA控制器，此时也需要选择你的SATA控制器，并勾选`All Functions`以保证其可以正常工作。

> 注意：在你直通SATA控制器后PVE便无法使用在这个控制器下面的所有硬盘了，如果你的PVE也安装在SATA硬盘中，请参考网络上直通SATA设备的方法

接下来可以启动虚拟机，切换到Console页面，此时就和操作普通的虚拟机一样了，具体安装TrueNAS的教程可以参考网络中安装教程。
