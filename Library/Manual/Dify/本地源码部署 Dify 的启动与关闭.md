# 启动

## 启动 Docker Desktop

必须先启动 Docker Desktop。

然后：

```powershell
cd E:\AI_Platform\dify\docker
docker compose --env-file middleware.env -f docker-compose.middleware.yaml up -d
docker ps
```

如果 Docker Desktop 还没启动，`docker compose` 也会失败。

## 打开三个 WSL 的终端

打开三个 PowerShell 窗口，并分别进 Ubuntu：

```powershell
wsl -d Ubuntu
```

**API 终端：**

```powershell
cd ~/dify/api
uv run flask run --host 0.0.0.0 --port=5001 --debug
```

**Worker 终端：**

```powershell
cd ~/dify/api
uv run celery -A app.celery worker -P solo --without-gossip --without-mingle --loglevel INFO -Q dataset,dataset_summary,priority_dataset,priority_pipeline,pipeline,mail,ops_trace,app_deletion,plugin,workflow_storage,conversation,workflow,schedule_poller,schedule_executor,triggered_workflow_dispatcher,retention,workflow_based_app_execution
```

**Web 终端：**

```powershell
cd ~/dify/web
pnpm dev --hostname 0.0.0.0
```

另外建议你把默认 WSL 从 `docker-desktop` 改成 `Ubuntu`，以后输入 `wsl` 就不会进错了。回到 PowerShell 执行：

```powershell
wsl --set-default Ubuntu
```

---
# 关闭

## 停止前端 Web

在运行 `pnpm dev` 的终端里按：

```
Ctrl + C
```

看到回到命令行提示符即可。

## 停止后端 API

在运行：

```
uv run flask run --host 0.0.0.0 --port=5001 --debug
```

的终端里按：

```
Ctrl + C
```

## 停止 Worker

在运行 `uv run celery ...` 的终端里按：

```
Ctrl + C
```

如果它提示是否 warm shutdown，等它退出即可；不退出再按一次 `Ctrl + C`。

## 停止 Docker 中间件

在 PowerShell 里执行：

```
cd E:\AI_Platform\dify\docker
docker compose --env-file middleware.env -f docker-compose.middleware.yaml down
```

这会停止：

```
PostgreSQL
Redis
Weaviate
Sandbox
Plugin daemon
SSRF proxy
```

一般不会删除数据卷，数据还在。

## （可选）关闭 WSL

确认 WSL 终端都不需要了之后，在 PowerShell 执行：

```
wsl --shutdown
```

## （可选）关闭 Docker Desktop

Docker 容器已经 `down` 之后，可以退出 Docker Desktop。

完整关机顺序就是：

```
停 Web
停 API
停 Worker
docker compose down
wsl --shutdown
关闭 Docker Desktop
关机
```

下次启动时顺序反过来：

```
Docker Desktop
docker compose up -d
WSL 启动 API
WSL 启动 Worker
WSL 启动 Web
```