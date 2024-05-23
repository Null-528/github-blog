---
title: 详解gRPC协议(二)
categories: 网络
tags: 协议
abbrlink: 5a25e7f5
date: 2024-03-01 15:37:23
---
我们在之前的文章中介绍了gRPC协议使用的protobuf，从而定义gRPC service和具体用到的对象模型。这篇文章中，我们来具体通过gRPC来调用一个服务，看看gRPC是如何工作的。

gRPC 使用 HTTP/2 作为传输协议。 虽然与 HTTP 1.1 也能兼容，但 HTTP/2 具有许多高级功能：

用于数据传输的二进制组帧协议 - 与 HTTP 1.1 不同，HTTP 1.1 是基于文本的。
对通过同一连接发送多个并行请求的多路复用支持 - HTTP 1.1 将处理限制为一次处理一个请求/响应消息。
双向全双工通信，用于同时发送客户端请求和服务器响应。
内置流式处理，支持对大型数据集进行异步流式处理的请求和响应。
减少网络使用率的标头压缩。
gRPC 是轻量型且高性能的。 其处理速度可以比 JSON 序列化快 8 倍，消息小 60% 到 80%。 在 Microsoft Windows Communication Foundation (WCF) 中，gRPC 的性能超过经过高度优化的 NetTCP 绑定的速度和效率。 与偏向于 Microsoft 堆栈的 NetTCP 不同，gRPC 是跨平台的。


建议在以下场景中使用 gRPC：

需要立即响应才能继续处理的同步后端微服务到微服务通信。
需要支持混合编程平台的 Polyglot 环境。
性能至关重要的低延迟和高吞吐量通信。
点到点实时通信 - gRPC 无需轮询即可实时推送消息，并且能对双向流式处理提供出色的支持。
网络受约束环境 - 二进制 gRPC 消息始终小于等效的基于文本的 JSON 消息。
在撰写本文时，gRPC 主要用于后端服务。 新式浏览器无法提供支持前端 gRPC 客户端所需的 HTTP/2 控制级别。 也就是说，支持使用 .NET 的 gRPC-Web，能够从使用 JavaScript 或 Blazor WebAssembly 技术构建的基于浏览器的应用进行 gRPC 通信。 gRPC-Web 使 ASP.NET Core gRPC 应用能支持浏览器应用中的 gRPC 功能：

强类型、代码生成的客户端
压缩 Protobuf 消息
服务器流式处理


