---
layout: post
title: Kubernetes初探
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
![Kubernetes REST对象](https://github.com/tragicjun/tragicjun.github.com/blob/master/images/restObjects.png)

###功能组件
如下图所示是官方文档里的集群架构图，一个典型的master/slave模型。
![Kubernetes架构图](https://raw.githubusercontent.com/GoogleCloudPlatform/kubernetes/master/docs/architecture.png)

master运行三个组件：
- **apiserver**：作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。
- **scheduler**：负责集群的资源调度，为新建的pod分配机器。这部分工作分出来变成一个组件，意味着可以很方便地替换成其他的调度器。
- **controller-manager**：负责执行各种控制器，目前有两类：
    - endpoint-controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
    - replication-controller：定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。
    
slave(称作minion)运行两个组件：
- **kubelet**：负责管控docker容器，如启动/停止、监控运行状态等。它会定期从etcd获取分配到本机的pod，并根据pod信息启动或停止相应的容器。同时，它也会接收apiserver的HTTP请求，汇报pod的运行状态。
- **proxy**：负责为pod提供代理。它会定期从etcd获取所有的service，并根据service信息创建代理。当某个客户pod要访问其他pod时，访问请求会经过本机proxy做转发。

###工作流
上文已经提到了Kubernetes中最基本的三个操作对象：pod, replicationController及service。 下面分别从它们的对象创建出发，通过时序图 来描述Kubernetes各个组件之间的交互及其工作流。
![Kubernetes工作流](https://github.com/tragicjun/tragicjun.github.com/blob/master/images/kubernetesWorkflow.png)
