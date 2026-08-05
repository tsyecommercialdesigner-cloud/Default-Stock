---
title: Pod
created: 2026-08-05
tags:
  - Kubernetes
  - kubectl
  - Kubernetes_Pod
  - Pod
---
---
# 将 Pod 部署到 Kubernetes

## 定义 `pod.yml`

```yaml
# 2022 version
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
  labels:
    project: qsk-book
spec:
  containers:
    - name: web-ctr
      image: rongruihouseholds/qsk-book:1.0
      ports:
        - containerPort: 8080
```

## 部署 Pod

首先获取当前 Pod 信息：

```bash
kubectl get pods
```

以指定文件的方式将 `pod.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f pod.yml
```

验证 Pod 部署结果：

```bash
kubectl get pods
```

（可选）获取更详细的信息：

```bash
kubectl describe pod ${podname}
```

---
# 验证 Kubernetes 集群

## 获取信息

获取节点信息：

```bash
kubectl get nodes
```

获取 Pod 信息：

```bash
kubectl get pods
```

## 删除

删除指定的 Pod ：

```bash
kubectl delete pod ${podname}
```

删除 Nodes ：

> - 删除工作节点对由单一节点组成的 Docker Desktop 不适用；
> - 对于云提供商的 Kubernetes 集群，需要在云提供商的控制台界面进行相关操作。

## 切换上下文

获取 `config` 的 `context` 信息：

```bash
kubectl config get-contexts
```

切换到指定的 `context`：

```bash
kubectl config use-context ${contextname}
```

---