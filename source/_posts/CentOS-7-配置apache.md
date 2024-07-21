---
title: Linux安装Apache
date: 2020-09-15 09:22:42
tags: [Linux,apache]
categories: 部署运维
thumbnail: https://s3.ax1x.com/2021/01/23/sHSjVP.png
cover: https://s3.ax1x.com/2021/01/23/sHSjVP.png
toc: true
---

操作系统：CentOS 7

Apache版本：2.4.x for Linux

 <!-- more -->

> Apache HTTP Serve 习惯上叫作 Apache，实则是httpd，apache旗下还有其他服务器软件，Apache Tomcat

# 安装

- CentOS 可以直接使用 `yum` 来安装，方便快捷，不过这里使用 *编译安装* 的安装方法，自定义性高一些
- 安装依赖：`apr,apr-util,pcre`。有可能会遇到没有安装依赖的情况，文章底部有安装依赖环境过程

下载地址：https://mirrors.bfsu.edu.cn/apache//httpd/httpd-2.4.46.tar.gz

```bash
wget https://mirrors.bfsu.edu.cn/apache//httpd/httpd-2.4.46.tar.gz
```

解压

```bash
tar -zxvf httpd-2.4.46.tar.gz
```

进入解压之后的文件夹

```bash
cd httpd-2.4.46
```

![压缩包内容](https://s3.ax1x.com/2021/01/23/s7ZuVJ.png)

运行可执行文件 `configure` 配置 apache

```bash
./configure
```

- 可选项什么都不选，默认安装在 <span style="color: darkcyan;">/usr/local/apache2</span>
- 还有很多配置选项，戳 <a href="http://httpd.apache.org/docs/2.4/programs/configure.html" style="color: green">configure手册页</a>
- 如果遇到<b style="color: red;">配置错误</b>：`configure: error: APR not found.  Please read the documentation.` 因为缺少`APR`和`APR-Util`，前往文章底部查看安装方法
- 如果遇到<b style="color: red;">配置错误</b>：`configure: error: pcre-config for libpcre not found. PCRE is required and available from http://pcre.org/` 因为缺少 `pcre`，前往文章底部查看安装方法

配置成功

![配置成功](https://s3.ax1x.com/2021/01/23/s7ULPx.png)

编译和安装

```bash
make && make install
```

- 如果遇到错误：`xml/apr_xml.c:35:19: 致命错误：expat.h：没有那个文件或目录`，缺少`expat`，前往文章底部查看安装方法

安装成功

![httpd安装成功](https://s3.ax1x.com/2021/01/23/s70cOU.png)

# 启动apache服务

## 修改配置文件

进入 apache 所处目录和目录结构 `/usr/local/apache2`

![apache目录](https://s3.ax1x.com/2021/01/23/s7BUc6.png)

修改配置文件

```bash
vim conf/httpd.conf
```

找到 `ServerName` 配置，去掉前面的 `#` 注释，并且修改它的值（或者直接增加上去）

```conf
ServerName localhost
```

保存退出，然后进入 `bin` 目录，运行可执行文件 `apachectl` 以开启 httpd 服务

![httpd目录的可执行文件](https://s3.ax1x.com/2021/01/23/s7vx5F.png)

```
./apachectl -k start
```

- 如果出现错误：`./apachectl:行95: lynx: 未找到命令`，运行 `yum install -y lynx` 安装即可

查看 httpd 服务监听的端口

```bash
netstat -antup | grep httpd
```

![默认监听80端口](https://s3.ax1x.com/2021/01/23/sHSdH0.png)

配置防火墙开放 httpd 服务默认监听的 `80` 端口

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent 
```

重启防火墙以实现端口的开放配置

```bash
systemctl restart firewalld
```

测试 httpd 服务，在浏览器访问**主机IP地址**

![成功配置](https://s3.ax1x.com/2021/01/23/s7zWtS.png)

## 加入系统命令

直接使用软链接将可执行文件链接到 `/usr/local/bin`

```bash
ln -s /usr/local/apache2/bin/apachectl /usr/local/bin
```

- 这样在任何目录下都可以使用`apachectl`来控制 httpd服务了

# 错误

## 没有APR

APR下载地址：https://mirrors.bfsu.edu.cn/apache//apr/apr-1.7.0.tar.gz

APR-Util下载地址：https://mirrors.bfsu.edu.cn/apache//apr/apr-util-1.6.1.tar.gz

下载和解压这两个压缩包

```bash 
wget https://mirrors.bfsu.edu.cn/apache//apr/apr-1.7.0.tar.gz https://mirrors.bfsu.edu.cn/apache//apr/apr-util-1.6.1.tar.gz
```

解压这两个压缩包

```bash
tar -zxvf apr-1.7.0.tar.gz
tar -zxvf apr-util-1.6.1.tar.gz
```

将`apr`包移到 `/解压后的apache包/srclib/apr`，`apr-util`包移到 `/解压后的apache包/srclib/apr-util`。<b style="color: red;">注意：</b>在安装这两个东西的时候，**包目录不允许有版本号**，所以移动的时候顺便改名

文中的包都下载在 <span style="color: gold">/root</span> 目录下（注意根据自己所处的目录执行命令）

```bash
mv apr-1.7.0 ./httpd-2.4.46/srclib/apr
```

```bash
mv apr-util-1.6.1 ./httpd-2.4.46/srclib/apr-util
```

![包的位置](https://s3.ax1x.com/2021/01/23/s71Izj.png)

> 完成后执行在apache包的根目录执行 `./configure`，接着上面的安装步骤

## 没有PCRE

pcre下载地址：https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz

其他版本👉<a href="https://ftp.pcre.org/pub/pcre/">戳</a>

```bash
wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz
```

解压，进入pcre文件夹

```bash
tar -zvxf pcre-8.44.tar.gz && cd pcre-8.44
```

配置

```bash
./configure
```

编译和安装

```bash
make && make install
```

![pcre安装](https://s3.ax1x.com/2021/01/23/s7UKC6.png)

> 完成后执行在apache包的根目录执行 `./configure`，接着上面的安装步骤

## 没有expat

expat下载地址：https://github.com/libexpat/libexpat/releases/download/R_2_2_10/expat-2.2.10.tar.gz

```bash
wget https://github.com/libexpat/libexpat/releases/download/R_2_2_10/expat-2.2.10.tar.gz
```

解压和进入目录

```bash
tar -zxvf expat-2.2.10.tar.gz && cd expat-2.2.10
```

配置

```bash
./configure
```

编译和安装

```
make && make install
```

![安装cxpat成功](https://s3.ax1x.com/2021/01/23/s70CdJ.png)

> 完成后执行在apache包的根目录执行 `make && make install`，接着上面的安装步骤



