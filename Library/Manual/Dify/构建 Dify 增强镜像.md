---
title: 构建 Dify 增强镜像
created: 2026-07-08
tags:
  - Dify
  - Docker
  - Docker_Compose
---
# 特别注意版本的一致性

*注意：Dify 版本、plugin_daemon 版本、自定义基础镜像三者必须对齐。*

你的 Dify 是：

```
Dify 1.15.0
```

它原本配套的 plugin daemon 是：

```
langgenius/dify-plugin-daemon:0.6.3-local
```

所以你自定义 MCP 插件基础镜像时，Dockerfile 必须基于此版本，而不是旧版本 `0.0.6-local`。

永远以当前 Dify 项目自带的 docker-compose.yaml 为准，不以书上的旧版本为准。

---
# 配置增强镜像的 Dockerfile

打开Cursor编辑器，创建`mingweiDockerfile`的新文件（或采用默认的`Dockerfile`命名）。

在文件中输入以下代码：

`docker/Dockerfile` 应该写：

```Dockerfile
FROM langgenius/dify-plugin-daemon:0.6.3-local

RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
RUN apt-get install nodejs -y
RUN curl -qL https://www.npmjs.com/install.sh | sh
RUN npm config set registry https://registry.npmmirror.com/
```

然后在 Dify 的 `docker-compose.yaml` 里，把 `plugin_daemon` 服务改成：

```plain
plugin_daemon:
  image: mcp-plugin-base:latest
```

注意：只改 `plugin_daemon` 这个服务的 `image`，其他环境变量、端口、volume 都保持 Dify 官方原样。

# 执行顺序

```bash
docker pull langgenius/dify-plugin-daemon:0.6.3-local
docker build -f docker/Dockerfile -t mcp-plugin-base .
docker compose up -d --force-recreate plugin_daemon
```

假设你的 Dockerfile 文件名是 `mingweiDockerfile`，那么对应的命令是：

```bash
docker build -t mcp-plugin-base:latest -f mingweiDockerfile .
```

含义是：

```
docker build        构建镜像
-t                  给镜像起名字和标签
mcp-plugin-base:latest   新镜像名
-f mingweiDockerfile            指定 Dockerfile 文件
.                   构建上下文是当前目录
```

如果你的文件名就叫标准的 `Dockerfile`，则可以简化成：

```bash
docker build -t mcp-plugin-base:latest .
```

构建完成后，可以用这个命令查看镜像是否生成：

```bash
docker images
```

你应该能看到类似：

```plain
mcp-plugin-base:latest
```

然后关键是：后续 Dify 的 `docker-compose.yaml` 或 `.env` 里，要让插件服务使用这个新镜像。例如原来可能是：

```
image: langgenius/dify-plugin-daemon:0.0.6-local
```

你需要改成：

```
image: mcp-plugin-base:latest
```

或者如果要求打成某个特定名字，例如`mcp-plugin-base:latest2`，那就用指定的镜像名。

确认运行的是新镜像：

```bash
docker compose ps
```

应看到：

```
plugin_daemon   mcp-plugin-base:latest
```

再看日志：

```bash
docker logs docker-plugin_daemon-1 --tail 50
```

没有持续报错后，再去 Dify 管理界面安装 `.difypkg` 插件。

如果安装插件时报：

```
404 Not Found
/management/decode/from_identifier
```

这不是插件包本身的问题，而是 **Dify API 调用了 plugin_daemon 的新接口，但当前 daemon 不支持**。

这几乎一定是版本不匹配：

```
Dify API/Web 版本较新
plugin_daemon 基础镜像较旧
```

---
# 错题反思

1. **把书里的镜像版本当成固定答案了**

书上用的是：

```
langgenius/dify-plugin-daemon:0.0.6-local
```

但你的 Dify 是 `1.15.0`，官方 compose 原本写的是：

```
langgenius/dify-plugin-daemon:0.6.3-local
```

永远以当前 Dify 项目自带的 docker-compose.yaml 为准，不以书上的旧版本为准。

2. **误以为 `mcp-plugin-base` 只是“装 Node 的镜像”**

它不只是装 Node。它继承了哪个 daemon，决定了它支持哪些 Dify 插件接口。

```
FROM 0.0.6-local  -> 旧 daemon + Node
FROM 0.6.3-local  -> 当前 Dify 配套 daemon + Node
```

真正重要的是 `FROM` 的版本。

3. **排查顺序一开始偏到了 Dockerfile 路径、volume、.env**

这些确实会出错，但今天最终安装失败的主因不是它们。以后遇到 Dify 插件安装 404，优先看：

```
git diff docker-compose.yaml
docker compose ps
docker logs docker-plugin_daemon-1 --tail 100
```

重点确认：

```
Dify 版本
plugin_daemon 原始镜像版本
自定义镜像 FROM 版本
```

一句话总结：**以后不要照抄书里的 daemon tag；先从当前 Dify compose 里查原始 `langgenius/dify-plugin-daemon` 版本，再用它作为自定义 Dockerfile 的 `FROM`。**