---
title: Nginx缓存
date: 2020-11-10 15:50:13
tags: nginx
categories: 部署运维
thumbnail: https://s1.ax1x.com/2020/11/10/BLFOhR.jpg
pcover: https://s1.ax1x.com/2020/11/10/BLkwE4.png
toc: true
---

操作系统：Linux CentOS 7

Nginx版本：1.18

<!-- more -->

>  课代表：① 永久缓存配置、② 临时缓存配置、③ 缓存清理配置

## 永久缓存配置

### 前提准备

准备两个虚拟机，一台作为**Web缓存服务器**（192.168.146.24），一台作为**内容源服务器**（192.168.146.27）

#### 配置 Web缓存服务器

在nginx配置文件中新建虚拟主机 cache.test.conf

```bash
cd /usr/local/nginx/conf/vhost
vim cache.test.conf
```

内容为

```nginx
server {
        listen	80;
        server_name	192.168.146.24;
        location / {
                root cache;
                proxy_store on;
                proxy_store_access user:rw group:rw all:rw;
                proxy_temp_path cache_tmp;
                if ( !-e $request_filename){
                	proxy_pass http://192.168.146.27;
                }
        }
}
```

在主配置文件（nginx.conf）中导入虚拟机

```nginx
include /usr/local/nginx/conf/vhost/cache.test.conf
```

在nginx的根目录中，创建保存缓存的文件夹 `cache`

```bash
mkdir cache
```

更改对此文件夹的用户权限，要求与Nginx工作进程的用户相同（比如nobody）

```bash
chown -R  nobody:nobody cache
```

重新加载Nginx配置文件

```bash
nginx -s reload
```

#### 配置内容源服务器

新建虚拟机并且使用 `include` 指令在主配置文件（nginx.conf）中引入

```bash
vim cache.test.conf
```

内容为

```nginx
server {
	listen 80;
	server_name localhost;
	location / {
		root html/cache.test;
		index cache.html;			
	}
```

创建对应的文件作为此服务器的测试文件。在Nginx根目录的*html*文件夹中

新建文件夹

```bash
mkdir cache.test
```

新建测试文件 cache.html

```bash
cd cache.test
vim cache.html
```

内容为

```html
<h1>Welcome to 192.168.146.27</h1>
```

在此目录下再创建一个新的目录`test`，在`test`里面创建创建 test.html，并且放一张图片(nginx.jpg)在此目录

```html
<h1>192.168.146.27/test/test.html</h1>
<img src="nginx.jpg" />
```

### 访问测试

分别在浏览器访问 *192.168.146.24/cache.html* 和 *192.168.146.24/test/test.html*

![访问缓存服务器](https://s3.ax1x.com/2020/11/13/DpyAOK.png)

![访问缓存服务器](https://s3.ax1x.com/2020/11/13/DpyfpR.png)

在刚刚创建的 `cache` 文件夹中，使用 `tree`查看目录内容

![缓存内容](https://s3.ax1x.com/2020/11/13/Dp6dED.png)

这些内容访问是web缓存服务器(192.168.146.24)的内容，就算改掉资源服务器(192.168.146.27)的内容，得到的还是不变的结果

- 如果没有 tree 命令，使用 `yum install -y tree` 下载就行

## 临时缓存配置

### 配置web缓存服务器

在`192.168.146.24`的主配置文件的 `http`模块中添加

```nginx
# 代理临时目录
proxy_temp_path  /usr/local/nginx/proxy_temp_dir;
# Web缓存目录和参数设置
proxy_cache_path /usr/local/nginx/proxy_cache_dir levels=1:2 keys_zone=cache_one:50m inactive=1m max_size=500m;
```

- /usr/local/nginx/proxy_cache_dir参数：表示用户自定义的缓存文件保存目录
- levels参数：表示缓存目录下的层级目录结构，它是根据哈希后的请求URL地址创建的，目录名称从哈希后的字符串结尾处开始截取
  - 假设哈希后的请求链接地址为“af7098a15e430326197ee01516fdace0”，则levels=1:2表示，第1层子目录的名称是长度为1的字符“0”，第2层子目录的名称是长度为2的字符“ce”。
- keys_zone参数：指定缓存区名称及大小，例如“cache_one:50m”表示缓存区名称为cache_one，在内存中的空间是50M
- inactive参数：表示主动清空在指定时间内未被访问的缓存。例如，“1m”清空在1分钟内未被访问过的缓存，“1h”表示1小时，“1d”表示1天等
- max_size参数：表示指定磁盘空间大小。例如，500m、10g

在`server`模块中添加临时缓存的相关配置(可以在之前配置中的虚拟主机中配置)

```nginx
server {
    	listen       80;
    	server_name  cache.test;
    	# 增加两个响应头信息，用于获知访问的服务器地址与缓存是否成功
    	add_header X-Via $server_addr;
    	add_header X-Cache $upstream_cache_status;
    	location / {
                 # 设置缓存区域名称
                 proxy_cache cache_one;
                 # 以域名、URI、参数组合成Web缓存的Key值，Nginx根据Key值哈希
                 proxy_cache_key $host$uri$is_args$args;
                 # 对不同的HTTP状态码设置不同的缓存时间
                 proxy_cache_valid 200 10m;     # 200缓存10分钟
                 proxy_cache_valid 304 1m;      # 304缓存1分钟
                 proxy_cache_valid 301 302 1h;  # 301、302缓存1小时
                 proxy_cache_valid any 1m;      # 其他未设置的状态码缓存1分钟
                 # 设置反向代理
                 proxy_pass      http://192.168.146.27;
         }
}
```

在浏览器访问测试结果

访问web缓存服务器（192.168.146.24）

![成功页面](https://s3.ax1x.com/2020/11/14/DPYn3T.png)

在 nginx 根目录下的 `proxy_cache_dir` 目录使用 tree 命令

![文件目录](https://s3.ax1x.com/2020/11/14/DPY4bj.png)

## 缓存清理配置

### 前提准备

#### 备份已安装的Nginx

先停掉Nginx，然后复制整个Nginx目录

```bash
cp -r  /usr/local/nginx /usr/local/nginx_old2
```

#### 重新编译安装Nginx

插件下载地址：https://github.com/FRiCKLE/ngx_cache_purge/releases

下载的内容会是一个*zip*包，使用`uzip`解压之后，移动

```bash
mv ngx_cache_purge-master /usr/local/ngx_cache_purge
```

去到 Nginx 的根目录

```bash
./configure \
--prefix=/usr/local/nginx \
--with-http_ssl_module \
--with-http_stub_status_module \
--add-module=/usr/local/ngx_cache_purge
```

- 这里的 `\` 表示的是当前命令还没有结束，只是换行处理

编译、安装

```bash
make && make install
```

在配置文件中（虚拟主机）加入

```nginx
location ~ /purge(/.*){
               allow 192.168.146.24;
               deny all;
               proxy_cache_purge cache_one $host$1$is_args$args;
              }

```

### 访问测试

先清除之前已经存在的缓存

在nginx根目录中进入`proxy_cache_dir`目录，把里面的内容全部删掉，然后重新访问web缓存服务器（192.168.146.24）。查看缓存：

![查看缓存](https://s3.ax1x.com/2020/11/14/DPRV9U.png)

要执行清除缓存操作，现在只需要在浏览器中输入`cache.test/purge/index.html`就可以清除缓存啦。



