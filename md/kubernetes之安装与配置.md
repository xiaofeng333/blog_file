---
title: kubernetes之安装与配置
date: 2019-12-11 14:30:08
tags: kubernetes
---

本文为观看`Kubernetes权威指南-第二版.pdf`第二章的笔记。

### 简单的安装方式

最简单的安装方式, 仍需修改各组件的启动参数, 才能完成kubernetes集群的配置。

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

#### Q&A

- apiserver端口占用？

  默认为8080。

  修改`/etc/kubernetes/apiserver`里的配置`KUBE_API_PORT`。

  同时修改`kubectl`的映射`alias kubectl='kubectl -s http://localhost:8081'`。

  

### 通过二进制文件和手动配置启动参数安装

下载地址为[kubernetes-v1.3.0](https://github.com/kubernetes/kubernetes/releases/download/v1.3.0/kubernetes.tar.gz), 解压后, server子目录中`kubernetes-server-linux-amd64.tar.gz`包含了kubernetes需要运行的全部服务程序文件。

#### Master

##### etcd

* 下载etcd, 使用版本为3.0.0

  ```shell
  curl -L https://github.com/coreos/etcd/releases/download/v3.0.0/etcd-v3.0.0-linux-amd64.tar.gz -o etcd-v3.0.0-linux-amd64.tar.gz
  ```

* 将`etcd`、`etcdctl`复制到`/usr/bin`目录下。

* 设置`systemd`服务文件`/usr/lib/systemd/system/etcd.service`。

  WorkingDirectory为etcd数据保存的目录, 需要在启动etcd服务之前进行创建。

  ```
  [Unit]
  Description=Etcd Server
  After=network.target
  
  [Service]
  Type=simple
  WorkingDirectory=/var/lib/etcd/
  EnvironmentFile=-/etc/etcd/etcd.conf
  ExecStart=/usr/bin/etcd
  
  [Install]
  WantedBy=multi-user.target
  ```

* 通过`systemctl start`命令启动etcd服务。

  ```shell
  systemctl daemon-reload
  systemctl enable etcd.service
  systemctl start etcd.service
  ```

* 通过执行`etcdctl cluster-health`, 可以验证etcd是否正确启动。

##### kube-apiserver

* 拷贝`kube-apiserver`复制到`/usr/bin`目录。

* 设置`systemd`服务文件`/usr/lib/systemd/system/kube-apiserver.service`。

  ```
  [Unit]
  Description=Kubernetes API Server
  Documentation=https://github.com/kubernetes/kubernetes
  After=etcd.service
  Wants=etcd.service
  
  [Service]
  EnvironmentFile=/etc/kubernetes/apiserver
  ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
  Restart=on-failure
  Type=notify
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  ```

* 在配置文件`/etc/kubernetes/apiserver`中配置`kube-apiserver`的启动参数。

  ```
  KUBE_API_ARGS="--etcd_servers=http://127.0.0.1:2379 
  			   --insecure-bind-address=0.0.0.0
                 --insecure-port=8081 
                 --service-cluster-ip-range=169.169.0.0/16
                 --service-node-port-range=1-65535
  --admission_control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota
                 --logtostderr=false 
                 --log-dir=/var/log/kubernetes 
                 --v=2" 
  ```

  * --etcd_servers: etcd服务的url。
  * --service-cluster-ip-range: kubernetes集群中Service虚拟IP地址段范围, 以CIDR格式表示, 该IP范围不能与物理机的真实IP段有重合。
  * --service-node-port-range: kubernetes集群中Service可映射的物理机端口号范围, 默认为30000~32767
  * --admission_control: kubernetes集群的准入控制设置, 各控制模块以插件的形式依次生效。
  * --logtostderr: 设置为false表示将日志写入文件, 不写入stderr。
  * --log-dir: 日志目录。
  * --v: 日志级别。

##### kube-controller-manager

- 拷贝`kube-controller-manager`复制到`/usr/bin`目录。

- 设置`systemd`服务文件`/usr/lib/systemd/system/kube-controller-manager.service`。

  ```
  [Unit]
  Description=Kubernetes Controller Manager
  Documentation=https://github.com/kubernetes/kubernetes
  After=kube-apiserver.service
  Requires=kube-apiserver.service
  
  [Service]
  EnvironmentFile=/etc/kubernetes/controller-manager
  ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
  Restart=on-failure
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  ```

- 在配置文件`/etc/kubernetes/controller-manager`中配置`kube-controller-manager`的启动参数。

  ```
  KUBE_CONTROLLER_MANAGER_ARGS="--master=http://127.0.0.1:8081 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
  ```

##### kube-scheduler

- 拷贝`kube-scheduler`复制到`/usr/bin`目录。

- 设置`systemd`服务文件`/usr/lib/systemd/system/kube-scheduler.service`。

  ```
  [Unit]
  Description=Kubernetes Scheduler
  Documentation=https://github.com/kubernetes/kubernetes
  After=kube-apiserver.service
  Requires=kube-apiserver.service
  
  [Service]
  EnvironmentFile=/etc/kubernetes/scheduler
  ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
  Restart=on-failure
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  ```

- 在配置文件`/etc/kubernetes/scheduler`中配置`kube-scheduler`的启动参数。

  ```
  KUBE_SCHEDULER_ARGS="--master=http://127.0.0.1:8081 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
  ```

##### start

- 通过`systemctl start`命令启动以上三个服务。

  ```
  systemctl daemon-reload
  systemctl enable <service_name>
  systemctl start <service_name>
  ```

- 通过`systemctl status <service_name>`来验证服务的启动状态, `running`表示启动成功。

- 将`kubectl`复制到`/usr/bin`目录。

- 到此, Master上所需的服务就全部启动完成了。

#### Node

在Node节点上需要预先安装好Docker Daemon并且正常启动。

##### kubelet

kubelet服务依赖于Docker服务。

- 拷贝`kubelet`复制到`/usr/bin`目录。

- 设置`systemd`服务文件`/usr/lib/systemd/system/kubelet.service`。

  ```
  [Unit]
  Description=Kubernetes Kubelet Server
  Documentation=https://github.com/kubernetes/kubernetes
  After=docker.service
  Requires=docker.service
  
  [Service]
  WorkingDirectory=/var/lib/kubelet
  EnvironmentFile=/etc/kubernetes/kubelet
  ExecStart=/usr/bin/kubelet $KUBELET_ARGS
  Restart=on-failure
  
  [Install]
  WantedBy=multi-user.target
  ```

- 在配置文件`/var/lib/kubelet`中配置`kubelet`的启动参数。

  ```
  KUBELET_ARGS="--api-servers=http://127.0.0.1:8081 --hostname-override=127.0.0.1 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
  ```

##### kube-proxy

kube-proxy服务依赖于network服务。

- 拷贝`kube-proxy`复制到`/usr/bin`目录。

- 设置`systemd`服务文件`/usr/lib/systemd/system/kube-proxy.service`。

  ```
  [Unit]
  Description=Kubernetes Kube-Proxy Server
  Documentation=https://github.com/kubernetes/kubernetes
  After=network.target
  Requires=network.service
  
  [Service]
  EnvironmentFile=/etc/kubernetes/proxy
  ExecStart=/usr/bin/kube-proxy $KUBE_PROXY_ARGS
  Restart=on-failure
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  ```

- 在配置文件`/etc/kubernetes/proxy `中配置`kube-proxy`的启动参数。

  ```
  KUBE_PROXY_ARGS="--master=http://127.0.0.1:8081 --logtostderr=false --log-dir=/var/log/kubernetes --v=2"
  ```

##### LAST

参考<a href="#start" target="_self">start</a>启动kubelet和kube-proxy服务。

因端口使用的不是默认的8080, 执行`alias kubectl='kubectl -s http://localhost:8081'`。

因为kubelet默认采用想Master自动注册本Node的机制, 可直接查看Node的状态, 状态为Ready表示Node已经成功注册并且状态为可用。

```shell
kubectl get nodes
```

#### Q&A

1. 出现问题, 非预期表现。

   使用`journalctl -xe`查看详细报错信息, 针对性解决。

2. 



### TODO

* 仍需关注的点包括安全设置, 其支持`基于CA签名的数字证书认证方式`和`基于HTTP BASE或TOKEN的简单认证方式`。

* 在多个Node组成的Kubernetes集群内, 跨主机的容器间网络互通式Kubernetes集群能够正常工作的前提条件。Kubernetes本身并不会对跨主机网络进行设置, 这需要额外的工具来实现, 开源工具包括flannel、Open vSwitch、Weave、Calico等都能够实现跨主机的容器间网络互通。

  也可以通过在每个Node上添加其他Node上的docker0的静态路由规则, 将不同物理机的docker0网桥互联互通。



