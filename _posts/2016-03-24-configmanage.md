---
layout: post
title:配置中心
tags: zookkeeper Consul go java 分布式
categories: 架构
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
###2.5 Consul
Consul是HashiCorp公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其他分布式服务注册与发现的方案，Consul的方案更"一站式"，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value存储、多数据中心方案，不再需要依赖其他工具（比如ZooKeeper等）。使用起来也较 为简单。Consul用Golang实现，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与Docker等轻量级容器可无缝配合。 

####2.5.1 Consul 的使用场景

docker 实例的注册与配置共享
coreos 实例的注册与配置共享
vitess 集群
SaaS 应用的配置共享
与 confd 服务集成，动态生成 nginx 和 haproxy 配置文件

####2.5.2 Consul 的优势

使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft.
支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等. zookeeper 和 etcd 均不提供多数据中心功能的支持.
支持健康检查. etcd 不提供此功能.
支持 http 和 dns 协议接口. zookeeper 的集成较为复杂, etcd 只支持 http 协议.
官方提供web管理界面, etcd 无此功能.
综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究.

####2.5.3 Consul 的角色

client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群. 
server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其他数据中心通讯. 每个数据中心的 server 数量推荐为 3 个或是 5 个.  

###2.6 比较etcd、zookeeper、consul
Jason Wilder的一篇博客分别对常见的服务发现开源项目Zookeeper、Doozer、etcd进行了总结。
http://jasonwilder.com/blog/2014/02/04/service-discovery-in-the-cloud/  

服务发现：Zookeeper vs etcd vs Consul
http://dockone.io/article/667   

Consul和ZooKeeper的区别
http://dockone.io/article/300  

Consul vs. ZooKeeper, doozerd, etcd（译文） 
https://www.douban.com/note/496362733/
### 总结
 
***
# 

