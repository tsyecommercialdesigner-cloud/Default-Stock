---
title: Deployment 常用操作
created: 2026-08-05
tags:
  - Deployment
  - Kubernetes
  - Kubernetes_Deployment
---
---

# 定义与部署

## 创建 YAML 模板

执行命令 `kubectl create` 可以创建一个 Deployment 的 YAML 模板文件：

```bash
export out="--dry-run=client -o yaml"
kubectl create deploy qsk-deploy --image=rongruihouseholds/qsk-book:1.0 $out
```

## 定义 Deployment 对象

定义 `deploy.yml` ：

```yaml
# 2025 version
# Start with 5 replicas on app version 1.0 and no update settings
apiVersion: apps/v1
kind: Deployment
metadata:
  name: qsk-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      project: qsk-book
  template:
    metadata:
      labels:
        project: qsk-book
    spec: 
      containers:
      - name: qsk-pod
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        image: rongruihouseholds/qsk-book:1.0
```
 
术语说明：

> `Pod` 、 `instance` 、`replica` 这都指同一样东西——运行容器化应用的 Pod 实例，即“副本”。


## 部署 Deployment 对象

以指定文件的方式将 `deploy.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f deploy.yml
```


## 验证部署结果

检查现有的 Deployment：

```bash
kubectl get deployment
```

如果要指定名称进行检查：

```bash
kubectl get deployment ${deployname}
```

查看 Pod 状态：

```bash
kubectl get pod
```


## 删除 Deployment 对象

删除指定的 Deployment  ：

```bash
kubectl delete deployment ${deployname}
```

提醒：同时别忘记清理不用的 Service 。

---

# 扩缩容

## 声明式方法（推荐）

通过更新 `deploy.yml` 来声明一个新的期望状态（记得按 `Ctrl + S` 保存对文件的修改）。

然后使用该文件更新集群：

```bash
kubectl apply -f deploy.yml
```

## 命令式方法

运行命令：

```bash
kubectl scale --replicas 5 deployment/$(deployname)
```

或者

```bash
kubectl scale --replicas=5 deploy $(deployname)
```

>[!warning] 使用命令式进行扩缩容容易疏漏的地方
>由于 `deploy.yml` 中配置的 Pod 数量并未得到修改，这意味着如果你忘记了该事实，
>那么在你下次通过修改 `deploy.yml` 来进行容器镜像版本更新时，可能产生意料之外的结果。

---

# 部署更新

## 部署更新的一般步骤

要将更新推送给应用，需要执行很多基本步骤：

1. 编写新版本应用的源代码；
2. 为新版本应用构建新的容器镜像；
3. 将新版本应用的容器镜像推送到容器仓库；
4. 在`deploy.yml` 中指定新版本镜像并配置更新设置；
5. 重新将 `deploy.yml` 发送给 Kubernetes API 服务器；
6. 观察这个更新过程，并对新版本应用进行测试。

## 配置 `rolling-update.yml`

以下是一个滚动更新的 `rolling-update.yml` 示例：

```yaml
# 2022 version
apiVersion: apps/v1
kind: Deployment
metadata:
  name: qsk-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      project: qsk-book
  minReadySeconds: 20
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    metadata:
      labels:
        project: qsk-book
    spec: 
      containers:
      - name: qsk-pod
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        image: rongruihouseholds/qsk-book:1.1
```

字段解释：

- `minReadySeconds: 20` ：更新每个副本后等待 20 秒再更新下一个副本
	- 实际上 Kubernetes 是删除现有的副本，并用一个运行新版本的全新副本来代替它们
- `type: RollingUpdate` ：强制 Kubernetes 以滚动更新方式执行对此 Deployment 的所有更新
- `maxSurge: 1` ：允许 Kubernetes 在更新操作中增加一个额外的 Pod （基于期望状态）
- `maxUnavailable: 0` ：禁止 Kubernetes 在更新期间减少 Pod 的数量（基于期望状态）

## 发送更新配置

以指定文件的方式将 `rolling-update.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f rolling-update.yml
```

## 监控和检查滚动更新

可以通过以下命令监控滚动更新工作的进展：

```bash
kubectl rollout status deployment ${deployname}
```