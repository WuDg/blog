# Kubernetes 简洁版

```describe
作者： 吴第广
日期： 2021年12月25日
```

## 参考文档

https://www.bookstack.cn/read/feiskyer-kubernetes-handbook-202005/README.md

## 入门

### 简介

Kubernetes 是 Google 开源的容器集群管理系统，是 Google 多年大规模容器管理技术 Brog 的开源版本，主要功能包括：

* 基于容器的应用部署、维护和滚动升级
* 负载均衡和服务发现
* 跨机器和跨地区的集群调度
* 自动伸缩
* 无状态服务和有状态服务
* 广泛的 Volume 支持
* 插件机制保证扩展性

### 核心组件

Kubernetes 主要由以下几个核心组件组成：

* etcd 保存整个集群的状态
* apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制
* controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
* scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上
* kubelet 负责维护容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理
* Container runtime 负责镜像管理以及 Pod 和容器的运行（CRI）
* kube-let 负责为 Service 提供 Cluster 内部的服务发现和负载均衡

![architecture](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/architecture.png)

## 核心原理

### 架构原理

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/components.png)

除了核心组件，还有一些推荐的插件

* kube-dns 负责为整个集群提供 DNS 服务
* Ingress Controller 为服务提供外网入口
* Heapster 提供资源监控
* Dashboard 提供 GUI
* Federation 提供跨可用区的集群
* Fluentd-elasticsearch 提供集群日志采集、存储与查询

**分层架构**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/14937095836427.jpg)

* 核心层：Kubernetes 最核心的功能，对外提供 API 构建高层的应用，对内提供插件式应用执行环境
* 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）
* 管理层：系统度量（如基础设施、容器和网络的度量）、自动化（如自动扩展、动态 Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）
* 接口层：kubectl 命令行工具、客户端 SDK 以及集群联邦
* 生态系统：
  * 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
  * 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理


**核心组件**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/core-packages.png)

**核心 API**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/core-apis.png)

**生态系统**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/core-ecosystem.png)

### 核心组件（通信）

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/components1.png)

**Pod 创建流程**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/workflow.png)

* 用于通过 REST API 创建一个 POD
* API Server 将其写入 etcd
* Scheduler 检测到未绑定 Node 的 Pod，开始调度并更新 Pod 的 Node 绑定
* Kubelet 检测到有新的 Pod 调度过来，通过 Container Runtime 运行该 Pod
* Kubelet 通过 Container Runtime 获取 Pod 状态，并更新到 API Server 中

**端口号**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/ports.png)

### ectd

Etcd 是 CoreOs 基于 Raft 开发的分布式 k-v 存储，可用于服务发现、共享配置以及一致性保证

**etcd 主要功能**

* 基本的 k- v 存储
* 监听机制
* key 的过期及续约机制，用于监控和服务发现
* 原子 CAS 和 CAD，用于分布式锁和 leader 选举

**etc 基于 Raft 的一致性**

leader 选举：通过心跳机制和投票

日志复制：过半同步成功则通知所有 Follower 将日志存储在本地磁盘

### API Server
> kube-apiserver 是 Kubernetes 最重要的核心组件之一

* 提供集群管理的 REST API 接口，包括认证授权、数据校验以及集群状态变更
* 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才能直接操作 etcd）

**REST API**

kube-apiserver 支持同时提供 https 和 http API，其中 http API 是非安全接口，不做任何认证授权机制，不建议生产环境启用

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/API-server-space.png)

实际使用中，通常通过 kubectl 来调用 apiserver，也可也通过 kubernetes 各个语言的 client 库来访问 apiserver

通过下面命令打开调试日志

```shell
kubectl get pod -n ns --v=8
```

**访问控制**

认证 -> 授权 -> 准入控制

**工作原理**

kube-apiserver 提供 k8s 的 REST API，实现了认证、授权、准入控制等安全校验功能，同时也负责集群状态的存储操作（通过 etcd）

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kube-apiserver.png)

以 `/apis/batch/v2alpha1/jobs` 为例

GET 请求的处理过程如下图：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/API-server-flow.png)

POST 请求的处理过程如下图：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/API-server-storage-flow.png)

**API 访问**

多种方式访问 k8s 提供的 REST API：

* kubectl
* SDK，支持多种语言
  * Go
  * Python
  * JS
  * Java
  * 其他

kubectl

```shell
kubectl get --raw /api/v1/namespaces
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

kubectl proxy

```shell
kubectl proxy --port=8080 &
curl http://localhost:8080/api/
```