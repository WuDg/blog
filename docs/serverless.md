# serverless-无服务架构


- [serverless-无服务架构](#serverless-无服务架构)
  - [写在前面](#写在前面)
  - [入门](#入门)
    - [简介](#简介)
    - [定义](#定义)
    - [Serverless 历史](#serverless-历史)
    - [Serverless 分类](#serverless-分类)
    - [Serverless 使用场景](#serverless-使用场景)
    - [Serverless 优缺点](#serverless-优缺点)
    - [开源的 Serverless 框架](#开源的-serverless-框架)
  - [Serverless 核心技术](#serverless-核心技术)
    - [函数计算](#函数计算)
    - [事件驱动](#事件驱动)
## 写在前面

```describe
作者： 吴第广
日期： 2022年01月02日
```

`写作要素：`

1. 内容第一
2. 格式整洁

`参考文档`

1. [书栈网](https://www.bookstack.cn/read/serverless-handbook/README.md)

## 入门

[Bilibili 半小时入门＋实操](https://www.bilibili.com/video/BV1fK4y1P76C)

###  简介

Serverless（无服务架构）是指服务端逻辑由开发者实现，应用运行在无状态的计算容器中，由时间触发，完全被第三方管理，其业务层面的状态则存储在数据库或存储起源中。

Serverless 是云原生技术发展的高级阶段，可以使开发者更聚集在业务逻辑，而减少基础架构的关注。

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/b2276d665a511c4404f25fe596cce34e.jpeg)

### 定义

**简单定义**

无服务架构并不是指完全没有服务器的服务架构，而是让开发者无需关注服务器层面，只需关注代码即可，其他资源交给第三方提供。

**进阶定义**

Server 是由事件（event）驱动（如 HTTP、pub/sub）的全托管计算服务。用户无需管理服务器等基础设施，只需编写代码和选择触发器（trigger），比如 RPC请求、定时器等，其余工作（如机器实例选择、扩缩容、部署、容灾、监控、日志、安全补丁等）全部由 serverless 系统托管。用户只需要为代码实际运行消耗的资源付费 -- 代码未运行则不产生费用

Serverless 相对于 serverful，对业务用户强调 noserver的运维理念，业务人员只需要聚焦业务逻辑代码

Serverless 相对于 serverful

1. `弱化了存储和计算之间的联系` 服务的存储和计算被分开部署和收费，存储不再是服务本身的一部分，而是演变成了独立的云服务，这使得计算变得无状态化，更容器调度和扩缩容，同时也降低了数据丢失的风险
2. `代码的执行不再需要手动分配资源` 只需要提供一份代码，剩下的交由 serverless 平台去处理即可。当前阶段的实现平台分配资源时还需要用户方提供一些策略，如单个实例的规模、最大并发数和单实例最大 CPU 使用率。理想情况时通过某些学习算法来进行完全自动的自适应分配
3. `按使用量计费` Serverless 按照服务的使用量（调用次数、时常等）计费，而不是像传统的 serverful 服务那样，按照使用的资源（ECS 实例、VM 的规模等）计费

**总结**

软件：C/S 和 MVC -> SOA -> 微服务架构 -> Cloud Native（云原生）

企业应用：单体架构 -> 服务化 -> 微服务化

云改变了我们对操作系统的认知，一个系统的资源、存储和网络时可以分离配置的

我们始终没有摆脱服务器的束缚，应用还是必须运行在实体或者虚拟的服务器上，必须经过部署、配置、初始化才可以提供服务，还需要对服务器和应用进行监控和管理，还需要保证数据的安全性

`让我们只要关注自己代码的逻辑就好了，其他部分云帮我们实现`

### Serverless 历史

Serverless  架构是云的自然延申，为了理解 Serverless，必须先了解云计算的发展

**IaaS**

2006 年 AWS 推出 EC2（Elastic Compute Cloud），作为第一代 Iaas（Infrastructure as a Service），用户可以用过 AWS 快速的申请到计算资源，并在上面部署自己的互联网服务。laas 从本质上来说是服务器租赁并提供基础设施外包服务。如我们用的水电一样，我们不会自己去引入自来水和发电，而是直接从自来水公司和电网公司购入，并根据实际使用付费

EC2 真正对 IT 的改变是硬件的虚拟化（更细粒度的虚拟化），而 EC2 给用户带来了以下五个好处：

1. `降低劳动力成本` 减少了企业本身雇佣 IT 人员的成本
2. `降低风险` 不用再像自己运维服务机那样，担心各种意外风险，EC2 有主机损坏，再申请一个就好了
3. `降低基础设置成本` 可以按小时、周、月或者年为单位租用 EC2
4. `扩展性` 不必过早的预期基础设施采购，通过云产商很快可以获取
5. `节约时间成本` 快速获取资源开展业务实验

**PaaS**

PaaS（Platform as a Service）是构建再 IaaS 之上的一种平台服务，提供操作系统安装、监控和服务发现等功能，用户只需要部署自己的应用即可，最早的一代是 Heroku。Heroku 是商业的 PaaS，开有一个开源的 PaaS -- Cloud Foundry，用户可以基于它来构建私有 PaaS，如果同事使用公有云和私有云，能在二者之间构建一个统一的 PaaS，即 "混合云"

在 PaaS 上最为广泛使用的技术要数 `Docker` 了，因为使用容器可以很清晰的描述应用程序，并保证环境一致性。管理云上的容器，可以称为 CaaS（Container as a Service），如 GCE（Google Container Engine）。也可也基于 kubernetes、Mesos 等开源软件构建自己的 CaaS

PaaS 是对软件的一个更高的抽象层面，已经接触到应用程序的运行环境本身，开源由开发者自定义，而不必接触更底层的 OS

**Serverless**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/2ee92aa5b8b4e7aa6e5b881094ff580a.jpeg)

Serverless 的商业化产品：

1. AWS lambda
2. Google Cloud Function
3. Microsoft Azure Cloud Function
4. Huawei Function Stage
5. 其他

**第一个 Serverless 平台**

2006 年，名为 Zimki（已倒闭），这个平台提供服务端 JavaScript 应用，虽然他们没有使用 Serverless 这个名词，但是他们是第一个 "按照实际调用付费" 的平台。第一个使用 Serverless 名词的是 `iron.io`

**总结**

Serverless 实际发展已有 10 年之久，而随着以 kubernetes 为基础的云原生应用平台的兴起，serverless 再度成为焦点

### Serverless 分类
> 主要包含两个领域：BaaS（Backend as a Service）和 FaaS（Function as a Service）

**BaaS**

BaaS（Backend as a Service）后端即服务，后端服务一般是一个个的 API 调用后端或者别人已经实现好的程序逻辑，如身份验证服务 Auth0，这些 BaaS 通常会用来管理数据，还有很多公有云上提供的开源商用软件服务，如亚马逊的 RDS 可以替代我们自己部署的 MySQL，还有各种其他数据库和存储服务

**FaaS**

FaaS（Function as a Service）函数即服务，FaaS 是无服务器计算的一种形式，当前使用最广泛的 AWS 的 lambda

FaaS 本质上是一种事件驱动的由消息触发的服务，FaaS 供应商一般会集成各种同步和异步事件源，通过订阅这些事件源，可以直接触发或者定期的触发函数运行

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/5dc80e9e306067e7d1c592e2d47c768f.jpeg)

传统的服务器端软件不同是经应用程序部署到拥有操作系统的虚拟机或者容器中，一般需要长时间驻留在操作系统中运行，而 FaaS 是直接将程序部署到平台即可，当有事件到来时触发执行，执行完了就可以直接卸载


Function as a Service 全景图

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/b19cb7f133a96f1702827aae2f2e9256.jpeg)


Serverless Landscape

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/bdcde0aa21cad57cec4d139c717268ce.jpeg)


### Serverless 使用场景

* 异步的并发，组件可独立部署和扩展
* 应用突发或服务使用量不可预测（主要时为了节约成本，因为 Serverless 应用在不运行时不收费）
* 短暂、无状态的业务（因为无需提前申请资源，可以加快业务上线速度）

使用场景实例：

1. ETL
2. 机器学习及 AI 模型处理
3. 图片处理
4. IoT 传感器数据分析
5. 流处理
6. 聊天机器人

**示例**

我们以一个移动游戏应用为例，来说明什么是 Serverless 应用

一款移动游戏至少包含如下几个特性：

1. 移动端友好的用户体验
2. 用户管理和权限认证
3. 关卡、升级等游戏逻辑，游戏排行、玩家等级、任务信息等信息

传统的应用程序架构可能如下：

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/aa9eaf9b461ca39ea6b4bf01fc5f129b.jpeg)

* 一个 app 前端（IOS 或 安卓）
* 用 Java 写开发的后端服务，使用 JBoss 或 Tocmat 做 server 运行
* 使用关系型数据库存储用户数据，如 MySQL

架构优点：可以让前端十分轻便，不需要做什么应用逻辑，只是负责渲染用户界面，将请求通过 HTTP 发送给后端服务，而所有的数据操作都是由后端的 Java 程序来完成
架构缺点：维护起来十分复杂，前端开发、后端开发都需要十分专业的人员，环境的配置以及数据库的维护、应用的更新和升级

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/37377bea471dae7039d9335f7b232a8e.jpeg)


而在 Serverless 架构中，我们不需要在服务器端代码中存储任何会话状态，而是直接将它们存储在 NoSQL 上，这样将使应用程序无状态，有助于弹性扩展。前端可以直接利用 BaaS 而减少后端的编码需求，这样架构的本质上使减少了应用程序开发的人力成本，减降低了自己维护基础设施的风险，而且利用云的能力更便于扩展和快速迭代

### Serverless 优缺点

**优点**

无需提前了解服务器要求、存储容量和数据库功能等

* 降低运营成本：Serverless 使非常简单的外包解决方案。可以委托服务提供商管理服务器、数据库和应用程序甚至逻辑
* 降低开发成本
* 扩展能力：横向扩展是完全自动的、有弹性的、且由服务提供者管理，只需要支付计算能力的费用
* 绿色计算
  
**缺点**

* 状态管理：要想实现自由缩放，无状态是前提
* 延迟：应用程序中不同组件的访问延迟是一个大问题，我们可以使用专有的网阔协议、RPC 调用、数据格式来优化，或者将实例放在同一个机架内或同一个主机实例来优化以减少延迟
* 本地测试：Serverless 应用本地测试困难

**总结**

生产力决定生产关系。

云计算的概念层出不穷，其本质上还是对生产关系和生产力的配置和优化

将技术产业化，提高生产力


###  开源的 Serverless 框架

kubernetes 的蓬勃发展催生出一系列以它为基础的 Serverless 应用，目前开源的 Serverless 框架大多以 kubernetes 为基础

* dispatch - Dispatch is a framework for deploying and managing serverless style applications.
* fas-netes - Enable Kubernetes as a backend for Functions as a Service (OpenFaaS) https://github.com/alexellis/faas
* firecamp - Serverless Platform for the stateful services
* fission - Fast Serverless Functions for Kubernetes http://fission.io
* fn - The container native, cloud agnostic serverless platform. http://fnproject.io
* funktion - a CLI tool for working with funktion https://funktion.fabric8.io/
* fx - Poor man’s serverless framework based on Docker, Function as a Service with painless.
* gloo - The Function Gateway built on top of Envoy.ironfunctions - IronFunctions - the serverless microservices platform. http://iron.io
* knative - Kubernetes-based platform to build, deploy, and manage modern serverless workloads.
* knative-lambda-runtime - Running AWS Lambda Functions on Knative/Kubernetes Clusters https://triggermesh.com
* kubeless - Kubernetes Native Serverless Framework http://kubeless.io
* nuclio - High-Performance Serverless event and data processing platform.
* openfaas - OpenFaaS - Serverless Functions Made Simple for Docker & Kubernetes https://blog.alexellis.io/introducing-functions-as-a-service/
* openwhisk - Apache OpenWhisk (Incubating) is a serverless, open source cloud platform that executes functions in response to events at any scale.
* riff - riff is for functions https://projectriff.io
* serverless - Serverless Framework – Build web, mobile and IoT applications with
* serverless architectures using AWS Lambda, Azure Functions, Google CloudFunctions & more! – https://serverless.com
* spec - CloudEvents Specification https://cloudevents.io
thanos - Highly available Prometheus setup with long term storage capabilities.


## Serverless 核心技术
> Serverless 是由事件驱动的全托管计算服务

* 函数的规范定义
* 函数部署流水线
* Woekflow 设置
* 0-m-n 扩缩容
* 快速冷启动

### 函数计算

FaaS 函数定义

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/e888f323592ff741aa91171e63ef8253.jpeg)

FaaS 中函数输入、context及输出

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/e11e5e4a99003a2b7f65557b7f4f57d0.jpeg)


**函数生命周期**

描述函数生命周期及如何使用 Serverless 框架/运行时来管理函数生命周期

函数创建完成后通过函数部署流水线

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/895f6edfd8bdcfb728a850101cf65faf.jpeg)

函数的生命周期始于编写代码并提供规范和元数据， `build` 实体将获取代码和规范进行编译，如何将其转换为工件（代码二进制文件、程序包或容器、镜像）。如何将工件部署到具有控制器实体的集群上，该控制器实体负责根据事件的流量和实例上的负载来扩展函数实例的数量

`函数操作`

Serverless 框架使用以下动作（Action）控制函数的生命周期

1. 创建：创建新函数，包括其规格（spec）和代码
2. 发布：创建（禁用、启用）可以在集群上部署的函数
3. 更新别名/标签：更新版本别名
4. 执行/调用：不通过事件源调用特定版本
5. 事件源关联：将函数的特定版本与事件源连接
6. 获取：返回函数元数据和规格
7. 更新：修改函数的最新版本
8. 删除：删除函数，可以删除特定版本或所有版本的函数
9. 列表：显示函数及其元数据的列表
10. 获取统计信息：返回有关函数运行时使用情况的统计信息（调用次数、瓶颈运行事件、瓶颈延迟、失败、重试等）
11. 获取日志：返回函数生成的日志（函数创建、删除、显式错误、警告、调试消息、函数 Stdout 或 Stderr）

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/e6eaf2ccd5ee2cd89e4ef9a9cd051010.jpeg)

创建函数时需要提供函数的元数据，创建的函数可能会被编译和发布。稍后可以启动（start）、禁用（disable）和启用（enable）函数。函数部署需要能够支持以下用例：

* 事件流式传输：在这种情况下，可能总是有事件在队列中，但是可能需要通过明确的请求来暂停/恢复处理
* 热启动：函数随时都有最少实例数量，由于函数已部署并准备为事件提供服务，因此当收到"第一个"事件时可以热启动（通过传入事件在第一次调用时部署函数）

用户可以发布函数，这将创建一个新版本（latest 版本的副本），该发布的版本可能被标记/标签（tag/label）或具有别名（aliases）

用户可能想要直接执行/调用函数，统计信息可以是当前指标值或值的时间序列

**函数版本控制和别名**

一个函数可能具有多个版本，通过多版本控制区分不同环境，"latest" 版本可以进行更新和修改，可能会在每次更改时触发新的构建过程

非最新的函数版本是不可变的，一旦发布就不能更改。函数未发布前不能被删除

当一个函数有多个版本时，用户必须指定要操作的函数版本以及如何在不同版本之间划分事件流量

**事件源到函数关联**

由于事件源触发而调用函数

函数和事件源之间存在 n：m 映射。每个事件源都可以用于调用多个函数，而一个函数可以由多个事件源触发

* 创建事件源关联
* 更新事件源关联
* 列出事件源关联

**事件源**

* 事件和消息传递服务：RabbitMQ、MQTT、SES、SNS、Google Pub / Sub
* 存储服务：S3、DynamoDB、Kinesis、Cognito、Google Cloud Storage
* 断电服务：物联网、HTTP 网关、移动设备、Alexa、Google Cloud Endpoint
* 配置存储库：Git、CodeCommit
* 使用特定于语言的 SDK 的用户应用程序
* SchEnable 定时调用函数

**函数要求**

* 函数必须与不同事件类的基础实现分离
* 可以从多个事件源调用函数
* 无需为每个调用方法使用不同的函数
* 事件源可以调用多个函数
* 函数可能需要一种与基础平台服务进行持久绑定的机制，可能是跨函数调用
* 同一个应用程序中每个函数可以使用不同的语言编写
* 函数运行时应尽可能减少事件序列化和反序列化的开销


**工作流相关要求**

* 函数可以作为工作流的一部分被调用，一个函数的结果可以作为另一个函数的触发
* 可以由事件或 "and/or 事件组合" 触发函数
* 一个事件可能触发按顺序或并行执行的多个函数
* "and/or 事件组合" 可能触发顺序运行、并行运行或分支运行的 m 个函数
* 在工作流的中间，可能会收到不同的事件或函数结果，这将触发分支切换到不同的函数
* 函数的部分或全部结果需要作为输入传递给另一个函数

**函数调用类型**

1. 同步请求：如 HTTP 请求、gRPC 调用（立即响应，属于阻塞调用）
2. 异步消息队列调用（Pub/Sub：如 RabbitMQ
3. 消息/记录流：如 Kafka

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/a67e06bb4966d4b11ee19b82e815fe7c.jpeg)

`函数代码`

函数代码、依赖项和/或二进制文件可以驻留在外部存储库中，或由用户直接提供。如果代码在外部促成农户库中，则需要用户指定路径和凭证

Serverless 框架还允许用户 warch 代码存储库中的更改（如 Webhook），并在每次提交时自动构建函数镜像/二进制文件

函数可能依赖外部库或二进制文件，这些需要由用户提供，包括描述构建过程的方式（如 Dockerfile、Zip）

**函数定义**

* 唯一 ID
* 名称
* 说明
* Label（或 tag）
* 版本 ID（或 别名）
* 版本创建事件
* 上次修改事件
* 函数处理程序
* 运行时程序
* 代码 + 依赖关系或代码路径和凭证
* 环境变量
* 执行角色和 secret
* 资源（所需的 CPU、内存）
* 执行超时
* 日志记录失败
* 网络策略（VPC）
* 数据绑定

**元数据详细信息**

* 版本：如 latest、production、beta
* 环境变量
* 执行角色：该函数应在特定的用户角色身份下允许
* 资源：定义所需或最大的硬件资源
* 超时：指定函数调用在平台终止之前可以运行的最长事件
* 故障日志（死信队列）：队列或流的路径，它将存储具有适当详细信息的失败函数执行队列
* 网络策略：分配给函数的网络域和策略
* 执行语义：指定如何执行函数（如 每个事件至少执行一次、最多执行一次、恰好一次）

**数据绑定**

某些 Serverless 框架允许用户指定函数使用的输入/输出数据资源，这使得开发变得更简单，性能更高

绑定数据可以采用文件、对象、记录、消息等形式，函数说明可以包括一组数据绑定定义，每个定义都指定数据资源、凭证和使用参数

**event**

* Event Class/Kind
* 版本
* 事件 ID
* Event Source/Prigin
* 来源身份
* 内容类型
* 邮件正文
* 时间戳

**context-一组输入属性、环境变量或全局变量**

* 函数名称、版本、ARN
* 内存限制
* 请求 ID
* Cloud Region
* 环境变量
* 安全密钥/令牌
* 运行时/绑定路径
* 日志
* 数据绑定

**函数输出**

* 将值返回给调用方（如 HTTP 请求/响应示例）
* 将结果传递到工作流程中的下一个执行阶段
* 将输出写入日志

### 事件驱动

Serverless 应用是由事件驱动的

Workflow 用于表示一系列事件或函数运行结果触发状态变更并运行相应函数的流程，如图

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/7e56fbc1bb180fef422e73f8e1511c70.jpeg)

**CloudEvents**

CloudEvents 是供应商立的事件数据定义规范

以通用格式描述事件数据的规范，以跨服务、平台和系统的互操作性

事件格式指定如何使用某些编码格式序列化 CloudEvent。支持这些编码兼容 CloudEvents 的实现必须遵守相应事件格式中指定的编码规则。所有实现都必须支持 JSON 格式


**术语**

![](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/59011d103a197db635d1c8d1baefdfa8.jpeg)

示例：

```json
{
    "specversion" : "1.0-rc1",
    "type" : "com.github.pull.create",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

* Occurrence
* Event
* Producer
* Source
* Consumer
* Intermediary
* Context
* Data
* Message
* Protocel


