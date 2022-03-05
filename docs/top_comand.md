# top-命令

```describe
作者： 吴第广
日期： 2021年12月20日
```

top命令经常用来监控linux的系统状况，比如cpu、内存的使用。

![top](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/blog/top.png)

---
## 一、命令格式
`top [-d number] | top [-bnp]`

```describe
-d:number代表秒数，表示top命令显示的页面更新一次的间隔。默认是5秒。
-b:以批次的方式执行top。
-n:与-b配合使用，表示需要进行几次top命令的输出结果。
-p:指定特定的pid进程号进行观察
```
---
## 二、使用top命令后
```describe
Z:改变颜色
B:加粗
t:显示和隐藏任务/cpu信息
m:内存信息
1:监控每个逻辑CPU的状况
f:进入字段显示配置模式，可增加或者移除显示字段，按相应的字母新增或去除；o:进入字段顺序设置模式，可配置显示位置顺序，按相应的字母往下移动，按“shift+相应的字母”往上移动     
F:进入字段排序配置模式，可设置排序的字段
R:正常排序/反向排序
s:设置刷新的时间
u:输入用户，显示用户的任务
i:忽略闲置和僵死进程。这是一个开关式命令
r:重新安排一个进程的优先级别。系统提示用户输入需要改变的进程PID以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10
c:切换显示命令名称和完整命令行
M:根据驻留内存大小进行排序
P:根据CPU使用百分比大小进行排序
H:显示线程
```
---
## 三、实践：统计信息

显示信息01:

 `top - 11:39:12 up 203 days, 14:17,  2 users,  load average: 0.28, 0.31, 0.16`


```describe
1. 11:39:12:表示当前时间
2. up 203 days, 14:17:系统运行时间 格式为时：分
3. 2 users:当前登录用户数
4. load average: 0.28, 0.31, 0.16:系统负载，即任务队列的平均长度。三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。
```
显示信息02:

`Tasks: 137 total,   4 running, 131 sleeping,   1 stopped,   1 zombie
%Cpu(s):  3.7 us,  3.1 sy,  0.0 ni, 93.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st`

```describe
137 total:进程总数
4 running:正在运行的进程数
131 sleeping:睡眠的进程数
1 stopped:停止的进程数
1 zombie:僵尸进程数
3.7 us:用户空间占用CPU百分比
3.1 sy:内核空间占用CPU百分比
0.0 ni:用户进程空间内改变过优先级的进程占用CPU百分比
93.2 id:空闲CPU百分比
0.0 wa:等待输入输出的CPU时间百分比
0.0 hi:硬中断（Hardware IRQ）占用CPU的百分比
0.0 si:软中断（Software Interrupts）占用CPU的百分比
0.0 st
```

显示信息03:

`KiB Mem :  1883724 total,    75580 free,  1528760 used,   279384 buff/cache
KiB Swap:        0 total,        0 free,        0 used.    99808 avail Mem`

```describe
1883724 total:物理内存总量
75580 free:空闲内存总量
1528760 used:使用的物理内存总量
279384 buff/cache:用作内核缓存的内存量
0 total:交换区总量
0 free:空闲交换区总量
0 used:使用的交换区总量
99808 avail Mem 代表可用于进程下一次分配的物理内存数量
```

---
## 四、进程信息

` PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND`

```describe
PID:进程id
USER:进程所有者的用户名
PR:优先级
NI:nice值。负值表示高优先级，正值表示低优先级
VIRT:进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES:进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR:共享内存大小，单位kb
S:进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU:上次更新到现在的CPU时间占用百分比
%MEM:进程使用的物理内存百分比
TIME+:进程使用的CPU时间总计，单位1/100秒
COMMAND 命令名/命令行
```

## 五、总结:

top命令是用来监控Linux系统状况，比如cpu、内存的使用。作为一个合格的开发人员和运维人员必须掌握的一项基本技能。

`本文收录于:`
* [Github仓库:wudg/blog](https://github.com/wudg/blog)
* [Gitee仓库:wudg/blog](https://githee.com/wudg/blog)