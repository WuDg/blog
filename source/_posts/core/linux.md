# linux

## awk
> 处理文本文件的语言，是一个强大的文本分析工具

### 基本用法

#### 用法一
> awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号

**示例**
`log.txt`
```text
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```
```shell
# 每行按空格或TAB分割，输出文本中的1、4项
awk '{print $1,$4}' log.txt
# 格式化输出
# awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
```

#### 用法二
> awk -F  #-F相当于内置变量FS, 指定分割字符

**示例**
```shell
# 使用逗号分割
awk -F, '{print $1,$4}' awk.txt
# 或者使用内建变量
# awk 'BEGIN{FS=","} {print $1,$2}' awk.txt
# 使用多个分隔符，先使用空格分割，然后对分割结果再使用","分割
# awk -F '[ ,]'  '{print $1,$2,$5}'   awk.txt
```

#### 用法三
> awk -v  # 设置变量

**示例**
```shell
awk -v a=1 '{print $1,$1+a}' awk.txt
# awk -va=1 -vb=s '{print $1,$1+a,$1b}' awk.txt
```

### 用法四
> awk -f {awk脚本} {文件名}

**示例**
```shell

awk -f cal.awk awk.txt
```

### 使用正则，字符串匹配

**示例**
```shell
awk '$2 ~ /th/ {print $2,$4}' awk.txt
# 输出包含 "re" 的行
# awk '/re/ ' awk.txt
# 忽略大小写
# awk 'BEGIN{IGNORECASE=1} /this/' awk.txt
```