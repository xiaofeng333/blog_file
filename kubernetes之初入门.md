---
title: kubernetes之初入门
date: 2019-12-04 16:37:02
tags: kubernetes
---

本文为观看`Kubernetes权威指南-第二版.pdf`第一章的笔记, 主要是k8s的核心概念。

### 概要

给`pod`(其中运行了服务进程容器)贴上`label`(标签), 然后给相应的Service定义Label Selector(标签选择器), 这样就将service和pod关联了起来。

pod运行在node(节点)的环境中, 通常在一个node中运行几百个pod。

每个pod里运行一个pause容器, 其他为业务容器。所有业务容器共享pause容器的网络栈和volume挂载卷。

在集群管理方面, Kubernetes将集群中的机器划分为一个Master节点和一群工作节点(Node)。

* Master节点上运行着集群管理相关的一组进程kube-apiserver、kube-controller-manager和kube-scheduler,这些进程实现了整个集群的资源管理、Pod调度、弹性伸缩、安全控制、系统监控和纠错管理等功能。
* Node作为集群中的工作节点, 运行真正的应用程序。 在Node上运行kubelet、kube-proxy服务进程, 负责Pod的创建、启动、监控、重启、销毁, 以及实现负载均衡器。



### 基本概念和术语

k8s中大部分概念如Node、Pod、Replication Controller、Service等都可以看作一种资源对象, 几乎所有对象都可以通过`kubectl`或`API编程调用`, 来进行增删查改并将其保存在etcd中持久化存储。

kubernetes是一个高度自动化的资源控制系统, 它通过跟踪对比etcd库里保存的`资源期望状态`与`实际资源状态`的差异来实现自动控制和自动纠错的高级功能。

#### Master

Master指的是集群控制节点, 负责整个集群的管理和控制。

基本上所有k8s的控制命令都是发给Master, 由其负责具体的执行过程。

Master节点运行如下一组关键进程。

* Kubernetes API Server(kube-apiserver), 提供HTTP Rest接口的关键服务进程, 是所有资源增删查改操作的唯一入口, 也是集群控制的入口进程。
* Kubernetes Controller Manager(kube-controller-manager), Kubernetes里所有资源对象的自动化控制中心。
* Kubernetes Scheduler(kube-scheduler), 负责资源调度(Pod调度)的进程。
* etcd Server进程, 所有资源对象的数据全部是保存在etcd中的。

#### Node

Node是k8s集群中的工作负载节点, 每个Node节点都会被Master分配一些工作负载, 当某个Node宕机时, 其上的工作负载会被转移到其他Node节点上去。

每个Node节点上运行着以下一组关键进程: 

* kubelet: 负责对Pod对应的容器的创建、启停等任务, 同时与Master节点密切协作, 实现集群管理的基本功能。
* kube-proxy: 实现Kubernetes Service的通信与负载均衡机制的重要组件。
* Docker Engine: Docker引擎, 负责容器的创建和管理工作。

Node可以在运行期间动态增加到Kubernetes集群中, 前提是上述关键进程咦正确安装、配置和启动。

kubelet会定时向Master节点上报自身的情况, 当超过指定时间未上报时, Master会判定其为失联, 进而出发工作负载的转移。

#### Pod

Pod包含Pause容器及一个或多个紧密相关的用户业务容器。

Pod分为普通的Pod及静态Pod, 静态Pod的不同之处在于, 它并不存放在k8s的etcd存储里, 而是存放在某一具体文件中, 并只在此Node上启动运行。

* Pause容器作为Pod的根容器, 它的状态代表了整个容器组的状态。
* 多个业务容器共享Pause容器的IP及其挂接的Volume。
* k8s的底层网络支持集群内任意两个Pod之间的TCP/IP直接通信(采用虚拟二层网络技术实现), 故一个Pod里的容器能与另外主机上的Pod容器直接通信。

* pod volume, 定义在Pod之上, 然后被各容器挂载到自己的文件系统中的。

* Event, 一个事件的记录, 记录事件的最早产生事件、 最后重现事件、重复次数等众多信息, 是排查故障的重要参考信息。

  ```shell
  kubectl describe pod XXX
  ```

* 每个Pod都可以对其能使用的服务器上的计算资源设置限额, 当前可设置的包括CPU和memory。

  1. <font color = 'red'>CPU的资源单位数量是绝对值</font>，以千分之一的CPU配额为最小单位, 用m标识。

     即100m这个配额所代表的CPU使用量, 在48个core的机器上与1个core的机器上的CPU使用量是一样的。

  2. Memory的配额也是一个绝对值, 它的单位是内存字节数。

     在kubernetes中, 一个计算资源进行配额限定需设定以下两个参数。

  * Requests: 该资源最小申请量, 系统必须满足的需求。

  * Limits: 该资源最大允许使用量, 不能被突破, 当超过时, 可能会被kill并重启。

    ```yaml
    spec:
      containers:
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits: 
            memory: "128Mi"
            cpu: "500m"
    ```

#### Label

一个Label是一个key=value的键值对, 其中key和value由用户自己指定。

Label可以附加到各种资源上, 例如Node、Pod、Service、RC等。

一个资源对象可以定义任意数量的Label, 同一个Label也可以被添加到任意数量的资源对象上去。

Label可以在资源对象被定义时确定, 也可以在对象创建后动态添加或者删除。

##### Label Selector

Label Selector作用于Pod时, 可以被类比为`select * from pod where pod's name = 'redis-slave'`这样的语句。

当前有两种label Selector表达式, 基于等式(Equality-based)和基于集合(Set-based)的。

* name=redis-slave或env!=production。

* name in (redis-master, redis-slave)或name not in (php-frontend)。

* 多个表达式之间用`,`分割, 几个条件之间是`AND`的关系, 即同时满足多个条件。

  ```sql
  name=redis-slave, name not in (php-fronted)
  ```

label selector的使用场景如下:

* `kube-controller`进程通过资源对象RC上定义的Label Selector来筛选要监控的Pod副本的数量, 从而实现Pod副本的数量始终符合预期设定的全自动控制流程。
* `kube-proxy`进程通过Service的Label Selector来选择对应的Pod, 自动建立起每个Service到Pod的请求转发路由表, 从而实现Service的智能负载均衡机制。
* 通过对某些Node定义特定的Label, 并在Pod定义文件中使用NodeSelector这种标签调度策略, `kube-selector`进程可实现Pod`定向调度`的特性。

#### Replication Controller(RC)

RC定义了一个期望的场景, 即声明某种Pod的副本数量在任何时刻都符合某个期望值, 其定义包括如下几个部分。

- Pod期待的副本数(replicas)。
- 用于筛选目标Pod的Label Selector。
- 当目标的副本数量小于预期数量时, 用于创建Pod的Pod模板。

Master节点的Controller Manager会定期巡检系统中存活的目标Pod, 确保其数量刚好等于此RC的期望值。

可以修改RC的副本数量, 来实现Pod的动态缩放(Scaling), 或通过`kubectl scale`命令来完成。

```shell
kubectl scale rc redis-slave --replicas=3
```

下一代的RC: `Replica Set`。

#### Deployment

kubernetes 1.2引入的新概念, 为了更好的解决Pod的编排问题, 其内部使用了Replica Set来实现目的。

#### HPA(Horizontal Pod Autoscaler)

HPA指Pod横向自动扩容, 其通过追踪分析RC控制的所有目标Pod的负载变化情况, 来确定是否需要针对性地调整目标Pod的副本数。

HPA包括的负载的度量指标有:

* CPUUtilizationPercentage, 此是目标Pod所有副本自身的CPU利用率的平均值。一个Pod自身的CPU利用率是该Pod当前CPU使用量除以它的Pod Request的值。
* 应用程序自定义的度量指标(TPS或QPS)。

#### Service

service可理解为微服务架构中的一个"微服务", 其通过`Label Selector`与Pod紧密结合在一起。

每个Service分配了一个全局唯一的虚拟Ip地址, 即CLuster IP。在Service的整个生命周期内, 它的Cluster IP不会发生改变。

#### Volume

存储卷, 定义在Pod上, 与Pod的生命周期相同。

#### Persistent Volume

PV是网络存储, 不属于任何Node, 但可以在每个Node上访问。

某个Pod想申请某种条件的PV, 首先需定义一个PersistentVolumeClaim(PVC), 然后在Pod的Volume定义中引用上述PVC即可。

#### Namespace

用于实现多租户的资源隔离, 属于逻辑隔离, 不同的分组可以共享整个集群的资源。

#### Annotation

定义用户任意定义的附加信息, 以便于外部工具进行查找。 

#### k8s里的三种"ip"

* Node IP: Node节点的IP地址。

  其是k8s集群中每个节点的物理网卡的IP地址, 这是一个真实存在的物理网络, 所有属于这个网络的服务器之间都能通过这个网络直接通信, 不管它们是否属于k8s集群。

* Pod IP: Pod的IP地址。

  它是Docker Engine根据docker0网桥的IP地址段进行分配的, 通常是一个虚拟的二层网络。

  真实的TCP/IP流量是通过Node IP所在的物理网卡流出的。

* Cluster IP: Service的IP地址。

  Cluster IP仅作用于Kubernetes Service这个对象, 并由Kubernetes管理和分配IP地址(来源于Cluster IP地址池), 无法直接在集群外部使用这个地址。

在kubernetes集群内, Node IP网、Pod IP网与Cluster IP网之间的通信, 采用的是Kubernetes自己设计的一种编程方式的特殊的路由规则。