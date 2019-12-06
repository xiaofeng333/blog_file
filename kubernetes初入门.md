---
title: kubernetes初入门
date: 2019-12-04 16:37:02
tags: kubernetes
---

### 基本知识

在k8s中, Service(服务)是分布式集群架构的核心, 一个Service拥有如下关键特征。

* 拥有一个唯一指定的名字。
* 拥有一个虚拟IP(Cluster IP、Service IP或VIP)和端口号。
* 能够提供某种远程服务能力。
* 被映射到了提供这种服务的一组容器应用上。

给`pod`(其中运行了服务进程容器)贴上`label`(标签), 然后给相应的Service定义Label Selector(标签选择器), 这样就将service和pod关联了起来。

pod运行在node(节点)的环境中, 通常在一个node中运行几百个pod。

每个pod里运行一个pause容器, 其他为业务容器。所有业务容器共享pause容器的网络栈和volume挂载卷。

在集群管理方面, Kubernetes将集群中的机器划分为一个Master节点和一群工作节点(Node)。

* Master节点上运行着集群管理相关的一组进程kube-apiserver、kube-controller-manager和kube-scheduler,这些进程实现了整个集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错管理等功能。
* Node作为集群中的工作节点, 运行真正的应用程序。 在Node上运行kubelet、kube-proxy服务进程, 负责Pod的创建、启动、监控、重启、销毁, 以及实现负载均衡器。

在k8s中扩容时, 只需为需要扩容的Service关联的Pod创建一个Replication Controller(RC)即可。

在RC定义文件中包含以下三个关键的信息。

* 目标Pod的定义。
* 目标Pod需要运行的副本数量(Replicas)。
* 要监控的目标Pod的标签(Label)。



### 环境安装

1. 安装etcd和kubernetes。

   ```shell
   yum install -y etcd kubernetes
   ```

2. 修改配置文件。

   ```shell
   # 将OPTIONS的内容设置为'--selinux-enabled=false --insecure-registry gcr.io'
   vim /etc/sysconfig/docker
   
   # 把--admission_control参数中的ServiceAccount删除
   vim /etc/kubernetes/apiserver
   ```

3. 按顺序启动所有服务。

   ```shell
   systemctl start etcd
   systemctl start docker
   systemctl start kube-apiserver
   systemctl start kube-controller-manager
   systemctl start kube-scheduler
   systemctl start kubelet
   systemctl start kube-proxy
   ```

4. 至此单机版的kubernetes搭建完成。

