# kubernetes网络

```describe
作者： 吴第广
日期： 2021年12月21日
```

k8s中网络相关概念：Pod网络，Service网络，Cluster IPs，NodePort，LoadBalance和Ingress等等

这里把k8s的网络分解为四个抽象层，编号0~3，除了第0层外，每一层都是构建于前一层之上，如图：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/k8s-network.png)

第0层Node节点网络，保证k8s节点间通信的网络，这个网络一般由底层（公有云或数据中心）支持

## 一、Pod网络
> Pod相当于k8s云平台的虚拟机，它是k8s的基本调度单位。
> Pod网络：保证k8s集群中的所有Pods互相通信。逻辑上看起来在同一个平面网络内，能够相互做IP寻址和通信的网络。

下面是Pod网络的简化概念模型：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/pod-network.png)

Pod网络构建于Node节点网络之上

### 1.1 同一节点间的Pod网络
> 一个Pod中可以存在多个容器，这些容器共享Pod的网络栈和其他资源，如Volume

什么是共享网络栈？同一节点上的Pod之间如何寻址和通信？

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/pod-network-detail.png)

上图节点中Pod网络依赖的3个网络设备：

* 主机网卡(eth0):是节点主机的网卡，支持节点流量出入，也支持集群节点间IP寻址和通信
* 虚拟网桥(docker0):可以理解为是一个虚拟交换机，支持该节点上的Pod之间的IP寻址和通信
* Pod虚拟网卡(veth0):支持该Pod内容器通信和对外访问的虚拟设备

`docker0网桥和veth0网卡，都是linux支持和创建的虚拟网络设备`

`共享网络栈：`
上图中Pod1内部由3个容器，它们共享同一个虚拟网卡veth0，内部的容器间可以通过`localhost`相互访问，但是不能使用同一个端口

`Pod1`中还有一个比较特殊的容器`pause`，这个容器运行的唯一的目的是为Pod建立共享的`veth0`网络接口，

`Pod` 的IP是由`docker0`网桥分配的，如上图`docker0`网桥IP是`172.17.0.1`,此时，如果再启动其他`Pod`，由于这些Pods都连在同一个网桥上，即再同一个网段内，因此Pods间可以进行IP寻址和通信，如下图：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/pod-network-between.png)

从上图得，节点内Pod网络在`172.17.0.0/24`这个地址空间内，而节点主机在`10.100.0.0/24`地址空间内，即Pod网络和节点网络在不同网络中

### 1.2 不同节点间Pod网络

假设我们有两个节点主机，host1(10.100.0.2)和host2(10.100.0.3)，它们都在`10.100.0.0/24`这个地址空间内。

`Pod网络的地址是由k8s统一管理和分配的，保证集群内Pod的IP地址唯一`

节点网络和Pod网络不在同一个网络地址空间内，此时host1上的PodX如何与host2上的PodY通信？


![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/two-node-pod-network.png)

实际上不同节点间的Pod网络通信有很多技术方案，这里介绍下面两种：
1. 路由：通过路由设备为k8s集群的Pod网络单独划分网段，并配置路由器支持Pod网络的转发。依赖底层网络设备，无额外性能开销
2. 覆盖（Overlay）：在现有网络上建立一个虚拟网络，实现技术包括：flannel、weavenet等。简单理解就是类似TCP5层或7层协议栈，出节点前，对数据包进行封装，到达目标节点后，数据包解封，再转发给节点内部Pod网络

### 1.3 CNI
> 为了简化Pod网络集成，k8s支持CNI(Container Network Interface)标准，不同的Pod网络技术可以通过CNI插件形式和k8s进行集成。

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/pod-network-cni.png)

### 1.4 总结
1. k8s网络分为4层，每一层都构建于上一层
2. 同一个节点内Pod网络依赖于虚拟网桥和虚拟网卡等linux虚拟设备。同一个Pod内容器共享该Pod网络栈，这个网络栈由pause容器创建
3. 不同节点Pod网络，可以采用路由和覆盖网络等技术方案。路由依赖底层网络设备，覆盖网络存在额外开销
4. CNI是一个Pod网络集成标准

## 二、Service网络

Pod 网络存在的前提下，下图是Service网络的简化概念模型

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-netword-model.png)

k8s通过引入一层`Account-Service`来实现下面的功能
* 服务发现：`Client Pod`发现并定位`Account-App`集群中的`Pod IP`。`Account-Service`提供统一的`ClusterIP`，`Client`通过`ClusterIP`就可以访问`Account-App`集群中的`Pod`，这里的`ClusterIP`是虚拟IP
* 负载均衡：Client Pod以负载均衡策略去访问`Account-App`集群中不同的Pod实例。默认使用RoundRobin

### 2.1 服务发现技术
> DNS 域名服务，可以认为是最早的一种服务发现技术
![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-find-map.png)

`方案1`
Pod运行时，k8s把Account-App的Pod集群信息自动注册到DNS，Client 应用通过域名查询DNS查找目标Pod，然后发起调用

`存在问题`
1. 不同DNS客户端实现功能存在差异，有些客户端每次调用都去查询DNS服务，造成不必要开销，而有些客户端则会缓存DNS信息，默认实践较长，当目标Pod IP发生变化时，存在缓存刷相信不及时，导致访问Pod失效
2. DNS客户端实现的负载均衡一般比较简单，有些甚至不支持负载均衡调用

`k8s是引入kube-DNS支持通过域名访问服务的，不过这是建立在ClusterIP/Service`

`方案2`
引入 Service Registry + Client配合，目前主流产品如：Eureka + Ribbon，Consul，Nacos等

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-find-registry-map.png)

k8s自身的分布式存储 etcd 就可以实现 Service Registry。运行时k8s把Account-App和Pod信息注册到Service Registry，Client 应用通过 Service Registry查询目标Pod。

`存在问题`
1. 需要客户端配合，对客户端有侵入性

`方案3`
Service Registry + DNS

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-find-service-registry-map.png)

k8s集群每个worker节点都部署有两个组件：
1. kubelet
2. kube-proxy

这两个组件 + Master 是k8s实现服务注册和发现的关键

`服务注册发现流程：`

1. 在服务Pod实例发布时，kubelet会服务启动Pod实例，启动完成后，kubelet会把服务的Pod IP列表注册到 Master 节点
2. 通过服务 Service 的发布，k8s会为服务分配 ClusterIP，相关信息也记录在 Master
3. 在服务发现阶段，Kube-Proxy会监听Master并发现服务ClusterIP和PodIP列表映射关系，并且修改本地的Linux iptables转发规则，指示iptables在接收到某个ClusterIP请求时，进行负载均衡并转发到对应的PodIP上
4. 当消费者Pod需要访问某个目标服务实例时，通过ClusterIP（服务名）发起调用，这个ClusterIP会被本地iptables机制截获，然后通过负载均衡算法，转发到目标服务Pod实例上

为了屏蔽ClusterIP的变化，k8s在每个worker节点上引入了一个kubeDNS组件，它也监听Master并发现服务名和ClusterIP之间映射关系，这样，消费者Pod通过KubeNDS可以间接发现服务的ClusterIP

Kube-Proxy：ClusterIP -> PodIP
Kube-DNS：ServiceName -> ClusterIP

### 2.2 总结
1. k8s的Service网络构建于Pod网络之上，主要用于解决服务发现和负载均衡问题
2. k8s通过ServiceName + ClusterIP统一屏蔽服务发现和负载均衡，底层技术时在DNS + Service Registry 演进而来
3. k8s服务发现和负载均衡是在客户端通过 Kube-Proxy + iptables转发实现，对应用无侵入，且不穿透 Proxy，无额外性能损耗
4. k8s服务发现机制，可以认为是现在微服务发现机制和传统linux内核机制的优雅结合

有了Service抽象，k8s中部署的应用都可以通过一个抽象的ClusterIP进行寻址访问。但是k8s的Service网络知识一个集群内可见的内部网络，集群外部时看不到Service网络的。如何将应用暴露出去，让外网访问

## 三、外部接入网络
> NodePort,LoadBalancer,Ingress

### 3.1 NodePort
> NodePort 是k8s将内部服务对外暴露的基础，后面的LoadBalancer底层依赖于NodePort

实际上是由节点网络可以直接对外暴露，具体的暴露方式看数据中心或公有云的底层网络部署

Kube-proxy：掌握Service网络的所有信息，可以和Service网络以及Pod网络通信，同事又可以和节点网络打通

NodePort：通过Kube-Proxy在节点上暴露一个监听端口，将k8s内部服务通过Kube-Proxy暴露出去的方式，就叫NodePort(端口暴露在节点)

下图是`NodePort`暴露服务的简化概念模型：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-network-nodeport.png)

我们要将k8s内部某个服务通过NodePort方式暴露出去，只需将服务 type设定为NodePort，同时指定一个范围在 3000~32767 内的对外暴露端口。服务发布后，k8s在每个worker节点都会开启这个监听端口，这个监听端口背后时Kube-Proxy

当外部Client要访问k8s集群内的某个服务，通过这个服务的NodePort端口发起调用，这个调用通过Kube-Proxy转发到内部的Service抽象层，再转发到目标Pod上

### 3.2 LoadBalancer

服务以NodePort方式暴露出去，k8s会在每个worker节点上都开启对应的NodePort端口。逻辑上看，k8s集群中所有的节点都会以集群的方式暴露这个服务。既然是集群，就会涉及到负载均衡问题，因此引入负载均衡器（Load Balancer）。下图是通过LoadBalancer将服务对外暴露的概念模型。

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-network-nodeport-loadbalancer.png)

`基于阿里云`
将k8s内部服务通过LoadBalancer方式暴露出去，可以将服务 type设定为 LoadBalancer。服务发布后，k8s集群会自动创建服务的NodePort端口转发，同时自动帮我们申请一个 SLB，有独立公网IP，并且k8s会自动把SLB映射到后台k8s集群对应的NodePort上。这样通过SLB的公网IP就可以访问到k8s内部服务。

### 3.3 Ingress

公网LB+IP是收费的，如果暴露一个服务就需要购买一个独立的LB + IP，那服务多了的话成本就高了。

思路：想办法在k8s内部部署一个独立的反向代理服务，让它做代理转发

Ingress：即k8s内部的福利部署的反向代理服务


![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-network-ingress-loadbalancer.png)

Ingress 就是一个特殊的Service，通过节点的 HostPort(80/443)暴露出去，前置一般也有LB做负载均衡。

这个Service提供的功能主要就是7层反向代理（可以提供安全认证，监控，限流和SSL证书等功能），类似Nginx

哪些软件可以实现Ingress功能？

Nginx、haproxy
现代的微服务网关 Zuul，Gateway，Kong，Envoy，Traefik等

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/service-network-ingress-detail.png)

### 3.4 Kubectl Proxy & Port Forward
> 上述的 NodePort/LoadBalancer/Ingress 主要针对正式环境，本地开发测试环境的调试可以使用下面方法

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/kubectl-proxy.png)

1. 通过`kubectl proxy` 命令，在本机开启一个代理服务，通过这个代理服务，可以访问k8s集群内的任意服务。原理是通过Master上的API Server简介访问k8s集群内服务，因为Master知道集群内所有服务信息
2. 通过`kubectl port-forward`命令，它可以在本机开启一个转发端口，间接转发到k8s内部的某个Pod端口上。这样，我们通过本机端口就可以访问k8s集群内的蘑菇额Pod。这种方式是TCP转发
3. 通过`kubectl exec` 命令直连Pod去执行Linux命令，如`curl`

### 3.5 总结
1. NodePort 是k8s内部服务对外暴露的基础。LoadBalancer底层依赖于NodePort。NodePort底层是Kube-Proxy，Kube-Proxy是沟通Service网络、Pod网络和节点网络的桥梁
2. 将k8s服务通过NodePort对外暴露是以集群方式暴露的，每个节点都会暴露相应的NodePort，通过LoadBalancer可以实现负载均衡，公有云提供的k8s都支持自动部署LB，且提供公网IP，LB对接NodePort
3. Ingress 是k8s内部的反向代理服务，通过Ingress，我们可以同时对外暴露多个服务，只需购买要给LB，Ingress 本质也是一种k8s的Service，通过暴露80/443端口
4. 通过`kubectl proxy`或者`kubectl port-forward`可以在本地环境快速调试访问k9s集群中的服务或者Pod
5. k8s中Service主要有3中type
   1. ClusterIP：表示仅内部可访问
   2. NodePort：表示通过NodePort对外暴露服务
   3. LoadBalancer：表示通过Loadbalancer对外暴露服务（底层对阶NodePort，一般公有云才支持）


`表格总结：`

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/k8s-network-table.png)


[原文地址](https://blog.csdn.net/qq_35789269/article/details/113922038)