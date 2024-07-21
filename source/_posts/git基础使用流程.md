---
title: git基础使用流程
toc: true
date: 2021-04-21 19:44:42
tags: [git,github]
categories: 部署运维
thumbnail: https://z3.ax1x.com/2021/04/22/cqsfLq.png
cover: https://z3.ax1x.com/2021/04/22/cqsfLq.png
---

简单记录<span style="color: orange;">git</span>和<span style="color: purple;">github</span>的使用

操作系统：Windows

<!-- more -->

# 安装和配置

## 安装

👉 <a href="https://git-scm.com/downloads">Git下载地址</a>

下载后安装的时候一直点下一步就行

安装成功后可以得到3个东西：git bash、git cmd、git gui。

![git install successfully](https://z3.ax1x.com/2021/04/21/cqkbFA.png)

git bash 和 git cmd 在基本上是没什么区别的，深入的现在先不管。git gui 就是图形界面，一般情况下命令行模式就行了。

## 配置

配置提交人的名字

```bash
git config --global user.name 你的名字
```

配置提交人的邮箱

```bash
git config --global user.email 你的邮箱
```

查看现有的配置信息

```bash
git config --list
```

如果要重新配置这些信息，重写就行。也可以使用配置文件配置，在*当前用户目录下的 `.gitconfig`* 文件中。

# 使用

介绍本地git仓库的基本使用和远程仓库github的基本使用

## git使用

在命令行中操作，git bash、git cmd、cmd、powershell 随便哪个都行

### 基本工作流程

1. `git init`：初始化 git 仓库
2. `git status`：查看文件状态（被git管理的文件和没被git管理的文件）
3. `git add 文件`：将文件交给git管理（一般直接`git add .`，把整个目录管理）
4. `git commit -m "要写的备注信息"`：向仓库提交内容，`-m`是messages的缩写
5. `git log`：查看提交记录

图片过程

![初始化git仓库](https://z3.ax1x.com/2021/04/21/cqZOAg.png)

![查看当前状态，红色代表还未被管理的文件](https://z3.ax1x.com/2021/04/21/cqepj0.png)

![将文件交给git管理，然后查看状态，绿色就ok了](https://z3.ax1x.com/2021/04/21/cqenjx.png)

![将内容提交到本地仓库](https://z3.ax1x.com/2021/04/21/cqeQHO.png)

![再次使用 git status 查看情况，已经空了](https://z3.ax1x.com/2021/04/21/cqeBVS.png)

![查看提交记录](https://z3.ax1x.com/2021/04/21/cqe5aF.png)

### 撤销

使用 `git commit` 提交的时候除了提交到本地仓库，还会存在*暂存区* 中。

用暂存区的文件覆盖本地工作目录下的文件

```bash
git checkout 文件
```

将文件从暂存区中删除（不会删除工作目录中的文件）

```bash
git rm --cached 文件
```

将仓库中指定的记录恢复出来（这个ID可以使用`git log`查看）

```bash
git rest --hard commitID
```

> **使用情况：**
>
> 有 **A → B → C → D** 四个记录，当前工作目录出现了问题，想回到 **B**，在回到 **B** 的时候，工作目录改变，**C**、**D** 记录也会被删除

### 分支

意思就是可以在一个新的分支上开发新的功能，另外分支做其他事情

👉 **比如**：

主分支（master）：第一次提交时自动创建的，一般情况下不会在主分支上直接进行开发

开发分支：从主分支中分出，用于开发，在开发完成之后再并入主分支

功能分支：基于开发分支中分出，如开发一个独立功能，功能开发完之后再并入开发分支，然后删掉这个功能分支以保持这个分支的干净

bug分支等等实现其他功能的分支

1. `git branch`：查看分支（在分支名称前面有`*`，代表当前选中的分支）

   ![分支](https://z3.ax1x.com/2021/04/21/cqJpvD.png)

2. `git branch 分支名`：创建分区。在哪个分支中操作，就是基于哪个分支创建。

   ![创建分支](https://z3.ax1x.com/2021/04/21/cqJJP0.png)

3. `git checkout 分支名`：切换分支

   ![切换分支](https://z3.ax1x.com/2021/04/21/cqJwqJ.png)

4. `git merge 来源分支`：合并分区。如：将开发分支合并到主分支，<span style="color: red;">要先将当前分支切换到主分支</span>，因为被合并的是开发分支

   ![合并分支](https://z3.ax1x.com/2021/04/21/cqJfqH.png)

5. `git branch -d 分支名称`：删除分支。分支没有被合并之前是不会被删除的，如果想强制删除，将`-d`改为`-D`。在当前分支删除当前分支也是不行的

   ![删除分支](https://z3.ax1x.com/2021/04/21/cqYom4.png)

### 暂时保存修改

出现情况：当前工作还没完成，临时有其他工作要做，但是又还不想使用`git commit`提交，分支在没`commit`之前是不可切换的。可以使用临时保存（分支临时切换）

1. `git stash`：存储临时改动
2. `git stash pop`：恢复改动

>这两个命令是独立于分支的，在其他分支上执行也会产生结果，所以要注意当前分支。

## github使用

### 多人工作的流程

1. 小明在本地创建仓库，然后`push`到远程仓库
2. 小李等其他人，将远程的代码`clone`下来（这些人不需要创建本地仓库）
3. 小李等人完成工作后，将代码`push`上远程仓库
4. 小明将远程代码`pull`回来，就得到来小李等人合作写好的代码

### 首次连接github

👉<a href="https://github.com/">github官网</a>

登录之后，点击右上角的➕，点`new repository`新建一个仓库

填上仓库的名字，点击公开（public）或者不公开（private）都行，其他的选项可以先不要管。点击绿色按钮（create repository）即可创建

![创建新仓库](https://z3.ax1x.com/2021/04/21/cqtHKS.png)

进入里面后有教你操作的提示，分为三部分。这里已经有了本地仓库，所以按照第二种操作就行

![连接远程仓库](https://z3.ax1x.com/2021/04/21/cqNYGt.png)

复制仓库的 HTTPS 或者 SSH 链接 (这里以上面的链接为例)，**在刚才操作git的目录中**

```bash
git remote add origin https://github.com/dsjerry/testGitHub.git
```

```bash
git branch -M main
```

```bash
git push -u origin main
```

-  `origin`：用于代替这一长串https或ssh的别名
-  `-M`：修改分支的名字。（git自动生成的是master分支，不过传到github的时候，github推荐的是将master改为main）因此不是必须的。
-  **第一次**要这么长，以后直接 **`git push`** 就行。

以上操作就可以实现连接远程的仓库，连接的操作已经完成了

> 其实第一条命令可以说是复合命令，下面分解具体步骤，当然现实中就随便怎么选都行，如果你不想了解，直接跳过就行

回到开始的话

首先

```bash
git push https://github.com/dsjerry/testGitHub.git master
```

后来因为这串HTTPS或者SSH是在太长，所以要给这串东西取个别名（**origin**就是别名）

```bash
git remote add origin https://github.com/dsjerry/testGitHub.git
```

以后使用一下命令就行。

```bash
git push origin master
```

> 第一次提交要填密码，以后就不用了。密码在这里看得到：控制面板 → 凭据管理器 → Windows凭据

使用`-u`记住仓库地址、别名和分支

```bash
git push -u origin master
```

以后直接使用`git push`就行

```bash
git push
```

这就是分解的步骤。

### 克隆 推送 拉取

#### 克隆(clone)

克隆（clone）到本地（小李克隆小明的，别名也会一起克隆）

```bash
git clone https://github.com/dsjerry/testGitHub.git
```

#### 推送(push)

小李将本地代码推送（push）到小明的远程仓库。在推送前，小李等人是没有权限推送到小明的仓库的。需要小明邀请小李。

打开当前github仓库页面，找到**settings**，选中**Manage access**，点击**Invite a collaborator**然后把小李的github账号添加上去就行

![仓库授权](https://z3.ax1x.com/2021/04/21/cqwgk4.png)

小李在接收到邀请邮件后，同意邀请就行了。

![同意邀请](https://z3.ax1x.com/2021/04/21/cqwqtH.png)

#### 拉取(pull)

经过一轮开发后，小明就可以把大家的成果拉去到自己本地啦

```bash
git pull https://github.com/dsjerry/testGitHub.git master
```

- 拉取属于*读* 操作，不需要密码验证

#### 可能的冲突

两个人修改同一个文件时可能会遇到冲突，小明发了新的版本，小李的旧版本推不上。

小李可以先把小明的新版 `pull` 下来，然后在终端中会有 `<<<......>>>`这样出现，指出了冲突的地方，然后以 `=====`将不同的冲突分隔开

### Fork

即使不是本地成员，也可以像仓库提交代码，前提是这些代码要经过审核才能并入主要代码。

**操作方法**

访问别人的仓库，然后点击**Fork**，相当于复制别人的；然后`clone`到本地修改，修改完成后`push`上去

![Fork](https://z3.ax1x.com/2021/04/22/cqriND.png)

对于仓库的管理人员，要接受外来的推送，在仓库页面中点**Pull request** → **New pull request** → **Create pull request**

![推送请求](https://z3.ax1x.com/2021/04/22/cqrtuq.png)

随后，点击**commits**，会看到修改的内容，点**file changed**可以查看到文件的改变。仓库的管理人员点击**comfirm merge**就可以接受合并。

### SSH免登陆

通过本机的私钥和服务器端的公钥进行连接认证，认证通过之后就行

**生成私钥**

输入命令，然后一直回车。生成的文件会在系统目录下的用户目录下的 **.ssh** 文件

```bash
ssh-keygen
```

- id_rsa：私钥，存在本地的
- id_rsa.pub：公钥，放在github上的

设置公钥，在github页面，点击自己的**头像**，点 **settings** → **SSH and GPG keys**，选中 **New SSH key**添加自己的公钥

以后再使用 `git push`的时候，使用 SSH 链接的时候就可以不用输入密码

### git文件忽略

有些文件是不需要推送到远程仓库的，比如通过**NPM**下载的包，只要保存**package.json**文件就行

在项目的根目录新建文件 `.gitignore`，不想要传什么内容，直接卸载这个文件里面（一项占一行），如：

```
node_modules
image
test.html
```

<!-- <span style="color: #f44d27; font-size: 150px">git</span>      <span style="font-size: 50px">和  </span><span style="color: #5f2781; font-size: 150px">github</span>-->



​    <!-- <span style="font-size: 50px">基本使用</span>-->













