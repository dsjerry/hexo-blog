---
title: LNMP分布式集群
date: 2020-11-30 14:46:05
tags: [Linux,nginx,php,mysql]
categories: 部署运维
thumbnail: https://s3.ax1x.com/2020/11/30/DWZfAJ.png
cover: https://s3.ax1x.com/2020/11/30/DWZfAJ.png
toc: true
---

操作平台：`WMware Workstation 15 Pro`

操作系统：`Linux CentOS 7`

<!-- more -->

根据课程总结出`LNMP`的配置过程

![LNMP分布式部署结构图, 转自黑马程序员](https://s3.ax1x.com/2020/11/30/DWZfAJ.png)

# 前提准备

达到上面的效果，需要准备9个节点，为了节省空间，这里使用一个`CentOS 7`作为白板，然后克隆此虚拟机。（转自黑马程序员）

| 编号 |       IP       |             服务器             |       硬件侧重点        |
| :--: | :------------: | :----------------------------: | :---------------------: |
|  1   | 192.168.146.31 |    Nginx (www.itshop.test)     |        网卡性能         |
|  2   | 192.168.146.32 |    Nginx (file.itshop.test)    |   内存容量、磁盘性能    |
|  3   | 192.168.146.33 | Nginx+PHP (upload.itshop.test) |        网卡性能         |
|  4   | 192.168.146.34 |           Nginx+PHP            |         CPU性能         |
|  5   | 192.168.146.35 |           Nginx+PHP            |         CPU性能         |
|  6   | 192.168.146.36 |              NFS               |        磁盘容量         |
|  7   | 192.168.146.37 |           MySQL (主)           | CPU、内存、磁盘整体性能 |
|  8   | 192.168.146.38 |           MySQL (从)           | CPU、内存、磁盘整体性能 |
|  9   | 192.168.146.39 |           Memcached            |        内存容量         |

每个节点的职责示意图

![LNMP节点](https://s3.ax1x.com/2020/11/30/DWm2FJ.png)

## 基于白板克隆节点

准备一个最小安装的 CentOS 白板（什么都没安装）

为了节省存储空间和内存，这里的 CentOS 选择*最小安装*，内存给每个节点配置*512M*，CPU选1个就行了。然后通过虚拟机克隆需要系统，配置对应的IP地址，*再根据需要安装相应的软件*。

为了节省时间，这边可以先配置节点`1,6,7,9`。因为有一些节点会安装相同的软件，后面的节点可以通过克隆来产生，这样就不用重复配置啦~

### 节点1部署Nginx

安装依赖包：

```bash
yum -y install gcc pcre-devel openssl-devel
```

下载 nginx 后，放在`/root`目录下然后解压（这里使用nginx1.18）：

```bash
tar -vzxf nginx-1.18.0.tar.gz
```

编译安装Nginx，增加`http_realip_module`模块

```bash
cd nginx-1.18.0
./configure  --with-http_ssl_module --with-http_realip_module
make && make install
```

添加环境变量、创建服务脚本、设置开机启动

```bash
cd /usr/local/nginx/sbin
ln -s `pwd`/nginx /usr/local/sbin/nginx
vim /etc/init.d/nginx
```

加入内容

```
#!/bin/bash
#chkconfig:35 85 15
DAEMON=/usr/local/nginx/sbin/nginx
case "$1" in
  start )
        echo "Starting nginx daemon..."
        $DAEMON && echo "SUCCESS"
  ;;
  stop )
        echo "Stopping nginx daemon..."
        $DAEMON -s quit && echo "SUCCESS"
  ;;
  reload )
        echo "Reloading  nginx daemon..."
        $DAEMON -s reload  && echo "SUCCESS"
  ;;
 restart )
        echo "Restarting nginx daemon..."
        $DAEMON -s quit 
        $DAEMON  && echo "SUCCESS"
  ;;
 * )
    echo "Usage:service nginx {start | stop |restart |reload }"
    exit 2
;;
esac

```

```bash
chmod +x /etc/init.d/nginx
```

设置开机自启

```bash
chkconfig --add nginx
```

创建`www`用户和站点目录`data/www`

```bash
useradd -s /sbin/nologin -M www
mkdir -p /data/www
```

将 nginx 默认页面内容复制到`www`目录下

```bash
cd /usr/local/nginx
cp html/* /data/www
```

赋予用户权限

```bash
chown -R www:www /data/www
```

修改*nginx主配置文件*：

```nginx
user www www;
server {
       listen 80;
       server_name localhost;
       root /data/www;
       index index.html index.htm;
 }
```

使用命令`nginx`或`service nginx start`启动 nginx，并且开通虚拟机的 **80** 端口

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
systemctl restart firewalld
```

测试访问 nginx 是否配置成功，访问默认页面 *192.168.146.31*

![nginx配置成功](https://s1.ax1x.com/2020/11/06/BhGTWn.png)

### 节点6部署NFS

### 节点7部署MySQL

### 节点8部署Memcached

## 基于节点1克隆节点2、节点3







