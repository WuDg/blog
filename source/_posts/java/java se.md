# Java SE

```md
author: WuDG
datetime： 15:2451 星期一 2021年8月2日
position： 杭州乐佳大厦/博彦
```
> 我曾七次鄙视自己的灵魂
第一次，当它本可进取时，却故作谦卑；
第二次，当它在空虚时，用爱欲来填充；
第三次，在困难和容易之间，它选择了容易；
第四次，它犯了错，却借由别人也会犯错来宽慰自己；
第五次，它自由软弱，却把它认为是生命的坚韧；
第六次，当它鄙夷一张丑恶的嘴脸时，却不知那正是自己面具中的一副；
第七次，它侧身于生活的污泥中，虽不甘心，却又畏首畏尾。 

## 一、tools

[Java 在线工具](https://www.runoob.com/manual/jdk11api/index.html)

[JDK 11 在线中文手册](https://www.runoob.com/manual/jdk11api/index.html)


## 2、入门

### 2.1 简介

* JavaSE/JavaEE/JavaME
* 语法类似 C 和 C++，丢弃 C++ 中操作符重载、多继承、自动强制类型转换，提供自动分配和回收内存
* 简单的/面向对象的/分布式的/健壮的/安全的/可移植的/解释型的/多线程的/动态的

### 2.2 开发环境配置

```properties
JAVA_HOME=
CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar
Path=$Path;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
```

### 2.3 基础语法
* 对象/类/方法/实例变量
* 大小写敏感/类名/方法名/源文件名/主方法入口/标识符/修饰符/变量（局部变量、类变量、成员变量）/数组/枚举
* 访问修饰符（default、public、protected、private），非访问修饰符（final、abstract、static、synchonized、transient、volatile）


### 2.4 对象和类
* 多态/继承/封装/抽象/类/对象/实例/方法/重载

### 2.5 基本数据类型

#### 数据类型
* 内置数据类型（byte、short、int、long、float、double、bool、char）
* 引用数据类型（对象、数组）
* 常量（final）

**自动类型换转**
> 低  ------------------------------------------------>  高
byte,short,char—> int —> long—> float —> double

* 大类型转小类型需要强制转换
* 隐式强制类型转换（整形默认 int，小数默认 double）

### 2.6 修饰符
* default：默认，当前包可见
* private：当前类可见
* public：所有类可见
* protected：当前包和所有子类可见

### 2.7 switch
* 支持 char、short、int、byte、String

### 2.8 Java 异常&错误
* 检查异常：无法预见的问题，如打开文件不存在
* 运行时异常：可被避免，可在编译时被忽略
* 错误：脱离程序控制，如 stack overflow、oom等


**Throwable**
* Error
  * OutOfMemoryError
* IOError
* Exception
  * IOException
    * FileNotFoundException
  * RuntimeException
    * NullPointerException