---
title: git-submodule
date: 2019-12-04 14:33:47
tags: git
top: true
---

### SYNOPSIS

```shell
.gitmodules, $GIT_DIR/config
```

```shell
git submodule
git <command> --recurse-submodules
```

```shell
# 同时克隆a项目的子模块
git clone --recursive a.git
```



### DESCRIPTION

submodule是将一个repository内嵌在另一个repository中。

submodule有自己的提交历史, 其通常位于父项目的`$GIT_DIR/modules/`。



### Define Submodule Properties

使用.gitmodules文件进行定义, 其是一个文本文件, 位于`$GIT_WORK_DIR/.gitmodules`。

文件由每一个submodule组成, 每个submodule包含如下属性。

#### 必选参数

* submodule.<name>.path: 指定submodule的相对路径, 不能以`/`结尾, 每个submodule路径是唯一的。
* submodule.<name>.url: 指定submodule仓库地址, 可以是git地址或父项目的相对路径。

#### 可选参数

* submodule.<name>.update: 指定submodule的更新策略, 包括checkout、rebase、merge、none, 只在`git submodule init`时生效。

* submodule.<name>.branch: 指定git仓库的远程分支, 默认为master。特殊符号为`.`, 其标识分支名与父项目相同。

* submodule.<name>.fetchRecurseSubmodules: 获取submodule的策略, 可被父项目中的配置覆盖或git fecth及git pull的参数`--[no-]recurse-submodules`覆盖。

  ```
  tips:
  git pull是git fetch及git merge FETCH_HEAD的缩写, 即执行git pull等同于执行git fetch和git merge FETCH_HEAD。
  ```

* submodule.<name>.ignore: 定义`git status`或`git diff`时是否展示submodule的修改, 支持的值为: all、dirty、untracked、 none。

* submodule.<name>.shallow: 为true时表示为浅克隆, 即不会克隆历史记录。

#### examples

```git
[submodule "libfoo"]
	path = include/foo
	url = git://foo.com/git/lib.git
```



### command line

```shell
# add a submodule
git submodule add <url> <path>
```

```shell
# See the list of submodules in a superproject
git submodule status
```

```shell
# occasionally update the submodule to a new version:
git -C <path> checkout <new version>
git add <path>
git commit -m "update submodule to new version"
```

```shell
# cloning missing submodules and updating the working tree of the submodules
git submodule update --init --recursive
git submodule update --remote --merge
```



### 总结

直接使用`git clone`不会同时下载submodule项目, 可使用`git clone --recursive`一并clone父项目与submodule项目; 或在父项目中使用`git submodule update`即可。



### 参考

[GIT子模块](https://yihui.org/cn/2017/03/git-submodule/)

