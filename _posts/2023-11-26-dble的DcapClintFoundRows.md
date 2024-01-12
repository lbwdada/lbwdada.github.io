---
title: dble的DcapClintFoundRows
categories:
- 工作记录
tags:
- 中间件
- dble
---

> 写这篇的原因是遇到一个奇怪的问题：在有些地方做修改的时候如果没有修改任何内容就点击保存的话前端页面会弹出操作失败的对话框。

<!-- more -->

## 1.查找原因

  首先我在自己本地搭建了一套环境(但是漏了个dble)用来测试，结果是没有任何异常。查看远程的后台日志发现了在执行update之后mybatis打印的返回值是0，而且直接把这个返回值放到了toAjax这个函数中，这就导致前端会弹出操作失败的对话框。

![image-20231126095131990](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-11-26/dble%E7%9A%84capClientFoundRows/0.png)

  为什么会返回0呢，在MySQL中执行update语句发现它会有这样的结果显示，其中有affected和matched，若这个时候update语句没有更新任何记录则affected的值就为0，matched的值表示where匹配到的记录数量。

![image-20231126095846928](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-11-26/dble%E7%9A%84capClientFoundRows/1.png)

  显然不知道为什么远程环境中的mybaits返回了affected导致这个奇怪的系统(正常人不会这么设计)出现了这个问题。而在spring.datasource.url中指定useAffectedRows=true可以使其返回**受影响行数**默认为false返回**匹配的行数**，遂查看nacos配置，发现根本没有这个参数￣へ￣。

  后来发现远程环境上使用了分库分表中间件dble，经过多方查证发现dble会持有数据库的连接放在连接池中，并发现了client_found_row这个标志和useAffectedRows是有对应关系的，因为多走了一层中间件，所以要先确保中间件中的MySQL连接是支持匹配的行数作为结果。

  至此原因已经确定。

## 2.DBLE 是否开启client_found_rows权能标志，默认关闭

> https://www.modb.pro/db/202800

### 介绍&背景

在客户端与服务器在初次连接的时候，服务端发送初始化握手包时会带上自己所支持的[权能标志](https://dev.mysql.com/doc/internals/en/capability-flags.html)；客户端接收后会对服务器发送的权能标志进行筛选，保留自身所支持权能标志且返回给服务器，从而保证服务器与客户端通讯的兼容性。因此权能标志是在初次连接确定，不能动态修改。
在DBLE中后端连接都是使用连接池预先建立好的，导致与前端请求不同导致行为不一样；因此在dble管理端中新增了对client_found_rows权能标志更改和后端连接池的刷新。

#### client_found_rows的作用

若初次连接handshake协议中启用client_found_rows权能标志，表示在DML等操作时结果集里返回发现行(found rows)，而不是影响行(affect rows)。

#### client_found_rows值设定：

MYSQL客户端：默认关闭client_found_row；（则返回结果集里为affect rows)
JDBC：默认开启client_found_rows；（则返回结果集里为found rows）

#### JDBC中useAffectedRows与client_found_row的关系

useAffectedRows=true 即关闭client_found_rows
useAffectedRows=false(默认) 即开启client_found_rows

###  具体

#### 在bootstrap.cnf里增加参数

\```

-DcapClientFoundRows=false ```

#### 管理端口增加命令

`show @@cap_client_found_rows; -- 查询client_found_row权能标志开启状态 0-关闭 1-开启 disable @@cap_client_found_rows; -- 关闭client_found_row权能标志 enable @@cap_client_found_rows; -- 开启client_found_row权能标志` 注意： 如果在不停dble服务的情况下，更改该权能标志，为了保证与后端连接的mysql的该权能标志一致，(强调)需要[刷新连接池](https://www.modb.pro/db/2.1_manager_cmd/2.1.21_fresh_conn.md)；否则insert的结果集不正确

## 3.附：

### dble管理端登录：需要用管理用户登录9066端口

  `mysql -u管理用户 -p管理用户 -h 地址 -P 9066`

在dble的user.xml中可以查看和配置管理用户

### dble刷新连接池命令

`fresh conn [forced] where dbGroup ='groupName' [and dbInstance='instanceName'];`

其中groupName为dble的db.xml文件中的配置

不包含forced时，则空闲的后端连接直接丢弃，⽽正在使⽤的后端连接等归还后丢弃

包含forced时，则⽆论空闲还是正在使⽤的后端连接都直接丢弃；另外正在使⽤的后端连接对应的前端连接也会全部断开

不指定dbInstance时，则dbGroup下所有的dbInstance都将刷新

指定dbInstance时，多个dbInstance以逗号隔开，则刷新dbGroup下指定的dbInstance

## 参考

[dble文档](https://actiontech.github.io/dble-docs-cn/)

[dble运维命令知多少](https://opensource.actionsky.com/20211012-dble/)

[Mybatis Update标签实战](http://681314.com/A/sKN85jbGyH#_UserMapperupdate_21)

