---
layout: post
title: Kubernetes初探（一）
published: true
---

Kubernetes是Google开源的容器集群管理系统。它构建于docker技术之上，为容器化的应用提供资源调度、部署运行、服务发现、扩容缩容等整一套功能，本质上可看作是基于容器技术的mini-PaaS平台。本文旨在梳理Kubernetes的架构、概念及基本工作流，并且通过运行一个简单的示例应用来介绍如何使用Kubernetes。

###总体概览
如下图所示是我初步阅读文档和源代码之后整理的总体概览，基本上可以从如下三个维度来认识Kubernetes。
![Kubernetes概览]({{ site.baseurl }}/images/Kubernetes.png)

