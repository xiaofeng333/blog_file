---
title: kubernetes之service
date: 2019-12-16 20:37:09
tags: kubernetes
---

通过创建Service, 可以为一组具有相同功能的容器应用提供一个统一的入口地址, 并将请求负载分发到后端的各个容器应用上。

### 定义详解

```yaml
apiVersion: v1      // Required
kind: Service       // Required
metadata:           // Required
  name: string      // Required
  # 不指定使用"default"的命名空间
  namespace: string // Required
  # 自定义标签列表
  labels:
    - name: string
  # 自定义注解属性列表  
  annotations:
    - name: string
  # 详细描述  
spec:             // Required
  # 选择具有指定Label标签的Pod作为管理范围
  selectors: []   // Required
  # 指定Service的访问方式, 默认为ClusterIP(虚拟的服务IP地址)。还可使用NodePort(宿主机IP+端口即可访问), LoadBalancer(使用外接负载均衡器完成到服务的负载分发)
  type: string    // Required
  # 当type为LoadBanlancer时, 需要指定。NodePort时如果不指定, 系统自定进行分配。
  clusterIP: string
  # 可选值为ClientIP, 表示将同一客户端的请求转发到同一后端Pod, 默认为空。
  sessionAffinity: string
  # service需要暴露的端口列表
  ports:
  - name: string
    # 端口协议, TCP/UDP
    protocol: string
    # 服务监听的端口号
    port: int
    # 转发到后端Pod的端口号
    targetPort: int
    # 当type为NodePort时, 映射到物理机的端口号。
    nodePort: int
  # 当type为LoadBalancer时, 用于设置外部负载均衡器的地址。  
  status:
    loadBalancer:
      ingress:
        ip: string
        hostname: string																
```



### 基本用法

使用expose快速创建Service的方法。

```shell
kubectl expose rc webapp
```

还可以通过上述配置文件来定义service。

```shell
kubectl create -f webapp-svc.yaml
```

#### 负载分发策略

* RoundRobin: 轮询模式, 即轮询将请求转发到后端的各个Pod上。

* SessionAffinity: 基于客户端的IP地址进行会话保持的模式, 可将sessionAffinity设置为`ClientIP`来启用。

* 自己控制负载均衡策略: 将Service的ClusterIP设置为`None`(无入口IP地址), 仅通过Label Selector将后端的Pod列表返回给调用的客户端。

  ```yaml
  ...
  clusterIP: None
  ...
  ```

#### 访问外部服务

定义一个不带标签选择器的Service, 即无法选择后端的Pod, 此时系统不会自动创建EndPoint, 因此手动创建一个与Service同名的EndPoint, 用于指向实际的后端访问地址。

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - adresses:
      - IP: 1.2.3.4
      ports:
        - port: 80
```



### DNS服务搭建

kubernetes提供的虚拟DNS服务名为skydns, 由四个组件组成。

* etcd: DNS存储。
* kube2sky: 将kubernetes Master的Service注册到etcd。
* skyDNS: 提供DNS域名解析服务。
* healthz: 提供对skydns服务的健康检查功能。



### Ingress: HTTP 7层路由机制

实现HTTP层的业务路由机制, 即不同的URL地址对应到不同的后端服务或者虚拟服务器。

在kubernetes集群中, Ingress的实现需要通过Ingress的定义与Ingress Controller的定义结合起来, 才能形成完整的HTTP负载分发功能。

#### 创建Ingress Controller

在定义Ingress之前, 需要先部署Ingress Controller, 以实现为所有后端Service提供一个统一的入口。

Ingress Controller需要实现基于不同HTTP URL向后转发的负载分发规则, 也可设置能够提供该类型HTTP路由的LoadBalancer为Ingress Controller。

Ingress Controller以Pod的形式运行, 监控apiserver的/ingress接口, 如果service发生变化, 则Ingress Controller应自动更新其转发规则。

#### 定义Ingress

设置到后端Service的转发规则。