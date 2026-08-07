---
title: namespace 常用操作
created: 2026-08-07
tags:
  - Namespace
  - Kubernetes_Domain
---
---

# 获取信息

## 查看名字空间

查看当前集群里的所有名字空间：

```bash
kubectl get ns
```

## 试验 DNS 域名

先运行命令“`kubectl exec`”进入 Pod：

```bash
kubectl exec -it $(podname) -- sh
```

然后通过 curl 访问“ngx-svc”“ngx-svc.default”等域名：

```bash
curl $(api-object-domain)
```

Service 对象域名的完整形式是“`对象.名字空间.svc.cluster.local`”，通常可以简写为“`对象.名字空间`”，甚至是“`对象名`”（默认使用对象所在的名字空间）。

Kubernetes 为每个 Pod 也分配了域名，形式是“`IP地址.名字空间.pod.cluster.local`”，但需要把 IP 地址里的“`.`”改成“`-`”，例如地址“10.10.0.11”对应的域名是“`10-10-0-11.default.pod`”。