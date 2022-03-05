# kubectl命令

```describe
作者： 吴第广
日期： 2021年12月20日
```

kubectl是操作k8s集群的命令行工具，安装在k8s的master节点。

kubectl在$HOME/.kube目录中查找一个名为config的文件, 可以通过设置Kubeconfig环境变量或设置--kubeconfig来指定其他的kubeconfig文件。

kubectl通过与apiserver交互可以实现对k8s集群中各种资源的增删改查。


![kubernetes](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/blog/kubernetes.jpg)


## 一、命令格式
`kubectl [command] [type] [name] [flags]`

## 二、常用命令

### 2.1 使用nginx镜像创建deployment
`kubectl create deployment nginx --image=nginx`

### 2.2 对外暴露端口
`kubectl expose deployment nginx --port=80 --type=NodePort`

### 2.3 查看资源

`kubectl get pod`

`kubectl get pod,svc`

`kubectl get pods -o wide`

### 2.4 尝试运行，并不会真正的创建deploy
`kubectl create deployment web --image=nginx -o yaml --dry-run`

### 2.5 创建deploy，输出到一个文件中
`kubectl create deployment web --image=nginx -o yaml --dry-run > hello.yaml`

### 2.6 查看一个已经部署的deploy
`kubectl get deploy`

### 2.7 查看后导出nginx的配置
`kubectl get deploy nginx -o=yaml --explort > nginx.yaml`

### 2.8 给节点新增标签
`kubectl label node node1 env_role=prod`

### 2.9 查看污点情况
`kubectl describe node k8smaster | grep Taint`

### 2.10 给节点添加污点
`kubectl taint node k8snode1 env_role=yes:NoSchedule`

### 2.11 删除污点
`kubectl taint node k8snode1 env_role:NoSchedule`

### 2.12 调整副本数量
`kubectl scale deployment web --replicas=5`

## 三、基础命令

1. `create` 通过文件名或标准输入创建资源
2. `expose` 将一个资源公开为一个新的 Service
3. `run` 在集群中运行一个特定的镜像
4. `set` 在对象上设置特定的功能
5. `get` 显示一个或多个资源
6. `explain` 文档参考资料
7. `edit` 使用默认的编辑器编辑一个资源
8. `delete` 通过文件名、标准输入、资源名称或标签来删除资源


## 四、部署命令

1. `rollout` 管理资源的发布
2. `rolling-update` 对给定的赋值控制器滚动更新
3. `scale` 扩缩容Pod数量，Deploy、RS、RC或Job
4. `autoscale` 创建一个自动选择扩缩容并设置Pod数量

## 五、集群管理命令

1. `certificate` 修改证书资源
2. `cluster-info` 显示集群信息
3. `top` 显示资源（CPU/内存）
4. `cordon/uncordon` 标记节点不可/可调度
5. `drain` 驱逐节点上的应用，准备下线维护
6. `taint` 修改节点taint标记


## 六、故障和调试命令

1. `describe` 显示特定资源或资源组的详细信息
2. `logs` 在一个Pod中打印一个容器日志，如果Pod中存在多个容器，则使用 -c 指定容器名
3. `attach` 附加到一个运行的容器
4. `exec` 执行命令到容器
5. `port-forward` 转发一个或多个
6. `proxy` 运行一个proxy到kubernetes API Server
7. `cp` 拷贝文件或目录到容器中
8. `auth` 检查授权

## 七、其他命令

1. `apply` 通过文件名或标准输入对资源应用配置
2. `patch` 使用补丁修改、更新资源的字段
3. `replace` 通过文件名或标准输入替换一个资源
4. `convert` 不同的API版本之间转换配置
5. `label` 更新资源上的标签
6. `annotate` 更新资源上的注释
7. `completion` 用于实现kubectl工具自动补全
8. `api-version` 打印受支持的API版本
9. `config` 修改kubeconfig文件(用用户访问API，比如配置认证信息)
10. `help` 所有命令帮助
11. `plugin` 运行一个命令行插件
12. `version` 打印客户端和服务版本信息

## 八、原图
![Kubetcl命令](https://cdn.jsdelivr.net/gh/wudg/picgo@master/images/blog/Kubetcl命令-1.jpg)

`原文:`
* [learning_mind_map](https://github.com/0voice/learning_mind_map)

`本文收录于:`
* [Github仓库:wudg/blog](https://github.com/wudg/blog)
* [Gitee仓库:wudg/blog](https://githee.com/wudg/blog)