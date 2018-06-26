---
title: MongoDB start
date: 2018-06-07 11:11:13
tags: MongoDB
categories: MongoDB
thumbnail: /img/MongoDB_bg.png
---

# 背景

最近在公司搞了个[聊天室](https://github.com/hongxuWei/let-s-talk)，用来和我家[小可爱](https://github.com/MiaoQ)说悄悄话❤。因为公司的 Skype 等会被监控啦。

初期版本特别简陋，用户信息是存放在一个服务端的 js 里， login 的时候，哈哈哈哈，去读取 js 里的变量，我都被我机智到了。

现在想加上数据库用户信息，聊天记录也要存放在数据库里。之前有弄过一段时间的 `MongoDB` 所以第一时间想到就是用它啦。

## 安装

### Windows

#### 下载安装

公司电脑是 Win10 的，先去官网下载对应的安装文件 [MongoDB Window 下载地址](https://www.mongodb.com/download-center?jmp=nav#community)

这种安装也不要什么说明啦，只是如果想要修改安装包的默认位置的话需要在  `Choose Setup Type` 这一步选择 **Custom**。之后就和安装普通的 Windows 软件一样。

这里我安装到了 D: 盘，安装完成后目录结构如下图

<p class="center">![files](files.png)</p>

#### 基础配置

**1. 配置环境变量**

为了下面敲命令方便这里把 MongoDB 的 `bin` 文件夹放入到 Windows 系统环境变量 `Path` 中。在我的安装目录中就是下面这样。

<p class="center">![path](path.png)</p>

**2. 创建数据库存放地址**

我是把数据库的内容放在 D 盘的 data 文件夹下

```shell
d:
mkdir data\db
mkdir data\log
```
**3. 创建配置文件**

创建配置文件，该文件必须设置 `systemLog.path`

配置文件在 `D:\mongodb\mongod.cfg`

内容如下
```cfg
systemLog:
    destination: file
    path: d:\data\log\mongod.log
storage:
    dbPath: d:\data\db
```

**4. 安装 MongoDB 服务**

```shell
# 管理员模式运行
# --serviceName 是注册 windows 服务名称
mongod --config "D:\mongodb\mongod.cfg" --install --serviceName "MongoDB"
```

启动 MongoDB 服务

```shell
net start MongoDB
```
关闭 MongoDB 服务

```shell
net stop MongoDB
```

至此 MongoDB 的基本安装和配置已经完成。

命令行执行

```shell
mongo
```

就可以连接数据库了