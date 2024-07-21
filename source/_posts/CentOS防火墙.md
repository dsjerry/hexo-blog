---
title: CentOS防火墙
date: 2020-12-01 11:33:09
tags: linux
categories: 部署运维
thumbnail: https://s3.ax1x.com/2021/01/25/sq7rtJ.png
cover: https://s3.ax1x.com/2021/01/25/sq7rtJ.png
toc: true

---

介绍 <span style="color: darkcyan">CentOS6</span>、<span style="color: gold">CentOS7</span>的防火墙的配置

<span style="color: gold">CentOS7</span>以下的版本和<span style="color: darkcyan">CentOS6</span>相同，<span style="color: skyblue">CentOS8</span>和<span style="color: gold">CentOS7</span>配置是相同的。🧐

<!-- more -->

# CentOS 6

## 安装配置

### 安装

```bash
yum -y install iptables-services
```

### 配置

开放 80 端口

```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

配置端口后，保存防火墙配置规则

```bash
service iptables save
```

或者修改默认配置文件 `/etc/sysconfig/iptables` ，增加开放的端口。添加内容：

```bash
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```

## 状态管理

```bash
# 查看状态
service iptables status
# 开启
service iptables start
# 关闭
service iptables stop
# 重启
service iptables restart
```

```bash
# 永久关闭
chkconfig iptables off
# 永久关闭之后想要开启
chkconfig iptables on
```



# CentOS 7

## 安装配置

### 安装

```bash
yum -y install firewalld
```

### 配置

添加 80 端口

```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

- 配置后**重启**防火墙
- firewall-cmd：防火墙的管理工具
- zone：指定作用范围

或者直接修改防火墙的配置文件，配置文件位置 `/etc/firewalld`，修改 `/etc/firewalld/zones/public.xml `，在 `<zone></zone>`里面添加：

```xml
<port protocol="tcp" port="80" />
```

## 状态管理

```bash
# 查看状态
systemctl status firewalld
# 开启
systemctl start firewalld
# 关闭
systemctl stop firewalld
# 重启
systemctl restart firewalld
```

```bash
# 开机启动
systemctl enable fierwalld
# 永久关闭
systemctl disenable fierwalld
```

# 其他

## 云服务器

云服务器的防火墙默认情况下是关闭的，所有的端口开放是在云服务器平台上的**安全组**中进行





<h1 style="font-size: 100px"><span style="color: gold">iptables</span> <span style="color: purple">VS</span> <span style="color: orange">firewalld</span></h1>



​	                                              <span style="text-align: center; font-size: 50px">🔥               🚒</span>



