---
title: nodePort、port 与 targetPort
created: 2026-08-05
tags:
  - Kubernetes
  - kubectl
---
---
# nodePort、port 与 targetPort 

targetPort 很好理解，就是 Kubernetes 把流量转发给后端 Pod 的端口地址，
而 port 可以理解成“前台号码”， nodePort 则相当于“大楼入口号码”。

```
Node:31111 → Service:80 → Pod:8080
```

- `Service:80`：Kubernetes 集群内部的“统一服务号码”。
    
    - 集群中的其他 Pod 访问 `svc-local:80`。
    - Kubernetes 再把流量转发给后端 Pod 的 `targetPort`，例如 `8080`。
- `Node:31111`：Kubernetes 节点对外开放的入口端口。
    
    - 外部流量先到某个节点的 `31111`。
    - 节点再将流量交给这个 Service 的 `80`。
    - 最后由 Service 转到 Pod 的 `8080`。

完整链路：

```
浏览器 / 外部客户端
    ↓
节点 IP:31111        ← nodePort
    ↓
svc-local:80         ← service port
    ↓
Pod IP:8080          ← targetPort / 应用端口
```

因此它们不是两个独立服务，而是同一条转发链路的不同层级端口。

例如：

```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 31111
```

意思是：

```
节点的 31111
  → Service 的 80
  → 应用容器的 8080
```

>[!info] 为什么不直接让 NodePort 也等于 `80`？
因为 NodePort 默认只能使用高位端口范围（通常 `30000–32767`），避免与节点操作系统的常用端口冲突。`80` 是给集群内部 Service 使用的稳定、易记端口；`31111` 是节点暴露给外部网络的实际入口。


---
# CLUSTER-IP 、 EXTERNAL-IP 与 PORT(S)

## kubectl get svc

对于：

```bash
kubectl get svc
```

它会返回类似于：

```
NAME        TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)
svc-local   LoadBalancer   10.96.35.53   172.18.0.5    80:31111/TCP
```

| 字段             | 含义                                                                                            |
| -------------- | --------------------------------------------------------------------------------------------- |
| `CLUSTER-IP`   | Kubernetes 集群内部的虚拟 Service 地址。只有集群内的 Pod/节点可访问，例如 `http://10.96.35.53:80`。                    |
| `EXTERNAL-IP`  | `LoadBalancer` 由集群外部负载均衡器分配的入口地址。你的 `172.18.0.5` 属于 Docker/Kind 内部网络地址，不一定能被 Windows 浏览器直接访问。 |
| `80:31111/TCP` | Service 端口为 `80`，NodePort 为 `31111`，协议是 TCP。                                                  |

流量关系：

```
集群内 Pod
  → 10.96.35.53:80       (ClusterIP)
  → Service
  → Pod 的 targetPort

节点网络
  → <节点IP>:31111       (NodePort)
  → Service:80
  → Pod 的 targetPort

集群外入口
  → 172.18.0.5:80        (External-IP / LoadBalancer)
  → Service:80
  → Pod 的 targetPort
```

其中：

- `port: 80`：Service 自己对外提供的端口。
- `nodePort: 31111`：节点上开放的高位端口。
- `targetPort`：最终转发到应用容器监听的端口，例如 `8080`。

所以若 YAML 是：

```yaml
port: 80
targetPort: 8080
nodePort: 31111
```

则含义是：

```
Service:80 / Node:31111 → 容器:8080
```

## 为什么 EXTERNAL-IP 不能被浏览器直接访问

因为 `EXTERNAL-IP` 是 Docker/Kind 虚拟网络里的负载均衡器 IP，不是 Windows 主机可直接路由的地址。

你的实际访问链路是：

```
Windows 浏览器
  → localhost:8080
  → Docker Desktop 的端口映射（127.0.0.1:8080）
  → Docker/Kind 负载均衡器（172.18.0.5:8080）
  → Kubernetes Service
  → Pod:8080
```

`kubectl get svc` 中的：

```
EXTERNAL-IP: 172.18.0.5
```

表示 Kubernetes 的 LoadBalancer 获得了这个内部 Docker 网络地址；它不等于“Windows 浏览器一定能直接访问的公网/宿主机地址”。

而你在 `docker ps` 中看到：

```
127.0.0.1:8080->8080/tcp
```

说明 Docker Desktop 额外把该负载均衡器的 `8080` 端口发布到 Windows 本机回环地址，因此：

```
http://localhost:8080
```

可以访问。

在 Windows + Docker Desktop 环境里，应优先使用 `localhost:8080`；
不要把 `EXTERNAL-IP` 当成可从 Windows 主机直接访问的地址。

## Docker 启动时的端口转发规则

假如我们执行了：

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

并获得结果：

```plain
127.0.0.1:8080->8080/tcp
```

含义是：

```plain
Windows 本机 127.0.0.1:8080
        ↓
Docker 容器内部 8080/TCP
```

同时代表 Docker Desktop 额外把负载均衡器的 `8080` 端口发布到 Windows 本机回环地址，使得
`localhost:8080`可以在 Windows 宿主机的浏览器中直接访问；

拆开看：

- `127.0.0.1`：只允许本机访问；同一局域网的其他设备无法访问。
- 左侧 `8080`：Windows 主机端口，因此浏览器用 `http://localhost:8080`。
- `->`：Docker 的端口转发关系。
- 右侧 `8080/tcp`：容器内部服务监听的 TCP 端口。

等价于 Docker 启动容器时使用：

```bash
docker run -p 127.0.0.1:8080:8080 ...
```

如果显示为：

```plain
0.0.0.0:8080->8080/tcp
```

则表示 Windows 主机所有网卡都可访问该端口，局域网其他机器也可以通过“你的电脑 IP:8080”访问。

---