# react
> 用于构建用户界面的 JavaScript 库

- [react](#react)
  - [一、Hello World](#一hello-world)
  - [二、JSX 简介](#二jsx-简介)
    - [2.1 例子](#21-例子)
    - [2.2 嵌入表达式](#22-嵌入表达式)
  - [三、元素渲染](#三元素渲染)
    - [3.1 React 只更新它需要更新的部分](#31-react-只更新它需要更新的部分)
  - [四、组件 & Props](#四组件--props)
    - [4.1 函数组件与 class 组件](#41-函数组件与-class-组件)
    - [4.2 渲染组件](#42-渲染组件)
    - [4.3 组合组件](#43-组合组件)
    - [4.4 提取组件](#44-提取组件)
    - [4.5 Props 的只读性](#45-props-的只读性)
  - [五、State & 生命周期](#五state--生命周期)
    - [5.1 将函数组件转换成 class 组件](#51-将函数组件转换成-class-组件)
    - [5.2 向 class 组件中添加局部的 state](#52-向-class-组件中添加局部的-state)
    - [5.3 将生命周期方法添加到 Class 中](#53-将生命周期方法添加到-class-中)

## 一、Hello World

```jsx
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>wudiguang-index</title>
</head>
<body>
    <div id="root"></div>
<script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

</body>

<script type="text/babel">
    const element = <h1>Hello, world!</h1>;
    ReactDOM.render( element,document.getElementById('root'));
</script>

</html>
```

## 二、JSX 简介
> 是一个 JavaScript 的语法扩展

### 2.1 例子

```jsx
const element = <h1>Hello, world!</h1>;
```

### 2.2 嵌入表达式

```jsx
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

JSX 语法中：大括号内防止任何有效的 JavaScript 表达式，如 `2+2`, `user.firstName` 或者 `formatName(user)`

```jsx
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

## 三、元素渲染

**将一个元素渲染为 DOM**

我们将其称为“根” DOM 节点

```html
<div id="root"></div>
```

**更新已渲染的元素**
> daduoshu1 React
>  大多数 React 应用只会调用一次 ReactDOM.render()

```jsx
function tick() {
    const element = (
        <div>
        <h1>Hello, world!</h1>
        <h2>It is {new Date().toLocaleTimeString()}.</h2>
        </div>
    );
    console.log(new Date().toLocaleTimeString())
    ReactDOM.render(element, document.getElementById('root'));
}
// 每秒执行一次
setInterval(tick, 1000);
```

### 3.1 React 只更新它需要更新的部分
> React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态

## 四、组件 & Props
> 组件允许将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思

### 4.1 函数组件与 class 组件

`该JS函数定义 React 组件，接收唯一参数 props，并返回 React 元素`

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

`ES6 的 class`

```es6
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### 4.2 渲染组件

React 元素可以是 DOM 标签，也可以是用户自定义的组件

```jsx
const element = <div />;
const element = <Welcome name="Sara" />;
```

当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性以及子组件转换为单个对象传递给组件，这个对象被称为 "props"

如：

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

1. 调用 ReactDOM.render() 函数，并传入 `<Welcome name="Sara" />`作为参数
2. React 调用 `Welcome` 组件，并将 `{name: 'Sara'}` 作为 props 传入
3. `Welcome` 组件将 `<h1>Hello,Sara</h1>`元素作为返回值
4. React DOM 将 DOM 高效地更新为 `<h1>Hello, Sara</h1>`

注意：组件名称必须以大写字母开头


### 4.3 组合组件
> 组件可以在其输出中引用其他组件

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

### 4.4 提取组件
> 将组件拆分为更小的组件

例如，如下 `Component` 组件：
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

由于复杂嵌套关系，难以维护且难以复用，因此可以提取一些组件出来

`Avatar` 组件

```jsx
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```

进一步提取 `UserInfo` 组件，该组件在用户名旁渲染 `Avatar` 组件：

```jsx
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

进一步简化 `Component` 组件

```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

### 4.5 Props 的只读性
> 组件无论是使用函数声明还是通过 class 声明，都绝不能修改自身的 props

`sum` 函数
```jsx
function sum(a, b) {
  return a + b;
}
```

这样的函数被称为"纯函数"，因为该函数不会尝试更改入参，且多次调用下相同的入参始终返回相同的结果

相反，下面的函数则不是纯函数，它更改了自己的入参

```jsx
function withdraw(account, amount) {
  account.total -= amount;
}
```

`所有 React 组件都必须像纯函数一样保护它们的 props 不被更改`

## 五、State & 生命周期

[详细](https://react.docschina.org/docs/react-component.html)

上一章节中时钟的例子中，我们通过调用 `ReactDOM.render()` 来修改我们想要渲染的元素：

```jsx
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

尝试真正复用 `Clock` 组件，它将设置自己的计时器并每秒更新一次

```jsx
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

### 5.1 将函数组件转换成 class 组件
通过以下五步将 `Clock` 的函数组件转换成 class 组件：

1. 创建一个同名的 ES6 class，并且继承于 React.Component
2. 添加一个空的 `render()` 方法
3. 将函数体移动到 `render()` 方法中
4. 在 `render()` 方法中使用 `this.props` 替换 `props` 
5. 删除剩余的空函数声明


```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

现在 `Clock` 组件被定义为 class，而不是函数

每次组件更新时 `render` 方法都会被调用，但只要在相同的 DOM 节点中渲染 `<Clock />`，就仅有要给 `Clock` 组件的 class 实例被创建使用。

### 5.2 向 class 组件中添加局部的 state

1. 把 `render()` 方法中的 `this.props.date` 替换成 `this.state.date`:

```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2. 添加一个 class 构造函数，然后在该函数中为 `this.state` 赋初值:

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
通过以下方式将 `props` 传递到父类的构造函数中：

```jsx
constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Clas 组件应该始终使用 `props` 参数来调用父类的构造函数

3. 移除 `<Clock />`元素中的 `date`属性：

```jsx
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

完整代码：

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
接下来设置 `Clock` 的计时器并每秒更新它

### 5.3 将生命周期方法添加到 Class 中

当 `Clock` 组件第一次被渲染到 DOM 中的时候，就为其设置一个计时器，这在 React 中被称为“挂载(mount)”
当 DOM 中 `Clock` 组件被删除时，应该清楚定时器，这被称为“卸载(unmount)”

我们可以为 class 组件声明一些特殊方法，当组件挂载或卸载时就会去执行这些方法：

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
  }

  componentWillUnmount() {
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

`这些方法叫做生命周期方法`

`componentDidMount()` 方法会在组件已经被渲染到 DOM 中后运行，所以最好在这里设置计时器

```jsx
componentDidMount() {
    this.timerID = setInterval(
        () => this.tick(),
        1000
    );
}
```

我们会在 `componentWillUnmount` 生命周期方法中清除计时器：

```jsx
componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

完整例子：

```jsx
class Clock extends React.Component {
  // 构造方法，初始化state中的date属性
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
  // 组件渲染到 DOM 中后运行
  componentDidMount() {
    // 设置计时器  
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
  // 清除计时器
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
  // 设置state中的date属性  
  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

细节:
1. 当`<Clock />` 被传给 `ReactDOM.render()` 时，React 会调用 `Clock` 组件的构造函数。同时初始化 `this.state`
2. 之后 React 会调用组件的 `render()`方法。即 React 确定该在页面上展示什么的方式
3. 当 `Clock` 的输出被插入到 DOM 中后，React 就会调用 `ComponentDidMount()` 方法，这个方法中 `Clock` 组件向浏览器请求设置一个计时器来每秒调用一次组件的 `tick()` 方法
4. 浏览器每秒都会调用一次 `tick()` 方法，这个方法中会通过调用 `setState()` 来更新 date 属性，React 知道 state 改变后会重新调用 `render()` 方法，将重新加载 `state` 的值
5. 一旦 `Clock` 组件从 DOM 中被移除，React 就会调用 `componentWillUnmount()`方法，这样计时器就会停止




