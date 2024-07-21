---
title: Nginx安装
date: 2020-11-06 16:08:41
tags: nginx
categories: 部署运维
thumbnail: https://z1.ax1x.com/2020/11/10/BLFOhR.jpg
cover: https://z1.ax1x.com/2020/11/10/BLFOhR.jpg
toc: true
---

操作系统：Linux CentOS 7

官网下载地址：www.nginx.org

<!-- more -->

预先安装：gcc、gcc-c++

Nginx 依赖包：pcre-devel、zlib-devel、openssl-devel

推荐安装：wget（用来从网络上下载文件）`yum install weget `

## 安装预先安装 gcc、gcc-c++

```bash
yum install gcc gcc-c++
```

## 安装依赖

```bash
yum install pcre-devel openssl-devel zlib-devel
```

## 安装 NGINX

通过 `wget`下载

```bash
wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

- 通过这种下载是下载官网上的压缩包（版本要根据实际情况）

在压缩包所在目录解压

```bash
tar -zxvf nginx-1.18.0.tar.gz
```

- z：表示压缩包格式为 _gzip_
- x：表示解压缩
- v：表示显示解压过程
- f：表示对指定文件进行操作

进入 Nginx 文件根目录

```bash
cd nginx-1.18.0
```

### Nginx 目录介绍

- auto 目录：存放大量的脚本文件，和 configure 脚本程序相关
- configure 文件：Nginx 自动安装脚本，用于检查环境，生成编译代码需要的 makefile 文件
- html 目录：存放默认网站文件
- src 目录：存放 Nginx 的源代码
- conf 目录：存放 Nginx 服务器的配置文件

### 安装 NGINX

在 Nginx 根目录

```bash
./configure --prefix=/usr/local/nginx --with-http_ssl_module
```

- `./configure`用于对即将安装的软件进行配置，检查当前的环境是否满足安装软件（Nginx）的依赖关系

- `--prefix`参数用于设置 Nginx 的安装目录，默认值“/usr/local/nginx”，可省略此参数或指定到其他位置

- `--with-http_ssl_module`参数用于设置在 Nginx 中允许使用 http_ssl_module 模块的相关功能（可以以后再安装）

  ![成功截图](https://s1.ax1x.com/2020/11/06/BhSrdJ.png)

```bash
make && make install
```

- make：编译
- make install：安装

## 启动 Nginx

```bash
cd /usr/local/nginx/sbin
```

```bash
./nginx
```

查看后台运行（是否启动成功）

```bash
ps aux | grep nginx
```

- ps：查看后台

  - a：显示现行终端下的所有程序，包括其他用户的程序
  - u：以用户为主的格式来显示
  - x：显示所有程序，不以终端机来区分

- grep：正则，这里匹配的是 _nginx_ 字符，并且突出显示

  ![查看后台运行](https://s1.ax1x.com/2020/11/06/BhCGIH.png)

  > ps aux 和 ps -aux 是用区别的，可能显示结果没什么区别

## 开启 Nginx 默认的 80 端口

查看端口占用

```bash
netstat -tlnp
```

![端口占用情况](https://s1.ax1x.com/2020/11/06/BheFLn.png)

开启允许外部访问 80 端口

```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

- -I INPUT：表示在 INPUT（外部访问规则）中插入一条规则
- -p tcp：指定数据包匹配的协议（tcp、udp、icmp 等），这里指定 tcp
- --dport 80：用于指定数据包匹配的目标端口号，这里指定 80 端口
- -j ACCEPT：指定对数据包的处理操作(ACCEPT、DROP、REJECT、REDIRECT 等)

## 访问测试

浏览器输入 IP 就可以访问到 Nginx 的默认页面

![Nginx默认页面](https://s1.ax1x.com/2020/11/06/BhGTWn.png)

## Nginx 的启动和停止

在`/usr/local/nginx/sbin`目录中

立即停止，无论当前进程是否在处理工作

```bash
./nginx -s stop
```

- -s：表示发送信号到主进程

从容停止，完成当前工作后再停止

```bash
./nginx -s quit
```

通过 `kill` 命令停止

```bash
kill nginx的主进程PID
```

通过`killall`也可以停止

```bash
killall nginx
```

## 将 Nginx 添加到环境变量

- 添加环境变量以使用*nginx*命令来启动和关闭 nginx

查看当前 PATH 环境变量

```bash
echo $PATH
```

在`/usr/local/nginx/sbin`目录中运行得到结果：

`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin`

创建软链接

```bash
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx
```

- 前一个是源文件路径，后一个是目标文件路径

### 使用 nginx 命令管理 Nginx 状态

停止

```bash
nginx -s quit
nginx -s stop
```

重新载入配置

```bash
nginx -s reload
```

重启 Nginx

```bash
nginx -s reopen
```
