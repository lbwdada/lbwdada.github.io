---
title: 两台Windows server 2008之间的MySQL(8.0)主从复制
categories:
- 工作记录
tags:
- mysql
- windows server
---

记录下两台Windows server 之间的MySQL如何进行主从复制。

环境：MySQL8 ， Windows server 2008 r2 ， VMware

<!-- more -->

## 主机配置

------

 服务器介绍：

两台Windows server 分为主机和从机，主机ip为192.168.159.134 ，从机ip为192.168.159.134；

从机使用VMware从主机克隆而来（有坑）；

------

### 1.配置my.ini

``

```
[mysqld]
log-bin=master-bin #开启二进制日志 binlog
server-id=1 #配置唯一的server-id
```

### 2.配置完成后必须重启MySQL服务才能生效

windows中可通过

``

```
net stop mysql 关闭MySQL服务
net start mysql 开启MySQL服务
```

或通过任务管理器关闭开启。

### 3.打开MySQL并查看主机信息

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515101757.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515101903.png)

信息中的 File 和 Position为关键信息，后面配置从机时需要用到。

### 4.在主机创建用于从机连接的用户

``

```
--创建用户copy用于从机进行连接
CREATE USER 'copy'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
--赋予用户copy权限
GRANT REPLICATION SLAVE ON *.* TO 'copy'@'%';
--刷新权限
FLUSH PRIVILEGES;
```

用户权限总结见：[https://www.cnblogs.com/felix-h/p/11072743.html](https://www.cnblogs.com/felix-h/p/11072743.html)

主机配置完成

------

## 从机配置

### 1.配置my.ini

``

```
[mysqld]
log-bin=slave-log-bin
relay_log=relay-log
relay_log_index=relay-log.index
#server-id不能与主机一样
server-id=2   
innodb_file_per_table=ON
```

注意：配置完成之后需要重启MySQL服务才能生效

### 2.配置并启动从机

通过主机的copy用户链接主机

``

```sql
change master to master_host='192.168.159.134',master_port=3306,master_user='copy',master_password='123456',master_log_file='主机信息中的File',master_log_pos=主机信息中的Position;　　# 这里的 master 日志文件和位置必须与主服务器当前状态一致！
```

启动从机复制进程(关闭使用stop slave)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515144006.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515184248.png)

#### 3.测试

查看一下数据库

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515184504.png)



![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515184548.png)

下面从主机操作下数据库看看效果：

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515185811.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515185859.png)

![image-20210515190027622](C:\Users\85115\AppData\Roaming\Typora\typora-user-images\image-20210515190027622.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515190103.png)

下面对数据库内的数据进行修改试试看：

为了方便我在物理机中用Navicat连接了两个服务器。Windows server 2008为主机，Windows server 2008_copy为从机。

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515190221.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515190535.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515190728.png)

![image-20210515190901621](C:\Users\85115\AppData\Roaming\Typora\typora-user-images\image-20210515190901621.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515190944.png)

测试结束！

------

## 问题总结

### 1.从机IO线程未启动

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515144834.png)

#### 排查问题

##### 1.首先是查看Master_Log_File有没有和主机对应

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515145053.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515145150.png)

发现对应，排除日志文件配置错误。

##### 2.在从机信息中发现Last_IO_Error中提示主机和从机的server_id重复了。（从前面的配置文件中可以看到我主机server-id配置的是1，从机的server-id配置的是2,并不相同。为什么两个server_id都是1呢？配置没有生效吗？暂时没有搞明白。）(2021-5-18 注：原因知道了，是因为Windows我忘开显示文件扩展名了，所以我一直在修改的是一个txt！！！kao， 大意了)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515145359.png)

查看两台机器的server_id：

``

```sql
show variables like 'server_id';
```



![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515145934.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515150243.png)

发现相同，通过

``

```sql
set global server_id=2;
```

修改从机server_id，但是**注意**该修改只是暂时修改，当MySQL服务重启后便失效，故保险办法是修改配置文件。



修改完server_id之后重启服务发现![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515183106.png)

向下查找原因发现：

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515183243.png)

这是因为从机是从主机克隆过来的，所以MySQL和主机一摸一样其UUID也不可避免，这时只需要删除掉从机的UUID文件并重启从机的MySQL服务即可重新生成UUID。这个文件在MySQL文件夹下的data文件夹中名字为auto.cnf。

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515183550.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515183719.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515183852.png)

![image-20210515184006940](C:\Users\85115\AppData\Roaming\Typora\typora-user-images\image-20210515184006940.png)

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/20210515184248.png)

