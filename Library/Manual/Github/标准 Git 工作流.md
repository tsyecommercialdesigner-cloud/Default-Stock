---
title: 标准 Git 工作流
created: 2026-07-18
tags:
  - git
source_url: "[atlassian](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)"
---
---
# 功能说明

这是一套适合 **main / dev / feature** 的标准 Git 工作流，偏实战、适合团队协作，核心思路是：

- `main` 保持可发布
- `dev` 作为集成分支
- `feature` 从 `dev` 分出来开发，完成后再合回 `dev`
-  `feature` 不直接进 `main`

## 分支职责

- `main`：正式发布分支，保持稳定、可上线。
    
- `dev`：开发集成分支，日常把功能汇总到这里。
    
- `feature/*`：单个功能分支，每个需求单独一个分支。
    

## 推荐习惯

- `main` 只放可发布版本，不在上面直接开发。
    
- `dev` 作为集成入口。
    
- `feature` 命名尽量统一，比如 `feature/login`、`feature/payments`。
    
- 团队协作时，尽量通过 PR/MR 合并，而不是直接 `merge` 到主分支。
    

---
# 创建项目的仓库

## 本地环境准备

先在项目根目录准备好 `.gitignore`，尤其不要提交 `.env`、API Key、数据库和构建产物。

然后执行：

```bash
cd C:\path\to\your-project

git init
git branch -M main
git add .
git commit -m "Initial commit"
```

## 创建远程仓库

在[Github](https://github.com/)上创建远程`Repository`，目标仓库最好先创建为空仓库，不要预先添加`README`。

如果项目已经是 Git 仓库，只需检查并推送：

```bash
git remote -v
git push -u origin main
```

## 连接远程仓库

```bash
git remote add origin <repo-url>
git push -u origin main
```

# 在 Codex Cloud 连接仓库

## 登录与授权

依次执行下列步骤：

1. 打开 [Codex Cloud](https://chatgpt.com/codex)。
2. 登录与 GitHub 关联的 OpenAI/ChatGPT 账号。
3. 授权 Codex GitHub App。
4. 选择刚推送的仓库和分支。
5. 创建 Cloud environment，然后启动任务。

>[!warning] 注意
>如果是私有仓库，需要在 GitHub App 设置中明确授权该仓库。

## 配置云端运行环境

Codex Cloud 会重新克隆仓库，不会继承你电脑上已安装的依赖。因此项目最好包含：

- `package.json` / `package-lock.json`
- `requirements.txt` / `pyproject.toml`
- 构建、测试和启动说明
- 可选的 `AGENTS.md`，告诉 Codex 安装、测试、格式化命令

例如环境初始化命令可以是：

```bash
npm ci
```

或者：

```bash
pip install -r requirements.txt
```

>[!warning] 注意
>`.env` 不要推到 GitHub；
>需要的密钥应在 Codex Cloud environment 中配置为 `secret/environment variable`。

## 与云端保持同步

本地改动上传：

```bash
git add .
git commit -m "Describe changes"
git push
```

云端改动通常由 Codex 创建分支或 Pull Request，再拉回本地：

```bash
git fetch origin
git switch <codex创建的分支>
```

或者先在 GitHub 合并 PR，再更新本地：

```bash
git switch main
git pull origin main
```

# 开发与维护

## 创建 `dev` 分支并推到远端：

```bash
git checkout -b dev
git push -u origin dev
```

或者现代写法：

```bash
git switch -c dev
git push -u origin dev
```

## 新建 feature 分支

从 `dev` 拉出一个功能分支：

```bash
git checkout dev
git pull origin dev
git checkout -b feature/login
git push -u origin feature/login
```

或者：


```bash
git switch dev 
git pull origin dev
git switch -c feature/login
git push -u origin feature/login
```

## 日常开发

在 `feature/*` 上正常提交：

```bash
git add .
git commit -m "feat: add login form"
git push
```

如果远端 `dev` 有更新，先同步再继续：


```bash
git checkout dev
git pull origin dev
git checkout feature/login
git merge dev
```

或者用 rebase：

```bash
git rebase dev
```

## 合并 feature 到 dev

功能完成后，切回 `dev` 合并：

```bash
git checkout dev
git pull origin dev
git merge feature/login
git push origin dev
```

合并完删除本地和远端 feature 分支：

```bash
git branch -d feature/login
git push origin --delete feature/login
```

# 发布更新

## 发布到 main

当 `dev` 稳定后，把它合并到 `main`：

```bash
git checkout main
git pull origin main
git merge dev
git push origin main
```

通常发布前还会打 tag：

```bash
git tag -a v1.0.0 -m "release v1.0.0"
git push origin v1.0.0
```

# 附录：常用命令速查

```bash
# 创建 dev
git switch -c dev
# 从 dev 创建 feature
git switch dev
git switch -c feature/x
# 提交
git add .
git commit -m "feat: ..."
# 推送 feature
git push -u origin feature/x
# 合回 dev
git switch dev
git merge feature/x
git push
# 删除 feature
git branch -d feature/x
git push origin --delete feature/x
```

