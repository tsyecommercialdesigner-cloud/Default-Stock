# 第 6 章 Dify 入门与应用实践

随着 RAG 技术的快速发展，企业对知识库的需求日益增加。一个高效、准确的知识库可以为用户提供更优质的服务，降本增效，同时提升企业的核心竞争力。为满足这一需求，市面上涌现了众多支持快速搭建知识库的工具和平台。其中，Dify 因其功能强大、开源灵活以及广泛的社区支持，逐渐成为企业和开发者的热门选择。

Dify 的优势在于其低门槛的使用体验以及强大的扩展能力。对于开发者来说，它能够快速集成并适配复杂的业务场景；而对于非技术用户，它的简化界面和智能功能可以显著降低使用门槛。此外，Dify 的开源特性使用户可以根据自身需求深度定制，构建符合业务逻辑的个性化解决方案，同时避免对闭源工具的过度依赖。

本章将详细介绍 Dify 的关键特性、使用方法以及其在知识库构建中的具体应用。通过实际案例分析，我们将探讨 Dify 与自研业务的深度结合，展示其在不同场景下的灵活性和高效性，为企业在 RAG 技术背景下实现知识库的最佳实践提供参考。

## 6.1 Dify 简介

Dify 是一款开源的大语言模型应用开发平台，融合了后端即服务（Backend as a Service）和 LLMOps（Large Language Model Operations，大语言模型运营）的理念。该平台旨在帮助开发者快速构建生产级的生成式人工智能应用，同时降低技术门槛，使非技术人员也能轻松参与到人工智能应用相关的工作中。

Dify 一词来源于 Define（定义）和 Modify（改进）的结合，寓意为用户提供一个能够持续定义和改进人工智能应用的平台，同时展现出其核心理念——“为你而做”（Do it for you）。正如 Dify.AI CEO 路宇所言：“我们的社区用户对 Dify 的评价可以归结为简单、克制、迭代迅速。”这一评价不仅体现了 Dify 以用户需求为导向的产品哲学，也凸显了其在技术易用性、功能设计和快速迭代能力上的独特优势。

Dify 内置了开发大语言模型应用所需的核心技术栈，包括对数百种模型的全面支持、直观的 Prompt 编排界面、高效的 RAG 引擎、强大的 Agent 框架、灵活的流程编排工具以及一套简洁易用的界面和 API。这种一站式的解决方案不仅大幅简化了开发流程，还为开发者节省了大量重复造轮子的时间，使他们能够将更多精力投入创新和满足业务需求上。

## 6.2 Dify 安装与部署

Dify 是一款支持私有化部署的开源 AI 应用开发平台，适用于构建问答机器人与智能助手等人工智能原生应用。平台支持多种部署方式，既提供免部署的云服务，也支持基于 AWS AMI 的 Dify Premium 一键部署方案。对于国内用户，推荐使用 Docker Compose 进行本地私有化部署。若选择源码安装，则需确保设备具备不低于 2 核 CPU 和 4GB 内存的基本配置，并满足其前后端分离架构所依赖的服务端、中间件及前端运行环境要求。

### 6.2.1 云服务

Dify 提供了便捷的云服务，用户只需要拥有一个 GitHub 或 Google 账号，即可直接使用 Dify 的全部功能。

云服务的优势显而易见：操作简单、快速上手。然而，这种方式对国内用户可能并不友好，因为需要 GitHub 或 Google 账号支持；此外，数据安全性也是需要考虑的一个重要问题。因此，接下来我们将探讨如何进行私有化部署，从而更好地解决这些潜在问题。

### 6.2.2 Dify Premium 简介

Dify Premium 是一款基于 AWS AMI 的解决方案，支持自定义品牌，并可通过一键部署轻松将其安装到 AWS VPC 中。用户可在 AWS Marketplace 中订阅并使用 Dify Premium，其非常适合以下场景：

- 中小企业私有化部署需求：在本地服务器上创建一个或多个应用程序，同时注重数据的安全性和私密性。
- 资源需求超出 Dify Cloud 计划：对 Dify Cloud 的订阅计划感兴趣，但需要的资源超出当前订阅所包含的范围。
- 企业级应用前的验证：希望在组织内大规模采用 Dify Enterprise 前进行 POC 验证。

### 6.2.3 Docker 部署

对于国内用户，如果想快速使用 Dify，可以选择通过 Docker Compose 进行私有化部署。在安装前需确保设备满足最低系统要求：CPU 不低于 2 核，内存不低于 4GB，以保障部署和运行的顺畅进行。

下面介绍 Docker Compose 部署的具体过程。

#### 1. 复制 Dify 代码仓库

将 Dify 的源代码复制到部署环境。

```bash
git clone https://github.com/langgenius/dify.git
```

#### 2. 启动 Dify

（1）进入 Dify 源代码的 Docker 目录。

```bash
cd dify/docker
```

（2）复制默认环境配置文件。

```bash
cp .env.example .env
```

（3）启动 Docker 容器。

根据系统上的 Docker Compose 的版本，选择合适的命令来启动容器。

如果版本是 Docker Compose V2，则使用以下命令：

```bash
docker compose up -d
```

如果版本是 Docker Compose V1，则使用以下命令：

```bash
docker-compose up -d
```

运行命令后，等待相关容器安装完成，随后检查所有容器是否正常运行。完成上述步骤后，在本地便成功部署了 Dify。

#### 3. 访问 Dify

找到机器的公有 IP，并通过默认的 HTTP 端口 80 访问 Dify。如果你的机器位于国内云厂商的云主机上，请务必调整安全组策略，确保 80 端口可以成功访问。

首先，需要前往管理员初始化页面设置管理员账户。

```text
http://your_server_ip/install
```

然后，访问 Dify 主页面进行登录，如图 6-1 所示。

```text
http://your_server_ip/apps
```

图 6-1 Dify 登录页面

登录成功之后，进入 Dify 的控制页面，如图 6-2 所示。

图 6-2 Dify 控制页面

#### 4. 自定义 Dify 配置

在 Dify 中，`.env` 文件用于配置系统所需的环境变量。如果需要调整相关配置，则可以直接编辑 `.env` 文件，然后重启 Dify，使修改生效。

```bash
docker-compose down
docker-compose up -d
```

### 6.2.4 本地源码部署

本地源代码安装 Dify 之前，请确保设备满足最低系统要求：CPU 不低于 2 核，内存不低于 4GB，以保障部署和运行的顺畅。Dify 采用前后端分离的开发模式，部署需要同时支持服务端和前端页面的运行，并依赖一些中间件。

因此，在本地启动服务之前，先复制 Dify 代码。

```bash
git clone https://github.com/langgenius/dify.git
```

然后部署以下中间件：PostgreSQL、Redis 和 Weaviate（如果本地尚未安装）。

```bash
cd docker
cp middleware.env.example middleware.env
docker-compose -f docker-compose.middleware.yaml up -d
```

#### 1. 服务端部署

服务器端运行 Dify 需要 Python 3.11 或 3.12 环境。为了更好地管理依赖项并避免与系统环境发生冲突，建议使用 Python 虚拟环境。以下是服务端具体的启动步骤。

（1）进入 `api` 目录。

```bash
cd api
```

（2）复制环境变量配置文件。

```bash
cp .env.example .env
```

（3）生成随机密钥，并替换 `.env` 中 `SECRET_KEY` 的值。

```bash
awk -v key="$(openssl rand -base64 42)" '/^SECRET_KEY=/ {sub(/=.*/,"=" key)} 1' .env > temp_env && mv temp_env .env
```

（4）安装依赖包。

Dify API 服务使用 Poetry 来管理依赖。你可以执行 `poetry shell` 来激活环境。

```bash
poetry env use 3.11
poetry install
```

（5）执行数据库迁移。

```bash
poetry shell
flask db upgrade
```

（6）启动 API 服务。

```bash
flask run --host 0.0.0.0 --port=5001 --debug
```

（7）启动 Worker 服务。

```bash
celery -A app.celery worker -P gevent -c 1 -Q dataset,generation,mail,ops_trace --loglevel INFO
```

#### 2. 前端页面部署

前端启动需要安装 Node.js v18.x（建议使用 LTS 稳定版）和 NPM 8.x.x 或 Yarn 的基础环境。访问 Node.js 官网，选择适合操作系统的 v18.x 安装包并完成安装。以下是前端具体的启动步骤：

（1）进入 `web` 目录：

```bash
cd web
```

（2）安装依赖包：

```bash
npm install
```

（3）配置环境变量。

在当前目录下创建文件 `.env.local` 并复制 `.env.example` 中的内容。根据需求修改这些环境变量的值：

```dotenv
# For production release, change this to PRODUCTION
NEXT_PUBLIC_DEPLOY_ENV=DEVELOPMENT

# The deployment edition, SELF_HOSTED
NEXT_PUBLIC_EDITION=SELF_HOSTED

# The base URL of console application, refers to the Console base URL of WEB service if console domain is
# different from api or web app domain.
# example: http://cloud.dify.ai/console/api
NEXT_PUBLIC_API_PREFIX=http://localhost:5001/console/api

# The URL for Web APP, refers to the Web App base URL of WEB service if web app domain is different from
# console or api domain.
# example: http://udify.app/api
NEXT_PUBLIC_PUBLIC_API_PREFIX=http://localhost:5001/api

# SENTRY
NEXT_PUBLIC_SENTRY_DSN=
NEXT_PUBLIC_SENTRY_ORG=
NEXT_PUBLIC_SENTRY_PROJECT=
```

（4）构建代码。

```bash
npm run build
```

（5）启动 Web 服务。

```bash
npm run start
# or
yarn start
# or
pnpm start
```

（6）本地访问 Dify。

访问 `http://127.0.0.1:3000`，即可使用本地部署的 Dify。
