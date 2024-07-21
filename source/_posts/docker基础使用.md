---
title: docker基础使用
toc: true
date: 2021-03-05 09:53:33
tags: docker
categories: 部署运维
thumbnail: https://s3.ax1x.com/2021/03/05/6e7IDP.jpg
cover: https://s3.ax1x.com/2021/03/05/6e7IDP.jpg
---

<span style="color: lightblue">docker</span>版本：19.03

<!-- more -->

# 安装

## 卸载旧版本

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## 安装

下载需要的安装包

```bash
yum install -y yum-utils
```

设置镜像仓库（使用阿里云）

```bash
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新 yum 软件包索引

```bash
yum makecache fast
```

安装 docker 社区版 `docker-ce`。企业版为 `docker-ee`

```bash
yum install docker-ce docker-ce-cli containerd.io
```

启动 docker

```bash
systemctl start docker
```

查看docker的版本、docker的系统信息、帮助

```bash
docker version
docker info
docker --help
```

## 卸载

卸载依赖

```bash
yum remove docker-ce docker-ce-cli containerd.io
```

删除资源（路径可能有所不同）

```bash
rm -rf /var/lib/docker
```

# 镜像命令

## 概览

查看本地镜像

```bash
docker images
或者
docker image ls
```

搜索镜像

```bash
docker search xxx
```

下载镜像

```bash
docker pull xxx
或者
docker image pull
```

删除镜像

```bash
docker rmi
或者
docker image rm
```

## docker images

### 标题

|    标题    |      意思      |
| :--------: | :------------: |
| REPOSITORY |  镜像的仓库源  |
|    TAG     |   镜像的标签   |
|  IMAGE ID  |    镜像的id    |
|  CREATED   | 镜像的创建时间 |
|    SIZE    |    镜像大小    |

### 可选项

|     选项      |       意思       |
| :-----------: | :--------------: |
|  -a 或 --all  |   列出所有镜像   |
| -q 或 --quiet |  只显示镜像的id  |
|      -aq      | 显示所有镜像的id |

## docker search

### 标题

|    标题     |          意思          |
| :---------: | :--------------------: |
|    NAME     |        镜像仓库        |
| DESCRIPTION |      镜像描述信息      |
|    STARS    |       镜像收藏数       |
|  OFFICIAL   |  是否为docker官方发布  |
|  AUTOMATED  | 是否为自动化构建的镜像 |

### 可选项

|      选项      |  类型  |        意思        |
| :------------: | :----: | :----------------: |
| -f 或 --filter | filter |  按提供的条件输出  |
|    --format    | string |     格式化输出     |
|    --limit     |  int   | 输出数量，默认25个 |
|   --no-trunc   |        |     不截断输出     |

```bash
docker search mysql --filter=STARS=3000
```

- 搜索收藏数大于3000的 `mysql` 镜像

## docker pull

## 可选项

|          可选           |  类型  |            意思            |
| :---------------------: | :----: | :------------------------: |
|    -a 或 --all-tags     |        |     下载所有标签的镜像     |
| --disable-content-trust |        | 跳过镜像验证（默认为true） |
|       --platform        | string | 如果是多平台的话，设置平台 |
|      -q 或 --quiet      |        |       不需要详细输出       |

使用格式 `docker pull 镜像名[:tag]`

```bash
docker pull tomcat:8
```

- 不写 *tag*，默认就是 *latest*
- Aready exists、Pull complete：是分层下载，docker image 的核心，联合文件系统
- Digest：签名、防伪码
- docker.io/library/tomcat:8：真实地址

## docker rmi

### 可选项

|     选项      |          意思          |
| :-----------: | :--------------------: |
| -f 或 --force |        强制删除        |
|  --no-prune   | 不要删除未标记的父对象 |

删除指定镜像

```bash
docker rmi -f 镜像id
docker rmi -f 镜像id 镜像id 镜像id
```

删除全部镜像

```bash
docker rmi -f $(docker images -aq)
```

# 容器命令

## 概览

新建容器并启动

```bash
docker run 镜像id
```

列出所有运行的容器 

```bash
docker ps
docker container list
```

删除指定容器

```bash
docker rm 容器id
```

启动容器、重启容器

```bash
docker start 容器id
docker restart 容器id
```

停止当前正在运行的容器

```bash
docker stop 容器id
```

强制停止当前容器

```bash
docker kill 容器id
```

## docker run

`docker run [可选参数] image` 或者 `docker container run [可选参数] image`

|      参数      |           意思            |
| :------------: | :-----------------------: |
| --name="Name"  |        容器的名称         |
|       -d       |       后台方式运行        |
|      -it       |   进入容器内查看、运行    |
|       -p       |   ip:主机端口:容器端口    |
|       -p       | 主机端口:容器端口（常用） |
|       -p       |         容器端口          |
| -P（这是大写） |       随机指定端口        |
|       -v       |        绑定一个卷         |

**例1**，运行 centos

```bash
docker run -it centos /bin/bash
```

- 如果没有事先下载好镜像，会先自动下载

使用 `exit` 退出

```bash
exit
```

使用键盘退出 `Ctrl + P + Q`

**例2**，运行 nginx

```bash
docker run --name nginx-test -p 8080:80 -d nginx
```

```bash
docker run --name nginx_web -p 8080:80 -v /usr/local/share/nginx:/usr/share/nginx  -d nginx
```

```bash
docker run -p 80:80 -v /data:/data -d nginx
```

**例3**，运行 mysql

```bash
docker pull mysql:5.7
```

```bash
docker run --name mysql -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

进入容器

```bash
docker exec -it mysql bash
```

## docker exec

格式：`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

### 可选项

|       选项        |  类型  |                    意思                    |
| :---------------: | :----: | :----------------------------------------: |
|   -d, --detach    |        |             分离模式，后台运行             |
|     -e, --env     |  list  |                设置环境可变                |
| -i, --interactive |        | 即使没有连接，也要保持标准输入保持打开状态 |
|     -t, --tty     |        |               分配一个伪tty                |
|    -u, --user     | string |               用户名或者UID                |
|   -w, --workdir   | string |              容器内的工作路径              |

```bash
docker exec -it nginx-test /bin/bash
```

## docker run 和 docker exec 区别

**docker run ：**根据镜像创建一个容器并运行一个命令，操作的对象是 **镜像**；

**docker exec ：**在运行的容器中执行命令，操作的对象是 **容器**。

## docker rm

删除指定容器，正在运行的不能删除

```bash
docker rm 容器id
```

删除指定容器，正在运行的强制删除

```bash
docker rm -f 容器id
```

删除所有的容器

```bash
docker ps -a -q | xargs docker rm
```

# 权限管理

