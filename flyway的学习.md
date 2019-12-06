---
title: flyway的学习
date: 2019-11-21 14:39:00
tags: database
top: true
---

#### 前言

数据库可以像代码一样, 使用flyway进行良好的版本控制。



#### 介绍

##### 空数据库-最简单的方式

* flyway找不到`schema history table`时, 将创建默认名为`flyway_schema_history `的表。

* flyway随后会扫描文件系统或application的classpath, 找到对应文件进行`migrations`。其可以是sql或java编写。

* 每次migration生效后, `flyway_schema_history `中会生成一条记录。

  ![flyway_schema_history](/images/flyway_schema_history.jpg)

* 每次创建一个版本更高的`migration`, 当下次flyway运行时, 其将会找到并运行该`migration`来升级数据库。

##### 非空数据库

指定baseline。

*  baselineOnMigrate  是否开启baseline, 默认为false。
*  baselineVersion 高于该指定版本的才会进行migrate。
*  baselineDescription 描述。

```shell
flyway -baselineOnMigrate=true -baselineVersion=1 -baselineDescription="aaa" migrate
```

##### command-line

以此种方式为例进行说明。

* 下载对应系统的flyway command-line tool。

* 进入到flyway文件夹。

  ```shell
  cd flyway-6.0.8
  ```

* 配置./conf/flyway.conf

  ```properties
  flyway.url=jdbc:mysql://127.0.0.1:3306/flyway?serverTimezone=UTC
  flyway.user=
  flyway.password=
  ```

* 在./sql目录下添加sql文件, 命名为`V1__Create_person_table.sql`, <font color="red">注意V1后的下划线为两个</font>, V是version的缩写, 1为版本号, __后的为此次操作的描述。

* 执行flyway migrate, 即可在数据库看到改变。

  ```sh
  flyway-6.0.8> flyway migrate
  ```

还可通过API、maven、Gradle来实现, 其具体操作参见官方文档。

##### Repeatable Migrations

命名规则: `R__people_view.sql`, 即使用`R`代替`V`,并且无需指定版本号, 其总是最后生效, 且可保证多次重复执行。一般用于创建 views/procedures/functions/packages/…

##### Undo Migrations

命名规则: ` U2__Add_people.sql `, U代替V, 其基于之前的migration均是成功的, 不能undo失败的migration。

当存在migration失败时, 需人工介入。

社区版不支持, 算了, 不用了。

##### Dry Runs

社区版不支持, 可生成该次会执行的sql改动, 便于检查。



#### SQL-based migrations

##### Naming

![flyway-naming](/images/flyway-naming.png)

Versioned可以省略Separator和Description(最好不要)。

##### Discovery

文件可以在filesystem或classpath, 分别使用对应的前缀, 即` filesystem: `或` classpath: `。

##### Syntax

flyway支持sql语义, 一行或多行语句。

单行注释(--)或多行注释(/**/)。

指定数据库的语义延伸。

##### Placeholder Replacement

flyway支持占位符。



#### Commands

7个命令: Migrate, Clean, Info, Validate, Undo, Baseline, Repair。

##### Migrate

flyway的中心功能, 其会扫描filesystem和classpath寻找可用的migration, 并与数据库里flyway_schema_history的版本进行比较, 执行需要执行的migration。

在启动时来执行migration, 避免数据库与代码的不一致。

##### Clean

<font color = 'red'>在开发和测试环境很有用, 但不要在生产环境使用。</font>

删除所有表, 试图, 存储过程, 触发器等。

##### Info

查看migrations的详细信息, 即flyway_schema_history的表信息。

##### Validate

校验可用的migration与已生效的migration。

#### 参考

[flyway官网](https://flywaydb.org/getstarted/)

[下载地址](https://flywaydb.org/download/)