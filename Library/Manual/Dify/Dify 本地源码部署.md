---
title: Dify 本地源码部署
created: 2026-07-08
tags:
  - Dify_local_source_deployment
  - Dify_deployment
---
# 启动服务前的准备

## 检查系统要求

本地源代码安装 Dify 之前，请确保设备满足最低系统要求：CPU 不低于 2 核，内存不低于 4GB，以保障部署和运行的顺畅。Dify 采用前后端分离的开发模式，部署需要同时支持服务端和前端页面的运行，并依赖一些中间件。

## 拉取官方仓库

选一个合适的目录：

```bash
cd /e/AI_Platform
```

在本地启动服务之前，需要先复制 Dify 代码。

```bash
git clone https://github.com/langgenius/dify.git
```

注意：clone 一份就够了，不要 api/web 各 clone 一份，多份之间容易出现分支、依赖、锁文件、接口版本不一致的问题。
## 部署中间件：PostgreSQL、Redis 和 Weaviate

进入目录：

```bash
cd /e/AI_Platform/dify/docker
```

运行：

```bash
cp envs/middleware.env.example middleware.env
```

注：在当前的 Dify 主分支， `middleware.env.example`位于`docker/envs/`, 而不是 `docker/`。

## 启动中间件

运行：

```bash
docker compose --env-file middleware.env -f docker-compose.middleware.yaml up -d
```

### Weaviate 端口占用错误的处理方法

注意：Dify middleware 里 Weaviate 默认会把容器内 `8080` 映射到宿主机 `8080`，即 Weaviate 要占用宿主机的 8080 端口，但可能会因 Windows 不允许绑定这个端口而报错，这通常不是被普通程序占用，而是 Windows/Hyper-V/WSL 保留端口段导致的。

解决办法最简单：**把宿主机端口改成 18080**。

在 `/e/AI_Platform/dify/docker/middleware.env` 里改：

```
EXPOSE_WEAVIATE_PORT=18080
```

建议顺手加上或确认这个端口，避免 gRPC 以后也撞：

```
EXPOSE_WEAVIATE_GRPC_PORT=15051
```

然后重新启动，启动时最好带上 `--env-file middleware.env`，
否则 compose 端口映射可能还是用默认的 `8080`。

运行

```bash
docker compose --env-file middleware.env -f docker-compose.middleware.yaml up -d
```

再检查：

```bash
docker ps
```

如果看到类似：

```
docker-weaviate-1   Up
0.0.0.0:18080->8080/tcp
```

就成功了，之后你启动源码版后端时，
记得把Linux文件系统下 `~/dify/api/.env` 里的 Weaviate 地址改成：

```
WEAVIATE_ENDPOINT=http://localhost:18080
```

想确认 Windows 保留了哪些端口，可以在 PowerShell 跑：

```
netsh interface ipv4 show excludedportrange protocol=tcp
```

---
# 服务端部署

Dify 官方本地源码文档里对 Windows 的建议是：**Windows with WSL 2 enabled**，
并且建议把源码放在 Linux 文件系统里，而不是 Windows 盘里。

如果你坚持在 Windows 原生环境里修，也可以尝试处理 `python-magic/libmagic`，但这条路后面还可能继续遇到别的原生库问题。

对 Dify 这种后端依赖很多的项目，WSL2 是更稳的源码开发路径。

参考：  [Dify Local Source Code Start](https://docs.dify.ai/en/self-host/deploy/advanced-deployments/local-source-code)

建议不要在 Git Bash 里硬跑后端，改用 **WSL2 Ubuntu** 跑 `api` 和 `web`。
Docker Desktop 可以继续用安装在Windows上的这个来跑中间件容器。

## 安装 Ubuntu

以管理员身份打开 **PowerShell** 。并执行：

```
wsl --install -d Ubuntu
```

安装位置通常由 Windows/WSL 管理，默认在系统盘用户目录下。

安装完成后，给 Ubuntu 创建一个 Linux 用户名。

你可以用`administrator`，这没问题。不过要区分这个用户名只是 WSL Ubuntu 里的普通用户，它和 Windows 的 `Administrator` 不是同一个账号，但叫一样也可以。

接下来设置并重复一遍对应此 Linux 账户的密码。

接着问你是否同意 Ubuntu/WSL 收集一些平台指标数据，你可以y或者n，这个不影响 Dify 部署。

创建完成后，你会进入类似这样的 Ubuntu Bash：

```
administrator@你的电脑名:~$
```

然后继续执行：

```bash
sudo apt update
sudo apt install -y git curl build-essential libmagic1
```

第一次 `sudo` 会让你输入刚刚设置的 Ubuntu 密码。

另请注意，另外你现在的位置是：

```
/mnt/c/Users/Administrator/Desktop
```

这是 Windows 的 C 盘桌面目录。
安装软件没关系，但后面 clone Dify 源码时，建议先回到 WSL 自己的 Linux home 目录：

```bash
cd ~
```

Dify 源码建议放在 WSL 的 Linux 文件系统里，也就是放在 Ubuntu 自己的 home 目录，比如：

```
/home/你的用户名/dify
```

不要放在：

```
/mnt/e/AI_Platform/dify
```

虽然 WSL 可以访问 E 盘，但 Dify 官方建议源码和绑定给 Linux 容器的数据放在 Linux 文件系统里，这样性能和兼容性更好。

## 拉取仓库

### 在 Ubuntu 自己的 home 目录拉取仓库

确认当前所在路径是`/home/administrator`，然后再 clone：

```bash
git clone https://github.com/langgenius/dify.git
```

如果报 GnuTLS，就改用 GitHub 镜像地址（Dify 仓库很大，浅克隆更稳）：

```bash
git clone --depth 1 https://gh-proxy.com/https://github.com/langgenius/dify.git
```

或者：

```bash
git clone --depth 1 https://hub.gitmirror.com/https://github.com/langgenius/dify.git
```

镜像不一定一直可用，但经常能绕过这种 TLS 中断。

注意：

> WSL 里的 SSH key 还没有配置到 GitHub，使用 SSH Key 方式拉取仓库会被拒绝；
> 不过Dify 是公开仓库，最简单的方式是直接用 HTTPS，不需要 SSH key。

此外：

> WSL 里访问 GitHub 时 TLS 连接被中断，常见于网络不稳定、代理/VPN 没进 WSL、GitHub 连接被重置等情况，而不是你的命令输错了。

### 备选方案：复用Windows 已经 clone 好的源码

如果镜像仓库也拉取不了，则考虑复用Windows 已经 clone 好的源码。

虽然长期开发不推荐放 `/mnt/e`，但为了先跑通后端，可以先复制到 WSL 的 Linux home：

```bash
cp -a /mnt/e/AI_Platform/dify ~/dify
```

然后再：

```bash
cd ~/dify/api
```

这个方法最稳，因为不用再从 GitHub 拉 400MB+ 的仓库。

如果你特别担心 C 盘空间，可以后续把 WSL 发行版迁移到 E 盘，但第一次先按默认安装最省事。等跑通后再迁移也可以。

## 进入API目录

接下来进入后端目录：

```bash
cd ~/dify/api
```

## 复制环境变量配置文件

这里需要运行 ：

```bash
cp .env.example .env
```


## 生成随机密钥

运行：

```bash
openssl rand -base64 42
```

如果你本机没有 `openssl`，也可以用 Python 生成：

```python
python -c "import secrets; print(secrets.token_urlsafe(42))"
```

然后编辑 `.env`：

```shell
nano .env
```

并将生成的密钥替换掉`.env`中`SECRET_KEY`等号后面的值，格式是这样：

```
SECRET_KEY=abc123xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

密钥后续不要频繁改，改了之后，已有登录态、session 之类可能会失效，不过本地开发问题不大。

还有别忘记更改 Weaviate 地址：

```
WEAVIATE_ENDPOINT=http://localhost:18080
```

另外顺手加上或确认这个端口，避免 gRPC 以后也撞：

```
WEAVIATE_GRPC_PORT=15051
```

`nano` 保存方式：

```shell
Ctrl + O
回车
Ctrl + X
```

## 安装UV

运行：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env
uv --version
```

## 安装后端依赖

运行：

```bash
uv sync --dev
```

## 执行数据库迁移

运行：

```bash
uv run flask db upgrade
```

## 启动 API 服务

运行：

```bash
uv run flask run --host 0.0.0.0 --port=5001 --debug
```

## 启动 Worker 服务

再新开一个终端，执行：

```
wsl -d Ubuntu
```

### 在 Ubuntu 中启用 Worker服务

继续运行：

```bash
cd ~/dify/api
uv run celery -A app.celery worker -P solo --without-gossip --without-mingle --loglevel INFO -Q dataset,dataset_summary,priority_dataset,priority_pipeline,pipeline,mail,ops_trace,app_deletion,plugin,workflow_storage,conversation,workflow,schedule_poller,schedule_executor,triggered_workflow_dispatcher,retention,workflow_based_app_execution
```

现在就已经把最关键的 API 服务跑起来了。

# 启动前端 Web

再新开一个终端：

```bash
cd ~/dify/web
```

---
## 安装依赖包

注意：要用在 Ubuntu/WSL 里单独安装的 Linux 版 Node.js，不要用 Windows 的 Node。使用 Windows 的 Node 去执行 `corepack enable` 会报错。

你可以先确认当前用的是哪里的Node.js：

```bash
which node
which corepack
```

如果看到 `/mnt/e/Program Files/nodejs/...`，就说明当前用的是 Windows 的 Node。

如果已经安装好 WSL 里的 Node ，`which node` 应该是：

```
/home/administrator/.nvm/versions/node/v22.x.x/bin/node
```

推荐用 `nvm` 安装 Node 22：

```bash
cd ~
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install 22
```

这一步可能出现 WSL 访问 `nodejs.org` 被断开的问题，和前面 GitHub 类似，属于网络问题。

可以直接给 `nvm` 换国内镜像。在 WSL 里执行：

```bash
export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node
nvm install 22
nvm use 22
node -v
npm -v
```

如果你希望以后一直走这个镜像，把它写进 `~/.bashrc`：

```bash
echo 'export NVM_NODEJS_ORG_MIRROR=https://npmmirror.com/mirrors/node' >> ~/.bashrc
source ~/.bashrc
```

然后再装：

```bash
nvm install 22
```

装好后继续再回到 web 目录：

```bash
cd ~/dify/web
```

继续：

```bash
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v
```

这将通过 Corepack 下载并启用 `pnpm`，如果你已经开了 V2rayN TUN，直接选 `Y` 继续就行。

如果下载很慢或失败，可以取消后改用国内源安装：

```
npm config set registry https://registry.npmmirror.com
npm install -g pnpm
pnpm -v
```

下载成功后再执行：

```bash
pnpm install --frozen-lockfile
```

很好，`pnpm install` 已经成功完成了。

## 配置环境变量

创建前端环境文件：

```bash
cp .env.example .env.local
nano .env.local
```

在 `.env.local` 里确认或修改这两项：

```
NEXT_PUBLIC_API_PREFIX=http://localhost:5001/console/api
NEXT_PUBLIC_PUBLIC_API_PREFIX=http://localhost:5001/api
```

如果文件里已经有这两项，就改成上面的值；如果没有，就加到文件末尾。

## 启动前端

运行：

```bash
pnpm dev
```

使用浏览器访问：

```
http://localhost:3000
```

此时你的本地源码部署应该是：

```
Docker middleware  已运行
API 后端           已运行，5001
Celery worker      需要保持运行
Web 前端           即将运行，3000
```

大功告成了。

---
# 注意事项

## 当需要修改后端、前端的环境变量时

你现在是本地源码部署，运行方式是：

```
api  -> ~/dify/api/.env
web  -> ~/dify/web/.env.local
docker -> 只跑中间件
```

所以：

```
~/dify/docker/.env
```

主要是完整 Docker Compose 部署时给容器里的 `api/web/worker` 用的。

区别于完整 Docker Compose 部署，本地源码部署的 API 是在 WSL 里用 `uv run flask run --host 0.0.0.0 --port=5001 --debug` 跑的，它不会读取 `docker/.env`。

如果某个环境变量真的需要改，应该改这里：

```
~/dify/api/.env
```

改完后重启：

```
API
Worker
```

## Dify 的 API 是跑在 WSL 里的

如果你的自定义模型服务跑在 Windows，比如 Ollama：

```
http://localhost:11434
```

而 Dify API 跑在 WSL 里，`localhost` 指的是 WSL 自己，不一定是 Windows。

这时候你可以在 WSL 里查 Windows host IP：

```
cat /etc/resolv.conf | grep nameserver
```

假设得到：

```
nameserver 172.29.xxx.1
```

那模型 Base URL 可以填：

```
http://172.29.xxx.1:11434
```

## 注意默认的 WSL 可能不是 Ubuntu

精准指定进入Ubuntu的命令是：

```
PS C:\Users\Administrator> wsl -d Ubuntu
```

进入成功后应该看到类似：

```
administrator@TSY-IceCream:~$
```

而不是：

```
docker-desktop:...#
```

如果想确认当前有哪些 WSL 发行版，在 PowerShell 里执行：

```
wsl -l -v
```

你可能会看到类似：

```
  NAME              STATE           VERSION
* docker-desktop    Running         2
  Ubuntu            Running         2
```

如果星号 `*` 在 `docker-desktop` 上，说明默认 WSL 是 Docker Desktop。

如果将默认的wsl改成 Ubuntu：

```
wsl --set-default Ubuntu
```

之后再输入：

```
wsl
```

就会进 Ubuntu 了，但是建议还是保持默认的 `docker-desktop` 。