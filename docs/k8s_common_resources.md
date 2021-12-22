# 私有化部署

- [私有化部署](#私有化部署)
  - [一、mysql-私有化部署](#一mysql-私有化部署)
    - [1. mysql-deployment](#1-mysql-deployment)
    - [2. mysql-service](#2-mysql-service)
    - [3. mysql-pvc](#3-mysql-pvc)
    - [4. mysql-pv](#4-mysql-pv)
    - [5. mysql-pvc-deployment](#5-mysql-pvc-deployment)
    - [6. mysql-configmap](#6-mysql-configmap)
    - [7. mysql-configmap-deployment](#7-mysql-configmap-deployment)
    - [8. mysql-secret](#8-mysql-secret)
    - [9. 探针](#9-探针)
  - [二、常用命令](#二常用命令)
    - [删除资源](#删除资源)
    - [启动资源](#启动资源)
      - [其他命令](#其他命令)
    - [10. springboot](#10-springboot)
  - [三、springboot 项目私有化部署：](#三springboot-项目私有化部署)
    - [1. 部署流程](#1-部署流程)
    - [2. mysql-deployment.yaml](#2-mysql-deploymentyaml)

参考文档路径: https://segmentfault.com/a/1190000014966962

kubectl 常用命令总结 https://www.cnblogs.com/klvchen/p/9585746.html

## 一、mysql-私有化部署

### 1. mysql-deployment

```yaml
# kubectl create -f mysql-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "5555"
```
### 2. mysql-service

```yaml
# kubectl create -f mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    nodePort: 30006
    protocol: TCP
    targetPort: 3306
  selector:
    app: mysql
```

* 通过本机访问
* 集群内部通过mysql service访问
* 集群外部，可通过任何一个K8S节点访问数据库

`kubectl exec -it mysql-8495658ff4-sc7rz -- mysql -uroot -p5555`

pvc storageClassName 属性设置

### 3. mysql-pvc

```yaml
# kubectl create -f mysql-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

### 4. mysql-pv

```yaml
# kubectl create -f mysql-pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

### 5. mysql-pvc-deployment
增加pvc后调整deployment

```yaml
# kubectl create -f mysql-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "5555"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql
```

### 6. mysql-configmap
增加 configmap，自定义配置

```yaml
# kubectl create -f mysql-config.yaml
apiVersion: v1
metadata:
  name: mysql-config
data:
  custom.cnf: |
        [mysqld]
        default_storage_engine=innodb
        skip_external_locking
        lower_case_table_names=1
        skip_host_cache
        skip_name_resolve
kind: ConfigMap
```

### 7. mysql-configmap-deployment
将 configmap 挂载到容器

```yaml
# kubectl create -f mysql-configmap-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "5555"
        volumeMounts:
        - name: mysql-config
          mountPath: /etc/mysql/conf.d/
      volumes:
        - name: mysql-config
          configMap:
            name: mysql-config
```

### 8. mysql-secret
加密敏感数据

```yaml
# kubectl create -f mysql-secret.yaml
# echo -n 5555 | base64
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pwd
data:
  mysql-root-pwd: NTU1NQ==
  mysql-app-user-pwd: NTU1NQ==
  mysql-test-user-pwd: NTU1NQ==
```

mysql-secret-deployment.yaml

```yaml
# kubectl create -f mysql-secret-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
                name: mysql-user-pwd
                key: mysql-root-pwd
        volumeMounts:
        - name: mysql-config
          mountPath: /etc/mysql/conf.d/
      volumes:
        - name: mysql-config
          configMap:
            name: mysql-config
```

### 9. 探针
容器健康检查

```yaml
# kubectl create -f mysql-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
                name: mysql-user-pwd
                key: mysql-root-pwd
        volumeMounts:
        - name: mysql-config
          mountPath: /etc/mysql/conf.d/
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - "-c"
            - MYSQL_PWD="${MYSQL_ROOT_PASSWORD}"
            - mysql -h 127.0.0.1 -u root -e "SELECT 1"
          initialDelaySeconds: 30
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - "-c"
            - MYSQL_PWD="${MYSQL_ROOT_PASSWORD}"
            - mysql -h 127.0.0.1 -u root -e "SELECT 1"
          initialDelaySeconds: 10
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 3
      volumes:
        - name: mysql-config
          configMap:
            name: mysql-config
```

## 二、常用命令
### 删除资源
```shell
kubectl delete deployment mysql
kubectl delete service mysql
kubectl delete configmap mysql-config
kubectl delete pvc mysql
```

### 启动资源
```shell
kubectl create -f mysql-config.yaml
kubectl create -f mysql-deployment.yaml
kubectl create -f mysql-service.yaml
```

#### 其他命令
```shell
kubectl get pod
kubectl describe deployment mysql
kubectl describe pod mysql
// 进入容器 mysql
kubectl exec -it mysql-77fdd66d54-cjt2x -- mysql -uroot -p5555
// 进入容器
kubectl exec -it k8s-example-85bdcf9797-85ldq -- bin/bash
curl -XGET http://localhost:8080/
kubectl get pod k8s-example-hz99h -o yaml

docker run 
--name k8s-example 
-d -p 8080:8080 
k8s-example:1.0.0

// tag
docker tag ed97e21c09c9 xxx/k8s-example:1.0.0
// 推送
docker push xxx/k8s-example:1.0.0
// 导出表结构文件
mysqldump -uroot  -p --skip-extended-insert --complete-insert -t nlp_knowledge_library example > /Users/wudiguang/code/k8s/structure.sql
mysqldump -uroot -p  --opt  -d  nlp_knowledge_library example > /Users/wudiguang/code/k8s/structure.sql
// 导出mysql insert语句
mysqldump -h 127.0.0.1 -P 3306 -uroot  -p  --skip-extended-insert --complete-insert -t  db_name example  > /Users/wudiguang/code/k8s/data.sql
```
### 10. springboot
* 1. springboot-rs.yaml
创建rs，部署springboot项目

```yaml
# kubectl create -f k8s-example-rs.yaml
apiVersion: v1
kind: ReplicationController
metadata:
 name: k8s-example  #必选，资源名称
spec:
 # 节点数，设置为多个可以实现负载均衡效果
 replicas: 1
 selector:
   app: k8s-example
 template:
   metadata:
     labels:
       app: k8s-example
   spec:
     containers:
     - name: k8s-example
       #镜像名
       image: xxx/k8s-example:1.0.0
       #本地有镜像就不会去仓库拉取
       imagePullPolicy: IfNotPresent
       ports:
       - containerPort: 8080
```

* 2. springboot-svc.yaml
创建svc对外暴露服务

```yaml
# vi k8s-example-svc.yaml
# kubectl create -f k8s-example-svc.yaml 
apiVersion: v1
kind: Service
metadata:
 name: k8s-example
spec:
 type: NodePort
 ports:
 - port: 8080
   targetPort: 8080
   # 节点暴露给外部的端口（范围必须为30000-32767）
   nodePort: 30001
 selector:
   app: k8s-example
```

* 3. springboot + mysql
springboot + mysql 项目部署

```yaml
# kubectl create -f k8s-example-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-example  #必选，资源名称
spec:
  # 节点数，设置为多个可以实现负载均衡效果
  replicas: 1
  selector:
    matchLabels:
      app: k8s-example
  template:
    metadata:
      labels:
        app: k8s-example
    spec:
      containers:
      - name: k8s-example
        #镜像名
        image: xxx/k8s-example:1.0.0
        #本地有镜像就不会去仓库拉取
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        env:
        - name: MYSQL_SERVICE_HOST
          value: "mysql"
        - name: MYSQL_USERNAME
          value: root
        - name: MYSQL_PASSWORD
          value: "5555"
```

## 三、springboot 项目私有化部署：
### 1. 部署流程

1. 修改项目中 application.yaml 环境变量，把本地配置改为环境变量配置，如 ${MYSQL_USERNAME}，${MYSQL_PASSWORD} 等
2. 修改项目中指向其他资源的地址，如 mysql 的地址，对应的 mysql 地址为 mysql-service 的名字
3. 打包 springboot 项目，mvn clean package -Dmaven.test.skip=true
4. 打包 docker 镜像，执行 ./build.sh 脚本或者配置一个启动的 docker configurations
5. 推送 docker 镜像到 docker 镜像仓库（目前使用：xxx/）
6. 配置 mysql（deployment，pvc），springboot（deployment）的资源文件，分别 apply
7. 进入 springboot pod 内部，curl 相应的 http 接口，返回数据正常则结束
8. 完成 springboot 项目私有化部署

代码源文件

### 2. mysql-deployment.yaml
可用的 mysql yaml 资源文件

```yaml
# kubectl create -f mysql-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: mysql
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "5555"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql
```

3. pvc

```yaml
# kubectl create -f mysql-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

4. pv

```yaml
# kubectl create -f mysql-pv.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

5. springboot-deployment.yaml
可用的 springboot yaml 资源文件

```yaml
# kubectl create -f k8s-example-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-example  #必选，资源名称
spec:
  # 节点数，设置为多个可以实现负载均衡效果
  replicas: 1
  selector:
    matchLabels:
      app: k8s-example
  template:
    metadata:
      labels:
        app: k8s-example
    spec:
      containers:
      - name: k8s-example
        #镜像名
        image: xxx/k8s-example:1.0.0
        #拉取最新镜像
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        env:
        - name: MYSQL_SERVICE_HOST
          value: "mysql"
        - name: MYSQL_USERNAME
          value: root
        - name: MYSQL_PASSWORD
          value: "5555"
```

踩坑记录
* 部署最新上传的镜像最新修改未生效。imagePullPolicy(镜像拉取策略)：IfNotPresent 改为 Always