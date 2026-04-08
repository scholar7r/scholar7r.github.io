+++
title = "K8s 深入浅出"
date = "2026-03-27T22:44:41+08:00"
author = "scholar7r"
authorTwitter = "scholar7r"
tags = ["墨技"]
keywords = ["k3s", "k8s"]
toc = true
draft = true
+++

Kubernetes (K8s) 是 Google 大规模容器管理技术 Borg 的开源版本。通过使用 K8s 可以实现容器集群的自动化部署、自动化扩缩容、维护等功能。

<!--more-->

## 引入

在容器的概念出现之前我们通常将服务部署在一台 Linux 主机中。通常在一台机器上部署多个服务，这些服务共享一个库，这样做就会导致应用的所有运行、配置、管理以及生存周期都和操作系统强绑定。不便于应用的版本更新等操作，当然也可以使用虚拟机来实现这种解耦的需求，但是虚拟机更加占用资源，且并不易于管理，不具有可移植性。

现代的方式是通过部署容器方式实现，容器之间相互隔离，每个容器有自己的文件系统，依赖关系隔离。资源占用少、部署快，每个应用可以被打包成一个容器镜像（进行打包可以通过 CI/CD 持续交付自动构建，且可以快速实现更新和回滚），应用和基础设施之间可以深度解耦。简单来说就是如下图的这种架构：

```goat
+-----+-----+ +----------+----------+
| App | App | | App      | App      |
+-----+-----+ | Libraies | Libraies |
| App | App | +----------+----------+
+-----+-----+ | App      | App      |
| Libraies  | | Libraies | Libraies |
+-----------+ +----------+----------+
| Kernel    | | Kernel              |
+-----------+ +---------------------+
```

所有的容器之间实现了相互隔离，不再共享同一个库，而是自己管理自己的依赖。

当然，这种部署方式的革命持续了一个又一个周期，在演变成容器化部署之前，人们便是使用庞大的虚拟机系统来部署应用的。

![部署方式的演变](https://kubernetes.io/images/docs/Container_Evolution.svg)

### K8s 能做什么

K8s 可以在物理或虚拟的集群上运行容器化应用，本文侧重于实用性，不描述 K8s 的理念，但是要学习这个东西，你必须理解他为什么存在，参阅 [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)。

### 集群架构

K8s 集群由一个控制平面和一组成为节点的机器组成，这些节点运行容器化应用。每个集群至少需要一个工作节点来运行 Pod。

工作节点托管应用运行负载的 Pod，控制平面管理集群中的工作节点和 Pod。在生产环境中，控制平面跨越多台计算机，集群运行多个节点，通过冗余的方式提高容错率和高可用性。

![K8s 集群架构](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

## K8s 组件

集群架构的组成离不开其组件：

![K8s 组件](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

### 控制平面组件

- kube-apiserver 作为 K8s 核心组件用于暴露 K8s HTTP API 服务
- etcd 是 K8s 默认提供的键值存储系统，保存所有集群数据。
- kube-scheduler 调度器用于监视未分配到工作节点的 Pod，它为每一个 Pod 调度工作节点。
- cloud-controller-manager 是可选的组件，用于集成云服务。

### 节点组件

节点组件在每个节点上运行，用于维护运行中的 Pod，并提供 K8s 运行环境。

- kubectl 用于确保和管理 Pod 运行，包括 Pod 内的容器。他可以安装 Pod 所需的 Volume，下载 Pod 的 Secrets。
- kube-proxy 是可选的网络组件，用于维护容器内的网络规则并执行网络转发来实现 K8s 服务抽象。

### 插件

- DNS 插件使用 CoreDNS，用于 K8s 的服务注册和发现。
- Web UI 提供 Web 界面的集群管理操作。
- Container Resource Monitoring 容器资源监控用于收集和存储容器指标。
- Cluster-level Logging 在一个中心日志存储服务中保存容器日志。

## K8s API

[Kubernetes API](https://kubernetes.io/docs/reference/kubernetes-api/) 中列出了详细的 API 资源，K8s 有着足够庞大的 API 体系，本文只提供 API 参考文档链接，建议将 API 作为 K8s 应用之后的一个独立章节进行学习。我们所使用的 K8s 操作基本上已由 kubectl 实现，如果有运维面板、自动化工具等需求，需要了解 K8s API。

## kubectl

Linux 可以使用下方命令下载最新版 kubectl：

```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

待下在完成之后使用 install 命令进行安装：

```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

建议使用虚拟机安装 K3s（K8s 的精简发行版）用于学习 K8s 的整个生态，K3s 会自动附带 kubectl 这些工具，通常我们会使用：

- apply 应用服务
- delete 删除服务
- rollout 更新服务
