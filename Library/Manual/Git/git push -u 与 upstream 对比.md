---
title: git push -u 与 upstream 对比
created: 2026-08-01
tags:
  - git
  - Github
  - Git_branch
  - Github_repository
---
---
# Git 命令解析：`git push -u` 与 `upstream` 概念对比

在 Git 版本控制中，很多初学者容易混淆 **`git push -u origin <branch>`** 与涉及 **`upstream`** 的相关操作。本文将对这两者进行清晰的梳理与对比。

## 1. `git push -u origin <branch_name>`（推送并绑定追踪关系）

### **主要作用**
将本地的指定分支推送到名为 `origin` 的远程仓库，并**建立“上游/追踪”（Upstream Tracking）关系**。

### **核心细节**
* **`-u`（即 `--set-upstream`）：** 告诉 Git：“以后这个本地分支默认和 `origin` 上的这个远程分支绑定在一起”。
* **带来的便利：** 首次使用 `-u` 推送成功后，以后在该分支下提交或拉取代码，只需直接输入 `git push` 或 `git pull`，无需再手动指定远程仓库名和分支名。

---

## 2.  `upstream` 命令格式

注意： `upstream` 并不是 `git remote` 的子命令，需要区分以下两条命令：

### ① `git branch --set-upstream-to=<remote>/<branch_name>`（仅绑定上游分支）
如果你已经将代码推送到远程，或者远程分支已存在，只需在本地建立（或修改）绑定关系，无需重新推送代码：

```bash
git branch --set-upstream-to=origin/feature/login
```

### ② `git remote add upstream <URL>`（添加上游远程仓库）
在 GitHub/GitLab 参与开源项目或 Fork 协同开发时，通常会面对两个远程仓库：
* **`origin`**：指向你 **Fork 出来的个人仓库**。
* **`upstream`**：指向 **原作者/官方团队的主仓库**。

添加官方主仓库为远程源的正确命令是：

```bash
git remote add upstream https://github.com/original-owner/repo.git
```

---

## 3. 核心概念对比汇总表

| 命令 / 概念                                | 命令类型        | 核心作用与使用场景                                      |
| :------------------------------------- | :---------- | :--------------------------------------------- |
| **`git push -u origin <branch>`**      | **推送 + 绑定** | 将本地分支推送到 `origin` 仓库，同时设置本地分支追踪该远程分支。          |
| **`git branch --set-upstream-to=...`** | **仅绑定**     | 在不推送代码的前提下，单纯修改或设置本地分支对应的远程上游分支。               |
| **`git remote add upstream <URL>`**    | **添加源**     | 为本地仓库添加一个名为 `upstream` 的远程仓库地址（常用于 Fork 协同开发）。 |

---

## 4. Fork 模式下的经典工作流参考

如果你在维护 Fork 项目，常用的命令流组合如下：

```bash
# 1. 从官方主仓库拉取最新代码
git fetch upstream

# 2. 合并主仓库改动到本地 master 分支
git checkout master
git merge upstream/master

# 3. 将更新后的 master 推送到你自己的 origin 仓库
git push origin master

# 4. 在本地新分支开发完成后，首次推送到 origin 并建立追踪
git checkout -b feature/new-idea
git push -u origin feature/new-idea
```
