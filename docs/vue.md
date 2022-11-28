# vue 学习笔记

[bilibili教程地址](https://www.bilibili.com/video/BV1Zy4y1K7SH/)

## vue 基础

### vue 简介

Q：vue 是什么
A：一套用于**构建用户界面**的**渐进式**的JavaScript框架

```
tips
构建用户界面：根据后台数据，渲染到UI界面
渐进式：vue可以自顶向上逐层的应用，即只需一个轻量小巧的核心库
```

Q：谁开发的
A：尤雨溪

- 2013年 受到Angular框架的启发，尤雨溪开发出了一款轻量框架--Seed。同年12月，Seed更名为Vue，版本号0.6.0.
- 2014年 Vue正式对外发布，版本号0.8.0。Taylor owell 在 Twitter 上发表动态，说自己正在学习 Vue.js
- 2015 年 10月27日，正式发布 Vue 1.0.0 Evangelion(新世纪福音战士)
- 2016年 10月1日，正式发布Vue 2.0.0 Ghost in the hell(攻壳机动队)
- 2020 年 9月18日，正式发布Vue 3.0.0 One Piece(海贼王)

后起之秀，生态完善，已然成为国内前端工程师必备技能

Q：Vue 的特点
A：
- 采用**组件化**模式，提高代码复用率、且让代码更好维护。(一个个vue文件)
- **声明式**编码，让编码人员无需直接操作DOM，提高开发效率。(使用声明式编码，如v-for等)
- 使用**虚拟DOM** + 优秀的**Diff 算法**(基于虚拟DOM)，尽量复用DOM结点(数据 -> 虚拟DOM -> 页面真实DOM)

*.vue = *.html + *.js + *.css

Q：学习Vue之前要掌握的JavaScript基础知识
A：ES6语法规范、ES6模块化、包管理器、原型、原型链、数组常用方法、axios、promise...

### 初识Vue

1. 想让vue工作，就必须创建一个vue实例，且要传入一个配置对象
2. root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法
3. root容器里的代码被称为【vue模板】
4. Vue实例和容器是一一对应的
5. 真是开发中只有一个Vue实例，并且会配合着组件一起使用
6. {{xxx}}中的xxx要写js表达式，且xxx可以自动读取data中的所有属性
7. 一旦data中的数据发生改变，那么模板中用到该数据的地方也会自动更新

注意区分：js表达式和js代码(语句)
1. 表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方
  a. a
  b. a+b
  c. demo(1)

2. js代码(语句)
  a. if(){}
  b. for(){}
  
### 模板语法

Vue模板语法有两大类：
1. 插值语法
    功能：用于解析标签体内容
    写法：{{xxx}}，xxx是js表达式，且可以直接读取到data中的所有属性
2. 指令语法
    功能：用于解析标签（包括：标签属性、标签体内容、绑定时间...）
    举例：v-bind:href="xxx" 或者简写为 :href="xxx"。xxx同样要写js表达式，且可以直接读取到data中的所有属性
    
    备注：Vue中有很多的指令，且形式都是：v-???。此处我们只是拿v-bind举个例子

### 数据绑定

Vue中有两种数据绑定的方式
1. 单向绑定(v-bind)：数据只能从data流向页面
2. 双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。
    备注：
        a. 双向绑定一般都应用在表单类元素上(如：input、select等)
        b. v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值

### MVVM模型

* M(model)：对应data中的数据
* V(view)：模板
* VM(ViewModel)：Vue实例对象



## vue-cli

## vue-router

## vuex

## element-ui

## vue3