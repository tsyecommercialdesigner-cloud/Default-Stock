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
# 将本地的`master`重命名为`main`

`git branch -M main` 是把当前分支重命名为 `main`。

创建第二个分支和切换分支通常这样写：

## 创建分支

- 只创建，不切换：`git branch new-branch`
    
- 创建并立即切换：`git checkout -b new-branch`
    
- 更现代的写法：`git switch -c new-branch`
    

## 切换分支

- 切换到已有分支：`git checkout new-branch`
    
- 更现代的写法：`git switch new-branch`
    

## 常见组合

如果你想从 `main` 创建第二个分支 `dev`，最常用的一句是：

bash

`git checkout -b dev`

或者：

bash

`git switch -c dev`

如果只是想把第二个分支先建好，不切过去：

bash

`git branch dev`
