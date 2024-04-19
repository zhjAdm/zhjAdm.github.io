---
title: Docker使用
tags: Docker
categories: Docker
description: 记录Docker使用命令
abbrlink: 5123
date: 2023-05-11 10:24:05
---

### Dockerfile

#### 是什么

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

#### 关键指令

| 指令       | 说明                                     |
| ---------- | ---------------------------------------- |
| FROM       | 指定基础镜像                             |
| RUN        | 在构建过程中在镜像中执行命令。           |
| CMD        | 指定容器创建时的默认命令。（可以被覆盖） |
| ENTRYPOINT | 设置容器创建时的主要命令。（不可被覆盖） |
| EXPOSE     | 声明容器运行时监听的特定网络端口。       |
| ENV        | 在容器内部设置环境变量。                 |
| ADD        | 将文件、目录或远程URL复制到镜像中。      |
| COPY       | 将文件或目录复制到镜像中。               |
| VOLUME     | 为容器创建挂载点或声明卷。               |
| WORKDIR    | 设置后续指令的工作目录。                 |

#### 构建镜像

```sh
docker build -t runoob/ubuntu:v1 . 
```

-t : 镜像标签













