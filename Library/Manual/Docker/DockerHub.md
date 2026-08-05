---
title: DockerHub
created: 2026-08-05
tags:
  - DockerHub
---
---
# 推送镜像到 DockerHub 的格式

Docker Hub 的 Docker ID（用户名）**不能修改**；可修改姓名、邮箱等资料，但不能改用户名。想换用户名只能注册新账号，再把镜像重新打标签并推送到新命名空间。 [Docker 官方 FAQ](https://docs.docker.com/accounts/general-faqs/)

推送 Docker Hub 镜像时，通常必须指定你有权限的命名空间：

```
<用户名>/<仓库名>:<标签>
```

或组织命名空间：

```
<组织名>/<仓库名>:<标签>
```

例如：

```bash
docker push rongruihouseholds/qsk-book:1.0
```

如果你的 Docker ID 不是 `rongruihouseholds`，重新打标签后推送：

```bash
docker tag rongruihouseholds/qsk-book:1.0 <你的DockerID>/qsk-book:1.0
docker login
docker push <你的DockerID>/qsk-book:1.0
```

不能只写：

```bash
docker push qsk-book:1.0
```

因为这会尝试推送到 Docker Hub 的保留 `library/qsk-book` 命名空间，普通用户没有权限。完整镜像格式是 `registry/namespace/repository:tag`；Docker Hub 可省略 registry，但 namespace 必须是你自己或你有推送权限的组织。 [Docker 镜像命名规则](https://docs.docker.com/reference/cli/docker/image/tag/)

---
# 本地标签不存在

如果 Git 推送时提示错误信息：

```plain
tag does not exist: rongruihouseholds/qsk-book:1.0
```

这表示本地标签不存在，不是登录或网络问题。

你当前构建出的镜像标签是：

```plain
tsyecommercialdesigner-cloud/qsk-book:1.0
```

先重新打上你的 Docker Hub 用户名标签，再推送：

```bash
docker image tag \
  tsyecommercialdesigner-cloud/qsk-book:1.0 \
  rongruihouseholds/qsk-book:1.0

docker login
docker image push rongruihouseholds/qsk-book:1.0
```

下次构建时，也可以直接将 `-t` 改为：

```bash
-t rongruihouseholds/qsk-book:1.0
```

这样构建后可直接 `docker image push rongruihouseholds/qsk-book:1.0`。

---
# 想要以另外的 namespace 推送

一般情况下，不可以使用与自己 Docker 账户名称不一致的 `namespace`，除非这个名称`tsyecommercialdesigner-cloud` 是：

- 你自己的 Docker ID；或
- 一个 Docker Hub 组织，且你的账号被授予该组织仓库的推送权限（通常为 Owner/Editor）。

本地镜像可以随便标成这个名字，但推送时 Docker Hub 会按命名空间鉴权；无权限会报 `denied: requested access to the resource is denied`。

你有两种选择：

```bash
# 推送到自己的个人命名空间
docker tag tsyecommercialdesigner-cloud/qsk-book:1.0 rongruihouseholds/qsk-book:1.0
docker push rongruihouseholds/qsk-book:1.0
```

或在 Docker Hub 创建/使用名为 `tsyecommercialdesigner-cloud` 的组织，并把 `rongruihouseholds` 加为具备推送权限的成员，然后才能：

```bash
docker push tsyecommercialdesigner-cloud/qsk-book:1.0
```

Docker Hub 仓库必须位于你自己的用户命名空间，或你有权限的组织命名空间下。 [Docker 官方仓库说明](https://docs.docker.com/docker-hub/repos/create/)