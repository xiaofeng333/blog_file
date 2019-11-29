---
title: Docker开发指南阅读总结
tags: 容器
date: 2019-02-25 23:03:26
---

#### 前言

容器是轻量且可移植的仓库，包含应用程序及其依赖的组件。

容器中主进程运行多久，容器就运行多久。

Docker容器使用联合文件系统(union file system, UFS)，它允许多个文件系统以层级方式挂载，并表现为一个单一的文件系统。镜像的文件系统以只读的方式挂载，任何对运行中容器的改变则只会发生在它之上的可读写层。

Dockerfile里的每个指令都会创建一个新的层，而这个层将位于前一个层之上。由于不必要的层会使镜像变得臃肿，因此Dockerfile会把多个UNIX命令放在同一个RUN指令中，以减少层的数量。

数据卷是直接在主机挂载的文件或目录，不属于常规联合文件系统的一部分，它们允许与其他容器共享，而任何修改都会直接发生在主机的文件系统里。声明一个目录为数据卷有两种方法,第一种是在Dockerfile 里使用VOLUME 指令,第二种是在执行docker run 的时候使用-v 参数, 执行docker run 命令的时候可以指定用于挂载的主机目录。

一个常用的做法是创建数据容器，这种容器的唯一目的就是与其他容器分享数据，没有必要让数据容器一直运行。

### 内容

#### 基础命令

docker run 

 -it  以交互的方式运行容器

 -h  设定一个新的主机名

 --rm 当容器退出时，容器和相关的文件系统会被一并删掉

 --link 把容器连接起来(不推荐使用)

  -v HOST_DIR:CONTAINER_DIR

  -p: 指定端口映射，格式为：主机(宿主)端口:容器端口

docker diff  (容器的名称或ID) 查看有哪些文件被改动过

docker inspect (容器的名称或ID) 获取更多关于容器的信息

docker logs (容器的名称或ID)  得知这个容器里曾经发生过的一切事情

docker commit 将容器转为镜像(无论是在运行状态还是停止状态)

docker search 搜索已有的镜像

docker push 推镜像到仓库中

docker rm 删除一个或多个容器

  -v 删除与容器关联的卷(不是绑定挂载，或正被其他容器使用)

  -l 移除容器间的网络连接,而非容器本身

  -f 通过SIGKILL信号强制删除一个正在运行的容器

docker ps 列出正在运行容器

  -a 显示所有的容器,包括未运行的

  -f 根据条件过滤显示的内容(条件为指定,不在此列出,使用时可进行查询)

  --format 指定返回值的模板文件

  -l 显示最后被创建的容器

  -n 列出最近创建的n个容器

  --no-trunc 不截断输出,显示完整输出

  -q 静默模式,只显示容器编号

  -s 显示总的文件大小

docker pause  暂停容器中所有的进程。

docker unpause  恢复容器中所有的进程。

docker history 查看组成镜像的所有层

docker attach 允许用户查看容器内的主进程，与它进行交互

docker cp 在容器和主机之间复制文件和目录

docker exec 在容器中执行一个命令，用于执行维护工作

#### 实用命令

docker ps -aq -f status=exited 得到所有已停止容器的id。

docker rm -v $(docker ps -aq -f status=exited) 删除所有已停止的容器。

docker run --restart on-failure:5 当退出值为非0时，将尝试重启5次，之后便会放弃。

#### Dockerfile 

Dockerfile 是一个描述如何创建Docker 镜像所需步骤的文本文件。

所有Dockerfile 一定要有FROM 指令作为第一个非注释指令，FROM 指令指定初始镜像。

RUN 命令指定在镜像中运行的shell命令。

ENTRYPOINT 指令让我们指定一个可执行文件，同时还能处理传给docker run 的参数。

#### Docker基本概念

##### Docker系统架构

![1551711321388](/images/docker_structure.png)

1.Docker 守护进程，它负责容器的创建、运行和监控，还负责镜像的构建和储存 ，容器和镜像都在图的右边。

2.Docker 客户端在图的左边，它通过HTTP 与Docker 守护进程通信，值得一提的是，Docker 客户端和守护进
程是由同一个二进制文件发布的。

3.Docker 寄存服务负责储存和发布镜像。

##### 镜像生成

docker build命令需Dockerfile和构建环境的上下文。

上下文是一组本地文件和目录，它可以被Dockerfile 的 ADD 或 COPY 指令所引用，通常以目录路径的形式指定。

从构建环境的上下文中排除不必要的文件，可以使用 .dockerignore 文件。

```XML
当构建失败时，可以把失败前的那个层启动起来，这非常有助于调试。
如果需要使缓存失效，可以在执行docker build 的时候加上--no-cache 参数。
不同的镜像会共享相同的基础镜像层，
```

###### Dockerfile 的指令

Dockerfile 的注释方法是以# 作为一行的开头。

1.ADD 从构建环境的上下文或远程URL 复制文件至镜像。

2.CMD 当容器启动时执行指定的指令。

3.COPY 用于从构建环境的上下文复制文件至镜像。

4.ENTRYPOINT 设置一个于容器启动时运行的可执行文件（以及默认参数）

5.ENV 设置镜像内的环境变量。

6.EXPOSE 向Docker 表示该容器将会有一个进程监听所指定的端口。

7.FROM 设置Dockerfile 使用的基础镜像。

8.MAINTAINER 通常用于设置镜像维护者的姓名和联系方式。

9.ONBUILD 指定当镜像被用作另一个镜像的基础镜像时将会执行的指令。

10.RUN 在容器内执行指定的指令，并把结果保存下来。

11.USER 设置任何后续的RUN、CMD 或ENTRYPOINT 指令执行时所用的用户（用户名或UID）

12.VOLUME 指定为数据卷的文件或目录。

13WORKDIR 对任何后续的RUN、CMD、ENTRYPOINT、ADD 或COPY 指令设置工作目录。

##### 通过compose实现容器自动化

Compose将使我们免于自己维护用于服务编排的脚本，包括启动、连接、更新和停止容器。

使用Compose常用的命令

```yaml
up:  启动所有在Compose 文件中定义的容器，并且把它们的日志信息汇集一起。通常会使
用-d 参数使Compose 在后台运行。
build: 重新建造由Dockerfile 构建的镜像。除非镜像不存在，否则up 命令不会执行构建的动
作，因此需要更新镜像时便使用这个命令。
ps: 获取由Compose管理的容器的状态信息。
run: 启动一个容器，并运行一个一次性的命令。被连接的容器会同时启动，除非用了--nodeps
参数。
logs: 汇集由Compose管理的容器的日志，并以彩色输出。
stop: 停止容器，但不会删除它们。
rm: 删除已停止的容器。不要忘记使用-v 参数来删除任何由Docker 管理的数据卷。
```

##### 镜像分发

从Dockerfile重新构建、从寄存服务器下载，或通docker load命令从归档文件安装。

待续。。。