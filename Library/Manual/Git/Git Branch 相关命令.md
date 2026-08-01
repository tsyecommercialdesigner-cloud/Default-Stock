---
title: Git Branch 相关命令
created: 2026-07-18
tags:
  - git
  - Branch
  - Git_branch
  - Github_repository
---
---
# 分支的重命名、创建、切换

把当前分支重命名为 `main`：

```bash
git branch -M main
```

创建第二个分支和切换分支通常这样写：

## 创建分支

- 只创建，不切换：
  
```bash
  git branch new-branch
```
 
- 创建并立即切换：

```bash
git checkout -b new-branch
```

- 更现代的写法：

```bash
git switch -c new-branch
```


## 切换分支

- 切换到已有分支：

```bash
git checkout new-branch
```


- 更现代的写法：

```bash
git switch new-branch
```


## 常见组合

如果你想从 `main` 创建第二个分支 `dev`，最常用的一句是：

```bash
git checkout -b dev
```

或者：

```bash
git switch -c dev
```

如果只是想把第二个分支先建好，不切过去：

```bash
git branch dev
```

---
# 分支的合并、Pull Request

将本地的 `computer` 分支合并到远程 `origin` 的 `main` 分支，通常有两种推荐的标准做法，
取决于你是采用**合并后推送**的方式，还是通过 **Pull Request (PR) / Merge Request (MR)** 流程：

## 方法一：通过本地合并后推送（最直观的标准步骤）

流程步骤：

1. 先切换到本地的 `main` 分支
2. 再将 `computer` 分支的代码合并进来
3. 然后再推送到远程 `origin/main`

对应命令是：

```bash
# 1. 切换到本地 main 分支
git checkout main

# 2. 拉取远程 origin/main 的最新代码，确保本地 main 是最新的
git pull origin main

# 3. 将本地 computer 分支的代码合并到当前的 main 分支
git merge computer

# 4. 将合并后的 main 分支推送到远程仓库 origin
git push origin main
```

## 方法二：推送本地分支并提交 PR/MR（团队协作规范流程）

在团队协作开发中，远程的 `main` 分支有保护（Protected Branch），通常不允许直接推送，
推荐以下步骤：

1. 将本地 computer 分支推送到远程 origin：

```bash
git push -u origin computer
```

2. 登录 GitHub / GitLab / Gitee 界面，发起一个 **Pull Request (PR)** 或 **Merge Request (MR)**：

	**Source（源分支）**：`computer`
	
	**Target（目标分支）**：`main`

3. 审查代码并点击界面上的 **Merge** 按钮完成合并。


## 💡 提示与注意事项

- 在合并前，建议确保本地 `computer` 分支的代码都已经提交（`git status` 干净）。
    
- 如果在执行 `git merge computer` 时产生冲突（Conflict），需要先手动解决冲突、提交改动，然后再执行 `git push origin main`。