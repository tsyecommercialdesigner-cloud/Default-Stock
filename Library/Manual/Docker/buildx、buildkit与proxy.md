---
title: buildx、buildkit与proxy
created: 2026-08-05
tags:
  - Buildx
  - BuildKit
  - Docker
---
# Windows + Docker Desktop + v2rayN：代理、Kubernetes 与镜像构建一遍直达手册

## 适用情形

本方案适用于以下组合：

- Windows 上使用 **Docker Desktop（Linux containers）**；
- 使用 **v2rayN** 提供 HTTP/混合代理；
- Docker Hub、Docker Desktop 内置 Kubernetes 或 `npm install` 因网络受限、代理未生效、TLS 超时而失败；
- 需要在 Git Bash 中执行 `docker buildx build`，Dockerfile 的 `RUN npm install` 还要访问外网。

本文以 v2rayN 的“本地混合监听端口”为 `10888` 为例。若你的端口不同，按下面的公式替换：

```text
本机代理端口 = P
局域网/Docker 代理端口 = P + 2
```

即 `P=10888` 时，局域网/Docker 端口为 `10890`。

> 不适用于：未使用 Docker Desktop 的原生 Linux Docker、Windows Containers 模式，或无需代理即可稳定访问 Docker Hub 的网络。

---

## 一遍直达：正确配置与构建流程

### 1. 配置 v2rayN

打开 **设置 → 参数设置 → Core：基础设置**：

1. 设置“本地混合监听端口”为 `10888`；
2. 开启“允许来自局域网的连接”；
3. 开启“为局域网开启新的端口”；
4. 点击“确定”，然后在 v2rayN 托盘菜单中重启服务/核心。

此时不要期待 `10888` 改为监听全部网卡：它仍是本机端口。新增的 `10890` 才是 Docker 应使用的局域网端口。

在 PowerShell 验证：

```powershell
Get-NetTCPConnection -LocalPort 10890 -State Listen
Test-NetConnection 192.168.1.54 -Port 10890
```

第二条应显示：

```text
TcpTestSucceeded : True
```

`LocalAddress` 显示为 `::` 也是正常的，表示监听在 IPv6 通配地址；本例已经由 IPv4 连通性测试证明 Docker 可访问。

### 2. 配置 Docker Desktop 代理

进入 **Docker Desktop → Settings → Resources → Proxies**：

- **Docker Desktop proxy**：选 *Manual configuration*；
- HTTP 与 HTTPS 都填写：

  ```text
  http://host.docker.internal:10890
  ```

- **Containers proxy**：选 *Same as Docker Desktop proxy*，或同样手工填写这两个地址；
- Bypass/No proxy 列表中不要加入 `docker.io`、`*.docker.io`、`auth.docker.io`、`registry-1.docker.io`；
- 点击 **Apply & restart**。

验证代理能访问 Docker Hub 的认证服务（在 Git Bash 或 PowerShell 均可运行）：

```bash
curl.exe -sS -o NUL -w "%{http_code}\n" \
  -x http://192.168.1.54:10890 \
  "https://auth.docker.io/token?service=registry.docker.io&scope=repository%3Alibrary%2Fnode%3Apull"
```

返回 `200` 即代表代理正常。不要把 `401` 当作唯一正确结果：调用具体 token 接口时正常结果是 `200`。

### 3. 为 Buildx 客户端和构建容器分别设置代理

这是最容易遗漏、也最关键的一步。Docker 构建过程中有两种不同的网络位置：

| 网络位置 | 应使用的代理地址 | 原因 |
| --- | --- | --- |
| Git Bash 中的 Buildx 客户端 | `http://127.0.0.1:10888` | 运行在 Windows 主机上，可直接连接 v2rayN 的本机端口；它负责取得 Docker Hub OAuth token。 |
| Docker 构建容器内的 `npm install` | `http://host.docker.internal:10890` | 容器内的 `127.0.0.1` 是容器自己，必须通过 Docker Desktop 的宿主机名称访问 v2rayN 局域网端口。 |

先在 **当前 Git Bash 会话**中执行：

```bash
export HTTP_PROXY=http://127.0.0.1:10888
export HTTPS_PROXY=http://127.0.0.1:10888
export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"
export NO_PROXY=localhost,127.0.0.1
```

这些变量只影响当前 Git Bash 窗口；关闭窗口后自动失效。

创建一次可复用的 BuildKit 构建器：

```bash
docker buildx create \
  --name v2ray-builder \
  --driver docker-container \
  --driver-opt default-load=true \
  --driver-opt env.HTTP_PROXY=http://host.docker.internal:10890 \
  --driver-opt env.HTTPS_PROXY=http://host.docker.internal:10890 \
  --use --bootstrap
```

然后在项目目录构建：

```bash
docker buildx build \
  --builder v2ray-builder \
  --load \
  --no-cache \
  --build-arg HTTP_PROXY=http://host.docker.internal:10890 \
  --build-arg HTTPS_PROXY=http://host.docker.internal:10890 \
  -t tsyecommercialdesigner-cloud/qsk-book:1.0 .
```

`--load` 会将构建结果导入本地镜像仓库。确认镜像已存在：

```bash
docker image ls tsyecommercialdesigner-cloud/qsk-book:1.0
```

---

## 内置 Kubernetes：应如何处理

`docker pull kindest/node:v1.36.1` 成功，只能证明该一个节点镜像已下载。Docker Desktop 创建 Kubernetes 集群还要创建证书、控制面、网络和存储组件，并下载其他镜像。

先列出当前 Docker Desktop 需要的镜像：

```powershell
docker desktop kubernetes images list
```

若 Kubernetes provisioning mode 为 **kind**，当前示例所需的辅助镜像包括：

```powershell
docker pull docker/desktop-containerd-registry-mirror:v0.0.4
docker pull docker/desktop-cloud-provider-kind:v0.6.0
docker pull envoyproxy/envoy:v1.36.7
```

镜像标签以 `docker desktop kubernetes images list` 的实际输出为准。若首次创建已卡住、且不存在需保留的 Kubernetes 工作负载，可使用 Docker Desktop → **Troubleshoot → Reset Kubernetes cluster** 后重新创建。

---

## 踩过的坑与避雷清单

### 1. 只检查 `10888`，误以为 v2rayN 设置没有生效

**现象**：

```text
127.0.0.1  10888  Listen
```

**原因**：启用“为局域网开启新的端口”后，v2rayN 不会把原本的本机端口 `10888` 改为全网卡监听；它会额外创建 `10890`。

**正确做法**：检查和使用 `P+2` 的端口，即本例的 `10890`。

### 2. Docker Desktop 仍指向 `10888`

**现象**：

```text
connecting to host.docker.internal:10888 ... actively refused
```

**原因**：Docker Desktop 从虚拟网络访问 Windows 主机，不能访问 v2rayN 的回环专用 `127.0.0.1:10888`。

**正确做法**：Docker Desktop 的 HTTP/HTTPS 代理都用：

```text
http://host.docker.internal:10890
```

不要用 Windows `portproxy` 将 `10888` 转发出去作为常规方案；v2rayN 自带局域网端口功能，配置更清晰，也不会在 IP 变化后遗留规则。

### 3. 看到 `docker info` 中的端口是 `3128`，误以为配置错误

**现象**：

```text
HTTP Proxy: http.docker.internal:3128
HTTPS Proxy: http.docker.internal:3128
```

**解释**：这是 Docker Desktop 的内部代理转发器，属于正常行为，并不要求显示 v2rayN 的 `10890`。

要看它实际是否转发到 v2rayN，请检查日志：

```powershell
Get-Content "$env:LOCALAPPDATA\Docker\log\host\httpproxy.log" -Tail 120 |
  Select-String 'host will use proxy|Linux will use proxy|auth.docker.io|registry-1.docker.io|error|timeout'
```

正确日志示例：

```text
container connecting via container settings HTTPS proxy http://host.docker.internal:10890
proxying to registry-1.docker.io:443
successful after ...
```

### 4. `docker pull`/Kubernetes 出现 `unexpected EOF`

**现象**：

```text
short read: expected ... bytes but got ...: unexpected EOF
```

**原因**：下载流在镜像层传输中被中断，通常是节点不稳定或代理连接断开，不是 Kubernetes YAML 或镜像名称写错。

**处理**：先重新执行同一条 `docker pull`；反复发生时换 v2rayN 节点，确认 Desktop 代理仍指向 `10890`，必要时重启 Docker Desktop。

### 5. `TLS handshake timeout` 或直连 IPv6 地址

**现象**：

```text
Post "https://auth.docker.io/token": dial tcp [IPv6-address]:443: ... timeout
```

**原因**：Docker/Buildx 未走代理，尝试直连 Docker Hub 的 IPv6 端点。

**处理顺序**：

1. 验证 v2rayN 局域网端口和 token 请求返回 `200`：

```bash
curl.exe -sS -o NUL -w "%{http_code}\n" \
  -x http://192.168.1.54:10890 \
  "https://auth.docker.io/token?service=registry.docker.io&scope=repository%3Alibrary%2Fnode%3Apull"
```

2. 确认 Docker Desktop 的两组代理配置：

```bash
docker buildx create \
  --name v2ray-builder \
  --driver docker-container \
  --driver-opt default-load=true \
  --driver-opt env.HTTP_PROXY=http://host.docker.internal:10890 \
  --driver-opt env.HTTPS_PROXY=http://host.docker.internal:10890 \
  --use --bootstrap  
```

3. 在 Git Bash `export HTTP_PROXY`、`HTTPS_PROXY` 到 `127.0.0.1:10888`：
   
```bash
export HTTP_PROXY=http://127.0.0.1:10888
export HTTPS_PROXY=http://127.0.0.1:10888
export http_proxy="$HTTP_PROXY"
export https_proxy="$HTTPS_PROXY"
export NO_PROXY=localhost,127.0.0.1
```

   
4. 使用上文的 `v2ray-builder` 和 `docker buildx build`。

```bash
docker buildx build \
  --builder v2ray-builder \
  --load \
  --no-cache \
  --build-arg HTTP_PROXY=http://host.docker.internal:10890 \
  --build-arg HTTPS_PROXY=http://host.docker.internal:10890 \
  -t tsyecommercialdesigner-cloud/qsk-book:1.0 .
```

### 6. 将 `127.0.0.1:10888` 作为 `--build-arg`

**现象**：`npm install` 报错：

```text
ECONNREFUSED 127.0.0.1:10888
```

**原因**：`--build-arg` 注入的是构建容器内部环境；容器中的 `127.0.0.1` 不等于 Windows 主机。

**正确做法**：构建参数必须使用：

```text
http://host.docker.internal:10890
```

### 7. 给 `--driver-opt env.NO_PROXY` 直接传逗号列表

**现象**：

```text
invalid value "127.0.0.1", expecting k=v
```

**原因**：Buildx 会把未转义的逗号拆分为多个 driver option。

**处理**：当前场景可省略该 `driver-opt`。若确有需要，必须按 Buildx 参数规则转义逗号；不要把它和普通 Shell 环境变量的写法混用。

---

## 日常使用与清理

每次新开 Git Bash 并需要构建时，只需重新执行“第 3 节”中的四条 `export`，然后运行构建命令。`v2ray-builder` 会保留，可查看：

```bash
docker buildx ls
```

若不再需要这个构建器，可删除：

```bash
docker buildx rm v2ray-builder
```

### 安全提醒

开启 v2rayN 的局域网端口会使代理监听在局域网可访问的地址上。请只在受信任的私有网络使用；不要在公共 Wi-Fi 或不受控网络上暴露无认证代理。若不再需要 Docker 通过代理访问，关闭“允许来自局域网的连接”和“为局域网开启新的端口”。

