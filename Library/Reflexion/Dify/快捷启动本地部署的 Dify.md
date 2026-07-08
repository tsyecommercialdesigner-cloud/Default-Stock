# 启动 Docker Desktop

必须先启动 Docker Desktop。

然后：

```powershell
cd E:\AI_Platform\dify\docker
docker compose --env-file middleware.env -f docker-compose.middleware.yaml up -d
docker ps
```

如果 Docker Desktop 还没启动，`docker compose` 也会失败。

# 打开三个 WSL 的终端

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
