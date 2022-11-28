# 【翟永超】spring-cloud微服务实战

DevOps、微服务、云原生架构、服务治理、CI/CD自动化流程和容器化部署

P442

Spring Cloud生态各个组件
* 服务治理组件 Eureka
* 客户端负载均衡组件 Ribbon
* 服务容错保护组件 Hystrix
* 声明式服务调用组件 Feign
* API 网关治理组件 Zuul
* 分布式配置中心组件 Config
* 消息总线组件 Bus
* 消息驱动组件 Stream
* 分布式服务跟踪组件 Sleuth

## 基础知识

### 什么是微服务

微服务是系统架构设计上的一种风格，主旨是将原本独立的一个系统拆分为多个小型服务，这些小型服务都在各自的进程中运行，服务之间通过基于 HTTP 的 Restful API 进行通信。

被拆分的每一个小型微服务都围绕着系统中的某一项或一些耦合度较高的业务功能进行构建，并且每个服务都维护着自身的数据存储、业务开发、自动化测试案例以及独立部署机制。

由于有了轻量级的通信写作基础，所以这些微服务可以使用不同的语言来编写。

### 与单体系统的区别

随着业务需求拓展，单体应用变得越来越臃肿。往往我们修改了一个很小的功能，为了部署上线会影响其他功能的运行。并且，单体应用中的这些功能模块的使用场景、并发量、消耗的资源类型都各有不同，对于资源的利用又互相影响，这使得我们对各模块的系统容量很难给出较为准确的评估。所以，单体系统在初期虽然可以非常方便的进行开发和使用，但是随着系统的发展，维护成本会变得越来越大，且难以控制。

微服务中将不同功能模块拆分多个不同的服务，这些服务都能够独立部署和扩展。由于每个服务都运行在自己的进程中，在部署上有稳固的边界，这样每个微服务的更新都不会影响其他服务的运行。同时，由于是独立部署的，我们可以更准确的为每个服务评估性能容量，通过配合服务将的协作流程也可以更容易地发现系统的瓶颈位置，以及给出较为准确的系统级性能容量评估。

### 如何实施微服务

* 运维的新挑战：微服务架构中，运维人员需要维护的进程数量会大大增加，有条不紊地将这些进程编排和组织起来不是一件容易的事
* 接口的一致性：服务间接口调用需要同步更新
* 分布式的复杂性：带来了网络延迟、分布式事务、异步消息等问题

#### 服务组件化

在微服务架构中，需要我们对服务进行组件化分解。服务，是一个进行外的组件，它通过 HTTP 等通信协议进行协作，而不是像组件那样以嵌入的方式协作工作。每一个服务都独立开发、部署，可以有效避免一个服务的修改引起整个系统的重新部署。

#### 按业务组织团队

如DBA团队，运维团队，后端团队，前端团队，设计师团队

#### 服务间调用方式

* 使用 HTTP 的 RESTful API 或轻量级的消息发送协议，实现信息传递与服务调用的触发。
* 通过在轻量级消息总线上传递消息，类似 RabbitMQ 等一些提供可靠异步交换的中间件。

#### 去中心化治理

#### 去中心化管理数据

把原来存储在 MySQL 中的表拆分后，存储到多个不同的 MySQL 实例中，也可以将一些具有特殊结构或业务特性的数据存储到一些其他技术的数据库实例中，如把日志信息存储到 MongoDB 或把用户登录信息存储到 Redis 中

#### 基础设施自动化

持续交付平台

* 自动化测试
* 自动化部署

#### 容错设计

微服务架构中，为了避免某个服务结点宕机而引发一系列故障引入服务降级等措施

通常，我们在每个服务中实现监控和日志记录的组件，比如路由器状态、吞吐量、网络延迟等关键数据的仪表盘等。

### 为什么选择 Spring Cloud

* 服务治理：Dubbo、DubboX、Eureka、Consul
* 分布式配置管理：Disconf、Archaius、QConf、Config、Diamond
* 批量任务：Elastic-Job、Azkaban、Task
* 服务跟踪：Hydra、Sleuth、Zipkin

### Spring Cloud 简介

Spring Cloud 是一个基于 Spring Boot 实现的微服务架构开发工具。它为微服务架构中设计的配置管理、服务治理、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

* Spring Cloud Config：配置管理工具，支持使用 Git 存储配置内容，可以使用它实现应用配置的外部化存储，并支持客户端配置信息刷新、加密/解密配置内容等
* Spring Cloud Netflix：核心组件，对多个 Netflix OSS 开源套件进行整合
  * Eureka：服务治理组件，包含服务注册中心、服务中注册与发现机制的实现
  * Hystrix：容错管理组件，实现断路器模式，帮助服务依赖中出现延迟和为故障提供强大的容错能力
  * Ribbon：客户端负载均衡的服务调用组件
  * Feign：基于 Ribbon 和 Hystrix 的声明式服务调用组件
  * Zuul：网关组件，提供智能路由、访问过滤等功能
  * Archaius：外部化配置组件
* Spring Cloud Bus：事件、消息总线，用于传播集群中的状态变化或事件，以触发后续的处理，比如用来动态刷新配置等
* Spring Cloud Cluster：针对 Zookeeper、Redis、Hazelcase、Consul的选举算法和通用状态模式的实现
* Spring Cloud Cloudfoundry：与 Pivotal Cloudfoundry 的整合支持
* Spring Cloud Consul：服务发现与配置管理工具
* Spring Cloud Stream：通过 Redis、Rabbit 或 Kafka 实现的消费微服务，可以通过简单的声明式模式来发送与接受消息
* Spring Cloud Security：安全工具包，提供在 Zuul 代理中对 OAuth2 客户端请求的中继器
* Spring Cloud Zookeeper：基于 Zookeeper 的服务发现与配置管理组件。
* Spring Cloud Starters：Spring Cloud 的基础组件，它是基于 Spring Boot 风格项目的基础依赖模块
* Spring Cloud CLI：用于在 Groovy 中快速创建 Spring Cloud 应用的 Spring Boot CLI 插件

### 版本说明

如 Angel.SR6、Brixton.SR5 等

#### 版本名与版本号

由于 Spring Cloud 是一个拥有多个子项目的大型综合项目，1.x.x 的版本明明规则会和其子项目的版本号混淆，因此没有采用版本号的方式，而是通过命名的方式。

![spring-cloud-version-list](http://wudiguang.top/images/springcloud/spring-cloud-version-list.png)

## 微服务构建：Spring Boot

### 框架简介

- 自动化配置：讲话 Spring 原有样板化配置
- 定义一系列 Starter POMs：方便整合各项功能
- 嵌入Web容器：如 Tomcat、Jetty 等

#### 快速入门

略

#### 实现 RESTful API

#### 编写单元测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(class = HelloApplication.class)
@WebAppConfiguration
public class HelloApplicationTests{
    // 略
}
```

#### 自定义参数

```properties
book.name=SpringCloudInAction
book.author=ZhaiYongchao
```

```java
@Component
public class Book {
    @Value("${book.name}")
    private String name;

    @Value("${book.author}")
    private String author;
}
```

#### 使用随机数
> 使用 ${random} 来生成随机值

#### 命令行参数

```shell
java -jar xxx.jar --server.port=8080
```

#### 多环境配置

多环境配置文件名需要满足：application-{profile}
* application-dev.properties 开发环境
* application-test.properties 测试环境
* application-prod.properties 生产环境

通过设置 spring.profile.active 来指定环境

#### 加载顺序

1. 在命令行中传入的参数
2. SPRING_APPLICATION_JSON 中的属性 系统环境变量中
3. java:comp/env 中的 JNDI 属性
4. Java 的系统属性，可以通过 System.getProperties() 获得
5. 操作系统的环境变量
6. 通过 random.* 配置的随机属性
7. 位于当前应用 jar 包之外，针对不同 {profile} 环境的配置文件内容，如 application-{profile}.properties
8. 位于当前应用 jar 包之内，针对不同 {profile} 环境的配置文件内容，如 application-{profile}.properties
9. 位于当前应用 jar 包之外的 application.properties
10. 位于当前应用 jar 包之内的 application.properties
11. 在 @Configuration 注解修改的类中，通过@PropertySource 注解定义的属性
12. 应用默认属性，使用 SpringApplication.setDefaultProperties 定义的内容

### 监控与管理

不断收集各个微服务应用的各项指标情况，并根据这些基础指标信息来制定监控和预警规则，更进一步甚至做到一些自动化的运维操作

#### 初识 actuator

Spring Boot 提供 `spring-boot-starter-actuator` 依赖

增加该依赖之后，重启应用。此时可以看到控制台中输出服务暴漏的端点，访问这些端点即可看到对应得指标内容

#### 原生端点

* 应用配置类：获取应用程序中加载的应用配置、环境变量、自动化配置报告等与Spring Boot 应用密切相关的配置类信息
* 度量指标类：获取应用程序运行过程中用于监控的度量指标，如内存信息、线程池信息、HTTP 请求统计等
* 操作控制类：提供了对应用的关闭等操作类功能

#### 应用配置类

略

## 服务治理：Spring Cloud Eureka

P58