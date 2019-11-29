---
title: gRPC的学习
date: 2019-11-20 20:33:45
tags: rpc
---

#### 概览

gRPC为Google的rpc实现, 使用proto buffer作为接口定义语言(Interface Definition Language, 即IDL)。

客户端可以像调用本地方法一样调用服务端方法。

在客户端保存有接口存根来进行方法调用，该存根有和服务器同样的方法; 服务端实现接口定义的方法, 来处理客户端的请求。

 ![Concept Diagram](/images/grpc-model.svg)



#### 相关概念

#####  Service definition

使用proto buffer定义接口(可以使用其它可选择的协议代替proto buffer)。

```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

有四种接口方法

* 像普通的函数,  一种请求类型, 一种返回类型。

  ```protobuf
  rpc SayHello(HelloRequest) returns (HelloResponse);
  ```

* 服务器返回流式数据, 在一次独立的调用中, gRPC保证其信息的顺序。

  ```protobuf
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
  ```

* 客户端发送流式数据, 在一次独立的调用中, gRPC保证其信息的顺序。

  ```protobuf
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
  ```

* 双向流式数据, 请求流和返回流独立处理, 服务端可以接受完所有数据后再返回, 也可以边接受边回传数据。

  在彼此的数据流中, 顺序得到保证。

  ```protobuf
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse)
  ```

##### Synchronous vs asynchronous

同步调用会阻塞，其也是RPC的方法调用希望的做法, 但是在很多场景下, 需要异步调用。

grpc同时支持同步编程和异步编程。

##### RPC life cycle

###### RPC

* 客户端调用方法时, 服务端将会收到通知, 并得到这次调用客户端的 [Metadata](#Metadata), 方法名, 及如果指定了的[Deadlines](#deadline)。
* 服务端也会立即返回自己的Metadata(必须在response前返回), 或者在收到请求信息后。
* 服务端进行处理, 返回对应的数据, 包括状态码和状态信息等。
* 当此次请求成功时, 客户端得到返回, 一次调用结束。

###### <span id="deadline">Deadlines/Timeouts</span>

客户端可以指定超时时间,  当超时时，会返回<font color = "red">DEADLINE_EXCEEDED</font>。

###### RPC termination

客服端和服务端都可以决定一次调用什么时候结束。

当请求超过客户端设定的超时时间,  客户端会认定请求失败, 即使服务端随后成功处理并发送返回。

服务端也可以在客服端发送完请求数据前开始返回来完成这次请求。

###### Cancelling RPCs

其不会<font color = "red">undo</font>, 在撤销前的改变不会回滚。

###### <span id="Metadata">Metadata</span>

键值对形式,  客户端通过metadata提供调用相关信息给服务端, 反之亦然。

###### Channels

提供到指定地址的gRPC server的连接。

clients可以通过指定channel参数来修改gRPC的默认行为, 例如打开或关闭消息压缩。

channel有状态, 包括 `connected`  和 `idle`。



#### Authentication

gRPC支持多种认证机制。

* SSL/TLS
* Token-based authentication with Google

##### Authentication API

###### Credential tpyes

* Channel credentials
* Call Credentials

java的认证与授权官方推荐OpenSSl,  [security](https://github.com/grpc/grpc-java/blob/master/SECURITY.md#transport-security-tls)。



#### Error Handling

##### Standard error model

发生错误时, gRPC返回错误的status codes及可选的错误信息字符串(提供更详细的信息)。

##### Richer error model

standard error model与grpc采用的数据交互形式无关, 但局限性较大, 不能详细描述错误信息。

[错误模型](https://cloud.google.com/apis/design/errors#error_model)



#### BenchMarking



#### 参考

[官方文档](https://www.grpc.io/docs/)