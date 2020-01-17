---
title: kubernetes之kubectl
date: 2019-12-13 11:15:58
tags: kubernetes
---

### 命令行语法

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

* command: 子命令, 用于操作kubernetes集群资源对象的命令, 如create、delete、describe、get、apply等。

* TYPE: 资源对象的类型, 区分大小写, 能以单数形式、复数形式或简写形式表示。

* NAME: 资源对象的名称, 区分大小写, 如果不指定名称, 则将返回属于TYPE的全部对象的列表。

* flags: kubectl子命令的可选参数。如`kubectl -s http://localhost:8081`， 指定apiserver的URL地址而不使用默认值。

* 在一个命令行中也可以同时对多个资源对象进行操作。

  * 获取多个pod的信息。

    ```shell
    kubectl get pods pod1 pod2
    ```

  * 获取多种对象的信息。

    ```shell
    kubectl get pod/pod1 rc/rc1
    ```

  * 同时应用多个yaml文件, 以-f file参数表示。

    ```shell
    kubectl get pod -f pod1.yaml -f pod2.yaml
    kubectl create -f pod1.yaml -f rc1.yaml
    ```

* 可使用`kubectl --help`查看kubectl命令行公共启动参数及支持的子命令, 每个子命令还有特定的flags参数, 可通过`kubectl [command] --help`命令进行查看。

### 操作示例

* 根据yaml文件一次性创建service和rc。

  ```shell
  kubectl create -f my-service.yaml -f my-rc.yaml
  ```

* 查看rc和service列表。

  ```shell
  kubectl get rc,service
  ```

* 描述资源对象。

  ```shell
  kubectl describle nodes <node-name>
  ```

* 删除资源对象。

  ```shell
  kubectl delete -f pod.yaml
  # 删除所有包含label的pod和service
  kubectl delete pods, services -l name=<label-name>
  # 删除所有pod
  kubectl delete pods --all
  ```

* 执行容器的命令。

  ```shell
  kubectl exec <pod-name> -c <container-name> date
  # 登录容器
  kubectl exec -it <pod-name> -c <container-name> /bin/bash
  ```

* 查看容器的日志。

  ```shell
  kubectl logs <pod-name>
  # tailf-f
  kubectl logs -f <pod-name> -c <container-name>
  ```

