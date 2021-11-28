# Ruby

[文档地址](https://www.bookstack.cn/books/the-book-of-ruby)

## 下载安装&使用
[下载地址](https://rubyinstaller.org/downloads/)

`CMD`中输入`irb`进入`Ruby`交互模式

## 第一章 字符串、数字、类和对象
`易于使用`

```shell
irb(main):005:0> puts 'hello, world'
hello, world
=> nil
```

```ruby
# hello.rb 
# puts() 方法会在末尾自动添加一个换行符，但 print() 方法则不会
# gets() 方法读取用户的输入并以字符串类型保存
# Ruby 不用预先声明变量，也不用指定数据类型
# Ruby 大小写敏感
# 变量名字必须以小写字母开头，如果以大写字母开头则 Ruby 认为是一个常量
# gets()、puts()、print() 方法的括号可以省略
# 双引号内部的 #{} 会被解释为变量（可以计算表达式），单引号则不会
# 单行注释
# =begin 以及末尾添加 =end（多行注释） 
# s.to_f 字符串转浮点数，如 "145.45".to_f，"Hello".to_f 返回 0.0
print('输入您的姓名:')
name = gets()
puts("Hello #{name}")
=begin
这是多行注释
这是多行注释
这是多行注释
这是多行注释
=end
```

`返回`
```shell
ruby hello.rb
输入您的姓名:wudiguang
Hello wudiguang
```

`双引号解析变量 #{}`
```shell
irb(main):024:0> puts "hello, #{name}"
hello, wudiguang
=> nil
irb(main):025:0> puts 'hello, #{name}'
hello, #{name}
```

`浮点数`
```shell
irb(main):070:0> taxrate = 0.175
=> 0.175
irb(main):071:0> subtotal = 100.00
=> 100.0
irb(main):072:0> tax = subtotal * taxrate
=> 17.5
irb(main):073:0> s = gets
145.45
=> "145.45\n"
irb(main):074:0> subtotal = s.to_f
=> 145.45
```

`条件语句`
```ruby
taxrate = 0.175
print "Enter price (ex tax): "
s = gets
subtotal = s.to_f
# 如果 if 后没有换行，则 then 不可省略
# 如果 if 后有换行，则 then 可以省略，但是为了代码可读性，建议统一加上 then
if (subtotal < 0.0) then
subtotal = 0.0
# end 必写，否则会报错
end
tax = subtotal * taxrate
puts "Tax on $#{subtotal} is $#{tax}, so grand total is $#{subtotal+tax}"
```

`局部变量与全局变量`
```ruby
# 三个名为 localVar 的局部变量，一个在 main 作用域，另外两个分别在独立的方法作用域内，互不影响
# 一个以 $ 字符开头的全局变量拥有全局作用域，类比 Java 中的 static 变量
localvar = "hello"
$globalvar = "goodbye"
def amethod
  localvar = 10
  puts(localvar)
  puts($globalvar)
end
def anotherMethod
  localvar = 500
  $globalvar = "bonjour"
  puts(localvar)
  puts($globalvar)
end

puts amethod
puts anotherMethod

puts amethod
puts localvar
```

`返回`

```shell
10
goodbye

500
bonjour

10
bonjour

hello
```

`类与对象`
> 概念同 Java 中的类和对象


```ruby
# 使用 class 关键字定义类，类名必须大写字母开头
class Dog
    # 定义修改名字方法
    def set_name(aName)
        @myname = aName
    end
    # 定义获取名字方法
    def get_name()
        # return 关键字时可选的。当被省略时，Runby 会返回最后一个表达式的值
        # 为了结构清晰，建议统一使用 return
        return @myname
    end
    # 定义狗叫方法
    def talk
    return 'woof!'
  end
end
# 使用：
=begin
irb(main):102:0> mydog = Dog.new
=> #<Dog:0x0000021a18471a88>
irb(main):103:0> yourdog = Dog.new
=> #<Dog:0x0000021a187cae28>
irb(main):106:0> mydog.set_name('Fido')
=> "Fido"
irb(main):107:0> mydog
=> #<Dog:0x0000021a18471a88 @myname="Fido">
=end
```

`构造方法-new 与 initialize`

```ruby
# 当一个类包含名为 initialize 的方法，它会在使用 new 方法创建对象时自动被调用。
# 使用 initialize 初始化变量不会出现get_xxx() 返回 nil 情况
# to_s 方法被定义在 Object 类中，该类时其他类的祖先
# new 方法可以创建一个对象，它可以被认为时对象的"构造方法"，如何通常不应该重写 new 方法
def initialize( aName, aDescription )
    @name = aName
    @description = aDescription
end
```
`查看对象内部细节`

```shell
irb(main):136:0> mydog.inspect
=> "#<Dog:0x0000021a18471a88 @myname=\"Fido\">"
irb(main):137:0> p(yourdog)
#<Dog:0x0000021a1841e180>
=> #<Dog:0x0000021a1841e180>
irb(main):138:0> p(mydog)
#<Dog:0x0000021a18471a88 @myname="Fido">
=> #<Dog:0x0000021a18471a88 @myname="Fido">
```

```ruby
# to_s.rb
puts(Class.to_s)     #=> Class
puts(Object.to_s)    #=> Object
puts(String.to_s)    #=> String
puts(100.to_s)       #=> 100
puts(Dog.to_s)  #=> Dog
```

## 第二章 类的层次结构、属性与变量

## 第三章 字符串和范围

## 第四章 数组和哈希表

## 第五章 循环（Loop）和迭代器（Iterator）

## 第六章 条件语句

## 第七章 方法（Methods）

## 第八章 传递参数和返回值

## 第九章 异常处理

## 第十章 Blocks，Procs and Lambdas

## 第十一章 符号（Symbols）

## 第十二章 模块（Modules）和混合（Mixins）

## 第十三章 Files 与 IO

## 第十四章 YAML

## 第十五章 Marshal

## 第十六章 正则表达式

## 第十七章 线程（Threads）

## 第十八章 调试与测试

## 第十九章 动态编程