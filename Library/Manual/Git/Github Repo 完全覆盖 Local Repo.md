---
title: Github Repo 完全覆盖 Local Repo
created: 2026-08-01
tags:
  - Github_repository
  - git
  - Github
---
---
想让Github上游仓库的内容完全覆盖掉本地仓库，并放弃本地仓库的一切更改，以保持和Github仓库完全一致，该怎么做？


可以，用 Git 的“硬重置 + 清理未跟踪文件”来让本地仓库完全等于 GitHub 远端。

假设你的远端叫 `origin`，目标分支是 `main`：

```bash
git fetch origin
git reset --hard origin/main
```

先确认是否真的属于未跟踪文件：

```bash
git status
git clean -fdn
```

其中 `git clean -fdn` 只会预览当前仍会删除的内容，不会恢复已删除文件。

然后执行命令删除未被 Git 跟踪的文件和目录：

```bash
git clean -fd
```

>[!warning] 注意
>此命令删除的是**未被 Git 跟踪的文件和目录**，Git 没有保存这些文件的历史，
>因此通常不能通过 `git reset`、`git reflog` 或 `git checkout` 回退。
>若文件从未提交/暂存且没有本地历史或备份，Git 本身无法恢复这些被删除的文件。
>为了避免误删，建议先固定使用预览模式 `git clean -fdn` ，
>确认输出无误后再执行 `git clean -fd`。

如果还想删除被 `.gitignore` 忽略的本地文件，例如构建产物、`.env`、缓存等，让目录更彻底地和远端一致：

```bash
git clean -fdx
```

所以最彻底版本是：

```bash
git fetch origin
git reset --hard origin/main
git clean -fdx
```

如果你的分支叫 `master`，把 `origin/main` 改成：

```bash
origin/master
```

如果不确定当前远端分支名，可以先看：

```bash
git branch -r
```

注意：这些命令会永久放弃本地修改、未提交文件、未跟踪文件。尤其是 `git clean -fdx` 会连 `.env` 这类被忽略文件也删掉。比较稳的做法是先确认当前状态：

```bash
git status
```

如果仓库用了子模块，还可以再执行：

```bash
git submodule update --init --recursive
git submodule foreach --recursive git reset --hard
git submodule foreach --recursive git clean -fdx
```

如果不小心误删了文件：

首先检查是否曾提交、暂存或被其他分支包含。如果文件其实曾被 Git 跟踪过，可以从提交恢复：

```bash
git log --all -- path/to/file
git restore --source=<commit> -- path/to/file
```

或查看悬空对象（仅对曾 `git add` 过、并且对象尚未被 GC 清理的文件偶尔有效）：

```bash
git fsck --lost-found
```

---