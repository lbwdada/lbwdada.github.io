---
title: Dbeaver连接TDengine数据库
categories:
- others
tags:
- 工作记录
- 工具
---

使用Dbeaver对TDengine数据库进行查看和数据操作

<!-- more -->

## 1.为Dbeaver添加新的驱动

数据库->驱动管理器

![image-20230919155833007](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/0.png)

新建驱动

![image-20230919160203495](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/1.png)

- 类名：`com.taosdata.jdbc.rs.RestfulDriver`
- URL模：`jdbc:TAOS-RS://your_ip:6041/your_db`
- 默认端口：`6041`
- 默认数据库：`your_db`
- 默认用户：`root`

![image-20230919161250686](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/2.png)

驱动->库

> 首先要获取对应的数据库库文件，就是一个jar包，从[这里](https://repo1.maven.org/maven2/com/taosdata/jdbc/taos-jdbcdriver/)获取，下载的是时候要选择和自己的TDengine版本对应的以免后面出问题。(下载后缀为-dist的jar包)

![image-20230919163651895](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/3.png)



##  2.新建连接

新添加好的驱动在others中

![image-20230919165017201](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/4.png)

输入密码，测试链接

![image-20230919165135213](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/5.png)

##  3.测试应用

可以像操作MySQL一样对TDengine进行操作啦

![image-20230919165312454](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-09-19/Dbeaver%E8%BF%9E%E6%8E%A5TDengine%E6%95%B0%E6%8D%AE%E5%BA%93/6.png)
