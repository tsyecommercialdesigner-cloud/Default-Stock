---
title: Service 常用操作
created: 2026-08-05
tags:
  - Kubernetes
  - Kubernetes_Service
  - Service
---
---
# 创建 `svc.yml`

## 使用命令创建

为 ngx-dep 对象生成 Service 对象，命令如下：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 --target-port=80 $out
```

如果 Service 对象的映射端口和目标端口相同，例如都是 80，那么可以省略“--target-port”参数：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 $out
```

生成的 Service YAML 模板文件如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ngx-svc

spec:
  selector:
    app: ngx-dep

  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

其中，`type` 字段的值是默认的 `ClusterIP` ：

```yaml
apiVersion: v1
kind: Service
...
spec:
  ...
  type: ClusterIP
```

如果在`kubectl expose`命令加上参数 `--type=NodePort`，或者在  Service 的 YAML 文件里添加字段 `type: NodePort`，那么 Service 不仅会对后端的 Pod 做负载均衡，还会在集群中的每个节点上创建一个独立的端口来对外提供服务：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 --target-port=80 --type=NodePort $out
```

可选字段包括：

- `--type=ClusterIP`
- `--type=ExternalName`
- `--type=LoadBalancer`
- `--type=NodePort`

其中“`ExternalName`”和“`LoadBalancer`”一般由云服务商提供。



## 从模板创建

### 本地

定义 `svc-local.yml`：

```yaml
# 2026 version
apiVersion: v1
kind: Service
metadata:
  name: svc-local
spec:
  type: LoadBalancer
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    project: qsk-book
```


### 云端

定义 `svc-cloud.yml`：

```yaml
# 2022 version
apiVersion: v1
kind: Service
metadata:
  name: cloud-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    project: qsk-book
```

---

# 创建 Service 对象

以指定文件的方式将 `svc.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f svc-local.yml
```

云端则改成：

```bash
kubectl apply -f svc-cloud.yml
```

验证 Service 部署结果：

```bash
kubectl get svc
```

---

# 删除 Service 对象

删除指定的 Service ：

```bash
kubectl delete svc ${svcname}
```
