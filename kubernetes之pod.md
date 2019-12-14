---
title: kubernetes之pod
date: 2019-12-13 19:58:25
tags: kubernetes
---

主要包括Pod和容器的使用、Pod的控制和调度管理、应用配置管理等内容。

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
    # 镜像拉去策略: Always、Never、IfNotPresent
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

### Pod的基本用法