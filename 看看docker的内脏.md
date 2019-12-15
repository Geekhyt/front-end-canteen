---
title: "看看Docker的内脏"
date: 2019-12-15T18:05:42+08:00
lastmod: 2019-12-15T18:05:42+08:00
draft: false
description: "看看docker的内脏"
show_in_homepage: true
show_description: false
license: ''

tags: ['Docker']
categories: ['前端']

featured_image: ''
featured_image_preview: ''

comment: true
toc: true
autoCollapseToc: true
math: true
---

有人把Docker比喻成运维人员的一把“瑞士军刀”，因为他们可以通过这把军刀实现终极梦想“一次创建，到处运行”。

<!--more-->

## Docker

也有人说Docker是给PaaS世界带来的“降维打击”，Docker镜像完美解决了应用打包这个根本性问题，直接将应用所运行的整个操作系统进行打包，让你的本地环境和云端环境高度一致，再也无需在不同的运行环境切换之间进行痛苦的试错。

究其本质，`Docker把纯后端的技术概念，进行了非常友好的封装，展现在广大开发者面前。`

回顾Docker的发展历史，Google、微软、IBM、RebHat等巨头推动着这只小鲸鱼在容器云的浪潮中掀起了阵阵波涛。云计算世界的运维方式从SSH的刀耕火种正式迈入了电气自动化时代。`从历史的本质中可以看出，运维不再仅仅是服务器的维护，更重要的是数据、服务与人的沟通。`看来新技术想要得到认可和发展，就要将开发者群体摆在最高位置。

在技术大爆发的时代，我们每天接收新知识的时候更多的都是进行快餐式的阅读，这样就会造成对待新技术仅仅停留在会用即可、了解即可的程度，这样的恶性循环对我们自身的技术发展是大大不利的。`对于核心的技术我们一定要做到能够真正理解它的原理和底层实现，这样才能对技术发展趋势建立起自己的判断和预言。`


作为前端工程师，项目里的Dockerfile一般都是脚手架自动生成配置好的，我们无需清楚它的内部实现。但当你有机会参与到公司的脚手架项目搭建，有机会和DevOps同事进行深度的交流(我工位离DevOps同事很近，经常找他们扯皮)，那么你一定会感受到这只小鲸鱼的魔力并想进入它的体内一探究竟。

来，看看内脏！

![看看内脏](/images/docker/docker0.png)

## Docker总架构图

Docker是一个C/S架构，我们可以通过客户端与服务器建立通信。

### Docker1.11版本之前的架构图


![Docker1.11版本之前的架构图](/images/docker/docker1.png)


从图中我们可以看到，它的主要模块有DockerClient、Docker Daemon、Docker Registry、Driver、Graph、libcontainer以及Docker Container。接下来我们来逐一理解他们。

### Docker Daemon
Docker Daemon是Docker中一个常驻在后台的守护进程，也就是“运行Docker”。

- Server接收Docker Client发送的请求，通过Router进行调度，分发到对应的Handler来处理请求。
- 管理所有的Docker容器

#### Engine
任务由Engine运行引擎完成，每项任务都由Job形式存在，也就是说Job是引擎内部最基本的工作执行单元。

### Docker Registry
Docker Registry是一个存储容器镜像(Docker Image)的仓库，可以以公有和私有的形式存在。Docker Hub就是全球范围内最大的公有Registry。在Docker的运行过程中，有三种情况能够和Docker Registry进行通信，分别是进行搜索、上传以及下载镜像。

### Graph
对比架构图，Graph有着保管着容器镜像的能力。当然它也支持多种不同的存储方式，除了图中的aufs，还有devicemapper、Btrfs等。

### Driver
Driver顾名思义，是一个驱动模块。可以实现对容器运行环境的定制，包括网络环境、存储方式以及容器执行方式。这一层的抽象意义在于，将与Docker容器相关的管理从Docker Daemon的所有逻辑中进行分门别类。

如架构图中所示，可以分为以下三类：

- graphdriver
- networkdriver
- execdriver

#### graphdriver
graphdriver主要管理容器镜像方面的内容。包括存储从Docker Registry上下载的镜像以及本地构建的镜像。

#### networkdriver
networkdriver负责管理容器网络环境。在Docker Daemon启动时为Docker环境创建网桥，在Docker容器创建前分配相应的网络接口资源，为Docker容器分配IP、端口并与宿主机进行NAT端口映射以及设置防火墙策略等。

#### execdriver
execdriver是Docker容器的执行驱动，负责创建容器运行时的命名空间、资源使用的统计与限制、负责管理内部进程的真正运行等。

### libcontainer
libcontainer抽象出了linux的内核特性，为Docker Daemon提供了完整的接口。
它是由Go语言设计实现的库，可以不依靠任何依赖，直接访问内核中与容器相关的系统调用。其本身也可以被上层多种不同的编程语言访问。

### Docker Container
如架构图中所示，最底下离我们最近的一层，Docker Container就是Docker架构中服务交付的最终体现形式。

## Docker架构的演进

`containerd核心化，引擎集群化。`

新增containerd和runc，原有的引擎功能下沉到containerd。

containerd负责管理容器的生命周期，向着独立于Docker作为通用容器运行时工具方向演进。

除此之外，将集群编排工具swarm整合进引擎，引擎内部也不断进行解耦出新模块buildkit等。

### Docker架构演进的原因

一部分原因是由于swarm与kubernetes的竞争失败，Docker公司为了避免自己在开源社区中被边缘化，于2017年将containerd捐献给CNCF基金会，将更多Docker引擎的功能下沉到containerd。另外就是容器世界的标准化在不断推进。

`“没有集装箱，就没有全球化。”`

这是《经济学家》杂志对集装箱的评价，看似普通的发明，推动了全球化的进程。这也是Docker的小鲸鱼身上背着的集装箱的魔力所在。

![集装箱](/images/docker/docker2.png)

## Docker的本质


容器其实就是一种特殊的进程。

Docker的核心功能，就是约束，“喜欢就会放肆，但爱是克制”。

看来小鲸鱼是走心真爱了，真爱三连问：

那么约束的又是什么呢？通过什么来进行约束呢？约束实现了什么呢？

- 约束的是进程的动态表现。

- 通过Cgroups技术(Linux的一种机制)来进行约束。

- 约束实现了“边界”。

除了约束以外，Docker还可以修改进行的动态表现，通过Namespace技术来实现。

`Namespace`也是Linux里面的一种机制，对被隔离应用的进程空间使用了`魅惑之术`，使得这些进程只能看到重新计算过的编号，但实际上，他们在宿主机的操作系统里，还是原来的编号。

![魅惑之术](/images/docker/docker3.png)

在Linux中使用也特别简单，就是Linux创建新进程时候的一个`可选参数(CLONE_NEWPID)`。除了PID Namespace，Linux操作系统还提供了Network、Mount、User、IPC、UTS等Namespace，来针对不同的进程进行魅惑。

`Linux Cgroups(Linux Control Group)`，它的功能是限制一个进程组能够使用的资源，包括CPU、内存、磁盘、网络带宽等。与Namespce配合起来使用就可以实现双剑合璧，隔离进程并且约束进程占用的资源量。

### 不足之处

- 相比于虚拟机而言，Linux的Namespace机制隔离的并不彻底，多个容器仍旧是共享宿主机内核。

- 时间并不能被隔离。

- Cgroups的/proc文件系统问题，/proc文件系统并不受Cgroups的限制(/proc渣男实锤)，在生产环境中可以使用`LXCFS`修正。

LXCFS其实也是一种魅惑之术，通过文件挂载的方式，把Cgroups中关于系统的相关信息读取出来，再通过Docker的Volume挂载给容器内部的proc系统。魅惑Docker内部应用，读取proc中信息的时候以为就是读取宿主机的真实proc。

参考：

- 《深入剖析Kubernetes》
- 《Docker源码分析》
- 《容器云运维实战》
- 《Docker技术入门与实战》





