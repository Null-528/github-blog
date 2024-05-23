---
title: Gradle使用手册
categories: 工具
tags: gradle
date: 2024-04-28 15:40:03
---

在整理老项目的过程中，部分老项目使用的是gradle，由于groovy的拓展脚本，学习成本比maven要高一些，因此记录一些常用的命令和配置方便进行回顾。

首先，一定要注意gradle使用的版本，我自己就是因为要兼容Mac M1，操作了gradle 2.0 -> 5.6.4的版本升级，导致许多插件、groovy语法都出现兼容问题，花费了很多时间去处理编译和运行的问题。

首先，最好的系统学习方法，就是在官网选择使用的版本，有详细实例和语法可以参考。例如[5.6.4版本使用说明](https://docs.gradle.org/5.6.4/userguide/tutorial_using_tasks.html)。

下面进入正题。

# 一、Gradle安装使用
目前gradle的使用主要有两种方式：
1. 在官网下载gradle安装包，解压后配置环境变量


# 二、Gradle常用命令
## 2.1 Gradle Task
首先，Gradle最核心的概念为task，不同task解决不同类型的功能，例如编译、打包，甚至大多数plugin也是通过拓展task类型来实现功能的。

gradle会根据用户的构建脚本，以及相关task之间配置的dependency，将这些task构建为有向无环图（DAG）。也就是说，build本质上是配置一组任务，并根据它们之间的依赖关系将它们连接起来，从而创建 DAG。任务图创建完成后，Gradle 会确定哪些任务需要按顺序运行，然后开始执行。

### 2.1.1 Gradle Task生命周期
Gradle 任务的生命周期包括以下阶段：
- clean: 清理任务，删除build目录
- check: 检查任务，用户也可以使用check.dependsOn(task)来做一些前置检查
- assemble: 编译与打包，
- build(depends on: check, assemble): 构建任务，包括运行运行测试、生成文档等。
- buildConfiguration: assemble那些附加到已命名配置的工件。例如，buildArchives 将执行创建附加到归档配置的任何工件所需的任何任务。
- uploadConfiguration: 与 buildConfiguration 的功能相同，但也会上传所有附加到给定配置的工件。
- cleanTask: 清理Task产出的output

## 2.2 Gradle 初始化顺序
以IDEA为例。在Gradle项目初始化时，会先读取setting.gradle脚本，这个最重要的是定义了整个项目的模块，idea需要感觉这个模块来识别各个子模块。

# 三、示例
## 3.1 task依赖后的执行顺序
```
task taskX {
    dependsOn 'taskY'
    doLast {
        println 'taskX1'
    }
    doLast {
        println 'taskX2'
    }
    configure {
        println 'taskX3'
    }
}
task taskY {
    doFirst {
        println 'taskY1'
    }
    doLast {
        println 'taskY2'
    }
    configure {
        println 'taskY3'
    }
}
```
执行：`gradle -q taskX`

执行结果：
```
taskX3
taskY3
taskY1
taskY2
taskX1
taskX2
```

也就是说，task会按照依赖关系，先执行被依赖的task，再执行自己。在单个task而言，执行顺序为configure-> doFirst -> doLast。
其中configure是个例外，即便依赖其他task，也要先初始化好自己的配置，才会加载被依赖对象的configure。很好理解，配置先行，然后才是执行阶段。

## 3.2 动态Task

