---
title: LDAP学习与介绍
tags: JAVA
date: 2019-04-04 23:23:48
---

#### 概念

LDAP是轻量目录访问协议(Lightweight Directory Access Protocol)的缩写，其结构用树来表示，而不是用表格。它有优异的读性能，但写性能差，并且没有事务处理、回滚等复杂功能，不适于存储修改频繁的数据。

LDAP是一种开放Internet标准，LDAP协议是跨平台的Interent协议。

类似以下的信息适合储存在目录中：

- 企业员工信息，如姓名、电话、邮箱等；
- 公用证书和安全密钥；
- 公司的物理设备信息，如服务器，它的IP地址、存放位置、厂商、购买时间等；

####  LDAP组织数据的方式

![LDAP组织数据的方式](/images/ldap_intro_dctree.png)

#### 基本概念

##### Entry

条目，也叫记录项，是LDAP中最基本的颗粒，就像字典中的词条，或者是数据库中的记录。通常对LDAP的添加、删除、更改、检索都是以条目为基本对象的。

`dn`：每一个条目都有一个唯一的标识名（distinguished Name ，DN），如上图中一个 dn："cn=baby,ou=marketing,ou=people,dc=mydomain,dc=org" 。通过DN的层次型语法结构，可以方便地表示出条目在LDAP树中的位置，通常用于检索。

`rdn`：一般指dn逗号最左边的部分，如cn=baby。它与RootDN不同，RootDN通常与RootPW同时出现，特指管理LDAP中信息的最高权限用户。

`Base DN`：LDAP目录树的最顶部就是根，也就是所谓的“Base DN"，如"dc=mydomain,dc=org"。

##### Attribute

每个条目都可以有很多属性（Attribute），比如常见的人都有姓名、地址、电话等属性。每个属性都有名称及对应的值，属性值可以有单个、多个，比如你有多个邮箱。

`属性不是随便定义的，需要符合一定的规则`，而这个规则可以通过schema制定。比如，如果一个entry没有包含在 inetorgperson 这个 schema 中的`objectClass: inetOrgPerson`，那么就不能为它指定employeeNumber属性，因为employeeNumber是在inetOrgPerson中定义的。

LDAP为人员组织机构中常见的对象都设计了属性(比如commonName，surname)。下面有一些常用的别名：

| 属性                   | 别名 | 语法             | 描述             | 值(举例)             |
| ---------------------- | ---- | ---------------- | ---------------- | -------------------- |
| commonName             | cn   | Directory String | 姓名             | sean                 |
| surname                | sn   | Directory String | 姓               | Chow                 |
| organizationalUnitName | ou   | Directory String | 单位（部门）名称 | IT_SECTION           |
| organization           | o    | Directory String | 组织（公司）名称 | example              |
| telephoneNumber        |      | Telephone Number | 电话号码         | 110                  |
| objectClass            |      |                  | 内置属性         | organizationalPerson |

##### ObjectClass

对象类是属性的集合，LDAP预想了很多人员组织机构中常见的对象，并将其封装成对象类。比如人员（person）含有姓（sn）、名（cn）、电话(telephoneNumber)、密码(userPassword)等属性，单位职工(organizationalPerson)是人员(person)的继承类，除了上述属性之外还含有职务（title）、邮政编码（postalCode）、通信地址(postalAddress)等属性。

通过对象类可以方便的定义条目类型。每个条目可以直接继承多个对象类，这样就继承了各种属性。如果2个对象类中有相同的属性，则条目继承后只会保留1个属性。对象类同时也规定了哪些属性是基本信息，必须含有(Must 活Required，必要属性)：哪些属性是扩展信息，可以含有（May或Optional，可选属性）。

对象类有三种类型：结构类型（Structural）、抽象类型(Abstract)和辅助类型（Auxiliary）。结构类型是最基本的类型，它规定了对象实体的基本属性，每个条目属于且仅属于一个结构型对象类。抽象类型可以是结构类型或其他抽象类型父类，它将对象属性中共性的部分组织在一起，称为其他类的模板，条目不能直接集成抽象型对象类。辅助类型规定了对象实体的扩展属性。每个条目至少有一个结构性对象类。

对象类本身是可以相互继承的，所以对象类的根类是top抽象型对象类。以常用的人员类型为例，他们的继承关系：

!(ObjectClass)[/images/ldap_objectclass.jpg]

##### Schema

对象类（ObjectClass）、属性类型（AttributeType）、语法（Syntax）分别约定了条目、属性、值，他们之间的关系如下图所示。所以这些构成了模式(Schema)——对象类的集合。条目数据在导入时通常需要接受模式检查，它确保了目录中所有的条目数据结构都是一致的。

![schema](/images/ldap_schema_attr_entry.jpg)

##### LDIF

LDIF（LDAP Data Interchange Format，数据交换格式）是LDAP数据库信息的一种文本格式，用于数据的导入导出，每行都是“属性: 值”对。

#### 工具安装

在ubuntu上安装未能成功。

可参考[CentOS 7 环境下 OpenLDAP 的安装与配置](https://myanbin.github.io/post/openldap-in-centos-7.html)在centos上安装成功。

其中不同点:

1.chdomain.ldif中{2}mdb中的mdb需通过

```shell
ls /etc/openldap/slapd.d/cn=config/ 
```

此命令查看如下格式文件。

我本机为olcDatabase={2}hdb.ldif，故我在此书写为hdb。

2.访问http://ip/phpldapadmin后出现This base cannot be created with PLA.

可参考[关于OpenLDAPAdmin管理页面提示“This base cannot be created with PLA“问题](https://blog.51cto.com/tingdongwang/1733005)修改。

#### 参考

[LDAP服务器的概念和原理简单介绍](https://segmentfault.com/a/1190000002607140)

[OpenLDAP管理工具之LDAP Admin](https://cloud.tencent.com/developer/article/1380076)

[CentOS 7 环境下 OpenLDAP 的安装与配置](https://myanbin.github.io/post/openldap-in-centos-7.html)

[关于OpenLDAPAdmin管理页面提示“This base cannot be created with PLA“问题](https://blog.51cto.com/tingdongwang/1733005)