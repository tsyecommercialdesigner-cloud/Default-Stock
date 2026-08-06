# Docker 容器端口映射不一致导致 HTTP 重定向故障分析与解决方案

## 一、 问题现象与根本原因

在 Docker 部署 Web 服务的场景中，当**宿主机对外暴露的端口**（如 `8080`）与**容器内部应用监听的端口**（如 `80` 或 `8000`）不一致时，若应用触发了 HTTP 301/302 重定向，客户端（浏览器）往往会被重定向到错误的端口（如直接丢失 `:8080` 变成默认的 `:80`），导致连接被拒绝（`ERR_CONNECTION_REFUSED`）或跳转失败。

---

### 1. 根本原因剖析

#### (1) Docker 端口映射工作在传输层（L4）
Docker 通过 `iptables` 或 `docker-proxy` 在网络传输层（TCP/IP）实现端口转发。它只负责将发送到宿主机 `8080` 端口的 TCP 数据包解包并转投给容器内 `80` 端口，**不会干预或修改 HTTP 协议层（L7）的数据包内容**。

#### (2) Web 应用构建绝对路径重定向 URL 的机制
当 Web 框架（如 Spring Boot、Django、Nginx、Apache 等）触发重定向（例如从 `/` 跳转到 `/login`）时，通常需要构建一个完整的绝对 URL：
$$\text{Location URL} = \text{Scheme} + \text{Host} + \text{Port} + \text{Path}$$

在默认无反向代理透传参数的情况下，容器内的 Web 服务依靠**自身的 Socket 通信上下文**推断访问环境：
* **Scheme**: `http`
* **Host**: 客户端请求标头中的 `Host` 字段
* **Port**: 本地绑定并监听的网络 Socket 端口 $\rightarrow$ **`80`**

因此，服务返回给浏览器的响应为：
```http
HTTP/1.1 302 Found
Location: http://example.com/login
```
*(忽略了外部客户端实际访问的 `:8080` 端口)*

#### (3) 浏览器请求闭环中断
浏览器收到 `302 Found` 后，会解析 `Location` 头中的 URL 发起新请求。由于目标地址变成了 `example.com:80`（或缺少端口），如果宿主机的 `80` 端口未开启，请求即刻宣告失败。

---

## 二、 Nginx 反向代理修复机制与端口网络架构

使用 Nginx 作为宿主机（或代理层）的反向代理可以完美解决此问题。

### 1. Nginx 对内与对外端口架构分析

在反向代理配置中，常出现两套不同层级的端口：

```
[客户端/浏览器]
       │
       │ HTTP 请求 (http://example.com:8080/)
       ▼
┌─────────────────────────────────────────┐
│ 宿主机 (Host)                           │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │ Nginx (反向代理)                │   │
│   │ - 监听外部端口: 8080             │   │
│   └──────────────┬──────────────────┘   │
│                  │                      │
│                  │ proxy_pass 内部转发  │
│                  ▼                      │
│   ┌─────────────────────────────────┐   │
│   │ Docker 容器 (Web 服务)          │   │
│   │ - 内部监听/映射端口: 8000        │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

* **对外监听端口 (`8080`)**：`listen 8080;` —— Nginx 接收外部用户访问的入口。
* **对内转发端口 (`8000`)**：`proxy_pass http://127.0.0.1:8000;` —— Nginx 在内部与 Docker 容器通信的通道，不对外开放。

---

## 三、 完整 Nginx 解决方案配置示例

将以下配置保存至 Nginx 配置目录（如 `/etc/nginx/conf.d/app.conf`）：

```nginx
server {
    # 1. Nginx 监听外部客户端访问的宿主机端口
    listen 8080;
    server_name example.com localhost;

    # 客户端上传文件大小限制
    client_max_body_size 50M;

    location / {
        # 2. 将请求转发至内部容器服务地址与端口
        proxy_pass http://127.0.0.1:8000;

        # ------------------- 核心逻辑：修复重定向端口丢失 ------------------- #

        # 核心指令 1：透传原始请求的完整 Host 和端口号（例如 example.com:8080）
        # 注意：必须使用 $http_host，不能使用 $host（$host 会剥离非标准端口）
        proxy_set_header Host $http_host;

        # 核心指令 2：透传客户端真实的端口号给后端应用
        proxy_set_header X-Forwarded-Port $server_port;

        # 核心指令 3：透传客户端请求协议（http 或 https）
        proxy_set_header X-Forwarded-Proto $scheme;

        # 核心指令 4：透传客户端真实 IP 链路
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # ------------------- 兜底保护：拦截并重写响应头 ------------------- #

        # 若后端容器忽略请求头依然返回了容器内部端口（8000），
        # proxy_redirect 会强制把 Location 头中的 127.0.0.1:8000 重写为外部 $http_host
        proxy_redirect http://127.0.0.1:8000/ http://$http_host/;

        # 支持 WebSocket 连接透传（按需配置）
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 四、 核心配置参数深度剖析

| 配置指令 | 详细作用原理 |
| :--- | :--- |
| **`proxy_set_header Host $http_host;`** | **最核心配置**。`$host` 仅包含域名，而 `$http_host` 能够保留原始请求中的 `域名:端口`（如 `example.com:8080`）。后端应用读取此 Header 后，生成的重定向 URL 就会带上 `:8080`。 |
| **`proxy_set_header X-Forwarded-Port $server_port;`** | 显式将代理层监听的外部端口告诉后端容器，常用于框架（如 Spring Boot、Django）开启 `use-forward-headers` 时的端口识别。 |
| **`proxy_set_header X-Forwarded-Proto $scheme;`** | 告知后端原始请求是 `http` 还是 `https`，防止后端在 HTTPS 场景下生成 HTTP 的重定向链接。 |
| **`proxy_redirect`** | 响应头重写策略。当后端应用依然返回包含内部 IP/端口（如 `http://127.0.0.1:8000/login`）的 `Location` 时，Nginx 会在推给客户端前动态替换为 `http://example.com:8080/login`。 |

---

## 五、 替代解决方案（不使用 Nginx）

如果架构中不希望引入 Nginx，可采用以下替代方案：

1. **采用 1:1 端口一致性映射**
   * 修改 Docker 映射与容器内部监听，使内外端口完全一致（例如 `-p 8080:8080`），消除端口不对称现象。

2. **配置 Web 应用使用相对路径重定向**
   * 修改容器内应用/服务器配置，返回相对路径 `Location: /login` 而非绝对路径 `Location: http://example.com/login`。

3. **设置应用的外部基准 URL（Base URL）**
   * 在后端应用配置中显式指定公开 URL（例如 Spring Boot 的 `server.forward-headers-strategy`，Django 的 `USE_X_FORWARDED_PORT = True`，或静态环境变量 `APP_URL=http://example.com:8080`）。

---

## 六、 验证与测试方法

配置完毕并 reload Nginx 后，使用 `curl` 检查响应头：

```bash
curl -I http://localhost:8080/
```

**期望的响应结果：**
```http
HTTP/1.1 302 Found
Server: nginx
Location: http://example.com:8080/login
```
*(若 `Location` 中的端口为 `:8080`，说明重定向已成功修正)*
