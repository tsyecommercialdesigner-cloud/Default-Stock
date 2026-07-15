---
title: n8n docker部署 + Cloudflare Tunnel 完整部署指南
created: 2026-07-15
source: lark
tags:
  - n8n_deployment
---
---
# 任务概览

## 📋 目标

通过 Docker Desktop + Cloudflare Tunnel，将本地 n8n 安全地暴露到公网，支持完整的 UI 界面和表单页面访问。

## 解决什么问题？

n8n + Cloudflare（通常指 Cloudflare Tunnel/Zero Trust + 你的域名）主要解决的是把自建 n8n 安全、稳定地对外提供访问和 Webhook 回调的问题，不需要公网 IP，也不需要自己在路由器/防火墙上做复杂端口映射，还能顺带加一层安全防护。

### 内网穿透/无公网 IP：

把你本机或内网的 `5678` 服务映射成公网可访问的 HTTPS 域名，第三方平台才能回调你的 Webhook（比如支付、表单、聊天机器人等）。

### 安全性：

你不必直接把 `5678` 裸露在公网；可以让所有外部流量先经过 Cloudflare，再转发到 n8n，从而降低被扫描、爆破的风险（并可叠加访问控制）。

### HTTPS 与证书：

外部访问天然用 HTTPS，更方便对接要求 HTTPS 的服务（很多 Webhook/OAuth 场景都更偏好或要求 HTTPS）。

## 🎯 最终效果

- ✅ 通过 `https://n8n.你的域名` 访问完整的 n8n 编辑器
- ✅ 表单节点可以正常显示和提交（不再 404）
- ✅ Webhook 功能完全可用
- ✅ 免费且稳定的公网访问
- ✅ 自动 HTTPS 和基础安全防护

## 📚 前置准备

### 必需条件

1. Windows 电脑（已安装 Docker Desktop）
2. 域名（需要托管在 Cloudflare，免费）
3. Cloudflare 账号（免费注册）

### 可选但推荐

- 文本编辑器（记事本即可，或者 notepad++）
- PowerShell 或命令提示符基础使用

# 🚀 第一部分：Cloudflare 账号和域名准备

## 步骤 1：注册 Cloudflare 账号

i. 访问 [<sup>1</sup>](https://cloudflare.com)

ii. 点击“注册”，使用邮箱注册账号

iii. 验证邮箱并登录

## 步骤 2：添加域名到 Cloudflare

a. 登录 Cloudflare 控制台

b. 点击“添加站点”

c. 输入你的域名（如：`example.com`）

d. 选择“Free”计划

e. 按照提示修改域名的 DNS 服务器为 Cloudflare 提供的地址

f. 等待域名状态变为“活跃”（通常 5-30 分钟）

**注意：** 如果你没有域名，可以：

- 购买便宜域名（如 `.tk`、`.ml`、`.ga` 等免费域名）
- 或使用 Freenom、Cloudflare Registrar 等服务

# 🔧 第二部分：创建 Cloudflare Tunnel

## 步骤 3：开通 Zero Trust

a. 在 Cloudflare 控制台，点击左侧“Zero Trust”

b. 如果是第一次使用，会提示创建团队名称

c. 输入团队名称（如：`your-team-name`），点击“下一步”

d. 选择“Zero Trust Free”计划（0 美元），点击“继续付款”

## 步骤 4：创建隧道

a. 进入 Zero Trust 控制台

b. 左侧菜单：**网络 → 隧道**

c. 点击“添加隧道”

d. 选择“Cloudflared”

e. 输入隧道名称（如：`n8n-tunnel`）

f. 点击“保存隧道”

## 步骤 5：获取 TUNNEL_TOKEN

a. 在“安装并运行连接器”页面

b. 选择“Docker”选项卡

c. 复制命令中的 token 部分，格式类似：

```text
eyJhIjoiMDNiMDcyMTcwYzdiMGJlMzcyZDkxYTE2MmZhZTVlNGMi...
```

**重要：** 保存这个 token，稍后需要用到

**暂时不要点击“下一步”**，先去配置 Docker

# 🐳 第三部分：Docker 配置

## 步骤 6：停止现有的 n8n 容器（如果有）

打开 PowerShell 或命令提示符，执行：

```bash
docker stop n8n
docker rm n8n
```

如果提示容器不存在，忽略错误继续。

## 步骤 7：创建项目目录

1. 在电脑上创建一个新文件夹，例如：

   - `D:\n8n_project` 或
   - `C:\Users\你的用户名\n8n_project`

## 步骤 8：创建 docker-compose.yml 文件

1. 直接下载文件放到一个新建的文件夹里：

   `docker-compose.yml`

2. 在刚创建的文件夹中，右键 → 新建 → 文本文档

3. 将文件名改为：`docker-compose.yml`

   - **注意：** 删除 `.txt` 后缀，确保文件名就是 `docker-compose.yml`

4. 用记事本打开这个文件

5. 粘贴以下内容：

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:stable
    container_name: n8n
    restart: always

    # 本机直连（快）：只绑定到本机，局域网其他设备访问不到
    ports:
      - "127.0.0.1:5678:5678"

    # Windows 建议用正斜杠，避免反斜杠转义问题
    volumes:
      - "G:/n8n_data:/home/node/.n8n"

    environment:
      # 对外基址（走 Cloudflare Tunnel 的域名）
      - N8N_HOST=n.你的域名
      - N8N_PROTOCOL=https
      - N8N_PORT=5678

      # 反代/隧道场景建议显式设置，确保 webhook 与编辑器链接正确
      - N8N_EDITOR_BASE_URL=https://n8n.你的域名/
      - WEBHOOK_URL=https://n8n.你的域名/

      # 反向代理/隧道常见需要：让 n8n 正确识别 X-Forwarded-*（有多层代理再按实际改）
      - N8N_PROXY_HOPS=1

      - N8N_DEFAULT_BINARY_DATA_MODE=filesystem
      - N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
      - TZ=Asia/Shanghai

    networks:
      - app-network

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: always
    command: tunnel --no-autoupdate run
    environment:
      - TUNNEL_TOKEN=你的TUNNEL_TOKEN
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

这里的 `app-network` 是给两个容器创建一个互通的网络，不然隔离会经常出错。

## 步骤 9：修改配置文件

需要修改两个地方：

### 1. 替换你实际文件夹的地方

```yaml
volumes:
  - "实际文件夹路径:/home/node/.n8n"
```

### 2. 替换域名：

- 将 `你的域名` 替换为你的实际域名
- 例如：如果你的域名是 `example.com`，则改为：

```yaml
- N8N_EDITOR_BASE_URL=https://n8n.example.com/
- WEBHOOK_URL=https://n8n.example.com/
```

### 3. 替换 TUNNEL_TOKEN：

- 将 `你的TUNNEL_TOKEN` 替换为步骤 5 中复制的 token
- 例如：

```yaml
- TUNNEL_TOKEN=eyJhIjoiMDNiMDcyMTcwYzdiMGJlMzcyZDkxYTE2MmZhZTVlNGMi...
```

### 4. 检查数据目录路径：

- 如果你之前使用的是其他路径（如 `G:\n8n_data`），保持不变
- 如果是新安装，可以改为相对路径：`./n8n_data:/home/node/.n8n`

## 步骤 10：启动 Docker 服务

1. 在 `docker-compose.yml` 所在目录，按住 Shift + 右键，选择“在此处打开 PowerShell 窗口”

2. 执行启动命令：

```bash
docker compose up -d
```

3. 等待镜像下载和容器启动（首次运行需要几分钟）

## 步骤 11：验证容器状态

执行以下命令检查状态：

```bash
docker compose ps
```

应该看到类似输出：

```text
NAME          IMAGE                            COMMAND                 SERVICE       STATUS
cloudflared   cloudflare/cloudflared:latest    "cloudflared --no-au..." cloudflared  ...
n8n           docker.n8n.io/n8nio/n8n          "tini -- /docker-ent..." n8n          ...
```

检查 cloudflared 连接状态：

```bash
docker compose logs cloudflared
```

应该看到类似 “Registered tunnel connection” 的成功连接信息。

# 🌐 第四部分：完成 Cloudflare 隧道配置

## 步骤 12：配置公网主机名

1. 回到 Cloudflare Zero Trust 控制台的隧道创建页面

2. 现在应该看到“连接器”状态为“已连接”

3. 点击“下一步”

4. 在“路由隧道”页面，填写以下信息：

**子域：**

```text
n8n
```

**域（必需）：**

从下拉菜单选择你的域名，例如：

```text
example.com
```

**路径：**

```text
留空（不填写）
```

**服务 - 类型：**

```text
HTTP
```

**服务 - URL：**

```text
n8n:5678
```

5. 点击“完成设置”

## 步骤 13：等待 DNS 生效

- Cloudflare 会自动创建 DNS 记录
- 通常 1-5 分钟生效
- 可以用 `nslookup n8n.你的域名` 检查 DNS 是否生效

# 🎉 第五部分：访问和验证

## 步骤 14：首次访问 n8n

1. 打开浏览器，访问：`https://n8n.你的域名`

2. 如果一切正常，应该看到 n8n 的初始设置页面

3. 按照向导创建管理员账号

## 步骤 15：测试功能

### 测试 UI 访问：

- 能正常打开 n8n 编辑器界面 ✅

### 测试表单功能：

1. 创建一个新的工作流
2. 添加“Form Trigger”节点
3. 添加“Respond to Webhook”节点
4. 连接两个节点并激活工作流
5. 点击 Form Trigger 节点生成的表单链接
6. 应该能正常打开表单页面并提交 ✅

### 测试 Webhook 功能：

1. 创建包含 Webhook 节点的工作流
2. 激活工作流
3. 使用生成的 webhook URL 进行测试
4. 应该能正常接收和处理请求 ✅

# 🔒 第六部分：安全加固（强烈推荐）

## 步骤 16：添加访问控制

1. 在 Zero Trust 控制台，进入：**Access → 应用程序**

2. 点击“添加应用程序”

3. 选择“Self-hosted”

4. 填写应用信息：

   - **应用名称：** n8n
   - **应用域名：** `n8n.你的域名`

5. 创建访问策略：

   - **策略名称：** 仅允许我访问
   - **操作：** 允许
   - **规则：** 包含 → 电子邮件 → 你的邮箱地址

6. 保存配置

现在访问 n8n 时会先要求登录验证，增加安全性。

# 🛠️ 常见问题和故障排除

## 问题 1：访问域名显示 404

**可能原因：**

- DNS 还未生效
- 隧道配置错误
- 容器未正常启动

**解决方法：**

1. 检查容器状态：`docker compose ps`
2. 检查 cloudflared 日志：`docker compose logs cloudflared`
3. 确认 Cloudflare 隧道状态为“已连接”
4. 等待 DNS 生效（最多 30 分钟）

## 问题 2：容器启动失败

**可能原因：**

- 端口被占用
- 配置文件格式错误
- Docker Desktop 未运行

**解决方法：**

1. 确保 Docker Desktop 正在运行
2. 检查端口占用：`netstat -an | findstr 5678`
3. 验证 YAML 格式是否正确
4. 查看详细错误：`docker compose logs`

## 问题 3：YAML 格式错误

**常见错误：**

- 缩进不正确（必须使用空格，不能用 Tab）
- 冒号后面缺少空格
- 引号不匹配

**解决方法：**

- 使用在线 YAML 验证器检查格式
- 确保所有缩进使用 2 个空格
- 重新复制提供的模板

## 问题 4：数据持久化问题

**现象：** 重启容器后数据丢失

**解决方法：**

- 确保 volumes 配置正确
- 检查挂载目录权限
- Windows 路径使用双反斜杠：`"G:\\n8n_data"`

# 📝 日常维护

## 启动服务

```bash
cd 你的项目目录
docker compose up -d
```

## 停止服务

```bash
docker compose down
```

## 查看日志

```bash
# 查看所有服务日志
docker compose logs

# 查看特定服务日志
docker compose logs n8n
docker compose logs cloudflared
```

## 更新镜像

```bash
docker compose pull
docker compose up -d
```

## 备份数据

定期备份 `G:\n8n_data` 目录（或你配置的数据目录）

# 🎯 总结

通过以上步骤，你已经成功：

1. ✅ 使用 Docker Desktop 部署了 n8n
2. ✅ 通过 Cloudflare Tunnel 实现了安全的公网访问
3. ✅ 配置了完整的 UI 和表单功能
4. ✅ 实现了稳定的 Webhook 服务
5. ✅ 添加了基础的安全防护

**相比之前的方案优势：**

- 🚫 不再需要复制长长的 `docker run` 命令
- ✅ 表单页面可以正常访问（不再 404）
- ✅ 稳定的域名（不会像 ngrok 那样变化）
- ✅ 免费且无流量限制
- ✅ 自动 HTTPS 和 DDoS 防护
- ✅ 简单的启停管理：`docker compose up -d` / `docker compose down`

**重要提醒：**

- 定期备份你的 n8n 数据
- 保管好你的 TUNNEL_TOKEN，不要泄露
- 建议启用 Cloudflare Access 进行访问控制
- 关注 n8n 和 cloudflared 的版本更新

现在你可以愉快地使用 n8n 进行工作流自动化了！🎉
