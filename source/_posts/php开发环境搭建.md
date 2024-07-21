---
title: php开发环境搭建
date: 2020-07-24 15:28:40
tags: [php,apache]
categories: 后端学习
thumbnail: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1597144364443&di=c901c7a58f2434d2c54d20f2baeecfed&imgtype=0&src=http%3A%2F%2Fimg11.weikeimg.com%2Fdata%2Fuploads%2F2013%2F05%2F14%2F3075164565191d7d9321e0.jpg
toc: true
---

在Windows平台上搭建~~~(*^▽^*)

其实在下载的文件中怎么配置上面都有说明，只是全是鸡肠看起来蛋疼

操作系统：Windows 10

<!-- more -->

# 安装 Apache

[Apache下载地址](https://www.apachehaus.com/cgi-bin/download.plx)

## 配置Apache

解压下载的压缩包，打开Apache的配置文件`conf/httpd.conf`

- 找到配置项`Define SRVROOT`，将里面的路径更改为自己下载的 apache 的根目录

> 路径要使用 `/` 而不是 `\`

- 找到 `ServerName`，去掉前面的`#`，这里配置自己的域名，如果没有域名，就写个IP地址上去。
  - 例如：`ServerName 127.0.0.1:80`，也可以写成 localhost:80

## 安装 Apache

以**管理员**的身份打开命令提示符，切换到 Apache目录下的 `bin` 目录，执行命令：

安装：

```
httpd.exe -k install
```

- 提示：这里安装之前Apache文件夹的名字叫什么，安装后Apache服务的名字就叫什么

卸载：

```
httpd.exe -k uninstall
```

## 启动Apache服务

有三种启动方式，通过命令行、通过Apache软件和通过系统服务

### 通过命令行启动 Apache

```
# 启动服务
net start Apache
# 停止服务
net stop Apache
```

### 通过软件启动 Apache

打开apache目录下的bin目录，双击 `ApacheMonitor.exe` 启动

### 通过系统服务启动Apache

任务管理 > 服务 > 打开服务 > 启动apache

### 启动失败

运行之后提示失败或者浏览器输入`localhost`没反应，可能**端口占用**问题

在*bin*目录下运行

```shell
httpd
# 或者powershell下运行
.\httpd
```

看看提示是不是端口占用，提示`80`端口占用，修改`httpd.conf`里面的`Listen`；提示`443`端口占用，修改`/extra`下的`httpd-ssl.conf`和`httpd-ahssl.conf`里面的`Listen`

## 访问测试

浏览器输入：`localhost`，如果看到有页面出现就是安装成功了。看到的页面是`/htdocs/index.html`。

# 安装PHP

[PHP 7.4 下载地址](https://windows.php.net/download#php-7.4)，找到 `Thread Safe` 版本，下载 zip 压缩包

## 创建配置文件

解压之后，找到 `php.ini-development` 文件，复制粘贴一份，然后改名为 `php.ini`

## 在Apache中引入PHP模块

- 打开Apache的配置文件 httpd.conf，引入 PHP 为 Apache 提供的DLL 模块

  ```
  LoadModule php7_module "php在电脑中的路径/php7apache2_4.dll"
  <FilesMatch "\.php$>
    setHandler application/x-httpd-php
  </FilesMatch>
  PHPIniDir "php在电脑中的路径/php-7.4.7"
  LoadFile "php在电脑中的路径/libssh2.dll"
  ```

- 配置 Apache 的索引页

  - 访问 localhost 的时候，实际上是在访问 localhost/index.html。在配置文件中搜索 `DirectoryIndex`

  - 起初的模样

    ```
    <IfModule dir_module>
        DirectoryIndex index.html
    </IfModule>
    ```

  - 加上另外的 index.php 

    ```
    <IfModule dir_module>
        DirectoryIndex index.php index.html
    </IfModule>
    ```

## 访问测试

修改了 Apache 配置文件后，要重新启动一下。

在 Apache 的 htdoc 目录下创建一个 test.php，编辑内容：

```php
<?php
	phpinfo();
?>
```

通过访问 `localhost/test.php`测试是都安装成功 

![img](/images/phpinfo.png)

## 打开常用扩展

打开 `php.ini`，搜索 `extension_dir`，增加一行 `extension_dir = "php安装的目录/ext"`

一般会启用的功能：

```ini
extension=curl
extension=gd2
extension=mbstring
extension=mysqli
extension=openssl
extension=pdo_mysql
```

# web服务器配置

在每次修改配置文件之后都要重启 Apache 才会生效。如果想恢复默认，在`conf/original`目录中获取

## 配置虚拟主机

其实在**真正使用**的时候，在购买域名平台上的控制台中就可以增加和修改 dns 的映射关系。

在**学习过程**中使用Windows系统配置，可以更改系统的 `hosts` 文件，这个文件用于配置域名和IP地址之间的解析关系

### 配置域名

用**管理员**打开 `C:/Windows/System32/drivers/etc` 目录下的 `hosts` 文件，配置域名和IP地址的映射关系

```
127.0.0.1 php.test
127.0.0.1 www.php.test
```

### 启用辅配置文件

辅配置文件是 Apache 配置文件 httpd.conf 的扩展文件，默认是不启动的，去掉 `#` 将它启动

```
Include conf/extra/httpd-vhost.conf
```

### 配置虚拟主机

打开 `httpd-vhost.conf`文件

将下面这一块注释掉，作为参考

```
#<VirtualHost *:80>
#    ServerAdmin webmaster@dummy-host.example.com
#    DocumentRoot "${SRVROOT}/docs/dummy-host.example.com"
#    ServerName dummy-host.example.com
#    ServerAlias www.dummy-host.example.com
#    ErrorLog "logs/dummy-host.example.com-error.log"
#    CustomLog "logs/dummy-host.example.com-access.log" common
#</VirtualHost>
```

新增虚拟主机

```
# 第一个虚拟主机
<VirtualHost *:80>
    DocumentRoot "D:/web/apache/htdocs"
    ServerName localhost
</VirtualHost>
# 第二个虚拟主机
<VirtualHost *:80>
    DocumentRoot "D:/web/www/thinkphp/public"
    ServerName thinkphp.test
    ServerAlias www.thinkphp.test
</VirtualHost>
```

- ServerAlias 取别名，不管访问哪个，都是指向同一网站

> 这时已经是可以访问配置好的页面了，访问 localhost 或者 thinkphp.test 测试配置
>
> 下面的是一些自定义的功能

[]~(￣▽￣)~* ，✿✿ヽ(°▽°)ノ✿，ヾ(◍°∇°◍)ﾉﾞ，(*^▽^*)，\(^o^)/~，Thanks♪(･ω･)ﾉ，٩(๑>◡<๑)۶，O(∩_∩)O哈哈~



## 访问权限控制

控制服务器中哪些文件允许被外部访问，在 httpd.conf 中，默认站点目录 `htdocs` 已经配置为允许外部访问，其他目录要手动配置。以 `www.admin.test` 为例

在 `httpd-vhost.conf` 文件中：

```
<VirtualHost *:80>
    DocumentRoot "D:/web/www.admin.test"
    ServerName admin.test`
</VirtualHost>
<Directory "D:/web/www.admin.test">
    Require local
    #Require all granted（充许局域网内其他电脑访问）
    #Require all denied（不充许局域网内其他电脑访问）
</Directory>
```

- Require local：只允许本地访问
- Require all granted：允许所有访问
- Require all denied：拒绝所有访问

## 分布式配置文件

为目录单独进行配置的文件，可以实现在不中期服务器的前提下更改某个目录的配置，编辑 httpd-vhost.conf 文件

```
<Directory "D:/web/www.admin.test">
    Require local
    AllowOverride All
</Directory>
```

- 添加 `AllowOverride All` 后， Apache 回到站点下各个目录中读取名称为 “ .htaccess ” 的分布式配置文件，该文件中的配置将会覆盖原有的配置
- 这可能会影响服务器运行效率，如果想关掉，改为 `AllowOverride None`

## 目录浏览功能

在目录“D:/web/w ww.admin.test”中创建文件 `.htaccess`，编写配置：

```
Options Indexes
# 不想使用的话
Options -Indexes
```

## 自定义错误页面

在遇到错误的时候，Apache会使用 error 目录中的模板显示一个页面，通过 `ErrorDocument` 指令对不同的状态码进行配置

在 <Directory> 或 `.htaccess` 中，加入：

```
ErrorDocument 403/403.html
ErrorDocument 404/404.html
ErrorDocument 500/500.html
```



✿✿ヽ(°▽°)ノ✿  如果深入配置的话，后面还可以配置 mysql 和 thinkPHP

# 了解一下

了解周边小知识，提高学习兴趣~~~奥利给！！！

## 关于Apache

因为 Apache HTTP Server Project 它本身不提供发行的软件下载，提供下载的比如有：[ApacheHaus](http://www.apachehaus.com/cgi-bin/download.plx)，[Apache Lounge](http://www.apachelounge.com/download/)等等。这里使用的服务器软件简称是 `httpd`，是其中的一项产品，在浏览器搜索的时候搜索`apache httpd`就可以找到下载的地方了。Apache 旗下还有别的产品，比如和 Java 比较般配的 `Tomcat`，也是 Apache 旗下的。

### 配置项

- 在上面配置Apache引入PHP文件的时候，LoadModule 是加载模块的指令，加载了 php7_module 模块，下面的代码时对PHP文件的解析，利用正则表达式匹配 " .php " 文件，然后通过 setHandler 提交给PHP处理。PHPIniDir 用于指定 php.ini 文件保存的目录
- 在上面的设置默认入口文件的时候，首先检测是否存在 inidex.php，然后再检测 index.html

### 配置虚拟主机相关知识

虚拟主机是 Apache 提供的**一个功能**，通过虚拟主机可以**在一台服务器上部署多个网站**，而不同的域名可以解析到同一个IP地址上。

因此，当用户通过不同的域名访问同一台服务器时，虚拟主机功能就可以让用户访问到不同的网站。

### 会遇到的问题

#### 端口占用

Apache 默认监听 80 端口，如果端口被占用，Apache将无法启动。

- 在Windows中可以在命令提示符中使用`net -ano` 查看当前的端口情况 
- 使用 `tasklist | findstr "PID"`可以查看当前占用端口的是什么应用程序

#### 安装多个Apache

Apache可以多个服务同时进行工作，前提是要服务名称和端口号不冲突。在Apache配置文件中，找到`Listen`，就可以修改监听的端口

安装多个Apache：

```
httpd.exe -k install -n apache2
```

卸载

```
httpd.exe -k uninstall -n apache2
```



## 关于PHP

### 版本

PHP除了版本的选择，还有一个选择就是 **Thread Safe**（线程安全）和 **No Thread Safe**（非线程安全）

PHP和Apache搭配的时候，选择Thread Safe

### 配置

php.ini-development：开发环境配置模板

php.ini-production：生产模式，适合上线的时候使用