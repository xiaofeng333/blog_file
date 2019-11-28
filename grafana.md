### grafana

#### install

在mac环境下安装。

* 安装

```shell
brew update
brew install grafana
```

* 升级

```shell
brew update
brew reinstall grafana
```

* 启动

```shell
brew services start grafana
```

* 默认访问地址: 127.0.0.1:3000
* 账号密码为admin/admin
* 安装路径: /usr/local/etc/grafana

#### Configuration

配置较多, 实际使用时查看[官方文档](https://grafana.com/docs/installation/configuration)即可, 以下只是随手记录, 并非最重要的参数。

* 可以通过.ini文件或指定环境变量进行配置。

  配置改变后, grafana需要重启生效。

  在ini文件中使用`;`来添加注释。

* 默认配置: $WORKING_DIR/conf/defaults.ini;

  自定义配置: $WORKING_DIR/conf/custom.ini;

  自定义配置可通过--config来重载。

##### environment variables

语义如下, SectionName为.ini文件中`[]`的内容。环境变量名称均为大写, `.`替换为`.`。

```shell
GF_<SectionName>_<keyName>
```

例子如下:

```ini
# default section
instance_name = ${HOSTNAME}

[security]
admin_user = admin

[auth.google]
client_secret = 0ldS3cretKey
```

```shell
export GF_DEFAULT_INSTANCE_NAME=my-instance
export GF_SECURITY_ADMIN_USER=true
export GF_AUTH_GOOGLE_CLIENT_SECRET=newS3cretKey
```

* temp_data_lifetime: 临时数据保存多久, 默认为`24h`, 支持的单位为`h(hour)`, `m(minutes)`, 0标识永久保存, 可设置`10h30m`。
* logs: 日志文件路径。
* 还可以配置使用的协议、插件路径、端口和域名等。

##### databse

默认使用内嵌的sqlite3。

* url: 数据库url。
* type: 数据库类型: mysql、postgres、sqlite3, 例如`mysql:user:secret@host:port/database`。
* path: 当使用sqllite3时, 用来指定文件地址。
* host: 当使用mysql或postgres, 使用ip(hostname):port或unix sockets。
* password: 当密码包含#或; 时, 使用三个引号包裹, 例如`"""#password;"""`。
* log_queries: 值为true时, 记录sql语句及执行时间。 
* max_idle_conn: 最大的空闲连接数。
* max_open_conn: 最大的并发连接数。

##### security

当账户名和密码在数据库user表存在时, 修改ini文件并不会起作用, 可清空表重启gragana, 默认为内嵌的sqlite3, 如下所示。

```sqlite
# cd /usr/local/var/lib/grafana/
sqlite3 grafana.db
select * from user;
delete from user;
```



* admin_user: 默认为admin。
* admin_password: 默认为admin。
* grafana也支持第三方授权, 如Google OAuth、Github OAuth、LDAP Authentication等。

##### analytics

* reporting_enabled: grafana发送匿名使用统计至`stats.grafana.org`。
* google_analytocs_ua_id: google的统计id。
* check_for_updates: 检查更新。

##### metrics

* enabled: 默认为true, 通过http接口`/metrics`。





