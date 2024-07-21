---
title: Linux虚拟机网络环境配置
date: 2020-09-20 22:31:49
tags: linux
categories: 部署运维
thumbnail: https://z1.ax1x.com/2020/10/13/0fQMid.png
cover: https://z1.ax1x.com/2020/10/13/0fQMid.png
toc: true
---

虚拟机软件：VMware Workstation Pro

操作系统：CentOS 7

<!-- more -->

桥接模式和 NAT 模式都可以，而且都可以配置静态 IP 地址。但是桥接模式在天翼校园网的情况下使用还得登录校园网账号才能上网，所以还是选择 NAT 模式。

配置静态 IP 地址的目的是为了方便其它 shell 软件连接，不配置照样可以上网。

## 虚拟机网络配置

- Windows 物理机的虚拟网卡 `VMnet8` 要开启

  ![open VMnet 8](https://s1.ax1x.com/2020/10/13/0hbt5n.png)

### 通过 DHCP 连接

- 若果不配置静态 ip，一般选择 NAT 模式之后，虚拟机就会获得一个同 VMnet8 网卡相同网段的 ip 地址

### 配置静态 IP 地址

- 配置的静态 IP 地址一定要和宿主机虚拟网卡 IP 处于同一个网段

#### 物理机

更改物理机的虚拟网卡 IP

![网络适配器](https://s1.ax1x.com/2020/10/13/0fwKnH.png)

NAT 模式使用的是 VMnet8

- 修改`TCP/IPv4`，给定一个 ip 地址，前面**192.168 是固定的**等下 VMware 里面配置相同的网段就行
- 网段：IP4 地址分为四段，第三个就是了。要在 0 ~ 255 之间
- 最后一位不能为 0，在 1 ~ 254 之间选

![VMnet8](https://s1.ax1x.com/2020/10/13/0fQ1zt.png)

#### VMware 虚拟网络编辑器

![wmware首页](https://s1.ax1x.com/2020/10/13/0fQ8QP.png)

这里的 VMnet8 的网段和上面物理机的网卡配成一样，然后点击右边的**NAT 设置**

![虚拟机网络配置](https://s1.ax1x.com/2020/10/13/0fQlRI.png)

这里的网关 IP 一般会自动分配了个 192.168.146.2，没有的就要自己手动写上，这个网关是**必须的**

![NAT设置](https://s1.ax1x.com/2020/10/13/0fQuIH.png)

#### CentOS 7 网络配置

选中虚拟机，选择**NAT**模式

![use NAT](https://s1.ax1x.com/2020/10/13/0h599U.png)

打开虚拟机，切换到 **root** 用户。root 密码是在安装系统的时候就设置了的那个

![root](https://s1.ax1x.com/2020/10/13/0hPUbt.png)

使用`cd`命令切换到网络配置目录（CentOS 7 版，其他的 Linux 发行版本略有不同）

![网络配置目录](https://s1.ax1x.com/2020/10/13/0fQMid.png)

在这里，白色字的是本机（CentOS 7）的一些网卡，默认情况下是有 `ifcfg-lo` 和 `ifcfg-ens33`，其他的是以后有需要再配置。配置网卡是修改 `ifcfg-ens33`的

使用`vi`或者`vim`编辑 `ifcfg-ens33`

`vim ifcfg-ens33`

![vim编辑](https://s1.ax1x.com/2020/10/13/0fQQJA.png)

红色框的是要改变的，根据上面已经配置的网段选一个 IP，然后保存退出。

- 保存：先按键盘的**ESC**键，然后在英文输入法的前提下输入`:wq`，w 表示写入保存，q 表示退出

![退出vim](https://s1.ax1x.com/2020/10/13/0hACM4.png)

重启 CentOS 的网络，使用命令：

```bash
systemctl restart network
```

查看自己配置的 IP，使用命令：`ifconfig` 或者 `ip a`

![查看IP地址](https://s1.ax1x.com/2020/10/13/0hEnkq.png)

如果出现就说明配置成功啦

### 测试是否连通

#### 是否与物理机连通

- 分别使用`ping`命令在物理机和虚拟机之间互 ping

在 Windows 上，打开命令提示符（CMD），使用 `ipconfig` 命令查看 Windows 本机的 IP 地址

![cmd](https://s1.ax1x.com/2020/10/13/0hfJEV.png)

- 注意：Windows 下是 ipconfig，不是 Linux 的 ifconfig

![查看ip](https://s1.ax1x.com/2020/10/13/0heF1K.png)

Windows 上 ping 虚拟机：`ping 192.168.146.24`，成功：

![Windows ping CentOS](https://s1.ax1x.com/2020/10/13/0hRTsI.png)

虚拟机 上 ping Windows：`ping 192.168.146.1`，成功：

![Linux ping Windows](https://s1.ax1x.com/2020/10/13/0hhiPU.png)

- 如果虚拟机 ping Windows 不通，是 Windows 防火墙拦住了，把防火墙的**专用网络**观点就行

> 正常情况下，虚拟机 ping 不通 Windows 物理机，没什么影响。最好还是打开 Windows 的防火墙，使自己电脑处于一个安全的网络环境。

#### 是否连接到互联网

ping 一下百度：`ping www.baidu.com`，成功：

![ping Baidu](https://s1.ax1x.com/2020/10/13/0h4N6J.png)

## 使用 shell 和 ftp 工具测试连接

#### 连接 shell

![成功连接](https://s1.ax1x.com/2020/10/13/0hTc5t.png)

#### 连接 sftp

![sftp](https://s1.ax1x.com/2020/10/13/0hHp6S.png)

> 这里不是连接 ftp，而是 sftp，ftp 服务要在操作系统中另外开启。不过效果是差不多的~

传输文件直接拖拽就行，如果直接在本地代开虚拟机上的文件，文件是同步的。

如果连接不上互联网，试着改一下 DNS，在 `ifcfg-ens33`中修改。
