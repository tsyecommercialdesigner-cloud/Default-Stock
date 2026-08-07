---
title: DaemonSet 常用操作
created: 2026-08-07
tags:
  - DaemonSet
---
---

# 创建 YAML 模板

Kubernetes 不提供自动创建 DaemonSet YAML 模板的功能，即不能使用命令“`kubectl create`”直接创建 DaemonSet 对象，但是我们可以用“`kubectl create`”先创建一个 Deployment 对象，然后把“kind”改成“DaemonSet”，再删除“`spec.replicas`”就行了，如下是一个例子：

```bash
export out="--dry-run=client -o yaml"
kubectl create deploy redis-ds --image=redis:7-alpine $out \
  | sed 's/Deployment/DaemonSet/g' \
  | sed '/replicas/d'
```


还可以从 Kubernetes 官网上复制任意一份 DaemonSet 的 YAML 示例，去掉多余部分做成一份 YAML 模板文件：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis-ds
  labels:
    app: redis-ds

spec:
  selector:
    matchLabels:
      name: redis-ds

  template:
    metadata:
      labels:
        name: redis-ds
    spec:
      containers:
      - image: redis:7-alpine
        name: redis
        ports:
        - containerPort: 6379
```

---

# 在控制面节点创建守护 Pod 


把 YAML 文件发送给 Kubernetes API 服务器：

```bash
kubectl apply -f ds.yml
```

控制面节点默认有污点“`node-role.kubernetes.io/control-plane`”，效果是“`NoSchedule`”，通常 Pod 不能容忍任何污点，所以默认无法将守护 Pod 调度到加上了污点属性的控制面节点上。

## 去除 `taint` 的方法

第一种方法是去除控制面节点上的污点。操作节点的污点属性需要使用命令“`kubectl taint`”，并指定节点名、污点名和污点效果，如果是去掉污点要额外加上一个“-”（减号）。

例如去掉控制面节点的“NoSchedule”效果，命令如下：

```bash
kubectl taint node k8s-master \
  node-role.kubernetes.io/control-plane:NoSchedule-
```

但这种方法修改的是节点的状态，影响面比较大，可能会导致很多 Pod 被调度到这个节点上运行。

如果要重新为控制面节点把污点添加回来，只需重新执行上述命令的去掉尾部"`-`"的版本。

## 增加 `tolerations` 的方法

第二种方法是保留节点的污点，为需要的 Pod 添加容忍度，只允许某些 Pod 被调度到个别节点上，实现精准调度。

在 `ds.yml` 中为 Pod 添加字段“tolerations”：

```yaml
tolerations:
- key: node-role.kubernetes.io/control-plane
  effect: NoSchedule
  operator: Exists
```

再运行命令“`kubectl apply -f ds.yml`”重新部署设置了容忍度的 DaemonSet。

---

# 管理静态 Pod

静态 Pod 的 YAML 文件默认放在节点的“`/etc/kubernetes/manifests`”目录下，这个目录是 Kubernetes 的专用目录。

运行以下命令查看 Kubernetes 集群控制面节点的目录：

```bash
ls /etc/kubernetes/manifests
```

使用静态 Pod 的方法是：

> 编写一个 YAML 文件放到“`/etc/kubernetes/manifests`”目录下。

节点的 kubelet 会定期检查目录里的文件，发现变化就会调用容器运行时创建或者删除静态 Pod。