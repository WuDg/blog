# 起步

1. 介绍版本控制工具的背景知识
2. 运行Git
3. 设置Git

## 1.1. 关于版本控制

版本控制系统（VCS）

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。`可以对任何类型的文件进行版本控制。`

文件回溯，比较文件变化细节，谁修改了哪个地方，问题出现原因，项目文件恢复



### 本地版本控制系统

通过复制整个项目目录来保存不同版本。易犯错且混淆所在的工作目录，不小心写错文件或者覆盖文件

![本地版本控制图解](https://git-scm.com/book/en/v2/images/local.png)

图1. 本地版本控制

原理是在硬盘上保存补丁集，通过应用所有的补丁，可以重新计算出各个版本文件内容



### 集中化的版本控制系统

让不同系统上的开发者协同工作

集中化的版本控制系统（CVCS Centralized Version Control System）

如：CVS，Subversion，Perforce等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，所有协同工作的人都通过客户端连接到这台服务器，取出最新的文件或者提交更新

![集中化的版本控制图解](https://git-scm.com/book/en/v2/images/centralized.png)

图2. 集中化的版本控制

每个人在一定程度上可以看到项目中其他人正在做什么，管理员也可以轻松掌握每个开发者的权限

缺点：中央服务器的单点故障

服务器宕机，无法提交更新，无法协同工作，只剩下各自机器上保留的单独快照



### 分布式版本控制系统

DVCS Distributed Version Control System

如Git，Mercurial，Bazaar，Darcs等

客户端不只提取最新版本的文件快照，而是保存代码仓库完整地镜像，包括完整地历史记录。如此，服务器宕机，可通过本地仓库来恢复中央仓库

![分布式版本控制图解](https://git-scm.com/book/en/v2/images/distributed.png)

图3. 分布式版本控制

## 1.2. Git简史

BitKeeper和Linux内核开源社区合作结束，收回Linux内核社区免费使用BitKeeper的权利

Git目标：

* 速度
* 简单的设计
* 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
* 完全分布式
* 有能力高效管理类似Linux内核一样的超大规模项目（速度和数据量）



## 1.3. Git是什么

**直接记录快照，而非差异比较**

Git对待数据的方式不同

其他大部分系统以文件变更列表的方式存储信息，这类系统（CVS，Subversion，Perforce，Bazaar等）将它们存储的信息看作是一组基本文件和每个文件随时间逐步积累的差异（`基于差异`的版本控制）

![存储每个文件与初始版本的差异。](https://git-scm.com/book/en/v2/images/deltas.png)

图4. 存储每个文件与初始版本的差异



Git把数据看做是对小型文件系统的一系列快照

在Git中，每当你提交更新或保存项目状态时，它基本上就就会对当时的全部文件创建一个快照并保存这个快照的索引。为了效率，如果文件没有修改，Git不再重新存储该文件，而只是保留一个链接指向之前存储的文件。Git对待数据更像是一个`快照流`

![Git 存储项目随时间改变的快照。](https://git-scm.com/book/en/v2/images/snapshots.png)

图5. 存储项目随时间改变的快照

**近乎所有操作都是本地执行**

无网络延迟开销

**Git保证完整性**

Git中所有的数据在存储前都计算校验和，然后以校验和来引用

计算校验和的机制叫做`SHA-1散列`（hash，哈希）.由40个十六进制字符组成的字符串，基于Git中文件的内容或目录结构计算出来的

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

**GIt一般只添加数据**

Git几乎不会执行任何可能导致文件不可恢复的操作

**三种状态**

* 已提交：表示数据已经安全保存在本地数据库中
* 已修改：表示修改了文件，但还没保存到数据库中
* 已暂存：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中

Git项目拥有三个阶段：工作区、暂存区以及Git目录

![工作区、暂存区以及 Git 目录。](https://git-scm.com/book/en/v2/images/areas.png)

图6. 工作目录、暂存区域以及Git仓库

`工作区`是对项目某个版本独立提取出来的内容。这些从Git残酷中提取出来的文件放在磁盘上供你使用和修改

`暂存区`是一个文件，保存了下次将要提交的文件列表信息，一般在Git仓库目录中。Git术语叫做：“索引”

`Git仓库目录`是git用来保存项目的元数据和对象数据库的地方。这是Git中最重要的部分，从其他计算机可从仓库时，复制的就是这里的数据



基本Git工作流程

1. 在工作区中修改文件
2. 将你想要下次提交的更改选择性暂存，这样只会将更改的部分添加到暂存区
3. 提交更新，找到暂存区，将快照永久性存储到Git目录

## 1.4. 命令行

命令行强于GUI工具

## 1.5. 安装Git

**在Windows上安装**

[下载地址](https://git-scm.com/download/win)

运行exe文件即可

## 1.6. 初次运行Git前的配置

Git自带一个Git config的工具来帮助设置控制Git外观和行为的配置变量

在 Windows 系统中，Git 会查找 `$HOME` 目录下（一般情况下是 `C:\Users\$USER` ）的 `.gitconfig` 文件。

可以通过下面命令查看所有的配置以及他们所在的文件

```shell
git config --list --show-origin
```

**用户信息**

设置用户名和邮件地址，每一个Git提交都会使用这些信息，它会写入到每一次提交中，不可更改：

```shell
$ git config --global user.name "wudiguang"
$ git config --global user.email 1490727316@qq.com
```

--global 为全局配置，去掉则只在当前目录下的仓库生效



**文本编辑器**

设置使用不同的文本编辑器，如Emacs

```console
$ git config --global core.editor emacs
```

```shell
$ git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

**检查配置信息**

使用git config --list列出所有Git配置

```shell
C:\Users\wudiguang>git config --list
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=D:/softs/Git/git-setting/mingw64/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=false
pull.rebase=false
credential.helper=manager-core
credential.https://dev.azure.com.usehttppath=true
user.name=wudiguang
user.email=wudiguang@ewell.cc
```

查询Git中变量原始值，且可以看到在哪个文件最后设置了该值

```shell
C:\Users\wudiguang>git config --show-origin user.name
file:C:/Users/wudiguang/.gitconfig      wudiguang
```



## 1.7. 获取帮助

```shell
$ git help <verb>
$ git <verb> --help
$ man git-<verb>
```

如想要获得`git config`命令的手册

```shell
$ git help config
```

获取可用选项的快速参考

```shell
$ git add -h
usage: git add [<options>] [--] <pathspec>...

    -n, --dry-run         dry run
    -v, --verbose         be verbose

    -i, --interactive     interactive picking
    -p, --patch           select hunks interactively
    -e, --edit            edit current diff and apply
    -f, --force           allow adding otherwise ignored files
    -u, --update          update tracked files
    --renormalize         renormalize EOL of tracked files (implies -u)
    -N, --intent-to-add   record only the fact that the path will be added later
    -A, --all             add changes from all tracked and untracked files
    --ignore-removal      ignore paths removed in the working tree (same as --no-all)
    --refresh             don't add, only refresh the index
    --ignore-errors       just skip files which cannot be added because of errors
    --ignore-missing      check if - even missing - files are ignored in dry run
    --chmod (+|-)x        override the executable bit of the listed filesxxxxxxxxxx $ git add -husage: git add [<options>] [--] <pathspec>...    -n, --dry-run         dry run    -v, --verbose         be verbose    -i, --interactive     interactive picking    -p, --patch           select hunks interactively    -e, --edit            edit current diff and apply    -f, --force           allow adding otherwise ignored files    -u, --update          update tracked files    --renormalize         renormalize EOL of tracked files (implies -u)    -N, --intent-to-add   record only the fact that the path will be added later    -A, --all             add changes from all tracked and untracked files    --ignore-removal      ignore paths removed in the working tree (same as --no-all)    --refresh             don't add, only refresh the index    --ignore-errors       just skip files which cannot be added because of errors    --ignore-missing      check if - even missing - files are ignored in dry run    --chmod (+|-)x        override the executable bit of the listed filesconsole
```

## 1.8. 总结

2021-1-15 15:32:35


---

# Git基础

[TOC]

本章涵盖使用Git完成各种工作时将会使用到的基本命令

* 配置并初始化仓库（`repository`）
* 开始或停止跟踪（`track`）文件
* 暂存（`stage`）或提交（`commit`）更改
* 配置`Git`来忽略指定文件和文件模式
* 迅速简单地撤销错误操作
* 浏览项目历史版本以及不同提交（`commit`）之间的差异
* 向远程仓库推送（`push`）以及从远程仓库拉取（`pull`）文件

## 2.1. 获取Git仓库

* 将尚未进行版本控制的本地目录转换为`Git`仓库
* 从其他服务器克隆一个已存在的`Git`仓库

**在已存在目录中初始化仓库**

```shell
$ git init
```

该命令将创建一个名为`.git`的子目录，其中包含初始化Git仓库的所有必须文件

[Git内部原理](https://git-scm.com/book/zh/v2/ch00/ch10-git-internals)

```shell
$ git add *.java
$ git add README.md
$ git commit -m 'initial project version'
```

**克隆现有的仓库**

```shell
$ git clone https://github.com/libgit2/libgit2 mylibgit2
```

`Git`支持多种数据传输协议。

`https://`

`git://`

## 2.2. 记录每次更新到仓库

![Git 下文件生命周期图。](https://git-scm.com/book/en/v2/images/lifecycle.png)

图8. 文件的状态变化周期

**检查当前文件状态**

`git status`

查看哪些文件处于什么状态

```shell
E:\desktop\git\git-learn>git status
On branch master

No commits yet
-- 已暂存状态
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   Hello.java
-- 未跟踪文件
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        nice.txt
```

**跟踪新文件**

`精确地将内容添加到下一次提交中`

```shell
$ git add Hello.java
```



**暂存已修改的文件**

```shell
E:\desktop\git\git-learn>git status
On branch master

No commits yet
-- 添加未暂存
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   Hello.java
-- 已跟踪文件的内容发生了变化，但还没有放到暂存区。修改未暂存
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   Hello.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        nice.txt
```

**状态简览**

使用 `git status -s` 命令或 `git status --short` 命令

```shell
$ git status -s
E:\desktop\git\git-learn>git status -s
A  Hello.java
?? nice.txt
```

* ??：新添加的未跟踪文件
* A：新添加到暂存区中的文件
* M：修改过的文件

**忽略文件**

无需纳入Git的管理

创建`.gitignore`文件

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配（正则），它会递归地应用在整个工作区中。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

[数十种项目语言.gitignore](https://github.com/github/gitignore)

**查看已暂存和未暂存的修改**

比较工作目录中当前文件和暂存区域快照之间的差异

```shell
$ git diff
```

比较已暂存与最后一次提交的文件差异

```shell
$ git diff --staged
```



查看已经暂存起来的变化

```shell
$ git diff --cached
```



**提交更新**

提交前用`git status` 查看一下所需要的文件是不是都已暂存，再运行提交命令`git commit`

提交时记录的放在暂存区域的快照。任何还未暂存文件的仍然保持已修改状态，可以在下次提交时纳入版本管理。每一次运行提交操作，都是对你项目做一次快照，以后可以回到这个状态，或者进行比较

**跳过使用暂存区域**

```shell
git commit -a -m 'commit message'
```

`Git`会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过`git add`步骤

**移除文件**

1. 从已跟踪文件清单中移除（从暂存区域移除）
2. 提交

```shell
$ git rm
$ git rm log/\*.log
```

**移动文件**‘

```shell
$ git mv file_from file_to
-- 相当于下面三条命令
$ mv file_from file_to
$ git rm file_from
$ git add file_to
```

## 2.3. 查看提交历史

[详细文档](https://git-scm.com/book/zh/v2/ch00/log_options)

按时间先后顺序列出所有的提交，最近的更新排在最上面

```shell
$ git log
```

查看最近两次的提交，显示每次提交所引入的差异

```shell
$ git log -p -2
```



查看提交统计信息

```shell
$ git log --stat
```

列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些被移除或是添加了



使用不同于默认格式的方式展示提交历史

* oneline
* short
* full
* fuller
* format

```shell
$ git log --pretty=oneline
$ git log --pretty=format:"%h - %an, %ar : %s"
```

format参数

| 选项  | 说明                                          |
| ----- | --------------------------------------------- |
| `%H`  | 提交的完整哈希值                              |
| `%h`  | 提交的简写哈希值                              |
| `%T`  | 树的完整哈希值                                |
| `%t`  | 树的简写哈希值                                |
| `%P`  | 父提交的完整哈希值                            |
| `%p`  | 父提交的简写哈希值                            |
| `%an` | 作者名字                                      |
| `%ae` | 作者的电子邮件地址                            |
| `%ad` | 作者修订日期（可以用 --date=选项 来定制格式） |
| `%ar` | 作者修订日期，按多久以前的方式显示            |
| `%cn` | 提交者的名字                                  |
| `%ce` | 提交者的电子邮件地址                          |
| `%cd` | 提交日期                                      |
| `%cr` | 提交日期（距今多长时间）                      |
| `%s`  | 提交说明                                      |

当 `oneline` 或 `format` 与另一个 `log` 选项 `--graph` 结合使用时尤其有用。 这个选项添加了一些 ASCII 字符串来形象地展示你的分支、合并历史：

**限制输出记录数量**

`--since` 和 `--until` 这种按照时间作限制的选项很有用。 例如，下面的命令会列出最近两周的所有提交：

```shell
$ git log --since=2.weeks
```



## 2.4. 撤销操作

将暂存区中的文件提交（合并上一次提交）

修改上一次提交信息

```shell
$ git commit --amend
```

```shell
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```



**取消暂存的文件**

如何操作暂存区和工作目录中已修改的文件

取消暂存文件

```shell
$ git reset HEAD <file>
```



**撤销对文件的修改**

`记住：会丢失本地修改`

```shell
$ git checkout -- <file>
```



## 2.5. 远程仓库的使用

`远程仓库可以在你的本地主机上`

**查看远程仓库**

origin：仓库服务器的默认名字

```shell
$ git remote
```

查看仓库URL

```shell
$ git remote -v
```

**添加远程仓库**

```shell
$ git remote add <shortname> <url>
```



如想要拉取plan仓库中有但你没有的信息，可以使用 `git fetch origin`



**从远程仓库中抓取与拉取**

```shell
$ git fetch <remote>
```

这个命令会访问远程仓库，从中拉取所有你还没有的数据

`可能会产生冲突`



**推送到远程仓库**

`git push <remote> <branch>`

```shell
$ git push origin master
```



**查看某个远程仓库**

`git remote show <remote>`

列出远程仓库的 URL 与跟踪分支的信息。

```shell
E:\desktop\ewell\code\shxhyy-plan>git remote show origin
* remote origin
  Fetch URL: http://192.168.150.61:2080/shxhyyemryfz/EmrGroup/shxhyy-plan.git
  Push  URL: http://192.168.150.61:2080/shxhyyemryfz/EmrGroup/shxhyy-plan.git
  HEAD branch: master
  Remote branches:
    develop        tracked
    master         tracked
    masterback     tracked
    mastertest     tracked
    mastertestback tracked
    release        tracked
  Local branches configured for 'git pull':
    develop merges with remote develop
    master  merges with remote master
  Local refs configured for 'git push':
    develop pushes to develop (up to date)
    master  pushes to master  (local out of date)

```



**远程仓库的重命名与移除**

`git remote rename`

```shell
$ git remote rename pb paul
```

删除远程仓库，远程跟踪分支以及配置信息将被删除

```shell
$ git remote remove paul
```

## 2.6. 打标签

给历史中某一个提交打上标签，以示重要。如V1.0，V2.0

**列出标签**

`git tag`

按特定模式查找标签（需要 -l 或 -list选项）

```shell
$ git tag
$ git tag -l "v1.8.5*"
```

**创建标签**

两种标签

* 轻量级标签（lightweight）
* 附注标签（annotated）

附注标签包含打标签者的名字、电子邮件地址、日期时间以及标签信息



**附注标签**

```shell
$ git tag -a
$ git tag -a v1.4 -m "my version 1.4"
```

`git show`命令可以看到标签信息和与之对应的提交信息

```shell
git show v1.4
```

输出显示了打标签者的信息、打标签的日期时间、附注信息，然后显示具体的提交信息。



**轻量标签**

轻量标签本质上是将提交校验和存储到一个文件中，没有保存任何其他信息

```shell
$ git tag v1.4-lw
```

**后期打标签**

```shell
$ git tag -a v1.2 9fceb02
```

**共享标签**

`git push`命令不会传送标签到远程仓库服务器中。必须显式推送标签到共享服务器

```shell
$ git push origin v1.5
```

将把所有不在远程仓库服务器上的标签全部传送

```shell
$ git push origin --tags
```

**删除标签**

删除本地轻量标签

```shell
$ git tag -d v1.4-lw
```

更新远程标签

操作含义：将冒号前面的控制推送到远程标签名，从而删除标签

```shell
$ git push origin :refs/tags/v1.4-lw
```

直接删除远程标签

```shell
$ git push origin --delete <tagname>
```

**检出标签**

2021-1-15 17:39:10

https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE