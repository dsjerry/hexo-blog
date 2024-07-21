---
title: CentOS 7 搭建FTP服务
date: 2021-01-20 15:52:19
tags: ftp
categories: 部署运维
thumbnail: https://s3.ax1x.com/2021/01/20/sWoyPH.png
cover: https://s3.ax1x.com/2021/01/20/sWoyPH.png
toc: true
---

操作系统：CentOS 7

<!-- more -->

## 安装 vsftpd

以 <span style="color: gold;">root</span> 身份执行

```bash
yum install -y vsftpd
```

设置开机自启

```bash
systemctl enable vsftpd
```

启动 FTP 服务

```bash
systemctl start vsftpd
```

查看FTP服务的占用端口以确认开启了此服务

```bash
netstat -antup | grep ftp
```

![ftp服务监听的端口](https://s3.ax1x.com/2021/01/20/sWcMkt.png)

- 此时，vsftpd 已默认开启匿名访问模式，无需通过用户名和密码即可登录 FTP 服务器。使用此方式登录 FTP 服务器的用户没有权修改或上传文件的权限。

## 配置 vsftpd

为 FTP 服务创建用户

```bash
useradd ftpuser
```

设置此用户的密码

```bash
passwd ftpuser
```

![更改密码](https://s3.ax1x.com/2021/01/20/sWgANq.png)

创建FTP服务使用的文件夹（在哪创建都ok）

```bash
mkdir /var/ftp/file
```

修改目录权限

```bash
chown -R ftpuser:ftpuser /var/ftp/file
```

- `:`左边的是用户，右边的是用户组

编辑 `vsftpd.conf` 配置文件

```bash
vim /etc/vsftpd/vsftpd.conf
```

> FTP 可通过主动模式和被动模式与客户端机器进行连接并传输数据。由于大多数客户端机器的防火墙设置及无法获取真实 IP 等原因，推荐使用<span style="color: darkcyan;">被动模式</span>

在配置文件中，修改一下内容

```conf
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
listen=YES
```

使用 <span style="color: darkcyan;">#</span> 注释关闭监听 IPv6 sockets

```conf
#listen_ipv6=YES
```

添加以下配置参数，开启被动模式

```conf
# 登录后所处的目录
local_root=/var/ftp/test
allow_writeable_chroot=YES
pasv_enable=YES
# 下面加入自己的IP地址
pasv_address=xxx.xx.xxx.xx
# 建立数据传输可使用的端口范围值
pasv_min_port=40000
pasv_max_port=45000
```

`:wq` 保存退出

修改 `chroot_list` 文件

```bash
vim /etc/vsftpd/chroot_list
```

- 这个文件中保存的是例外允许的用户
- 一个用户名占用一行

重启 FTP 服务

```bash
systemctl restart vsftpd
```

## 开放端口（非云服务器）

开放 FTP 使用的 `21` 端口

```bash
firewall-cmd --zone=public --add-port=21/tcp --permanent 
```

- 如果要关闭，将 <span style="color: green">add</span> 改为 <span style="color: red">remove</span> 就行

## 设置安全组（云服务器）

以 <span style="color: skyblue">腾讯云为例</span>，打开腾讯云的控制台，找到 **安全组** 点击 **修改规则**，然后 **添加规则**

![配置安全组](https://s3.ax1x.com/2021/01/20/sW5JwF.png)

## 备注

### 备注1：登录失败

如果配置之后，使用用户名和密码登录登录不成功，修改配置文件 `/etc/pam.d/vsftpd`

```bash
vim /etc/pam.d/vsftpd
```

添加 **#** 注释掉这一行

```ini
#auth       required    pam_shells.so
```

### 备注2：上传文件失败

遇到文件上传失败，先检查FTP服务使用的文件夹是否有权限

```bash
ls -l /var/ftp
```

![文件夹权限](https://s3.ax1x.com/2021/01/20/sWIUHS.png)

如果 `d`后面没有`w`则表示没有写入权限（上图是有的），就添加权限

```bash
chmod +w /var/ftp/file
```

### 备注3

如果连接失败，检查一下FTP连接工具是否使用的是 *被动模式*