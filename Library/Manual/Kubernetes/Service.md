---
title: Service
created: 2026-08-05
tags:
  - Kubernetes
  - Kubernetes_Service
  - Service
---
---
## 定义 `svc.yml`

### （本地）`svc-local.yml`

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


### （云端）`svc-cloud.yml`

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


## 部署 `svc.yml`

以指定文件的方式将 `service.yml` 发送给 Kubernetes API 服务器：

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

## 删除 svc

删除指定的 Service ：

```bash
kubectl delete svc ${svcname}
```
