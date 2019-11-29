---
title: Docker实操
tags: JAVA
date: 2019-04-21 20:30:43
---

#### DOCKER安装

在菜鸟教程中详细提供了linux，windows, mac下的安装操作。

#### DOCKER认知

container是运行的image。容器中的程序不能后台运行，否则容器会终止

每个容器中只运行一个程序，故可将日志直接输出到控制台上。

Docker镜像含有启动容器所需要的文件系统及其内容，因此，用于创建并启动docker容器。

docker inspect 查看docker信息

docker login/logout 登陆/登出

docker push 将本地镜像推到远程仓库

docker save/load 保存/加载镜像

#### 参考

[Docker 教程](http://www.runoob.com/docker/docker-tutorial.html)



