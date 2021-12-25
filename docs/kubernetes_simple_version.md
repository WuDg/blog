# Kubernetes 简洁版

```describe
作者： 吴第广
日期： 2021年12月25日
```

## 参考文档

[书栈网](https://www.bookstack.cn/read/feiskyer-kubernetes-handbook-202005/README.md)

[官网](https://kubernetes.io/docs/home/)

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

### kube-scheduler

kube-scheduler 负责分配调度 Pod 到集群内的节点上，它监听 kube-apiserver，查询还未分配 Node 的 Pod，然后根据调度策略为 Pod 分配节点（更新 Pod 的 Nodename
字段）

调度器需要充分考虑的因素：

* 公平调度
* 资源高效利用
* QoS
* affinity 和 anti-affinity
* 数据本地化
* 内部负载干扰
* deadlines

**指定 Node 节点调度**

三种方式指定 Pod 只允许在执行的 Node 节点上

  1. nodeSelector：只调度到匹配指定 label 的 Node 上
  2. nodeAffinity：功能更丰富的 Node 选择器，如支持集合操作
  3. podAffinity：调度到满足条件的 Pod 所在的 Node 上

**nodeSelector（指定 Node 节点调度）**

首先给 Node 打标签

```shell
kubectl label nodes node-01 disktype=ssd
```

然后在 damonset 中指定 nodeSelector 为 disktype=ssd

```yaml
spec:
    nodeSelector: 
        disktype: ssd
```

**nodeAffinity 示例（指定 Node 节点调度）**

nodeAffinity 目前支持两种：

1. requiredDuringSchedulingIgnoredDuringExecution ：必须满足条件
2. preferredDuringSchedulingIgnoredDuringExecution：优选条件

下面例子代表调度到：包含标签 `kubernetes.io/e2e-az-name` 并且值为 `e2e-az1` 或 `e2e-az2`，并且优选还带有标签 `another-node-label-key=another-node-label-value`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: gcr.io/google_containers/pause:2.0
```

**podAffinity 示例（指定 Node 节点调度）**

同时支持标签的正向选择和反向选择

**Taints 和 tolerations**

用于保证 Pod 不被调度到不合适的 Node 上，其中 Taint 应用于 Node 上，而 toleration 应用于 Pod 上

目前支持 taint 类型

* NoSchedule：新的 Pod 不调度到该 Node 上，不影响正在运行的 Pod
* PreferNoSchedule：sofe 版 NoSchedule，尽量不调度到该 Node
* NoExecute：新的 Pod 不调度到该 Node 上，并且删除（evict）已在运行的 Pod，Pod 可以增加一个延迟删除时间（tolerationSeconds）

`注意`
DaemonSet 创建的 Pod 会自动加上 `node.alpha.kubernetes.io/unreachable` 和 `node.alpha.kubernetes.io/notReady` 的 `noExecute` Toleration，避免被删除

**优先级调度**

从 v1.8 开始，kube-scheduler 支持定义 Pod 的优先级，从而保证高优先级的 Pod 优先调度，并从 v1.11 开始默认开启

在指定 Pod 优先级之前需要先定义一个 PriorityClass(非 namespace 资源)，如

```yaml
apiVersion: v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

其中：

* value：为 32 位整数，越大优先级越高
* globalDefault：用户未配置 PriorityClass 的 Pod，值为 True 的 PriorityClass 集群唯一

**自定义调度器**

**调度器扩展**

**其他影响调度的因素**

1. 如果 Node Condition 处于 MemoryPressure，则所有 BestEffort 的新 Pod（未指定 resource limits 和 requests）不会调度到该 Node 上
2. 如果 Node Condition 处于 DiskPressure，则所有新 Pod 都不会调度到该 Node 上
3. 为了保证 Critical Pods 的正常运行，当它们处于异常状态时会自动重新调度。Critical Pods 是指
   1. annotation 包括 scheduler.alpha.kubernetes.io/critical-pod=''
   2. tolerations 包括 [{"key":"CriticalAddonsOnly", "operator":"Exists"}]
   3. priorityClass 为 system-cluster-critical 或者 system-node-critical

**启动 kube-scheduler 示例**

```shell
kube-scheduler --address=127.0.0.1 --leader-elect=true --kubeconfig=/etc/kubernetes/scheduler.conf
```

**kube-scheduler 工作原理**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kube-scheduler-choose-node.png)

`kube-scheduler` 调度分为两个阶段，predicate 和 priority

* `predicate：`过滤不符合条件的节点
* `priority：`有限级排序，选择优先级最高的节点

### Controller Manager

由 kube-controller-manager 和 cloud-controller-manager 组成，是 k8s 的大脑，它通过 apiserver 监控整个集群的状态，并确保集群处于预期的工作状态

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/post-ccm-arch.png)

kube-controller-manager 由一系列的控制器组成

* Replication Controller
* Node Controller
* CronJob Controller
* Daemon Controller
* Deployment Controller
* Endpoint Controller
* Garbage Collector
* Namespace Controller
* Job Controller
* Pod AutoScaler
* RelicaSet
* Service Controller
* ServiceAccount Controller
* StatefulSet Controller
* Volume Controller
* Resource quota Controller

cloud-controller-manager 在 k8s 启用 Cloud provider 时才需要，用来配合云服务供应商的控制，也包括一系列的控制器，如：

* Node Controller
* Route Controller
* Service Controller

**Metrics**

提供了控制器内部逻辑的性能度量

默认监听在 `kube-controller-manager` 的 10252 端口，提供 Prometheus 格式的性能度量数据，可以通过 `http://localhost:10252/metrics` 来访问

**三类控制器**

1. 必须启动的控制器：如 EndpointController、NamespaceController、ServiceAccountController等
2. 默认启动的可选控制器，可通过选项设置是否开启：如 NodeController、ServiceController等
3. 默认禁止的可选控制器，可通过选项设置是否开启：如 BootstrapSignerController、TokenCleanerController 等

**高可用**

**高性能**

引入 Informer，提供了基于事件通知的缓存机制，减少 API 调用

**Node Eviction**

Node 控制器在节点异常后，对异常 Node 进行驱逐

### kubelet

每个 Node 节点都运行一个 kubelet 服务进程，默认监听 10252 端口，接收并执行 Master 发来的指令、管理 Pod 及 Pod 中的容器。每个 kubelet 进程会在 API Server 上注册所在 Node 节点的信息，定期向 Master 节点汇报该节点的资源使用情况，并通过 cAdvisor 监控节点和容器的资源

**节点管理**

节点自注册和节点状态更新：

* kubelet 通过设置启动参数 --register-node 来确定是否向 API Server 注册自己
* 如果 kubelet 没有选择自注册模式，则需要用户配置 Node 资源信息，同时需要告知 kubelet 集群上的 API Server 的位置
* kubelet 在启动时通过 API Server 注册节点信息，并定时向 API Server 发送节点新消息，API Server 在接收到新消息后，将信息写入 etcd

**Pod 管理**

**获取 Pod 清单（Pod 管理）**

kubelet 以 PodSpec 的方式工作，PodSpec 时描述一个 Pod 的 YAML 或 JSON 对象。kubelet 采用一组机制提供的 PodSpecs（主要通过 apiserver），并确保 PodSpecs 中描述的 Pod 正常健康运行

向 kubelet 提供节点上需要运行的 Pod 清单的方法：

* 文件：启动参数 —config 指定的配置目录下的文件 (默认 / etc/kubernetes/manifests/)。该文件每 20 秒重新检查一次（可配置）。
* HTTP endpoint (URL)：启动参数 —manifest-url 设置。每 20 秒检查一次这个端点（可配置）。
* API Server：通过 API Server 监听 etcd 目录，同步 Pod 清单。
* HTTP server：kubelet 侦听 HTTP 请求，并响应简单的 API 以提交新的 Pod 清单。

**通过 API Server 获取 Pod 清单及创建 Pod 的过程（Pod 管理）**

kubelet 通过 API Server 监听 `register/nodes/$nowNode` 和 `register/pod`，将获取的信息同步到本地缓存

kubelet 监听 etcd，所有的 Pod 操作都会被 kubectl 监听到

Kubelet 读取监听到的信息，如果是创建和修改 Pod 任务，则执行如下处理：

* 为该 Pod 创建一个数据目录；
* 从 API Server 读取该 Pod 清单；
* 为该 Pod 挂载外部卷；
* 下载 Pod 用到的 Secret；
* 检查已经在节点上运行的 Pod，如果该 Pod 没有容器或 Pause 容器没有启动，则先停止 Pod 里所有容器的进程。如果在 Pod 中有需要删除的容器，则删除这些容器；
* 用 “kubernetes/pause” 镜像为每个 Pod 创建一个容器。Pause 容器用于接管 Pod 中所有其他容器的网络。每创建一个新的 Pod，Kubelet 都会先创建一个 Pause 容器，然后创建其他容器。
* 为 Pod 中的每个容器做如下处理：
  * 为容器计算一个 hash 值，然后用容器的名字去 Docker 查询对应容器的 hash 值。若查找到容器，且两者 hash 值不同，则停止 Docker 中容器的进程，并停止与之关联的 Pause 容器的进程；若两者相同，则不做任何处理；
  * 如果容器被终止了，且容器没有指定的 restartPolicy，则不做任何处理；
  * 调用 Docker Client 下载容器镜像，调用 Docker Client 运行容器。

**Static Pod**

**容器健康检查**

Pod 通过两类探针检查容器的健康状态：

* LivenessProbe：用于判断容器是否健康，如果探测到容器不健康，则 kubelet 删除该容器，并根据容器的重启策略做相应的处理，如果容器中不包含 LivenessProbe，则认为 LivenessProbe 返回的值永远是 `Success`
* ReadinessProbe：用于判断容器是否已准备好接收请求。如果 ReadlinessProbe 探测失败，则 Pod 的状态将被修改。Endpoint Controller 将从 Service 的 Endpoint 中删除包含该容器所在 Pod 的 IP 地址的 Endpoint 条目。

kubelet 定期调用容器中的 LivenessProbe 来诊断容器的健康状况。包含以下三种方式：

1. ExecAction：在容器中执行指定的命令，该命令退出码为 0，则表示容器健康
2. TCPSocketAction：通过 IP 地址和端口号执行 TCP 检查，如果能访问，则容器健康
3. HTTPGetAction：通过 `curl http://ip:port` 方式，如果响应的状态码 200<=statusCode<400，则容器健康

**cAdvisor 资源监控**

k8s 集群中不同级别上的进检测：容器、Pod、Service 和整个集群。

cAdvisor 是一个开源的分析容器资源使用率和性能特性的代理工具，集成到 kubelet 中，当 kubelet 启动时会同时启动 cAdvisor，只监控当前节点的容器。自动采集 CPU、内存、文件系统和网络使用的统计信息

**kubelet Eviction（驱逐）**

kubelet 会监控资源使用情况，并使用驱逐机制防止计算和存储资源耗尽

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kubelet_eviction.png)


`驱逐信号分为软驱逐和硬驱逐`

`驱逐动作包括回收节点资源和驱逐用户 Pod`：

* 回收节点资源
  * 配置了 imagefs 阈值时：
    * 达到了 nodefs 阈值：删除已停止的 Pod
    * 达到 imagefs 阈值：删除未使用的镜像
  * 未配置 imagefs 阈值时：：
    * 达到了 nodefs 阈值时：删除已停止的 Pod 和删除未使用的镜像

* 驱逐用户 Pod
  * 驱逐顺序为：BestEffort、Burstable、Guaranteed
  * 配置了 imagefs 阈值时
    * 达到 nodefs 阈值，基于 nodefs 用量驱逐（local volume + logs）
    * 达到 imagefs 阈值，基于 imagefs 用量驱逐（容器可写层）
  * 未配置 imagefs 阈值时
    * 达到 nodefs阈值时，按照总磁盘使用驱逐（local volume + logs + 容器可写层）

**容器运行时**
> Container Runtime

负责真正管理镜像和容器的生命周期。kubelet 通过容器运行时接口（CRI）于容器运行时交互，以管理镜像和容器

CIR 将 kubelet 与容器运行时解耦，将原来完全面向 Pod 级别的内部接口拆分成面向 Sandbox 和 Container 的 gRPC 接口，并将镜像管理和容器管理分离到不同服务

目前基于 CRI 容器引擎已经非常丰富了，包括：

* Docker
* OCI
* PouchContainer
* Frakti
* Rktlet
* Virtlet
* 等

**kubelet 工作原理**

* kubelet API：包括 10252 端口的认证 API、4194 端口的 cAdvisor API、10255 端口的只读 API 以及 10248 端口的健康检查 API
* syncLoop：从 API 或者 manifest 目录接收 Pod 更新，发送到 podWorkers 处理，大量使用 channel 处理异步请求
* 辅助 manager：如 cAdvisor、PLEG、Volume manager 等
* CRI：容器执行引擎接口，负责与 container runtime shim 通信
* 容器执行引擎：如 dockershim、rkt 等
* 网络插件：目前支持 CNI 和 kubenet

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kubelet.png)

**Pod 启动流程**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/pod-start.png)

**查看 Node 汇总指标**

通过 kubelet 的 10255 端口可以查询 Node 汇总指标。

* 集群内部可以直接访问 kubelet 的 10255 端口，如：`http://<node-name>:10255/stats/summary`
* 在集群外可以借助 `kubectl proxy&`：`curl http://localhost:8001/api/v1/proxy/nodes/<node-name>:10255/stats/summary
`

### kubectl-proxy

每个节点都运行一个 kube-proxy，它监听 API Server 中的 service 和 endpoint 的变化情况，并通过 iptables 等来为服务配置负载均衡（TCP/UDP）

kube-proxy 可以直接运行在物理机，也可也以 static pod 或者 daemnoSet 方式运行

kube-pryx 目前支持以下几种实现：

* userspace：最早的负载均衡方案，它在用户空间监听一个端口，所有服务通过 iptables 转发到这个端口，然后在其内部负载均衡到实际的 Pod。效率低，有明显性能瓶颈
* iptables：目前推荐的方案，完全以 iptables 规则方式实现 service 负载均衡。存在当服务多的时候产生太多的 iptables 规则，更新时存在一定时延，存在性能问题
* ipvs：为解决 iptables 模型性能问题，采用增量式更新，保证 service 更新期间连接保持不断
* winuserspace：同 userspace，但仅工作在 windows 节点

`注意:`使用 ipvs 模式时，需要预先在每台 Node 上加载内核模块：`nf_conntrack_ipv4`、`ip_vs`、`ip_vs_rr`、`ip_vs_wrr`、`ip_vs_sh` 等

```shell
# load module <module_name>
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
# to check loaded modules, use
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
# or
cut -f1 -d " "  /proc/modules | grep -e ip_vs -e nf_conntrack_ipv4
```


**Iptables 示例**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/iptables-mode.png)

**ipvs 示例**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/ipvs-mode.png)

**kube-proxy 工作原理**

kube-proxy 监听 API Server 中的 service 和 endpoint 的变化情况，并通过 userspace、iptables、ipvs 或 winuserspace 等 proxier 来为服务配置负载均衡

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kube-proxy.png)

**kube-proxy 不足**

kube-proxy 目前只支持 TCP 和 UDP，不支持 HTTP 路由，并且没有健康检查机制。这些可以通过自定义 Ingress Controller 的方法来解决

### Federation

k8s 的设计定位是单一集群在同一地域内
集群联邦，为了提供跨区、跨服务商 k8s 集群服务而设计的

略

### kubeadm 工作原理

kubeadm 是 k8s 主推的部署工具之一

**参考文档**

[书栈网](https://www.bookstack.cn/read/feiskyer-kubernetes-handbook-202005/components-kubeadm.md)
[官网](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

略

### hyperkube

hyperkube 是 k8s 的 all-in-one 二进制包，可以用来启动多种 k8s 服务

hyperkube支持的子命令包括

* kubelet
* apiserver
* controller-manager
* federation-apiserver
* federation-controller-manager
* kubectl
* proxy
* scheduler

### kubectl

kubectl 是 k8s 的命令行工具，是 k8s 用户和管理员必备的管理工具

kubectl 提供了大量的子命令，方便管理 k8s 集群

* `kubectl -h` 查看子命令列表
* `kubectl options` 查看全局选项
* `kubectl <command> --help` 查看子命令的帮助
* `kubectl [command] [PARAMS] -o=<format>` 设置输出格式（如 json、yaml、jsonpath 等）
* `kubectl explain [RESOURCE]` 查看资源的定义

**配置**

配置 k8s 集群以及认证方式，包括：

* cluster 信息：k8s server 地址
* 用户信息：用户名、密码或密钥
* Context：cluster、用户信息以及 NameSpace 等

```shell
kubectl config set-credentials myself --username=admin --password=secret
kubectl config set-cluster local-server --server=http://localhost:8080
kubectl config set-context default-context --cluster=local-server --user=myself --namespace=default
kubectl config use-context default-context
kubectl config view
```

**常用命令**

* 创建：`kubectl run <name> --image=<image>` 或者 `kubectl create -f manifest.yaml`
* 查询：`kubectl get` <resource>
* 更新 `kubectl set` 或者 `kubectl patch`
* 删除：`kubectl delete <resource> <name>` 或者 `kubectl delete -f manifest.yaml`
* 查询 Pod IP：`kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'`
* 容器内执行命令：`kubectl exec -ti <pod-name> sh`
* 容器日志：`kubectl logs [-f] <pod-name>`
* 导出服务：`kubectl expose deploy <name> --port=80`
* Base64 解码：`kubectl get secret SECRET -o go-template='{{ .data.KEY | base64decode }}'`

注意，`kubectl run` 仅支持 `Pod、Replication Controller、Deployment、Job` 和 `CronJob` 等几种资源。具体的资源类型是由参数决定的，默认为 `Deployment`

**命令自动补全**

Linux 

```shell
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```

MacOs

```shell
source <(kubectl completion zsh)
```

**自定义输出列**

**日志查看**

```shell
# Return snapshot logs from pod nginx with only one container
kubectl logs nginx
# Return snapshot of previous terminated ruby container logs from pod web-1
kubectl logs -p -c ruby web-1
# Begin streaming the logs of the ruby container in pod web-1
kubectl logs -f -c ruby web-1
```

**连接到一个正在运行的容器**

```shell
  # Get output from running pod 123456-7890, using the first container by default
  kubectl attach 123456-7890
  # Get output from ruby-container from pod 123456-7890
  kubectl attach 123456-7890 -c ruby-container
  # Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod 123456-7890
  # and sends stdout/stderr from 'bash' back to the client
  kubectl attach 123456-7890 -c ruby-container -i -t
Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
  -i, --stdin=false: Pass stdin to the container
  -t, --tty=false: Stdin is a TTY
```

**在容器内部执行命令**

```shell
  # Get output from running 'date' from pod 123456-7890, using the first container by default
  kubectl exec 123456-7890 date
  # Get output from running 'date' in ruby-container from pod 123456-7890
  kubectl exec 123456-7890 -c ruby-container date
  # Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod 123456-7890
  # and sends stdout/stderr from 'bash' back to the client
  kubectl exec 123456-7890 -c ruby-container -i -t -- bash -il
Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
  -p, --pod='': Pod name
  -i, --stdin=false: Pass stdin to the container
  -t, --tty=false: Stdin is a TT
```

**端口转发**

`kubectl port-forward` 用于将本地端口转发到指定的 Pod

```shell
# Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in the pod
kubectl port-forward mypod 5000 6000
# Listen on port 8888 locally, forwarding to 5000 in the pod
kubectl port-forward mypod 8888:5000
# Listen on a random port locally, forwarding to 5000 in the pod
kubectl port-forward mypod :5000
# Listen on a random port locally, forwarding to 5000 in the pod
kubectl port-forward mypod 0:5000
```

也可以将本地端口转发到服务、rs 等

```shell
# Forward to deployment
kubectl port-forward deployment/redis-master 6379:6379
# Forward to replicaSet
kubectl port-forward rs/redis-master 6379:6379
# Forward to service
kubectl port-forward svc/redis-master 6379:6379
```

**API Server 代理**

`kubectl proxy` 命令提供了一个 k8s API 服务的 HTTP 代理

```shell
$ kubectl proxy --port=8080
Starting to serve on 127.0.0.1:8080
```

可以通过代理地址 `http://localhost:8080/api/` 来直接访问 k8s API，比如查询 Pod 列表

```shell
curl http://localhost:8080/api/v1/namespaces/default/pods

```

**文件拷贝**

`kubectl cp` 支持从容器和节点双向拷贝

```shell
  # Copy /tmp/foo_dir local directory to /tmp/bar_dir in a remote pod in the default namespace
  kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir
  # Copy /tmp/foo local file to /tmp/bar in a remote pod in a specific container
  kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>
  # Copy /tmp/foo local file to /tmp/bar in a remote pod in namespace <some-namespace>
  kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
  # Copy /tmp/foo from a remote pod to /tmp/bar locally
  kubectl cp <some-namespace>/<some-pod>:/tmp/foo /tmp/bar
Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
```

`注意：文件拷贝依赖于 tar 命令，所以容器中需要能够执行 tar 命令`

**权限检查**

**查看事件**

```shell
# 查看所有事件
kubectl get events --all-namespaces
# 查看名为nginx对象的事件
kubectl get events --field-selector involvedObject.name=nginx,involvedObject.namespace=default
# 查看名为nginx的服务事件
kubectl get events --field-selector involvedObject.name=nginx,involvedObject.namespace=default,involvedObject.kind=Service
# 查看Pod的事件
kubectl get events --field-selector involvedObject.name=nginx-85cb5867f-bs7pn,involvedObject.kind=Pod
```

**插件**

**原始 URL**

kubectl 可以用来直接访问 API Server 的 REST API

```shell
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```

`附录`

kubectl 安装方式

```shell
# OS X
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
# Linux
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
# Windows
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/windows/amd64/kubectl.exe
```

## 资源对象
