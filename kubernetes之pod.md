---
title: kubernetes之pod
date: 2019-12-13 19:58:25
tags: kubernetes
---

### 定义详解

```yaml
# 版本号
apiVersion: v1    
kind: Pod  
# 元数据
metadata:
  # Pod的名称
  name: string
  # 所属的命名空间
  namespace: string    
  # 自定义标签列表
  labels:     
    - name: string
  # 自定义注解列表  
  annotations:       
    - name: string
# 详细定义    
spec:
  # 容器列表
  containers:   
    # 容器名称
  - name: string
    # 容器镜像名称
    image: string
    # 镜像拉取策略: Always、Never、IfNotPresent
    imagePullPolicy: 
    # 容器的启动命令列表, 如果不指定, 则使用镜像打包时使用的启动命令
    command: [string]    
    # 启动命令参数列表
    args: [string] 
    # 工作目录
    workingDir: string    
    # 挂在到容器内部的存储卷配置
    volumeMounts:   
      # 引用Pod定义的共享存储卷的名称, 需使用volumes[]部分定义的共享存储卷名称
    - name: string     
      # 存储卷在容器内Mount的绝对路径, 应少于512个字符
      mountPath: string    
      # 是否为只读模式, 默认为读写模式
      readOnly: boolean    
    # 容器需要暴露的端口号列表  
    ports:       
      # 端口的名称
    - name: string 
      # 容器需要监听的端口号
      containerPort: int 
      # 容器所在主机需要监听的端口号, 默认与containerPort相同, 设置hostPort时, 同一宿主机无法启动该容器的第二副本
      hostPort: int
      # 端口协议, 支持TCP(默认)和UDP
      protocol: string     
    # 容器运行前需要设置的环境变量列表  
    env:      
      # 环境变量的名称
    - name: string  
      # 环境变量的值
      value: string    
    # 资源限制和资源请求的设置  
    resources:      
      limits:  
        # CPU限制, 单位为core
        cpu: string    
        # 内存限制, 单位可以为MiB/GiB等
        memory: string  
      # 最低要求
      requests:        
        cpu: string    
        memory: string
    # 对Pod内容器健康检查的设置    
    livenessProbe:     
      exec:      
        command: [string]  
      httpGet:       
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     
         port: number
       # 容器启动完成后进行首次探测的时间, 单位为秒  
       initialDelaySeconds: 0  
       # 容器健康检查的探测等待响应超时时间设置, 单位为秒, 超过该时间, 将重启容器
       timeoutSeconds: 0   
       # 容器健康检查的定期探测时间设置, 单位为秒, 默认10秒探测一次
       periodSeconds: 0    
       # 失败后最少成功几次才会认为容器健康, 最小为1
       successThreshold: 0
       # 失败多少次后才会重启容器, 默认为3, 最小为1
       failureThreshold: 0
       # pod的安全设置
       securityContext:
         privileged:false
    # 重启策略。Always(Pod一旦终止运行, kubelet将会重启它)、OnFailure(非正常结束时, 才会重启该容器)
    # Never(Pod终止后, kubelet将退出码报告给Master, 不会再重启该Pod)
    restartPolicy: 
    # 表示将Node调度到包含这些Label的Node上
    nodeSelector: obeject  
    # pull镜像时, 使用的secret名称
    imagePullSecrets:    
    - name: string
    # 是否使用主机网络模式, 为true则使用宿主机网络, 不再使用Docker网桥
    hostNetwork:false     
    # 在该Pod上定义的共享存储卷列表
    volumes:       
      # 每个共享存储卷名称唯一
    - name: string 
      # 与Pod生命周期相同的一个临时目录
      emptyDir: {}
      # 挂载在所在宿主机的目录
      hostPath:      
        path: string  
      # 挂载集群预定义的secret对象到容器内部  
      secret:      
        scretname: string  
        items:     
        - key: string
          path: string
      # 关在集群预定义的configMap到容器内部    
      configMap:    
        name: string
        items:
        - key: string
          path: string
```

属于一个Pod的多个容器应用之间相互访问仅需要通过localhost就可以通信, 使得这一组容器被绑定在一个环境中。



### 静态Pod

其是由kubelet进行管理的仅存在于特定Node上的Pod, 不能通过API Server进行管理, 无法与ReplicationController、Deployment、DaemonSet进行关联, 并且kubelet无法对它们进行健康检查。

静态Pod总是由kubelet进行创建, 并且总是在kubelet所在的Node上运行。

创建静态Pod有两种方式: 配置文件或者HTTP方式。



### Pod的配置管理-ConfigMap

#### 典型用法

* 生成容器内的环境变量。
* 设置容器启动命令的启动参数(需设置为环境变量)。
* 以volume的形式挂载为容器内部的文件或目录。

configmap以`key:value`的形式保存在kubernetes系统中供应用使用, 可以通过`yaml`或`kubectl create configmap`命令行的方式来创建ConfigMap。

```shell
kubectl create -f cm-appvars.yaml
kubectl create configmap NAME --from-file=config-files-dir --from-literal=
```

#### 使用方法

* 通过环境变量获取ConfigMap中的内容。

  ```yaml
  ...
  spec:
    containers:
    - name: cm-test
      env:
      --name: APPLOGLEVEL
        valueFrom:
          configMapKeyRef:
            name: cm-appvars
            value: apploglevel
      ...
  ```

  

* 通过Volume挂载的方式将ConfigMap中的内容挂载为容器内部的文件或目录。

#### 限制条件

* ConfigMap必须在Pod之前创建。
* ConfigMap也可以定义为属于某个namespace, 只有处于相同namespace中的Pod可以引用它。
* 静态Node无法引用ConfigMap。



### 生命周期和重启策略

#### 生命周期

| 状态      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Pending   | API Server已创建该Pod, 但Pod内还有一个或多个容器没有创建, 包括正在下载镜像的过程。 |
| Running   | Pod内所有容器均已创建, 且至少有一个容器处于运行状态、正在启动状态或正在重启状态。 |
| Succeeded | Pod内所有容器均成功执行退出, 且不会再重启。                  |
| Failed    | Pod内所有容器均已退出, 但至少有一个容器退出为失败状态。      |
| Unknown   | 由于某种原因无法获取该Pod的状态, 可能由于网络通信不畅导致。  |

#### 重启策略

Pod的重启策略(RestartPolicy)应用于Pod内的所有容器, 并且仅在Pod所处的Node上由`kubelet`进行判断和重启操作。

* Always: 当容器失效时, 有kubelet自动重启该容器。
* OnFailure:  当容器终止运行且退出码不为0时, 由kubelet自动重启该容器。
* Never: 不论容器运行状态如何, kubelet都不会重启该容器。

kubelet重启失效容器的时间间隔以sync-frequency乘以2n来计算, 最长延时5分钟, 并且在成功重启后10分钟后重置该时间。

Pod的重启策略与控制方式息息相关。

* RC和DaemonSet: 必须设置为Always, 需要保证该容器持续运行。
* Job: OnFailure或Never, 确保容器执行完成后不再重启。
* kubelet管理的静态Pod: 在Pod失效时自动重启它。



### 健康检查

可以通过两类探针来检查: LivenessProbe和ReadinessProbe。

* LivenessProbe: 用于判断容器是否存活, 当不包含时, 默认返回值永远是"Success"。

  有以下三种实现方式。

  * ExecAction: 在容器内部执行一个命令, 如果该命令返回码为0, 则表明容器健康。
  * TCPSocketAction: 通过容器的IP地址和端口号执行TCP检查, 如果能建立TCP连接, 则表明容器健康。
  * HTTPGetAction: 调用HTTP Get方法, 如果响应码大于等于200且小于等于400, 则认为容器健康。

* ReadinessProbe: 用于判断容器是否启动完成, 可以接受请求。如果检测到失败, `Endpoint Controller`将从Service的Endpoint找那个删除该容器所在Pod的Endpoint。

对于每种探测方式, 都需要设置initialDelaySeconds和timeoutSeconds两个参数。



### 调度

Master上的Scheduler服务负责实现Pod的调度。

#### NodeSelector: 定向调度

通过Node的标签和Pod的`nodeSelector`属性相匹配, 来达到上述目的。

如果指定了Pod的nodeSelector, 且集群中不存在包含相应标签的Node, 则即使集群中还有其他可供使用的Node, 这个Pod也无法被成功调度。

#### NodeAffinity: 亲和性调度

替换NodeSelector的下一代调度策略。

NodeAffinity增加了In、NotIn、Exiss、DoesNotExist、Gt、Lt等操作符来选择Node, 能够使调度策略更加灵活。

如果同时设置了NodeSelector和NodeAffinity, 则系统将需要同时满足两者的设置才能进行调度。

#### DaemonSet: 特定场景调度

用于管理集群中每个Node上仅运行一份Pod的副本实例。

* 在每个Node上运行一个GlusterFS存储或Ceph存储的daemon进程。
* 在每个Node上运行一个日志采集程序, 如fluentd或logstach。
* 在每个Node上运行一个健康程序, 采集该Node的运行性能数据。

也可以在`DaemonSet`的yaml中使用NodeSelector或NodeAffinity来指定满足条件的Node范围进行调度。

#### Job: 批处理调度

* Job Template Expansion模式: 一个Job对象对应一个待处理的Work item。
* Queue with Pod Per Work Item模式: 采用一个队列存放Work Item, 一个Job作为消费者去完成这些Work Item, 在这种某事下, Job启动N个Pod, 每个Pod对应一个Work Item。
* Queue with Variable Pod Count模式: 与上面模式不同的是, Job启动的Pod数量是可变的。



### 扩容和缩容

* 通过rc的scale机制来完成这些工作。

  ```shell
  kubectl scale rc redis --replicas= 3
  ```

* 使用HPA(HorizontalPodAutoscaler)。



### 滚动升级

使用rolling-update命令。

```shell
kubectl rolling-update redis -f redis-controller-v2.yaml
kubectl rolling-update redis --image=redis-master:2.0
```

定义v2版本的ReplicationController.yaml时需注意。

* RC的name不能与旧的RC名字相同。
* 在selector中至少有一个Label应与旧的不同, 以便区分。