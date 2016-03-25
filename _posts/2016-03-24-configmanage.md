---
layout: post
title:GoLang 学习之路
tags: zookkeeper java 分布式
categories: Java
---
# 开源分布式配置中心选型
## 一、目标
实现分布式配置中心：  
（1）集中管理外部依赖的服务配置和服务内部配置  
（2）提供web管理平台进行配置和查询  
（3）支持配置版本控制  
（4）支持客户端拉取配置  
（5）支持订阅与发布，配置变更主动通知到client，实时变更配置  
（6）client支持.Net，Java 

##二、开源解决方案调研 

### 2.1 disconf
百度开源
与spring集成的很好，有web管理，client只支持java。
#### 2.1.1 介绍  
https://github.com/knightliao/disconf/wiki
#### 2.1.2 源码  
https://github.com/knightliao/disconf
### 2.2 diamond
阿里开源
阿里内部应用广泛，由http server(nameservers), diamond-server ，web组成，diamond-server连接同一个mysql，数据同步通过mysql dump文件同步（同步效率？），支持订阅发布，client只支持java。
####2.2.1 介绍
http://code.taobao.org/p/diamond/wiki/index/
####2.2.2 源码
http://code.taobao.org/svn/diamond/trunk
### 2.3 zookeeper
Zookeeper是为分布式应用程序提供高性能协调服务的工具集合，也是Google的Chubby一个开源的实现，是Hadoop的分布式协调服务。分布式应用程序可以基于它实现配置维护、命名服务、分布式同步、组服务等。它主要用来解决分布式应用中经常遇到的数据管理问题，如集群管理、统一命名服务、分布式配置管理、分布式消息队列、分布式锁、分布式协调等。
ZooKeeper 是相对成熟的框架
####2.3.1 特性-
* 
* 简单、健壮、有序、快速  
* 以目录树的的方式存储和管理  
(1) ZooKeeper目录树中每一个节点对应一个Znode。每个Znode维护着一个属性结构，它包含着**版本号(dataVersion)**，时间戳(ctime,mtime)等状态信息
(2) **Znode是客户端访问ZooKeeper的主要实体 ,当节点状态发生改变时(数据的增、删、改)将会触发watch所对应的操作，这以特性很重要 可以帮我们完成配置管理、信息同步、分布式锁等等。**
(3) 分临时节点（临时znode不能有子znode）和永久节点，每个节点都拥有着自己的访问控制列表**（可用来做权限控制）**
(4) 节点唯一性  
#### 2.3.2版本
官方资料(https://zookeeper.apache.org/doc/r3.3.2/index.html) 
官方支持Java  以及.Net客户端
#### 2.3.3 学习资料
* CSDN优秀博客 (http://blog.csdn.net/tswisdom/article/details/41522069)
* cnblogs学习笔记(http://www.cnblogs.com/sunddenly/category/620563.html)
* Windows下安装使用 (http://www.cnblogs.com/shanyou/p/3221990.html)
###2.4 etcd
CoreOS开源。
轻量级分布式key-value数据库，同时为集群环境的服务发现和注册而设计。
它提供了数据TTL失效（通过TTL更新来判断机器下线，来避免一定的网络分区问题）、数据改变监视、多值、目录监听、分布式锁原子操作等功能，来管理节点状态。

####2.4.1 特性

简单: curl可访问的用户的API（HTTP+JSON）
安全: 可选的SSL客户端证书认证
快速: 单实例每秒 1000 次写操作
可靠: 使用Raft保证一致性
注：zookeeper使用的比较复杂基于Paxos的Zab协议，etcd使用的Standford新的一致性算法Raft，概念上会简单些，0.4.6版本依赖于go-raft，0.5.0以上重新设计。
####2.4.2 目前版本

网上的介绍文章基本停留到稳定版0.4.6上。目前版本已从0.x直接跳到了2.0版本。（2015-1-28）
https://coreos.com/blog/etcd-2.0-release-first-major-stable-release/
2.0介绍（需要翻墙）：
https://www.youtube.com/watch?v=z6tjawXZ71E

使用go语言，部署简单，同时项目较年轻。

####2.4.3 介绍

从实现原理到应用场景多方位解读（2015-1-30）
http://www.infoq.com/cn/articles/etcd-interpretation-application-scenario-implement-principle

####2.4.4 源码
https://github.com/coreos/etcd

###2.6 比较etcd和zookeeper
Jason Wilder的一篇博客分别对常见的服务发现开源项目Zookeeper、Doozer、etcd进行了总结。
http://jasonwilder.com/blog/2014/02/04/service-discovery-in-the-cloud/

etcd特性略胜于zookeeper两点：  
（1）etcd在订阅发布机制上能提供的功能与zookeeper相似。但是更轻量级，使用api更简单，依赖少，可直接使用curl/http+json或etcdctl的方式。  
（2）etcd的TTL机制能避免一定的网络分区问题（如网络间断误认为注册服务下线）  

zookeeper胜于etcd两点：  
（1）成熟，稳定性高，多数坑已被踩过。  
（2）配套工具：etcd没有web监控平台，client有node-etcd 3.0，较年轻。zookeeper有简单易用的exhibitor监控，java client的curator替代zkclient，非常成熟易用，避免掉坑，还支持.Net client。

### 总结
 继续目标和成熟性  个人倾向于选取ZooKeeper来封装开发
***
# ZooKeeper详细分析

