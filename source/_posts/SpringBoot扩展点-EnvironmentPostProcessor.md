---
title: 'SpringBoot扩展点:EnvironmentPostProcessor'
categories: 框架
tags: SpringBoot
abbrlink: cd861ef5
date: 2024-03-18 19:41:26
---

在一些时候，例如我们在服务中需要进行集成测试时，需要将多个模块依赖的底层数据库先拉起，才能进行完整数据通路的测试。例如业务依赖的mysql、 
一些外部存储容器如redis、minio等。

这个时候我们就可以通过SpringBoot提供的EnvironmentPostProcessor扩展点，在环境加载后，服务启动前的阶段，对环境进行扩展。

> 官网文档：https://docs.spring.io/spring-boot/docs/2.5.2/reference/htmlsingle/#howto.application.customize-the-environment-or-application-context

该类的作用是在SpringBoot项目启动之前自定义环境变量，可以在项目启动之前从非标准springboot配置文件中读取相关的配置并填充到springboot上下文中。
例如：

```java

```