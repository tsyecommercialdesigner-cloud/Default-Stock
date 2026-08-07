---
title:
created: 2026-08-07
tags:
---
---

# 获取信息

使用 `kubectl api-resources` 命令查看当前 Kubernetes 版本支持的所有对象：

```bash
kubectl api-resources
```

使用`kubectl get ns` 命令查看当前集群里的所有名字空间，即 API 对象的分组：

```bash
kubectl get ns
```

使用 `kubectl` 命令时加上参数 `--v=9`，就会显示详细的命令行执行过程：

```bash
kubectl get pod --v=9
```
