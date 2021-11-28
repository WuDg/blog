# 核心技术
---

[文档地址](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html)

这部分参考文档覆盖了`Spring`框架绝对不可或缺的所有技术。

其中最重要的就是`Spring`框架的控制反转(IoC Inversion of Control)容器。之后就是面向切面编程（AOP Aspect-Oriented Programming）技术。`Spring`框架拥有自己的`AOP`框架，它在概念上更容易理解，并且成功解决了`Java`企业编程中 `80%` 的`AOP`需求。

`Spring`还提供了和`AspectJ`（目前Java企业领域中最成熟的`AOP`实现）集成的内容



## 1. IoC 容器

### 1.1. 介绍 Spring IoC 容器和 Beans

本章介绍了Spring框架中控制反转原理的实现。

Ioc 也被成为依赖注入（DI dependency injection），是一个对象定义其依赖（一同工作的其他对象）关系的过程。只有通过构造函数参数、工厂方法参数或者对象实例被构造或从工厂方法返回后设置的属性。

然后容器在创建bean时注入这些依赖项，这个过程从根本上说是bean创建的逆过程，因此被称为控制反转。

`org.springframework.beans` 和 `org.springframework.context` 包是`Spring`框架`IoC`容器的基础。

`BeanFactory`接口提供了能够管理任何类型对象的高级配置机制。`ApplicationContext`是`BeanFactory`的子接口。增加了以下功能

* 更容易整合`Spring`的`AOP`特性
* 消息资源处理（对于使用国际化）
* 事件发布
* 应用层特殊上下文，如`web`应用的 `WebApplicationContext`

总的来说，`BeanFactory`提供了配置框架和基础功能，而`ApplicationContext`添加了更多特定于企业的功能

在`Spring`中，构成应用程序主干并由`Spring IoC`容器管理的对象称为`bean`。

`bean`是由`Spring IoC`容器`实例化、组装和管理`的对象。`bean`及其依赖关系反映在容器使用的`配置元数据`中

## 1.2. 容器概览

`ApplicationContext`接口代表`Spring IoC`容器和它负责实例化、配置和组装的bean。容器通过读取配置元数据获取关于要实例化、配置和组装哪些对象的指令。配置元数据用`XML、Java注解或Java代码`表示，它允许表示组成应用程序的对象以及这些对象之间丰富的相互依赖关系。

`Spring`提供了`ApplicationContext`接口的几个实现。在独立应用程序中，通常创建 `ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例。

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

图1 `Spring IoC`容器

### 1.2.1. 配置元数据

作为应用程序开发人员，告诉`Spring`容器实例化、配置和组装应用程序中的对象

配置元数据通常以简单直观的XML格式提供

其他形式的配置元数据

* [基于注解配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config) `Spring2.5`引入了对基于注解的配置元数据的支持
* [基于Java代码配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) `Spring3.0`开始，Spring `JavaConfig`项目提供的许多特性都成为了`Spring`核心框架的一部分。`@Configuration,@Bean,@Import,@DependsOn`注解

`Spring`配置的`IoC`容器管理至少一个`bean`

* 基于XML的配置元数据将bean配置为顶级<beans></beans>中的<bean></bean>元素

* 基于Java配置的通常在`@Configuration`类中使用`@Bean`注解



下面例子中展示基于`XML`配置元数据的基本结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- 这个bean的协作者和配置在此 -->
        <!-- id：表示单个bean定义的字符串，class：定义bean的类型，使用类完全限定类名 -->
    </bean>

    <bean id="..." class="...">
    </bean>

</beans>
```

### 1.2.2 初始化容器

`ApplicationContext`构造函数指定资源路径（本地文件系统、`Java`类路径等）来加载配置元数据

`services.xml` 和 `daos.xml` 都是`resources`目录下的配置文件

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

下面的示例显示了服务层对象（`services.xml`）的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="com.wdg.spring.service.impl.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
    </bean>

</beans>
```

下面的示例显示了数据访问对象（`daos.xml`）文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="com.wdg.spring.dao.AccountDao">
    </bean>

    <bean id="itemDao" class="com.wdg.spring.dao.ItemDao">
    </bean>

</beans>
```

**组合基于xml配置元数据**

```xml
<beans>
    <import resource="daos.xml"/>
    <import resource="services.xml"/>
    <bean id="bean1" class="com.wdg.spring.bean.Bean1"></bean>
    <bean id="bean2" class="com.wdg.spring.bean.Bean2"></bean>

</beans>
```

### 1.2.3. 使用容器

`ApplicationContext`允许读取bean定义并访问它们

```java
public class LoadBeanConfig {
    private static final Gson GSON = new Gson();
    private static final Logger logger = Logger.getLogger(LoadBeanConfig.class);
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
        PetStoreServiceImpl petStore = context.getBean("petStore", PetStoreServiceImpl.class);
        List<String> usernameList = petStore.getUsernameList();

        logger.info(GSON.toJson(usernameList));
        logger.info(GSON.toJson(petStore));
    }
}
```

## 1.3. Bean概览

`Spring IoC`容器管理一个或多个`bean`。这些bean是使用提供给容器的配置元数据创建的（如`XML`定义的<bean/>形式）

在容器内部，这些`bean`定义表示为`BeanDefinition`对象，其中包含以下元数据

* 包限定类名：通常是被定义的`bean`的实际实现类
* `Bean`行为配置元素，它声明Bean在容器中的行为（作用域、声明周期回调等）
* 对其他`bean`的引用是该`bean`完成其工作所需要的。这些引用也被成为协作者或依赖
* 管理连接池的`bean`中使用连接数量

表1 bean定义

| 属性         | 解释           |
| ------------ | -------------- |
| Class        | 实例化Bean     |
| Name         | 命名Bean       |
| Scope        | Bean作用域     |
| 构造参数     | 依赖注入       |
| Properties   | 依赖注入       |
| 自动注入模式 | 自动注入合作者 |
| 懒加载模式   | 懒加载Bean     |
| 初始化方法   | bean实例化回调 |
| 销毁方法     | bean销毁回调   |

### 1.3.1. Bean命名

每个`bean`都有一个或多个标识符。这些标识符必须是唯一的，一个`bean`通常只有一个标识符，如果要求有多个标识符，则其它的可以看做是别名

如果没有指定`id`和`name`属性，则`Spring`会给`bean`自动生成唯一的名字

**bean定义之外对bean设定别名**

```xml
<alias name="fromName" alias="toName"/>
```

### 1.3.2. 实例化bean

* 通过反射调用构造函数直接创建`bean`
* 容器调用类的静态工厂方法来创建`bean`

**内部类名字**

`com.example.SomeThing$OtherThing`

**构造函数实例化**

只需要有个空构造函数

```xml
<bean id="bean1" class="com.wdg.spring.bean.Bean1"/>

<bean name="bean2" class="com.wdg.spring.bean.Bean2"/>
```

**静态工厂方法实例化**

在遗留代码中调用静态工厂

```xml
<bean id="clientService"
    class="com.wdg.spring.bean.ClientService"
    factory-method="createInstance"/>
```

```java
public class ClientService {

    private static ClientService clientService = new ClientService("吴第广");
    private String name;
    private ClientService(String name){
        this.name = name;
    }
    private static ClientService createInstance(){
        return clientService;
    }
}
```

**实例工厂方法实例化**

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="com.wdg.spring.bean.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService1"
      factory-bean="serviceLocator"
      factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {
    private static ClientService clientService = new ClientService("云云");

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

**确定bean的运行时类型**

```java
BeanFactory.getBean()
```

## 1.4. 依赖

多对象共同协作实现目标

### 1.4.1. 依赖注入

依赖注入（DI Dependency injection），就是对象通过构造函数参数，工厂方法参数，`setter`属性定义其依赖项的过程。

依赖注入有两种主要的变体：基于构造函数的依赖注入和基于`setter`的依赖注入

**基于构造的依赖注入**

基于构造函数的依赖注入是由容器调用带有多个参数的构造函数来完成的，每个参数代表一个依赖项。和调用带有特定参数的静态工厂方法来构造`bean`几乎是等价的

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

**构造函数参数解析**

* 类类型匹配

  ```xml
  <bean id="beanOne" class="com.wdg.spring.bean.ThingOne">
      <constructor-arg ref="beanThree"/>
      <constructor-arg ref="beanTwo"/>
  </bean>
  
  <bean id="beanTwo" class="com.wdg.spring.bean.ThingTwo"/>
  
  <bean id="beanThree" class="com.wdg.spring.bean.ThingThree"/>
  ```

* 基本类型匹配

  ```xml
  <bean id="exampleBean" class="com.wdg.spring.bean.ExampleBean">
      <constructor-arg type="int" value="7500000"/>
      <constructor-arg type="java.lang.String" value="42"/>
  </bean>
  ```

* 索引下标匹配（从0开始）

  ```xml
  <bean id="exampleBean" class="com.wdg.spring.bean.ExampleBean">
      <constructor-arg index="0" value="7500000"/>
      <constructor-arg index="1" value="42"/>
   </bean>
  ```

解决多个简单值得模糊性还可以解决构造函数有两个以上相同类型参数的模糊性

**构造参数名字**

使用`@ConstructorProperties`指定名字

```xml
<bean id="exampleBean2" class="com.wdg.spring.bean.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

```java
@ConstructorProperties({"years", "ultimateAnswer"})
public ExampleBean(int years, String ultimateAnswer) {
    this.years = years;
    this.ultimateAnswer = ultimateAnswer;
}
```

**基于`setter`依赖注入**

调用无参构造函数或无参数静态工厂方法实例化之后，通过调用`bean`的`setter`方法来实现

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

对于强制依赖使用构造函数，对于可选依赖使用setter方法或配置方法是一个很好的经验法则。

**依赖解析过程**

* 通过配置元数据中描述的bean来创建 `ApplicationContext`，配置元数据可以使用`XML，Java代码，或注解`
* 在创建`bean`时提供依赖项
* 每个`bean`的构造函数参数和`setter`都是有实际定义的或者引用的其他类
* `Spring`可以将字符串格式提供的值转换为所有内置类型，如`int、long、string、boolean`等

`循环依赖`

**依赖注入示例**

2021年1月13日16:18:51

* 构造参数

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <!-- constructor injection using the nested ref element -->
      <constructor-arg>
          <ref bean="anotherExampleBean"/>
      </constructor-arg>
  
      <!-- constructor injection using the neater ref attribute -->
      <constructor-arg ref="yetAnotherBean"/>
  
      <constructor-arg type="int" value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  ```

  ```java
  public class ExampleBean {
  
      private AnotherBean beanOne;
  
      private YetAnotherBean beanTwo;
  
      private int i;
  
      public ExampleBean(
          AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
          this.beanOne = anotherBean;
          this.beanTwo = yetAnotherBean;
          this.i = i;
      }
  }
  ```

* setter参数

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean">
      <!-- setter injection using the nested ref element -->
      <property name="beanOne">
          <ref bean="anotherExampleBean"/>
      </property>
  
      <!-- setter injection using the neater ref attribute -->
      <property name="beanTwo" ref="yetAnotherBean"/>
      <property name="integerProperty" value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  ```

  ```java
  public class ExampleBean {
  
      private AnotherBean beanOne;
  
      private YetAnotherBean beanTwo;
  
      private int i;
  
      public void setBeanOne(AnotherBean beanOne) {
          this.beanOne = beanOne;
      }
  
      public void setBeanTwo(YetAnotherBean beanTwo) {
          this.beanTwo = beanTwo;
      }
  
      public void setIntegerProperty(int i) {
          this.i = i;
      }
  }
  ```

* 静态工厂方法参数

  ```xml
  <bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
      <constructor-arg ref="anotherExampleBean"/>
      <constructor-arg ref="yetAnotherBean"/>
      <constructor-arg value="1"/>
  </bean>
  
  <bean id="anotherExampleBean" class="examples.AnotherBean"/>
  <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
  ```

  ```java
  public class ExampleBean {
  
      // a private constructor
      private ExampleBean(...) {
          ...
      }
  
      // a static factory method; the arguments to this method can be
      // considered the dependencies of the bean that is returned,
      // regardless of how those arguments are actually used.
      public static ExampleBean createInstance (
          AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
  
          ExampleBean eb = new ExampleBean (...);
          // some other operations...
          return eb;
      }
  }
  ```

### 1.4.2. 依赖项和配置详细

<property/> 和 <constructor-arg/>

**值（基本数据类型，Strings等）**

<property/>的`value`属性将属性或构造函数参数指定为字符串形式

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

下面例子中使用`p名称空间`进行更简洁的XML配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```

也可以配置一个`java.util.Properties`实例，如下：

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器将<value/>元素内的文本转换为`java.util`。通过使用JavaBeans PropertyEditor机制来创建属性实例

**idref元素**

idref元素只是将容器中另一个bean的id传递给<constructor-arg/>或<property/>元素的一种防止错误的方法

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

上面的bean定义代码片段在运行时与下面的代码片段完全等价：

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

使用`idref`标记可以让容器在部署时验证所引用的命名`bean`是否确实存在。

**引用其他bean（协作者）**

略

**内部bean**

在<property/>或<constructor-arg/>元素内部使用<bean/>元素定义内部bean

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean总是匿名的，无法重用，所以指定`id`和`name`属性是多余的

**集合**

* List:<list/>

* Set:<set/>

* Map:'<map/>

* Prpperties:<props/>



```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

Map的键或者值、Set的值可以是下面元素：

```xml
bean | ref | idref | list | set | map | props | value | null
```

**集合合并**

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

**集合合并的限制**

不能合并不同类型的集合

**强类型集合**

泛型注入，自动类型转换

```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

**null和空字符串**

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

等价于下面：

```java
exampleBean.setEmail("");
```



```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

等价于下面：

```java
exampleBean.setEmail(null);
```

**带有p名称空间的XML快捷方式**

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```



多种bean定义规则

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

**带有c名称空间的XML快捷方式**

允许内联属性来配置构造函数参数

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

使用索引：

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```

**复合属性名**

必须保证路径上的对象不为空

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```



### 1.4.3. 使用`depends-on`

一些依赖项不那么直接。如数据库驱动程序注册（静态初始化）

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

bean对多bean的依赖表示（逗号分隔）：

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```



### 1.4.4. 延迟实例化bean

默认情况下，ApplicationContext 急切创建和配置所有的单例bean，作为初始化过程的一部分，可以尽早暴露问题

也可以惰性初始化bean来防止预实例化单例bean。延迟初始化bean告诉IoC容器在第一次被请求时创建bean实例，而不是在启动时

<bean/>元素的`lazy-init`属性控制

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

如果声明了为延迟初始化的bean是另外一个没有声明的bean的依赖时，这个延迟初始化的bean仍会在启动时创建



容器级控制延迟初始化bean

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```



### 1.4.5. 自动装配协作者

Spring容器可以自动装配协作bean之间的关系

自动装配有一下优点：

* 显著减少指定属性或构造函数参数
* 可以随着对象的发展而更新配置

使用<bean/>元素的`autowire`属性为bean指定autowire模式

| autowire模式 | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| no           | 默认，没有自动装配                                           |
| byName       | 通过属性名自动装配                                           |
| byType       | 通过类型自动配置，如果存在多个，则抛出异常                   |
| constructor  | 类似于byType，但适用于构造函数参数。如果容器中没有确切的构造函数参数类型的bean则会报错 |

**自动装配的局限性和缺点**

* 属性和构造函数参数设置中的显示依赖总是覆盖自动装配。不能自动装配简单的属性，如字符串，基本数据类型
* 自动装配注入精确装配。自动装配可能产生歧义
* 从Spring容器生成文档工具可能无法使用连接信息
* 容器中的多个bean定义可能与要自动连接的setter方法或者构造函数参数指定的配置匹配

为了避免上述问题，可以选择下面方案解决问题：

* 放弃自动装配，支持精确装配
* 通过`autowire-candidate`属性设置为`false`，避免对bean定义进行自动装配
* 通过将<bean/>元素的`pripmary`属性设置为`true`，将当个bean定义指定为主候选对象
* 实现基于注释的配置提供的更细粒度的控制，如基于注释的容器配置中所述

**从自动装配中排除一个bean**

<bean>元素中`autowire-candidate`属性置位`false`

### 1.4.6. 方法注入

单例使用非单例依赖

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

**检查方法注入**

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

这部分看不懂



## 1.5. bean作用域

在创建`bean`定义时，就是创建一个模板。用于创建改`bean`定义的类的实际实例。

控制特定的`bean`对象的范围

`Spring`框架支持6个作用域，其中4个只有在使用`web`感知的`ApplicationContext`才能使用。

| 作用域      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| singleton   | 默认，为每个SPring IoC容器将bean定义范围为单个对象实例       |
| prototype   | 将单个bean定义为任意数量的对象实例                           |
| request     | 将单个bean定义限定为单个HTTP请求的生命周期，即每个HTTP请求都有自己的bean实例 |
| session     | 将单个bean定义为HTTP回话的生命周期                           |
| application | 将单个bean定义到ServletContext的生命周期                     |
| websocket   | 将单个bean定义为WebSocket的生命周期                          |



### 1.5.1. 单例作用域

一个单例bean只有一个共享实例被管理

`Spring IoC`容器为`Bean`创建一个实例，并存储在单例`bean`缓存，后续的请求和引用都会返回缓存中的对象

![singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/images/singleton.png)

`Spring`的单例`bean`概念不同于`GoF`模式书籍中定义的单例模式。`GoF`单例对对象的作用域进行硬编码，这样每个类加载器只创建一个特定类的实例。

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```



### 1.5.2. 原型作用域

`bean`部署的非单例原型作用域导致每次对特定`bean`发出请求时都会创建一个新的`bean`实例

也就是说，通过容器的`getBean()`方法调用返回对象不同

对所有有状态`bean`使用原型作用域，对无状态`bean`使用单例作用域

![prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/images/prototype.png)

定义为原型作用域：

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相比，`Spring`并不管理原型`bean`的完整生命周期。容器实例化、配置和组装一个原型对象，并将其交给客户端，而不需要进一步记录该原型实例

初始化生命周期回调方法在所有作用域上都会被调用，而原型作用域的bean不会调用销毁生命周期的回调方法

在某些方法，`Spring`容器对于原型作用域bean的作用就是替代了`Java new`操作符



### 1.5.3. 带有原型bean依赖的单例bean



依赖关系是在实例化时解析的

先实例化一个新的原型bean，然后将依赖注入到单例`bean`中



#### 1.5.4. request、session、application和websocket作用域

以上作用域只在使用`web`相关的`Spring ApplicationContext`实现是有效，如`XmlWebApplicationContext`。

否则将抛出异常

**初始化Web配置**

如果需要使用以上作用域，则在定义`bean`之前需要进行一些初始配置

如何完成这个初始设置取决于特定的`Setvlet`环境

略，不重要且难

**request作用域**

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

**session作用域**

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

**application作用域**

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

### 1.5.5. 自定义作用域

略

## 1.6. 定制bean的性质

* 生命周期回调
* `ApplicationContextAware` 和 `BeanNameAware`
* 其他 `Aware` 接口

### 1.6.1. 生命周期回调

要与容器管理的bean生命周期交互，可以实现Spring InitializingBean和DisposableBean接口。

对前者调用afterPropertiesSet()，对后者调用destroy()，以便bean在初始化和销毁bean是执行某些操作

JSR-250中 `@PostConstruct`和`@PostConstruct`与Spring接口解耦

还有可以在bean定义时使用 `init-method`和`destroy-method`属性

在内部，`Spring`框架使用`BeanPostProcessor`实现来处理它可以找到的1任何回调接口并调用适当的方法，也可自定义特性或其他`Spring`默认不提供的生命周期行为，通过实现`BeanPostProcessor`接口

**初始化回调**

`org.springframework.beans.factory.InitializingBean`

允许在容器设置了bean的所有必要的属性后执行特定逻辑

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

或

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

**销毁回调**

让bean在容器被销毁时执行

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

或

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

**默认初始化和销毁方法**

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

**结合生命周期机制**

从Spring 2.5开始，有三个选项来控制bean的生命周期行为：

* `InitializingBean` 和 `DisposableBean`回调接口
* 自定义 `init()`和 `destroy()`方法
* `@PostConstruct` 和 `@PreDestroy` 注解。

为同一个bean配置的多个生命周期机制，使用不同的初始化方法（销毁方法也是相同的调用顺序），如下所示：

1. `@PostConstruct`（@PreDestroy）方法注解
2. 作为`InitializingBean`（Disposable）接口回调方法`afterPropertiesSet()`（destroy()）
3. 自定义的 `init()`（destroy()）方法

**启动和关闭回调**

`Lifecycle`接口为任何有自己生命周期需求的接口定义了基本方法

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现`Lifecycle`接口，通过LifecycleProcessor实现

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

```java
public interface Phased {

    int getPhase();
}
```

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

**在非Web应用中优雅停止Spring IoC容器**

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

### 1.6.2. `ApplicationContextAware`和`BeanNameAware`

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

### 1.6.3. 其他`Aware`接口

略



## 1.7. Bean定义继承

beann故意可以包含很多配置信息，包括构造函数参数、属性值和特定于容器的信息，比如初始化方法、静态工厂方法名称等子bean定义从父bean定义继承配置数据。

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

## 1.8. 容器扩展点

### 1.8.1. 通过`BeanPostProcessor`定制bean

在Spring容器完成实例化、配置和初始化bean之后实现一些定制逻辑

**例子**

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

### 1.8.2. 使用`BeanFactoryPostProcessor`自定义配置元数据

### 1.8.3. 使用FactoryBean定制实例化逻辑

* Object getObject()：返回工厂创建的对象实例
* boolean isSingleton()：是否是单例
* Class getObjectType()：返回由getObject()方法返回的对象类型



## 1.9. 基于注解的从其配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

隐式注册的后处理器包括：

AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor, 和RequiredAnnotationBeanPostProcessor.



### 1.9.1. @Required

应用于bean属性设置方法

强制依赖，不存在则空指针

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

Spring Framework 5.1正式弃用

### 1.9.2. 使用@Autowired

> JSR 303的@Inject注解可以用来代替Spring的@Autowired注解

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;
	// 可以用于构造函数，setter函数
    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

用于构造和setter，混用

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

注入该类型的数组的字段或方法

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

`Map`类型映射实例

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

若`@Autowired`为空，则空指针异常，使用`required = false`允许可以为空

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

### 1.9.3. 使用@Primary对基于注解的自动装配进行微调

@Primary指出，当多个bean是自动连接到单指依赖项的候选对象时，应该优先考虑特定bean

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

上述例子中`firstMovieCatalog()`方法返回的`MovieCatalog`实例将会被注入

对应下面的XML配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 1.9.4. 使用@Qualifier对基于注解的自动装配进行微调

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

其他功能略（如自定义@Qualifier注解）

### 1.9.5. 使用泛型作为自动装配限定符

假如有以下配置

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

### 1.9.6. 使用 `CustomAutowireConfigurer`

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```



### 1.9.7. 使用 @Resource注入

> JSR-250

@Resource带有一个name属性。默认情况下，Spring将该值解释为要注入的bean名

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果没有显式指定名称，则默认名称派生自字段名或setter方法

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

### 1.9.8. 使用@Value

通常用于注入外部化的属性

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

```java
catalog.name=MovieCatalog
```



### 1.9.9. 使用@PostConstruct和@PreDestroy

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```



## 1.10. 类路径扫描和管理组件



### 1.10.1. @Component和进一步的原型注解

@Repository

@Component：是Spring管理组件的通用原型

@Service

@Controller



### 1.10.2. 使用元注解和组合注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

### 1.10.3. 自动检测类和注册bean

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，需要在使用@Configuration的类上使用@ComponentScan注解，其中basePackages属性扫描包路径

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

使用XML扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```