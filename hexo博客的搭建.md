---
title: hexo博客的搭建
date: 2019-11-27 17:27:28
tags: hexo
---

本文旨在讲述个人使用hexo及GithubPages搭建的过程。

主要参考[最全Hexo博客搭建+主题优化+插件配置+常用操作+错误分析](<https://www.simon96.online/2018/10/12/hexo-tutorial/>)。

### 主题

经对比, 选择[hexo-theme-matery](<https://github.com/blinkfox/hexo-theme-matery>), demo为[闪烁之狐](<https://blinkfox.github.io/>), 贼拉好看。

* 下载主题至themes文件夹下, 然后修改`_config.yml`的theme字段为主题文件夹名称。

  ```yaml
  theme: hexo-theme-matery
  ```

* 根据[readme配置文件](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)做进一步配置。

* 评论使用[gitalk](https://github.com/gitalk/gitalk), 创建[OAuth application](https://github.com/settings/applications/new)后, 在matery的_config.yml中配置即可。

