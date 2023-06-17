---
title: Ubuntu创建systemd服务
categories:
- linux
- clash
tags:
- 瞎折腾
---

Unit 文件按照 Systemd 约定，应该被放置指定的三个系统目录之一中。这三个目录是有优先级的，如下所示，越靠上的优先级越高。因此，在三个目录中有同名文件的时候，只有优先级最高的目录里的那个文件会被使用。1 /etc/systemd/system：系统或用户自定义的配置文件2 /run/systemd/system：软件运行时生成的配置文件3 /usr/lib/systemd/system：系统或**第三方软件安装时添加的配置文件**。Systemd 默认从目录 /etc/systemd/system/ 读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录 /usr/lib/systemd/system/，真正的配置文件存放在那个目录

<!--  more  -->

[阮一峰](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

##### 文件存放到哪里的问题？

> 我从Google搜索得到的结果是在Ubuntu系统下，与Debian软件包无关的systemd相关文件(.service)文件等最好是直接就存放在etc/systemd/system 目录下即可。

.service 文件配置的服务常用systemd管理。然而，systemd有系统和用户区分；系统/user/lib/systemd/system/）、用户（/etc/lib/systemd/user/）。一般系统管理员手工创建的单元文件建议存放在/etc/systemd/system/目录下面。

#### 以Clash的开机自启为例子演示

```shell
1. sudo touch /etc/systemd/system/clash.service
2. sudo vim /etc/systemd/system/clash.service
```

```
[Unit]
Description=clash daemon


[Service]
ExecStart=/data/zhangjianghui/clash/clash -d /data/zhangjianghui/clash


[Install]
WantedBy=default.target
```

```shell
sudo systemctl daemon-reload #重新加载服务配置文件
sudo systemctl enable clash #开机启动
sudo systemctl start clash #启动服务
sudo systemctl stop clash #停止服务
sudo systemctl status clash #查看服务状态
```
