---
title: Linx命令
tags:
  - Linux
categories: Linux
description: 记录常用Linux命令
abbrlink: 57987
date: 2023-03-09 23:15:55
---

# 

# Linux命令

#### 后台运行

```shell
nohup 执行脚本 > 输出日志.log 2>&1 & 
```

#### 添加可执行权限

```shell
chmod +x 文件
```

#### 查找指定进程

```shell
 ps -aux | grep "进程名" 
```

#### 压缩

```shell
tar czvf 压缩文件名.tar.gz 被压缩文件夹
```

#### 解压

```shell
tar zvxf 压缩文件名.tar.gz -C 目标文件夹
```

#### 从服务器上下载文件

```shell
scp 用户名@服务器地址:要下载的文件路径 保存文件的文件夹路径
```

#### 上传本地文件到服务器

```shell
scp 要上传的文件路径 用户名@服务器地址:服务器保存路径 
```

#### 从服务器下载整个目录

```shell
scp -r 用户名@服务器地址:要下载的服务器目录 保存下载的目录
```

#### 上传目录到服务器

```shell
scp  -r 要上传的目录 用户名@服务器地址:服务器的保存目录
```
