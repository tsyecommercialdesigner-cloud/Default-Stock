---
title: Control Plane 和 Worker Nodes
created: 2026-08-07
source: Cherry Studio
tags: Kubernetes
---
---

在 Kubernetes 官方文档和社区语境中：

- **控制面**：**Control Plane**
- **工作节点**： **Worker Nodes**

---

# Control Plane（控制面）

这是 Kubernetes 的正式核心术语。它负责集群的全局决策和状态管理，例如：

- 调度 Pod 到哪个节点
- 维护期望状态与实际状态一致
- 管理 API、认证、授权等

典型组件包括：

- `kube-apiserver`
- `etcd`
- `kube-scheduler`
- `kube-controller-manager`
- `cloud-controller-manager`（可选）

官方文档中也常把它称为：

```text
Kubernetes control plane
```

---

# Worker Nodes（工作节点）

**Data Plane** （数据面）在 Kubernetes 网络、服务流量和 Service Mesh 等上下文中很常见，泛指实际承载或转发业务流量的部分，例如：

- Node 上的 `kube-proxy`
- CNI 网络插件（如 Calico、Cilium、Flannel）
- Ingress Controller
- Service Mesh 的 sidecar / ztunnel / Envoy
- 承载工作负载的 Pod 和 Node 网络路径

但工作节点在 Kubernetes 核心架构的官方文档更常用的对应分类不是 “data plane”，而是：

```text
Control Plane Components
Node Components
```

其中节点侧组件包括：

- `kubelet`
- `kube-proxy`
- `Container Runtime`

因此，较严谨地描述 Kubernetes 标准架构的说法是：

```text
Control Plane（控制面）
Worker Nodes / Node Components（工作节点 / 节点组件）
```

如果讨论网络流量转发或服务网格，则通常说：

```text
Control Plane（控制面）
Data Plane（数据面）
```
