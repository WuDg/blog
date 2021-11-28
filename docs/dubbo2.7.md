# 一、入门

dubbo入门



## 1.1 背景

网站应用演进

互联网发展，网站应用规模不断扩大

垂直应用架构 ❎

分布式服务架构✅

流动计算架构✅

![image](https://dubbo.apache.org/imgs/user/dubbo-architecture-roadmap.jpg)



* 单一应用架构：All in One
* 垂直应用架构：将应用拆成互不相干的几个应用
* 分布式服务架构：核心业务抽取独立，多实例
* 流动计算架构：容量评估，小服务资源浪费。需要一个调度中心基于访问压力实时管理集群容量，提高集群利用率。关键是提高机器利用率的资源调度和治理中心



## 1.2 需求

dubbo解决的需求

![image](https://dubbo.apache.org/imgs/user/dubbo-service-governance.jpg)

* 服务URL配置管理困难，F5应急负载均衡器单点问题（解决：服务注册中心，冬天注册和发现服务，软负载均衡）

* 服务间依赖管理错综复杂，甚至分不清应用启动顺序（解决：自动画出应用间依赖关系图）

* 服务调用量增大，服务容量问题暴露，资源利用率问题（解决：根据服务日常调用量和响应时间作为容量规划参考指标）



## 1.3 架构

dubbo架构

![dubbo-architucture](https://dubbo.apache.org/imgs/user/dubbo-architecture.jpg)

节点角色说明

* Provider：暴露服务的服务提供方
* Consumer：调用远程服务的服务消费方
* Registry：服务注册与发现的注册中心
* Monitor：统计服务的调用次数和调用时间的监控中心
* Container：服务运行容器

调用关系说明

* 服务容器负责启动、加载、运行服务提供者
* 服务提供者在启动时，向注册中心注册自己提供的服务
* 服务消费者在启动时，向注册中心订阅自己所需的服务
* 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
* 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用
* 服务消费者和提供者，再内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心



Dubbo架构特点：连通性、健壮性、伸缩性、升级性（面向对来架构）

### 1.3.1 连通性

* 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
* 监控中心负责统计各服务调用次数，调用时间等，统计现在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
* 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
* 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
* 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
* 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
* 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
* 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

### 1.3.2 健壮性

* 监控中心宕机不影响使用，只是丢失部分采样数据
* 数据库宕机后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
* 注册中心对等集群，任意一台宕机后，将启动切换到另一台
* 注册中心全部宕机后，服务提供者和服务消费者仍能通过本地缓存通讯
* 服务提供者无状态，任意一台宕机后，不影响使用
* 服务提供者全部宕机后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

### 1.3.3 伸缩性

* 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
* 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

### 1.3.4 升级性

当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，先有分布式服务架构不会带来阻力。下图是未来可能的一种架构：

![dubbo-architucture-futures](https://dubbo.apache.org/imgs/user/dubbo-architecture-future.jpg)

节点角色说明

Deployer：自动部署服务的本地代理

Repository：仓库用于存储服务应用发布包

Scheduler：调度中心基于访问压力自动增减服务提供者

Admin：同意管理控制台

Registry：服务注册与发现的注册中心

Monitor：统计服务的调用次数和调用时间的监控中心



## 1.4 用法

dubbo简单入门

### 1.4.1 本地服务Spring配置

`local.xml`

```xml
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />
<bean id=“xxxAction” class=“com.xxx.XxxAction”>
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

### 1.4.2 远程服务Spring配置

服务提供方`remote-provider.xml`

```xml
<!-- 和本地服务一样实现远程服务 -->
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
<!-- 增加暴露远程服务配置 -->
<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> 
```

服务消费方`remote-consumer.xml`

```xml
<!-- 增加引用远程服务配置 -->
<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
<!-- 和本地服务一样使用远程服务 -->
<bean id=“xxxAction” class=“com.xxx.XxxAction”> 
    <property name=“xxxService” ref=“xxxService” />
</bean>
```

---
# 二、快速开始

`dubbo`采用全`Spring`配置方式，透明化接入应用，无API入侵，只需用`Spring`加载`dubbo`配置即可

如果不使用`Spring`，也可通过`API`方式进行调用

## 2.1 服务提供者

**定义接口服务**

`DemoService.java`

```java
package org.apache.dubbo.demo;

public interface DemoService {

    String sayHello(String name);

}
```

**在服务提供方实现接口**

`DemoServiceImpl.java`

```java
package org.apache.dubbo.demo.provider;
 
import org.apache.dubbo.demo.DemoService;
 
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

**使用Spring配置声明暴露服务**

`provider.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

**加载Spring配置**

`Provider.java`

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Provider {
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
        context.start();
        System.in.read(); // 按任意键退出
    }
}
```

## 2.2 服务消费者

**通过Spring配置引用远程服务**

`consumer.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
</beans>
```

**加载Spring配置，并调用远程服务**

`Consumer.java`

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.apache.dubbo.demo.DemoService;
 
public class Consumer {
    public static void main(String[] args) throws Exception {
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF/spring/dubbo-demo-consumer.xml"});
        context.start();
        DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法
        System.out.println( hello ); // 显示调用结果
    }
}
```

1. `api`接口服务单独打包，在服务提供方和消费方共享
2. 对服务消费方隐藏实现
3. `DemoService`也可以使用`IoC`注入


---

# 三、依赖

dubbo依赖介绍

## 3.1 必须依赖

JDK 1.6+



## 3.2 缺省依赖

```fallback
[INFO] +- com.alibaba:dubbo:jar:2.5.9-SNAPSHOT:compile
[INFO] |  +- org.springframework:spring-context:jar:4.3.10.RELEASE:compile
[INFO] |  +- org.javassist:javassist:jar:3.21.0-GA:compile
[INFO] |  \- org.jboss.netty:netty:jar:3.2.5.Final:compile
```

## 3.3 可选依赖

使用相应实现策略时自行加入

- netty-all 4.0.35.Final
- mina: 1.1.7
- grizzly: 2.1.4
- httpclient: 4.5.3
- hessian_lite: 3.2.1-fixed
- fastjson: 1.2.31
- zookeeper: 3.4.9
- jedis: 2.9.0
- xmemcached: 1.3.6
- hessian: 4.0.38
- jetty: 6.1.26
- hibernate-validator: 5.4.1.Final
- zkclient: 0.2
- curator: 2.12.0
- cxf: 3.0.14
- thrift: 0.8.0
- servlet: 3.0 [5](https://dubbo.apache.org/zh/docs/v2.7/user/dependencies/#fn:5)
- validation-api: 1.1.0.GA [5](https://dubbo.apache.org/zh/docs/v2.7/user/dependencies/#fn:5)
- jcache: 1.0.0 [5](https://dubbo.apache.org/zh/docs/v2.7/user/dependencies/#fn:5)
- javax.el: 3.0.1-b08 [5](https://dubbo.apache.org/zh/docs/v2.7/user/dependencies/#fn:5)
- kryo: 4.0.1
- kryo-serializers: 0.42
- fst: 2.48-jdk-6
- resteasy: 3.0.19.Final
- tomcat-embed-core: 8.0.11
- slf4j: 1.7.25
- log4j: 1.2.16



---

# 四、成熟度

介绍dubbo各功能、策略的成熟度

## 4.1 功能成熟度

| Feature    | Maturity | Strength                                                     | Problem                            | Advise         | User     |
| ---------- | -------- | ------------------------------------------------------------ | ---------------------------------- | -------------- | -------- |
| 并发控制   | Tested   | 并发控制                                                     |                                    | 试用           |          |
| 连接控制   | Tested   | 连接数控制                                                   |                                    | 试用           |          |
| 直连提供者 | Tested   | 点对点直连服务提供方，用于测试                               |                                    | 测试环境使用   | Alibaba  |
| 分组聚合   | Tested   | 分组聚合返回值，用于菜单聚合等服务                           | 特殊场景使用                       | 可用于生产环境 |          |
| 参数验证   | Tested   | 参数验证，JSR303验证框架集成                                 | 对性能有影响                       | 试用           | LaiWang  |
| 结果缓存   | Tested   | 结果缓存，用于加速请求                                       |                                    | 试用           |          |
| 泛化引用   | Stable   | 泛化调用，无需业务接口类进行远程调用，用于测试平台，开放网关桥接等 |                                    | 可用于生产环境 | Alibaba  |
| 泛化实现   | Stable   | 泛化实现，无需业务接口类实现任意接口，用于Mock平台           |                                    | 可用于生产环境 | Alibaba  |
| 回声测试   | Tested   | 回声测试                                                     |                                    | 试用           |          |
| 隐式传参   | Stable   | 附加参数                                                     |                                    | 可用于生产环境 |          |
| 异步调用   | Tested   | 不可靠异步调用                                               |                                    | 试用           |          |
| 本地调用   | Tested   | 本地调用                                                     |                                    | 试用           |          |
| 参数回调   | Tested   | 参数回调                                                     | 特殊场景使用                       | 试用           | Registry |
| 事件通知   | Tested   | 事件通知，在远程调用执行前后触发                             |                                    | 试用           |          |
| 本地存根   | Stable   | 在客户端执行部分逻辑                                         |                                    | 可用于生产环境 | Alibaba  |
| 本地伪装   | Stable   | 伪造返回结果，可在失败时执行，或直接执行，用于服务降级       | 需注册中心支持                     | 可用于生产环境 | Alibaba  |
| 延迟暴露   | Stable   | 延迟暴露服务，用于等待应用加载warmup数据，或等待spring加载完成 |                                    | 可用于生产环境 | Alibaba  |
| 延迟连接   | Tested   | 延迟建立连接，调用时建立                                     |                                    | 试用           | Registry |
| 粘滞连接   | Tested   | 粘滞连接，总是向同一个提供方发起请求，除非此提供方挂掉，再切换到另一台 |                                    | 试用           | Registry |
| 令牌验证   | Tested   | 令牌验证，用于服务授权                                       | 需注册中心支持                     | 试用           |          |
| 路由规则   | Tested   | 动态决定调用关系                                             | 需注册中心支持                     | 试用           |          |
| 配置规则   | Tested   | 动态下发配置，实现功能的开关                                 | 需注册中心支持                     | 试用           |          |
| 访问日志   | Tested   | 访问日志，用于记录调用信息                                   | 本地存储，影响性能，受磁盘大小限制 | 试用           |          |
| 分布式事务 | Research | JTA/XA三阶段提交事务                                         | 不稳定                             | 不可用         |          |

## 4.2 策略成熟度

| Feature           | Maturity | Strength                                                     | Problem                                    | Advise                   | User |
| ----------------- | -------- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------ | ---- |
| Zookeeper注册中心 | Stable   | 支持基于网络的集群方式，有广泛周边开源产品，建议使用dubbo-2.3.3以上版本（推荐使用） | 依赖于Zookeeper的稳定性                    | 可用于生产环境           |      |
| Redis注册中心     | Stable   | 支持基于客户端双写的集群方式，性能高                         | 要求服务器时间同步，用于检查心跳过期脏数据 | 可用于生产环境           |      |
| Multicast注册中心 | Tested   | 去中心化，不需要安装注册中心                                 | 依赖于网络拓扑和路由，跨机房有风险         | 小规模应用或开发测试环境 |      |
| Simple注册中心    | Tested   | Dogfooding，注册中心本身也是一个标准的RPC服务                | 没有集群支持，可能单点故障                 | 试用                     |      |

| Feature        | Maturity | Strength               | Problem                                           | Advise         | User |
| -------------- | -------- | ---------------------- | ------------------------------------------------- | -------------- | ---- |
| Simple监控中心 | Stable   | 支持JFreeChart统计报表 | 没有集群支持，可能单点故障，但故障后不影响RPC运行 | 可用于生产环境 |      |

| Feature     | Maturity | Strength                                                     | Problem                               | Advise         | User    |
| ----------- | -------- | ------------------------------------------------------------ | ------------------------------------- | -------------- | ------- |
| Dubbo协议   | Stable   | 采用NIO复用单一长连接，并使用线程池并发处理请求，减少握手和加大并发效率，性能较好（推荐使用） | 在大文件传输时，单一连接会成为瓶颈    | 可用于生产环境 | Alibaba |
| Rmi协议     | Stable   | 可与原生RMI互操作，基于TCP协议                               | 偶尔会连接失败，需重建Stub            | 可用于生产环境 | Alibaba |
| Hessian协议 | Stable   | 可与原生Hessian互操作，基于HTTP协议                          | 需hessian.jar支持，http短连接的开销大 | 可用于生产环境 |         |

| Feature             | Maturity | Strength                              | Problem                                          | Advise         | User    |
| ------------------- | -------- | ------------------------------------- | ------------------------------------------------ | -------------- | ------- |
| Netty Transporter   | Stable   | JBoss的NIO框架，性能较好（推荐使用）  | 一次请求派发两种事件，需屏蔽无用事件             | 可用于生产环境 | Alibaba |
| Mina Transporter    | Stable   | 老牌NIO框架，稳定                     | 待发送消息队列派发不及时，大压力下，会出现FullGC | 可用于生产环境 | Alibaba |
| Grizzly Transporter | Tested   | Sun的NIO框架，应用于GlassFish服务器中 | 线程池不可扩展，Filter不能拦截下一Filter         | 试用           |         |

| Feature               | Maturity | Strength                                             | Problem                                                      | Advise         | User    |
| --------------------- | -------- | ---------------------------------------------------- | ------------------------------------------------------------ | -------------- | ------- |
| Hessian Serialization | Stable   | 性能较好，多语言支持（推荐使用）                     | Hessian的各版本兼容性不好，可能和应用使用的Hessian冲突，Dubbo内嵌了hessian3.2.1的源码 | 可用于生产环境 | Alibaba |
| Dubbo Serialization   | Tested   | 通过不传送POJO的类元信息，在大量POJO传输时，性能较好 | 当参数对象增加字段时，需外部文件声明                         | 试用           |         |
| Json Serialization    | Tested   | 纯文本，可跨语言解析，缺省采用FastJson解析           | 性能较差                                                     | 试用           |         |
| Java Serialization    | Stable   | Java原生支持                                         | 性能较差                                                     | 可用于生产环境 |         |

| Feature                | Maturity | Strength                                       | Problem                                                      | Advise         | User    |
| ---------------------- | -------- | ---------------------------------------------- | ------------------------------------------------------------ | -------------- | ------- |
| Javassist ProxyFactory | Stable   | 通过字节码生成代替反射，性能比较好（推荐使用） | 依赖于javassist.jar包，占用JVM的Perm内存，Perm可能要设大一些：java -XX:PermSize=128m | 可用于生产环境 | Alibaba |
| Jdk ProxyFactory       | Stable   | JDK原生支持                                    | 性能较差                                                     | 可用于生产环境 |         |

| Feature           | Maturity | Strength                                                     | Problem                                | Advise         | User     |
| ----------------- | -------- | ------------------------------------------------------------ | -------------------------------------- | -------------- | -------- |
| Failover Cluster  | Stable   | 失败自动切换，当出现失败，重试其它服务器，通常用于读操作（推荐使用） | 重试会带来更长延迟                     | 可用于生产环境 | Alibaba  |
| Failfast Cluster  | Stable   | 快速失败，只发起一次调用，失败立即报错,通常用于非幂等性的写操作 | 如果有机器正在重启，可能会出现调用失败 | 可用于生产环境 | Alibaba  |
| Failsafe Cluster  | Stable   | 失败安全，出现异常时，直接忽略，通常用于写入审计日志等操作   | 调用信息丢失                           | 可用于生产环境 | Monitor  |
| Failback Cluster  | Tested   | 失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作 | 不可靠，重启丢失                       | 可用于生产环境 | Registry |
| Forking Cluster   | Tested   | 并行调用多个服务器，只要一个成功即返回，通常用于实时性要求较高的读操作 | 需要浪费更多服务资源                   | 可用于生产环境 |          |
| Broadcast Cluster | Tested   | 广播调用所有提供者，逐个调用，任意一台报错则报错，通常用于更新提供方本地状态 | 速度慢，任意一台报错则报错             | 可用于生产环境 |          |

| Feature                    | Maturity | Strength                                                     | Problem                                                      | Advise         | User    |
| -------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------- | ------- |
| Random LoadBalance         | Stable   | 随机，按权重设置随机概率（推荐使用）                         | 在一个截面上碰撞的概率高，重试时，可能出现瞬间压力不均       | 可用于生产环境 | Alibaba |
| RoundRobin LoadBalance     | Stable   | 轮询，按公约后的权重设置轮询比率                             | 存在慢的机器累积请求问题，极端情况可能产生雪崩               | 可用于生产环境 |         |
| LeastActive LoadBalance    | Stable   | 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差，使慢的机器收到更少请求 | 不支持权重，在容量规划时，不能通过权重把压力导向一台机器压测容量 | 可用于生产环境 |         |
| ConsistentHash LoadBalance | Stable   | 一致性Hash，相同参数的请求总是发到同一提供者，当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动 | 压力分摊不均                                                 | 可用于生产环境 |         |

| Feature      | Maturity | Strength                               | Problem                                      | Advise         | User    |
| ------------ | -------- | -------------------------------------- | -------------------------------------------- | -------------- | ------- |
| 条件路由规则 | Stable   | 基于条件表达式的路由规则，功能简单易用 | 有些复杂多分支条件情况，规则很难描述         | 可用于生产环境 | Alibaba |
| 脚本路由规则 | Tested   | 基于脚本引擎的路由规则，功能强大       | 没有运行沙箱，脚本能力过于强大，可能成为后门 | 试用           |         |

| Feature          | Maturity | Strength                                                     | Problem                                  | Advise         | User    |
| ---------------- | -------- | ------------------------------------------------------------ | ---------------------------------------- | -------------- | ------- |
| Spring Container | Stable   | 自动加载META-INF/spring目录下的所有Spring配置                |                                          | 可用于生产环境 | Alibaba |
| Jetty Container  | Stable   | 启动一个内嵌Jetty，用于汇报状态                              | 大量访问页面时，会影响服务器的线程和内存 | 可用于生产环境 | Alibaba |
| Log4j Container  | Stable   | 自动配置log4j的配置，在多进程启动时，自动给日志文件按进程分目录 | 用户不能控制log4j的配置，不灵活          | 可用于生产环境 | Alibaba |


---

# 五、Dubbo配置

以不同方式配置dubbo应用

## 5.1 XML配置

以XML配置方式来配置dubbo

[参考手册](https://dubbo.apache.org/zh/docs/v2.7/user/references/xml/)

`provider.xml`

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

`consumer.xml`

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

所有标志都支持自定义参数，用于不同扩展点实现特殊配置，如

```xml
<dubbo:protocol name="jms">
    <dubbo:parameter key="queue" value="your_queue" />
</dubbo:protocol>
```

或

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">  
    <dubbo:protocol name="jms" p:queue="your_queue" />  
</beans>
```

**配置之间的关系**

![dubbo-config](https://dubbo.apache.org/imgs/user/dubbo-config.jpg)

| 标签        | 用途         | 解释                                                         |
| ----------- | ------------ | ------------------------------------------------------------ |
| service     | 服务配置     | 用于暴露服务，定义服务元信息，一个服务可以用多协议暴露，也可注册到多个注册中心 |
| reference   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| protocol    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
| application | 应用配置     | 用于配置当前应用信息，包括服务提供者和消费者                 |
| module      | 模块配置     | 用于配置当前模块信息                                         |
| registry    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| monitor     | 监控中心配置 | 用于配置连接监控中心相关信息                                 |
| provider    | 提供方配置   | 当ProtocolConfig和ServiceConfig某属性没有配置时，采用此缺省值 |
| consumer    | 消费方配置   | 当ReferenceConfig某属性没有配置时，采用此缺省值              |
| method      | 方法配置     | 用于ServiceConfig和ReferenceConfig指定方法级的配置信息       |
| argument    | 参数配置     | 用于指定方法参数配置                                         |



### 5.1.1 不同粒度配置覆盖关系

以timeout为例，其他retries，loadbalance，actives类似

* 方法级优先，接口级其次，全局配置最低
* 如果级别一样，则消费方优先，提供方其次

服务提供方配置，通过URL经过注册中心传递给消费者

![dubbo-config-override](https://dubbo.apache.org/imgs/user/dubbo-config-override.jpg)





## 5.2 动态配置中心

配置中心职责

* 外部化配置。如启动配置的集中式存储
* 服务治理。服务治理规则的存储与通知

启用动态配置，以Zookeeper为例

`xml`

```xml
<dubbo:config-center address="zookeeper://127.0.0.1:2181"/>
```

或者`properties`

```fallback
dubbo.config-center.address=zookeeper://127.0.0.1:2181
```

或者`Java代码`

```java
ConfigCenterConfig configCenter = new ConfigCenterConfig();
configCenter.setAddress("zookeeper://127.0.0.1:2181");
```



### 5.2.1 外部化配置

目的之一：实现配置的集中式管理

业界成熟的专业配置系统：`Apollo`，`Nacos`等

外部化配置和本地配置在内容和格式上无区别

适合将一些公共配置，如注册中心、元数据中心配置等抽取做集中管理

```fallback
# 将注册中心地址、元数据中心地址等配置集中管理，可以做到统一环境、减少开发侧感知。
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.registry.simplified=true

dubbo.metadata-report.address=zookeeper://127.0.0.1:2181

dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

dubbo.application.qos.port=33333
```

**优先级**

外部化配置 > 本地配置，覆盖本地配置值

[各配置形式间覆盖关系](https://dubbo.apache.org/zh/docs/v2.7/user/configuration/configuration-load-process/)

调整配置中心优先级

```fallback
-Ddubbo.config-center.highest-priority=false
```

**作用域**

外部化配置有全局和应用两个级别

全局配置是所有应用共享的，应用级配置是每个应用自己维护且只对自身可见

### 5.2.2 Zookeeper

 ```xml
<dubbo:config-center address="zookeeper://127.0.0.1:2181"/>
 ```

默认所有的配置都存储在 `/dubbo/config` 节点，具体结构图如下

![zk-configcenter.jpg](https://dubbo.apache.org/imgs/user/zk-configcenter.jpg)

* namespace，用于不同配置的环境隔离
* config，Dubbo约定的固定节点，不可更改，所有配置和服务治理规则都存储在此节点下
* dubbo/application，分别用来隔离全局配置、应用级别配置。dubbo是默认group值，application对应应用名
* dubbo.properties，此节点的node value存储具体配置内容

### 5.2.3 Apollo

```xml
<dubbo:config-center protocol="apollo" address="127.0.0.1:2181"/>
```

### 5.2.4 自己加载外部化配置

dubbo对配置中心的支持，本质上就是把远程的配置拉取到本地，然后和本地的配置做一次融合

```java
// 应用自行加载配置
Map<String, String> dubboConfigurations = new HashMap<>();
dubboConfigurations.put("dubbo.registry.address", "zookeeper://127.0.0.1:2181");
dubboConfigurations.put("dubbo.registry.simplified", "true");

//将组织好的配置塞给Dubbo框架
ConfigCenterConfig configCenter = new ConfigCenterConfig();
configCenter.setExternalConfig(dubboConfigurations);
```

### 5.2.5 服务治理

`Zookeeper`

![zk-configcenter-governance](https://dubbo.apache.org/imgs/user/zk-configcenter-governance.jpg)

* namespace，用于不同配置的环境隔离
* config，dubbo约定的固定节点，不可更改，所有配置和服务治理规则都存在此节点下
* dubbo，所有服务治理规则都是全局性的，dubbo为默认节点
* configurators/tag-router/condition-router，不同的服务治理规则类型，node value存储具体规则内容

`Apollo`

所有的服务治理规则都是全局性的，默认从公共命名空间`dubbo`读取和订阅







## 5.3 属性配置

以属性配置的方式来来配置你的`dubbo`应用

`dubbo.properties`

`dubbo`自动加载`classpath`根目录下的`dubbo.properties`，也可使用JVM参数指定路径

```
-Ddubbo.properties.file=xxx.properties
```

### 5.3.1 映射规则

`xml`和`properties`配置映射

`dubbo.application.name=foo` 相当于 `<dubbo:application name="foo" />`

如果同一tag有多个，则使用`id`进行区分

`dubbo.proptocol.rmi.port=1099` 相当于 `<dubbo:protocol id="rmi" name="rmi" port="1099" />`

### 5.3.2 重写与优先级

![properties-override](https://dubbo.apache.org/imgs/user/dubbo-properties-override.jpg)

JVM -D > XML -> Properties



## 5.4 自动加载环境变量

在dubbo自动加载环境变量

`dubbo.labels`

```fallback
# JVM
-Ddubbo.labels = "tag1=value1; tag2=value2"
# 环境变量
DUBBO_LABELS = "tag1=value1; tag2=value2"
```

`dubbo.env.keys`

```fallback
# JVM
-Ddubbo.env.keys = "DUBBO_TAG1, DUBBO_TAG2"
# 环境变量
DUBBO_ENV_KEYS = "DUBBO_TAG1, DUBBO_TAG2"
```



## 5.5 API配置

以API配置的方式来配置Duboo应用

[参考手册](https://dubbo.apache.org/zh/docs/v2.7/user/references/xml/)

如 `ApplicationConfig.setName("xxx") `对应 `<dubbo:application name="xxx" />`

### 5.5.1 服务提供者

```java
import org.apache.dubbo.rpc.config.ApplicationConfig;
import org.apache.dubbo.rpc.config.RegistryConfig;
import org.apache.dubbo.rpc.config.ProviderConfig;
import org.apache.dubbo.rpc.config.ServiceConfig;
import com.xxx.XxxService;
import com.xxx.XxxServiceImpl;
 
// 服务实现
XxxService xxxService = new XxxServiceImpl();
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("xxx");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 服务提供者协议配置
ProtocolConfig protocol = new ProtocolConfig();
protocol.setName("dubbo");
protocol.setPort(12345);
protocol.setThreads(200);
 
// 注意：ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
 
// 服务提供者暴露服务配置
ServiceConfig<XxxService> service = new ServiceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接，请自行缓存，否则可能造成内存和连接泄漏
service.setApplication(application);
service.setRegistry(registry); // 多个注册中心可以用setRegistries()
service.setProtocol(protocol); // 多个协议可以用setProtocols()
service.setInterface(XxxService.class);
service.setRef(xxxService);
service.setVersion("1.0.0");
 
// 暴露及注册服务
service.export();
```

### 5.5.2 服务消费者

```java
import org.apache.dubbo.rpc.config.ApplicationConfig;
import org.apache.dubbo.rpc.config.RegistryConfig;
import org.apache.dubbo.rpc.config.ConsumerConfig;
import org.apache.dubbo.rpc.config.ReferenceConfig;
import com.xxx.XxxService;
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("yyy");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 注意：ReferenceConfig为重对象，内部封装了与注册中心的连接，以及与服务提供方的连接
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
reference.setApplication(application);
reference.setRegistry(registry); // 多个注册中心可以用setRegistries()
reference.setInterface(XxxService.class);
reference.setVersion("1.0.0");
 
// 和本地bean一样使用xxxService
XxxService xxxService = reference.get(); // 注意：此代理对象内部封装了所有通讯细节，对象较重，请缓存复用
```



### 5.5.3 特殊场景

**方法级设置**

```java
...
 
// 方法级配置
List<MethodConfig> methods = new ArrayList<MethodConfig>();
MethodConfig method = new MethodConfig();
method.setName("createXxx");
method.setTimeout(10000);
method.setRetries(0);
methods.add(method);
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
...
reference.setMethods(methods); // 设置方法级配置
 
...
```

**点对点直连**

```java
...
 
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
// 如果点对点直连，可以用reference.setUrl()指定目标地址，设置url后将绕过注册中心，
// 其中，协议对应provider.setProtocol()的值，端口对应provider.setPort()的值，
// 路径对应service.setPath()的值，如果未设置path，缺省path为接口名
reference.setUrl("dubbo://10.20.130.230:20880/com.xxx.XxxService"); 
 
...
```

`API使用范围说明：仅使用于OpenAPI，ESB，Test，Mock等系统集成，普通服务提供方或消费方，请使用XML配置`



## 5.6 注解配置

2021-1-15 14:15:08



## 5.7 配置加载流程