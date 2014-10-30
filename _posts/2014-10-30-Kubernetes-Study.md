---
layout: post
title: Kubernetes初探（一）
published: true
---

Kubernetes是Google开源的容器集群管理系统。它构建于docker技术之上，为容器化的应用提供资源调度、部署运行、服务发现、扩容缩容等整一套功能，本质上可看作是基于容器技术的mini-PaaS平台。本文旨在梳理Kubernetes的架构、概念及基本工作流，并且通过运行一个简单的示例应用来介绍如何使用Kubernetes。

###总体概览
如下图所示是我初步阅读文档和源代码之后整理的总体概览，基本上可以从如下三个维度来认识Kubernetes。
![Kubernetes概览](https://github.com/tragicjun/tragicjun.github.com/blob/master/images/Kubernetes.png)
###操作对象
Kubernetes以RESTFul形式开放接口，用户可操作的REST对象有三个：
- **pod**：是Kubernetes最基本的部署调度单元，可以包含container，逻辑上表示某种应用的一个实例。比如一个web站点应用由前端、后端及数据库构建而成，这三个组件将运行在各自的容器中，那么我们可以创建包含三个container的pod。
- **service**：是pod的路由代理抽象，用于解决pod之间的服务发现问题。因为pod的运行状态可动态变化(比如切换机器了、缩容过程中被终止了等)，所以访问端不能以写死IP的方式去访问该pod提供的服务。service的引入旨在保证pod的动态变化对访问端透明，访问端只需要知道service的地址，由service来提供代理。
- **replicationController**：是pod的复制抽象，用于解决pod的扩容缩容问题。通常，分布式应用为了性能或高可用性的考虑，需要复制多份资源，并且根据负载情况动态伸缩。通过replicationController，我们可以指定一个应用需要几份复制，Kubernetes将为每份复制创建一个pod，并且保证实际运行pod数量总是与该复制数量相等(例如，当前某个pod宕机时，自动创建新的pod来替换)。

可以看到，service和replicationController只是建立在pod之上的抽象，最终是要作用于pod的，那么它们如何跟pod联系起来呢？这就要引入label的概念：label其实很好理解，就是为pod加上可用于搜索或关联的一组key/value标签，而service和replicationController正是通过label来与pod关联的。如下图所示，有三个pod都有label为"app=backend"，创建service和replicationController时可以指定同样的label:"app=backend"，再通过label selector机制，就将它们与这三个pod关联起来了。例如，当有其他frontend pod访问该service时，自动会转发到其中的一个backend pod。
![Kubernetes概览](https://github.com/tragicjun/tragicjun.github.com/blob/master/images/restObjects.png)
