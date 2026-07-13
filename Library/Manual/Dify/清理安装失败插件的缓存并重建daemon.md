---
title: 清理安装失败插件的缓存并重建daemon
created: 2026-07-13
tags:
  - Cache_clean
  - Plugin_installation
---
**清理失败插件缓存并重建 daemon**

先停止 daemon：

```
cd /opt/dify/docker
docker compose stop plugin_daemon
```

查看失败插件对应目录，例如查找 OpenAI：

```
sudo find ./volumes/plugin_daemon/cwd -maxdepth 1 -mindepth 1 -type d -iname '*openai*' -printf '%f\n'
```

确认目录名后，只删除该插件目录。OpenAI 的示例：

```
sudo rm -rf ./volumes/plugin_daemon/cwd/langgenius/openai-1.0.0@*
```

最后让新 `.env` 配置生效并重建 daemon：

```
docker compose up -d --force-recreate plugin_daemon
docker compose ps plugin_daemon
docker compose logs --tail=50 plugin_daemon
```

确认 `plugin_daemon` 为 `Up` 后，在 Dify 页面重新安装插件即可。不要删除整个 `volumes/plugin_daemon`，也不要执行 `docker compose down -v`。