### centos安装问题

#### dracut_initqueue timeout

原因是UltroISO在Window下写入U盘的安装文件路径，没有被linux安装程序识别。

在U盘启动界面，根据提示，按tab/e键，修改系统引导命令行，修改为：

vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdb4 quiet

这sdb4的值是默认值，可能会不同

[U盘启动安装CentOS 7和遇到错误](https://www.cnblogs.com/zhangtingzu/p/8972063.html)

#### 网络问题

安装后未能连接网络，因未开启网络连接。

修改网卡配置文件，把网络连接打开。

将etc/sysconfig/network-scripts/目录下的配置文件，将ONBOOT改为yes，然后service network restart。

[开启网络连接](https://blog.csdn.net/sfeng95/article/details/62239539)

#### 关闭报警声音

编辑/etc/inputrc，找到 #set bell-style none项，去掉注释。