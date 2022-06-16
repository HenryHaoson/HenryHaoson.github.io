---
layout: post
title: 【Sentry】 sentry 服务搭建
date: 2022-6-14
categories: blog
tags: [Sentry]
description: 着重看用于表示模型的3种模型元素模式。细品讲的有道理的话。作为以后建模的参考原则。
---

# 安装

参照 https://github.com/getsentry/self-hosted 说明，将项目 clone 下来，然后执行 install.sh

根据文档要求，需要 docker 环境、还有一些硬件基础


- Docker 19.03.6+
- Compose 1.28.0+
- 4 CPU Cores
- 8 GB RAM
- 20 GB Free Disk Space


遇到的问题：
1. docker-compose 版本过低，需要安装新版本：https://stackoverflow.com/questions/49839028/how-to-upgrade-docker-compose-to-latest-version

2. install 报错

``` shell
Creating ../relay/config.yml...
relay Pulling
error getting credentials - err: exit status 1, out: `GDBus.Error:org.freedesktop.DBus.Error.ServiceUnknown: The name org.freedesktop.secrets was not provided by any .service files`
An error occurred, caught SIGERR on line 27
Cleaning up...
```

解法：

``` shell
apt install gnupg2 pass
```

第一次安装的时候会卡在拉取镜像上很久，要耐心等待

``` shell
▶ Fetching and updating Docker images ...
```

安装完成之后的页面
![image](https://raw.githubusercontent.com/HenryHaoson/HenryHaoson.github.io/source/source/images/sentry_welcome.png)



