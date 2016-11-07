---
title: PostgreSQL集群的一种解决方案
date: 2016-11-07 09:40:49
tags: 集群
---
...............
<!--more-->
## 集群要考虑的问题                                                                                                                                                                                                                                   
当作PostgreSQL集群时应考虑数据间的同步问题，并发读写问题，故障自动切换问题，数据库备份问题，以下为这些问题的一种解决方案
       
## 数据同步采用流复制
       
关于配置可参考[官方文档高可用性与负载均衡与复制部分](http://www.postgres.cn/docs/9.4/high-availability.html)
       
## 用pgpool-II实现负载均衡
       
pgpool-II 是一个位于 PostgreSQL 服务器和 PostgreSQL 数据库客户端之间的中间件，[具体介绍及使用看官方文档](http://www.pgpool.net/docs/latest/pgpool-zh_cn.html)，它提供以下功能：        
- 连接池
- 复制
- 负载均衡
- 限制超过限度的连接                                                                                  
       
## 用repmgr作流复制主从间复制流向的切换
虽然pg本身有流复制功能可以保证有多个备库，可要做到自动切换还是不容易的，缺点是自动切换后原来的主库就要重新手工调整，这个太麻烦了，而repmgr可以用来解决此问题，[使用可以参考这篇文章](https://my.oschina.net/lianshunke/blog/223896)
       
## pgpool-II单点故障问题
至于pgpool-II的高可用问题，自从pgpool 3.2以后提供了watchdog功能以消除pgpool的单点故障,不过需要对外暴露一个虚拟IP,考虑在阿里云服务器上IP是分配好的不好再虚拟一个出来,这块可以在客户端连接数据库的配置处同时配置两个pgpool2的连接，以实现让客户
端自己处理pgpool2的高可用问题
       
## 测试
使用 pgbench 进行数据库压力测试,验证集群的效果，[使用参考](http://blog.yaodataking.com/2016/02/postgresql-pgbench.html)
       
##  数据库备份
数据库备份可以自己写脚本也可用现场的工具，这里可以使用barnman,[使用手册](http://docs.pgbarman.org/release/2.0/)，[参考文章](https://my.oschina.net/firxiao/blog/505951)
       
## 小结       
以上repmgr，barnman工具都来自[这个公司](https://2ndquadrant.com/en/resources/),还有其它工具也可尝试   
       
整体配置可参考[这篇文章](http://jensd.be/591/linux/setup-a-redundant-postgresql-database-with-repmgr-and-pgpool)

