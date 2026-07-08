打开三个 PowerShell 窗口，并分别进 Ubuntu：

```
wsl -d Ubuntu
```

**API 终端：**

```
cd ~/dify/api
uv run flask run --host 0.0.0.0 --port=5001 --debug
```

**Worker 终端：**

```
cd ~/dify/api
uv run celery -A app.celery worker -P solo --without-gossip --without-mingle --loglevel INFO -Q dataset,dataset_summary,priority_dataset,priority_pipeline,pipeline,mail,ops_trace,app_deletion,plugin,workflow_storage,conversation,workflow,schedule_poller,schedule_executor,triggered_workflow_dispatcher,retention,workflow_based_app_execution
```

**Web 终端：**

```
cd ~/dify/web
pnpm dev --hostname 0.0.0.0
```

另外建议你把默认 WSL 从 `docker-desktop` 改成 `Ubuntu`，以后输入 `wsl` 就不会进错了。回到 PowerShell 执行：

```
wsl --set-default Ubuntu
```
