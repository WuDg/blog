# Ruby

- [Ruby](#ruby)
  - [下载安装&使用](#下载安装使用)
  - [第一章 字符串、数字、类和对象](#第一章-字符串数字类和对象)
    - [1.1 易于使用](#11-易于使用)
    - [1.2 双引号解析变量 #{}](#12-双引号解析变量-)
    - [1.3 浮点数](#13-浮点数)
    - [1.4 条件语句](#14-条件语句)
    - [1.5 局部变量与全局变量](#15-局部变量与全局变量)
    - [1.6 类与对象](#16-类与对象)
    - [1.7 构造方法-new 与 initialize](#17-构造方法-new-与-initialize)
    - [1.8 查看对象内部细节](#18-查看对象内部细节)
  - [第二章 类的层次结构、属性与变量](#第二章-类的层次结构属性与变量)
    - [2.1 类的层次结构、属性与变量](#21-类的层次结构属性与变量)
    - [2.2 超类&子类](#22-超类子类)
    - [2.3 类变量](#23-类变量)
    - [2.4 类常量](#24-类常量)
  - [第三章 字符串和范围](#第三章-字符串和范围)
    - [3.1 字符串和范围](#31-字符串和范围)
    - [3.2 反引号](#32-反引号)
    - [3.3 字符串处理](#33-字符串处理)
    - [3.4 格式化字符串](#34-格式化字符串)
    - [3.5 范围](#35-范围)
  - [第四章 数组和哈希表](#第四章-数组和哈希表)
  - [第五章 循环（Loop）和迭代器（Iterator）](#第五章-循环loop和迭代器iterator)
  - [第六章 条件语句](#第六章-条件语句)
  - [第七章 方法（Methods）](#第七章-方法methods)
  - [第八章 传递参数和返回值](#第八章-传递参数和返回值)
  - [第九章 异常处理](#第九章-异常处理)
  - [第十章 Blocks，Procs and Lambdas](#第十章-blocksprocs-and-lambdas)
  - [第十一章 符号（Symbols）](#第十一章-符号symbols)
  - [第十二章 模块（Modules）和混合（Mixins）](#第十二章-模块modules和混合mixins)
  - [第十三章 Files 与 IO](#第十三章-files-与-io)
  - [第十四章 YAML](#第十四章-yaml)
  - [第十五章 Marshal](#第十五章-marshal)
  - [第十六章 正则表达式](#第十六章-正则表达式)
  - [第十七章 线程（Threads）](#第十七章-线程threads)
  - [第十八章 调试与测试](#第十八章-调试与测试)
  - [第十九章 动态编程](#第十九章-动态编程)

[文档地址](https://www.bookstack.cn/books/the-book-of-ruby)

## 下载安装&使用
[下载地址](https://rubyinstaller.org/downloads/)

`CMD`中输入`irb`进入`Ruby`交互模式

## 第一章 字符串、数字、类和对象

### 1.1 易于使用

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

### 1.2 双引号解析变量 #{}
```shell
irb(main):024:0> puts "hello, #{name}"
hello, wudiguang
=> nil
irb(main):025:0> puts 'hello, #{name}'
hello, #{name}
```

### 1.3 浮点数
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

### 1.4 条件语句
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

### 1.5 局部变量与全局变量
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

### 1.6 类与对象
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

### 1.7 构造方法-new 与 initialize

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
### 1.8 查看对象内部细节

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

### 2.1 类的层次结构、属性与变量
> 继承

### 2.2 超类&子类
> 同 Java

```ruby
class Treasure < Thing
# 定义get
attr_reader :description
# 定义set
attr_writer :description
# 定义get、set
attr_accessor :value
# 同时定义多个变量
=begin
attr_reader :name, :description
attr_writer(:name, :description)
attr_accessor(:value, :id, :owner)
=end
```

### 2.3 类变量
```ruby
@@num_things = 0
```

### 2.4 类常量
> 以大写字母开头

```ruby
class X
  A = 10
  
  class Y
  end
end
=begin
(main):162:0> X::A
=> 10
irb(main):163:0> X::Y.new
=> #<X::Y:0x0000021a18b94c98>
=end
```

```ruby
class A
  def b
    puts( "b" )
  end
end
class B < A
  def ba2
    puts( "ba2" )
  end
end
```

可以将功能添加到 Ruby 的标准库中，如 Array
```ruby
class Array
  def gribbit
    puts( "gribbit" )
  end
end
# [1,2,3].gribbit
```


## 第三章 字符串和范围
### 3.1 字符串和范围
> 在双引号嵌入 #{} 可以解析变量
> 双引号替代分隔符：%Q和/，%/和/
> 单引号替代分隔符：%q和/

```ruby
print('Enter your name: ')
name = gets()
puts("Hello #{name}")
```

### 3.2 反引号
> 将命令传递给操作系统执行

```ruby
puts(`dir`)
puts(%x/dir/)
puts(%x{dir})
```

### 3.3 字符串处理

```ruby
# 使用 << 方法时，后面可以直接附加一个整数
# 使用 + 号或者空格时，整数必须使用 to_s 方法先转换为字符串
# 一些字符串方法实际上是在不创建新的字符串对象的情况下改变其本身，这些方法通常以感叹号结尾，如 capitalize!
# $/ 作为 "记录分隔符"
# 使用 chop 和 chomp 方法删除换行符，建议使用 chomp，它仅会删除最后的记录分隔符
s = "Hello " << "world"
s = "Hello " + "world"
s = "Hello " "world"
s4 = "This ", "is", " not a string!", 10, print("print (s4):", s4, "\n")
```

```shell
irb(main):290:0> s = "Hello World"
=> "Hello World"
irb(main):291:0> puts(s[1])
e
=> nil
irb(main):292:0> puts(s[1, 1])
e
=> nil
irb(main):293:0> puts(s[1..3])
ell
=> nil
```

### 3.4 格式化字符串

```
%d – decimal number
%f – floating point number
%o – octal number
%p – inspect object
%s – string
%x – hexadecimal number
```

```ruby
printf("%0.02f", 10.12945)  # displays 10.13
```

### 3.5 范围
```ruby
a = (1..10)
b = (-10..-1)
c = (-10..10)
d = ('a'..'z')
# 可以使用三个点或者两个点，三个点则不包括最后一个值
e = ('a'..'z')   # this two-dot range = 'a'..'z'
f = ('a'...'z')   # this three-dot range = 'a'..'y'
(1..10).to_a
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

`范围迭代器`
```ruby
for i in (1..10) do
    puts( i )
end
```

`跨行字符串-HereDoc`
```ruby
hdoc1 = <<EODOC
I wandered lonely as a #{"cloud".upcase},
That floats on high o'er vale and hill...
EODOC
```

默认情况下，HereDoc 被视为双引号字符串，因此表达式 #{"cloud".upcase} 将会被执行

```ruby
p %q/dog cat #{1+2}/  #=> "dog cat \#{1+2}"
p %Q/dog cat #{1+2}/  #=> "dog cat 3"
p %/dog cat #{1+2}/  #=> "dog cat 3"
p %w/dog cat #{1+2}/  #=> ["dog", "cat", "\#{1+2}"]
p %W/dog cat #{1+2}/  #=> ["dog", "cat", "3"]
p %r|^[a-z]*$|  #=> /^[a-z]*$/
p %s/dog/ #=> :dog
p %x/vol/ #=> " Volume in drive C is OS [etc...]"
```


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