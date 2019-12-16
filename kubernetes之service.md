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

