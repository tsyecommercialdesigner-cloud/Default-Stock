---
title: Pod、Node 常用操作
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

以指定文件的方式将 `pod.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f pod.yml
```

记得在部署前检查当前节点和 Pod 状态，在部署实施后进行结果验证。

---

# 获取信息
## 获取一般信息 

获取节点信息：

```bash
kubectl get nodes
```

查看 Pod 列表和运行状态：

```bash
kubectl get pods
```

（可选）获取更详细的信息：

```bash
kubectl describe pod $(podname)
```

显示 Pod 的运行日志信息：

```bash
kubectl logs $(podname)
```

## 持续获取监控信息

在获取信息的命令后添加 `-w` 可以对信息进行实时追踪，且不会主动退出：

```bash
kubectl get pod -w
```


---
# 文件操作

## 删除

删除指定的 Pod ：

```bash
kubectl delete pod $(podname)
```

删除 Nodes ：

> - 删除工作节点对由单一节点组成的 Docker Desktop 不适用；
> - 对于云提供商的 Kubernetes 集群，需要在云提供商的控制台界面进行相关操作。

## 复制文件

例如有一个 `a.txt` 文件，可以使用以下命令把这个文件拷贝到 Pod `ngx-pod`的 `/tmp` 目录里：

```bash
kubectl cp a.txt ngx-pod:/tmp
```

## 创建 YAML 模板

使用 kubectl 的两个特殊参数 `--dry-run=client` 和 `-o yaml`，前者表示空运行，后者表示生成 YAML 格式，组合使用会让 kubectl 不会有实际的创建动作，只生成 YAML 文件。

创建 Pod：

```bash
kubectl run
```

例如，想要生成一个 Pod 的 YAML 样板示例，在 `kubectl run` 后面加上这两个参数：

```bash
kubectl run ngx --image=nginx:alpine --dry-run=client -o yaml
```

注意：“`kubectl run`”命令只能创建 Pod，要创建 Pod 以外的其他 API 对象需要使用“`kubectl create`”命令，并在命令中加上对象的类型：

```bash
kubectl create $(objecttype)
```

例如，用 busybox 创建一个“echo-job”，命令如下：

```bash
export out="--dry-run=client -o yaml"    # 定义 Shell 变量
kubectl create job echo-job --image=busybox $out
```

如果不知道资源对象都有哪些，可以使用以下命令来枚举 API 资源对象：

```bash
kubectl api-resources
```

如果不理解字段的含义，可以使用以下命令来解释字段说明：

```bash
kubectl explain $(fieldname)
```


---

# 运行指令

## 运行 Shell 命令

`kubectl exec` 的命令格式与 Docker 有所区别，需要在 Pod 后面加上 `--`，把 kubectl 的命令与 Shell 命令分隔开，使用的时候需要注意：

```bash
kubectl exec -it ngx-pod -- sh
```

如果 ngx-pod 不在当前默认命名空间，需要加上 `-n <命名空间>`。

---

# 切换上下文

获取 `config` 的 `context` 信息：

```bash
kubectl config get-contexts
```

切换到指定的 `context`：

```bash
kubectl config use-context $(contextname)
```

---