---
title: proto buffer 学习
date: 2019-11-14 16:54:57
tags: data-serialization
---

### 简介

protobuffer是与语言无关、与平台无关、可扩展的数据格式,  可用于数据通信及存储等, 比XML更小、更快、更简洁。

### 实战

#### 定义proto file

```protobuf
syntax = "proto3";

// this is request
message SearchRequest {

  // reserved filed number which cannot be used
  reserved 4;	
	
  optional string voice_box = 1;
  required int32 host = 2;
  repeated int32 display = 3;
}

/* this is response
*/
message SerachResponse {
	...
}
```

* 第一行标识使用proto3语义， 如未在第一个非注释行指定, 则默认使用proto2。
* 每个filed有命名和类型, 类型可以是 numbers(整型或浮点型)、  booleans 、 strings 、enumerations、bytes, 甚至是proto buffer类型。proto字段类型与不同语言的对应关系: [scalar value types](https://developers.google.com/protocol-buffers/docs/proto3#scalar)。
* 每个字段类型后都有一个唯一标识号, 此标识号会在proto二进制文件中使用, 标识号在1-15内使用一个字节去编码, 16至2047使用两个字节去编码, 故1-15为频繁使用的字段保留,  也应留存以便未来使用。
*  <font style="color:red"> reserved, 保证不被后来的使用者重新使用</font>
* 不能在.proto文件中使用19000-19999作为字段标识号, 因为它们是proto保留字段数。
* 使用//或/**/添加注释。

* singular(optional 或 required): 至多1个, repeated: 不限制个数, 0或多个, 相对而言使用required弊多于利。




#### 枚举字段

```protobuf
message SearchRequest {
  ...
  enum Corpus {
   	reserved 2, 15, 9 to 11, 40 to max;
  	option allow_alias = true;
  	
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
    START = 7;
    BEGIN = 7;
  }
  Corpus corpus = 4;
}
```

* 必须有0值，此为默认值。
* 0值必须是第一个元素, 与proto2兼容(其默认第一个枚举值为默认值)。
* allow_alias = true, 即允许在enum中使用别名,  即START与BEGIN。
* 可在其他message type中使用该enum, 即MessageType.EnumType。
* <font color = "green">reserved, 保证不被后来的使用者重新使用</font>



#### 定义message type field

proto编译器找寻proto文件的路径通过命令行里的 -I/--proto_path指定, 如若不存在, 其在编译器执行的当前目录   下找寻。

* 在同一proto文件中, 可直接使用。

  ```protobuf
  message SearchResponse {
    repeated Result results = 1;
  }
  
  message Result {
    string url = 1;
  }
  ```

* 在同一目录下的其他文件中,  需导入该proto文件。

  ```protobuf
  import "myproject/other_protos.proto";
  ```

* 非同一目录下,  使用import public

  ```protobuf
  import public "new.proto"
  ```



#### 更新Message Type

 注意事项:

* 不要改动任何已存在的字段标识号。
* 添加新字段时, 应明确知晓其默认值, 以便新代码可以正常处理老proto生成的信息。
* 字段可以被删除, 但不能被再次使用, 可以将其置为<font color = "blue">reserved</font>或添加<font color = "blue"> OBSOLETE_ </font>前缀。



#### 未知字段

当老版本解析新版本中新增字段时，其新增字段属于未知字段。

版本3.5后, proto会保留未知字段, 而非丢弃。



#### Any

Any可以允许使用message tpye, 而不引入其proto文件。

```protobuf
import "google/protobuf/any.proto";

message ErrorStatus {
	String message = 1;
	repeated google.protobuf.Any details = 2;
}
```



#### oneof

每次最多只有一个字段类型被设置时, 可以使用oneof, 不能在oneof中使用repeated字段。

```protobuf
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

* 设置oneof字段时, 如同时设置多个,  只有最后一个会被保留。
* 如果解析器同时发现多个相同的oneof, 只有最后一个oneof会被保留。
* 谨慎添加或删除oneof字段。



#### Map

```protobuf
...
// map<key_type, value_type> map_field = N;
map<String, Project> projects = 3;
...
```

* key_type可以是int或string类型, enum、floating和enum不是有效的key_type。
* value_type可以是任何类型, 但不能是map。
* Map字段不能使repeated。

 ##### 向后兼容

```protobuf
// 此与map等价, 任何支持map的proto协议都必须接受如下定义。
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```



#### Packages

可以在proto文件中使用package来防止名称冲突。

```protobuf
package foo.bar;
message Open { ... }
```



#### Defining Services

在proto文件中定义rpc接口。

```protobuf
service SearchService {
	rpc Search (SearchRequest) returns (SearchResponse);
}
```



#### 转为JSON

[JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json)



#### Options

* java_package, 如果不是生成java代码, 其不起作用。

  ```protobuf
  option java_package = "com.example.foo";
  ```

* java_multiple_files, 将proto文件内定义的message type生成多个java类。

  ```protobuf
  option java_multiple_files = true;
  ```

* java_outer_classname, 指定生成的java类名。

  ```protobuf
  option java_outer_classname = "Ponycopter";
  ```

* optimize_for,  可用的值有三个，SPEED、CODE_SIZE、LITE_RUNTIME。默认为SPEED, 对于后台应用, 使用默认的即可。

  ```protobuf
  option optimize_for = CODE_SIZE;
  ```

* deprecated,  对于java语言, 对应字段标注@Deprecated注解。

  ```protobuf
  int32 old_field = 6 [deprecated=true];
  ```

* 还可以自定义options, 感兴趣可查看[custom options](https://developers.google.com/protocol-buffers/docs/proto.html#customoptions)。



#### 生成类

```protobuf
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

* proto_path, 指定proto文件所在位置, 可指定多次, -I是其简化写法。
* java_out即可生成java代码, 其DST_DIR可指定为zip或jar, 则输出文件即为对应格式。



#### 安装

[package download](https://developers.google.com/protocol-buffers/docs/downloads.html)

找到对应系统的安装包:  protoc-$VERSION-$PLATFORM.zip, 下载解压后,  配置环境变量, 即可使用protoc生成类文件。



### 参考

[protocol buffers overview](https://developers.google.com/protocol-buffers/docs/overview)

[language guide(proto3)](https://developers.google.com/protocol-buffers/docs/proto3)

[encoding](https://developers.google.com/protocol-buffers/docs/encoding)

[Protobuf语言指南-很全-官方文档中文版](https://www.cnblogs.com/dkblog/archive/2012/03/27/2419010.html)