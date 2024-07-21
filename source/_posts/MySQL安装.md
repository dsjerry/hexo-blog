---
title: MySQL安装
date: 2020-11-26 11:25:59
tags: mysql
categories: 部署运维
thumbnail: https://s3.ax1x.com/2020/11/26/D0Dmh4.png
cover: https://s3.ax1x.com/2020/11/26/D0Dmh4.png
toc: true
---

操作系统：`Linux CentOS`

MySQL版本：`5.7.3`

<!--more-->

下载地址：https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz

## 前提准备

### 检查预装

检查是否预先安装了`Mariadb`

```bash
rpm -qa|grep mariadb
```

![是否安装mariadb](https://s3.ax1x.com/2020/11/26/Dww9Gn.png)

删除预装

```bash
yum -y remove mariadb-libs-5.5.60-1.el7_5.x86_64
```

![删除成功](https://s3.ax1x.com/2020/11/26/DwwWzq.png)

### 下载、解压

在 Windows 中下载好安装包或者在此系统中使用`wget`下载

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz
```

解压并移动、重命名文件夹（一般会解压到 */usr/local* 目录下），在压缩包所在的目录中

```bash
tar -zxvf mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.32-linux-glibc2.12-x86_64 /usr/local/mysql
```

### 创建MySQL用户组和用户

```bash
groupadd mysql
useradd -g mysql mysql
```

创建 *data* 目录备用

```bash
cd /usr/local/mysql
mkdir data
```

### 修改mysql目录所属的用户

```bash
chown -R mysql:mysql ./
```

### 准备MySQL配置文件

在 **/etc** 下新建 *my.cnf* 文件

```bash
cd /etc
vim my.cnf
```

加入内容

```mysql
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql/mysql.sock
[mysqld]
skip-name-resolve
#设置3306端⼝
port = 3306
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装⽬录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放⽬录
datadir=/usr/local/mysql/data
# 允许最⼤连接数
max_connections=200
# 服务端使⽤的字符集默认为8⽐特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使⽤的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
```

创建 **/var/lib/mysql** 目录，并且修改权限

```bash
mkdir /var/lib/mysql
chmod 777 /var/lib/mysql
```

## 安装MySQL

### 安装

```bash
cd /usr/local/mysql/bin
```

```bash
./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --
datadir=/usr/local/mysql/data
```

> 注意，记住下面的红线，这是初始的 *root* 的密码

![mysql密码](https://s3.ax1x.com/2020/11/26/Dwgpi8.png)

### 复制脚本到启动目录

```bash
cd /usr/local/mysql
cp ./support-files/mysql.server /etc/init.d/mysqld
```

修改 **/etc/init.d/mysqld** ，修改其 *basedir* 和 *datadir* 为实际对应⽬录

```bash
vim /etc/init.d/mysqld
```

```mysql
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

### 配置MySQL系统服务并且开机自启

⾸先增加 *mysqld* 服务控制脚本执⾏权限：

```bash
chmod +x /etc/init.d/mysqld
```

同时将 *mysqld* 服务加⼊到系统服务：

```bash
chkconfig --add mysqld
```

最后检查 mysqld 服务是否已经⽣效即可：

```bash
chkconfig --list mysqld
```

![mysql的状态](https://s3.ax1x.com/2020/11/26/D08r7R.png)

- 这样就表明 mysqld 服务已经⽣效了，在2、3、4、5运⾏级别随系统启动⽽⾃动启动，以后可以直接使 ⽤ service 命令控制 mysql 的启停。

## 启动MySQL

### 启动

```bash
service mysqld start
```

![mysql成功启动](https://s3.ax1x.com/2020/11/26/D0GpEn.png)

### 将MySQL添加到环境变量

将mysql的 *bin* 添加到 *path* 环境变量，添加到环境变量就可以在任何目录使用mysql命令

编辑 **~/.bash_profile** 文件，在末尾添加：

```bash
export PATH=$PATH:/usr/local/mysql/bin
```

![~/.bash_profile文件](https://s3.ax1x.com/2020/11/26/D0YR3V.png)

执行命令使得环境变量生效

```bash
source ~/.bash_profile
```

## 登录MySQL

### 登录

```bash
mysql -u root -p
```

![登录成功](https://s3.ax1x.com/2020/11/26/D0N0oj.png)

### 修改MySQL的root密码

在 mysql 中执行命令：

```mysql
alter user user() identified by "123456";
flush privileges;
```

![修改密码root](https://s3.ax1x.com/2020/11/26/D0NjTH.png)

### 配置远程登录

在 *mysql* 中执行：

```mysql
use mysql;
update user set user.Host='%' where user.User='root';
flush privileges;
```

#### Navicat测试连接

![连接成功](https://s3.ax1x.com/2020/11/26/D0aQPA.png)

#### Windows CMD测试连接

![cmd连接成功](https://s3.ax1x.com/2020/11/26/D0didg.png)



<h1 style="text-align: center;">安装配置完成</h1>



## 在浏览器中查找MySQL其他版本

### 社区版的其他版本

https://www.mysql.com/downloads/

![下载主页](https://s3.ax1x.com/2020/11/26/D00nVU.png)

#### Windows 版本

![win version](https://s3.ax1x.com/2020/11/26/D0BFoD.png)

#### Linux 版本

![Linux 版本](https://s3.ax1x.com/2020/11/26/D0Bj78.png)