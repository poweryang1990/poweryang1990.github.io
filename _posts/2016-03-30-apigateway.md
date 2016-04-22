---
layout: post
title: 微服务API Gateway
tags: 微服务 服务化 apigateway
categories: 架构
---

#微服务 API网关

## API网关 
一个API 网关，他是所有客户端的入口。 API网关有两种方式来处理请求。有些请求被简单地代理/路由到合适的服务上，其他的请求被转给到一组服务 
该API网关可以为每个客户端提供不同的API，而不是提供一个适合所有情况下的API。例如，Netflix的API网关运行客户端特定适配器代码，提供给每个客户端它们需要的API。
API网关还可以实现安全性，例如： 验证客户端被授权执行请求
![Use an Api GateWay]( {{"/apigateway/apigateway001.jpg" | prepend: site.imgrepo }} "Use an Api GateWay")

  使用API​​网关具有以下优点： 

　*  使服务和客户端解耦。

　* 使客户端和服务部署环境解耦。

　*  提供了最佳的API给每个客户端。

　*  减少的请求/往返次数。例如，API网关可以一次性检索多个服务的数据。更少的请求也意味着更少的开销，提高了用户体验。一个API网关对于移动应用至关重要。

　* 简化了客户端的调用，因为API网关可以组合服务，并提供组合后的façade接口。

　　API网关模式也有一些缺点：

　*  增加的复杂性，API网关是必须开发、部署和管理的另一个应用。

　* 增加的响应时间，因为通过API网关多了一层网络跳转。然而，对于大多数应用的额外往返的成本是微不足道的。 

### How implement a API Gateway with Consul?
https://github.com/hashicorp/consul/issues/1012
http://www.mammatustech.com/Microservice-Service-Discovery-with-Consul
http://www.mammatustech.com/consul-service-discovery-and-health-for-microservices-architecture-tutorial

### consul  data Persisten
 
用 -data-dir 指定 数据持久化目录

### consul Multi environment config  
 We have multiple data centers, and we also have environments like DEV, TEST, QA, PROD  
 https://groups.google.com/forum/#!topic/consul-tool/R61ISkfAzwo  
 https://github.com/hashicorp/consul-template/issues/374   
 https://groups.google.com/forum/#!topic/consul-tool/0P_27-j3SYQ
 
 我们也可以用 Tags来区分多环境
 
### 用Consul实现服务注册和发现  
http://blog.leanote.com/post/proyang/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E4%B8%AD%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0
 http://tonybai.com/2015/07/06/implement-distributed-services-registery-and-discovery-by-consul/
 我们建立Consul Cluster是为了实现服务的注册和发现。     
 Consul支持两种服务注册的方式，一种是通过Consul的服务注册HTTP API，
 由服务自身在启动后调用API注册自己，另外一种则是通过在配置文件中定义服务的方式进行注册。Consul文档中建议使用后面一种方式来做服务 配置和服务注册。 
 
 Consul提供了两种发现服务的方式，一种是通过HTTP API查看存在哪些服务；另外一种是通过consul agent内置的DNS服务来做。
 两者的差别在于后者可以根据服务check的实时状态动态调整available服务节点列表。我们这里也着重说明适用 DNS方式进行服务发现的具体步骤
 consul为服务编排的内置域名为 “NAME.service.consul"
 
 ### How to use Consul-template?
 
 https://github.com/hashicorp/consul-template/tree/master
 
 
 ## consul load balancer 
 DNS NGINX