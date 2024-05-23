---
title: 详解RPC
categories: 网络
abbrlink: b9eb0f9
date: 2024-02-27 14:22:43
---

# 什么是RPC
在最早的时候，进程只能获取自己栈帧内的数据，例如有2个进程，两者是无法数据通信的。为了解决这个问题，出现了本地进程间通信IPC（Inter-Process Communication）技术，通过如共享内存，管道等手段，来交换数据。但后来业务逐渐复杂，磁盘、CPU等资源逐渐达到瓶颈，人们开始考虑将多台计算机组成一个集群来提高服务的吞吐能力，这就是分布式。

RPC（Remote Procedure Call），即远程程序调用，可以理解为RPC是将进程间通信的范围从单机扩大到了一个共享网络中，这样不同服务间调用方法像同一服务间调用本地方法一样，而不需要调用者了解底层网络技术的协议。如cassandra集群节点接通信，以及SpringCloud 微服务中各个服务之间的通信。

RPC 现在所指的概念也有些模糊，一般来说我们将其理解为一种request-response的网络传输思想，但在wiki等一些网站上，rpc也可以狭义地理解为一种协议，并且需要通过stub等动态代理来实现消息传输。

# RPC协议模型
这里介绍在wiki等网站**狭义**定义的RPC协议模型：
![rpc调用](../images/详解RPC/rpc调用流程.jpeg)
整个过程可以分为以下几个阶段：
1. 客户机调用本地的Stub，将参数传递进去。
2. Stub将参数marshall（打包）到消息中。这个过程包括将参数标准化的过程。
3. Stub将消息传递给传输层，再由传输层发送到远程服务器机器。
4. 服务器的Stub收到消息后，存根对参数进行unmarshall（解包），然后根据参数调用本地服务。
5. 服务器服务完成调用后，将结果参数返回给服务器Stub，服务器Stub再将结果参数标准化并marshall（打包）到消息中。
6. 服务器Stub将消息传递给传输层，传输层再发送给客户端Stub。
7. 客户端Stub解析参数，将结果返回给调用者。

在上面的流程中，看上去两个进程是1对1同步阻塞的，但实际上rpc并没有限制异步调用，可以选择异步来提高并发。

> 这里用Stub存根来指代远程服务的本地代理程序或者软件，它是客户端和应用程序的中间层，用于处理一些调用细节如负责封装请求、解析响应、处理网络通信等。例如在java中，Stab就是通过动态代理技术实现的。
>
> Stub一般都是RPC框架自动生成的，开发者不需要关心。

# RPC系统
为了让不同的客户端访问服务器，人们创建了许多标准化的 RPC 框架。这些框架大多使用接口描述语言IDL(Interface description language)来让各种平台调用 RPC。IDL 文件随后可用于生成客户端与服务器之间的接口代码。目前有以下这些常见的 RPC 框架：

1. gRPC：由Google开发的高性能、跨语言的RPC框架，基于HTTP/2协议，支持多种语言，如C++, Java, Python, Go等。gRPC使用Protocol Buffers作为默认的序列化协议，支持双向流、流式处理等特性。

2. Apache Thrift：由Facebook开发的跨语言的RPC框架，支持多种语言，如C++, Java, Python, PHP等。Thrift使用自定义的IDL（Interface Definition Language）来定义服务接口，支持多种传输协议和序列化协议。如cassandra执行cql时使用的就是Thrift。

3. Apache Dubbo：由阿里巴巴开发的高性能、轻量级的RPC框架，支持多种语言，如Java, Go, Python等。Dubbo提供了丰富的功能，如负载均衡、服务注册与发现、服务治理等。但官网的性能测试数据可能并不准确，实际使用中性能表现不如gRPC，可以参考[性能测试应该怎么做？](https://coolshell.cn/articles/17381.html)

# 其他常见概念
## Restful & RPC
两者不是可以放在一起比较的概念。
Restful，即 Representational State Transfer(表现层状态转移)，是一种设计风格，它面向资源，对接口做出了一系列要求。
```
获取所有用户： GET /users
获取指定用户： GET /users/{id}
创建用户：    POST /users
更新用户：    PUT /users/{id}
删除用户：    DELETE /users/{id}
```
从宏观来讲，RPC是一种网络通信思想，而Restful是一种API规范。RPC本身并不限制实现框架的传输协议，只要是基于传输层之上的就可以。但Restful一定都是作用于HTTP/HTTPS协议之上的。

## MQTT & RPC
从定义来讲，MQTT的发布订阅模式并不完全符合RPC『远程程序调用』的定义，即MQTT的接收方不会返回调用运行计算结果。笔者认为这应该是两种网络通信方式。
另外，从定义来讲，RPC要求调用方和被调用方必须同时存在，而MQTT是发布订阅模式，即消息的发送者不需要知道消息接收者的存在。PC是一对一，MQTT是多对一。

