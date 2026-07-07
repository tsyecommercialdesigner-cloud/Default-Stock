注意：**Dify 版本、plugin_daemon 版本、自定义基础镜像三者必须对齐**。

# 正确路线

你的 Dify 是：

```
Dify 1.15.0
```

它原本配套的 plugin daemon 是：

```
langgenius/dify-plugin-daemon:0.6.3-local
```

所以你自定义 MCP 插件基础镜像时，Dockerfile 必须基于这个版本，而不是旧书里的 `0.0.6-local`。

`docker/Dockerfile` 应该写：

```
FROM langgenius/dify-plugin-daemon:0.6.3-local

RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
RUN apt-get install nodejs -y
RUN curl -qL https://www.npmjs.com/install.sh | sh
RUN npm config set registry https://registry.npmmirror.com/
```

然后在 Dify 的 `docker-compose.yaml` 里，把 `plugin_daemon` 服务改成：

```
plugin_daemon:
  image: mcp-plugin-base:latest
```

注意：只改 `plugin_daemon` 这个服务的 `image`，其他环境变量、端口、volume 都保持 Dify 官方原样。

执行顺序：

```
docker pull langgenius/dify-plugin-daemon:0.6.3-local
docker build -f docker/Dockerfile -t mcp-plugin-base .
docker compose up -d --force-recreate plugin_daemon
```

确认运行的是新镜像：

```
docker compose ps
```

应看到：

```
plugin_daemon   mcp-plugin-base:latest
```

再看日志：

```
docker logs docker-plugin_daemon-1 --tail 50
```

没有持续报错后，再去 Dify 管理界面安装 `.difypkg` 插件。

**核心判断方法**

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

正确原则是：**永远以当前 Dify 项目自带的 docker-compose.yaml 为准，不以书上的旧版本为准。**

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