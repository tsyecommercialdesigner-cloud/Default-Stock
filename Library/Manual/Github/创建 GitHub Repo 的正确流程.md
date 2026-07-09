# 最推荐的完整正确流程

这套流程适合第一次把本地项目上传到 GitHub，尤其适合 Obsidian 仓库、代码仓库、包含附件的知识库项目。

核心原则：

- 先准备 `.gitignore`，再 `git add .`
- 先确认没有敏感文件和大文件，再提交
- 推荐使用 SSH remote
- 如果 GitHub 远程仓库已经有 README、`.gitignore` 或 LICENSE，先 `pull --rebase`
- 不要把 `.env`、SSH 私钥、Token、OAuth Secret、Obsidian 插件缓存提交到 GitHub

## 1. 进入项目目录

```bash
cd "你的项目目录"
```

示例：

```bash
cd "D:/Documents/Obsidian Stock/claude-obsidian"
```

## 2. 初始化 Git 仓库

```bash
git init
git branch -M main
```

## 3. 准备 `.gitignore`

在项目根目录创建或检查 `.gitignore`。

Obsidian 仓库推荐至少忽略：

```gitignore
/.obsidian/
/_attachments/
/.codex/*
!/.codex/hooks.json

.env
.env.local
*.pem
*.key
id_rsa*
id_ed25519*
credentials*
secrets.y*ml
auth.json

__pycache__/
*.pyc
.venv/

node_modules/

.DS_Store
Thumbs.db

*.mkv
*.mp4
*.mov
*.avi
*.gif
*.png
*.jpg
*.jpeg
*.webp
*.epub
*.pdf
*.mobi
*.azw3
```

## 4. 检查将要提交的内容

```bash
git status
```

如果看到 `.env`、私钥、Token、`.obsidian/plugins/`、大文件电子书、视频等敏感或大体积文件，先处理 `.gitignore`，不要急着提交。

## 5. 添加并提交

```bash
git add .
git commit -m "Initial commit"
```

## 6. 添加 GitHub 远程仓库

推荐使用 SSH：

```bash
git remote add origin git@github.com:你的用户名/你的仓库名.git
```

示例：

```bash
git remote add origin git@github.com:tsyecommercialdesigner-cloud/claude-obsidian.git
```

检查 remote：

```bash
git remote -v
```

## 7. 如果远程仓库已有内容，先拉取

如果你在 GitHub 创建仓库时勾选了 README、`.gitignore` 或 LICENSE，远程仓库已经有提交。

这时先执行：

```bash
git pull origin main --rebase --allow-unrelated-histories
```

如果没有冲突，就继续推送。

如果有冲突，先解决冲突，然后：

```bash
git add .
git rebase --continue
```

## 8. 推送到 GitHub

第一次推送建议绑定上游分支：

```bash
git push -u origin main
```

以后普通推送可以用：

```bash
git push origin main
```

## 9. 确认同步成功

```bash
git status
```

如果看到：

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

说明本地和 GitHub 已经同步完成。

## 10. 常见问题快速处理

### src refspec main does not match any

通常是还没有 commit，或者本地没有 `main` 分支。

```bash
git branch -M main
git add .
git commit -m "Initial commit"
git push -u origin main
```

### fetch first

远程已有提交，本地没有，需要先拉取。

```bash
git pull origin main --rebase --allow-unrelated-histories
git push origin main
```

### Push cannot contain secrets

提交历史里有密钥、Token、OAuth Secret 等敏感信息。

不要点 GitHub 的 allow secret。应从提交历史中移除敏感文件后再推送。

### 大文件已经提交进历史

彻底删除历史中的大文件可用：

```bash
git filter-repo --force --path "文件路径" --invert-paths
git remote add origin git@github.com:你的用户名/你的仓库名.git
git push --force origin main
```

注意：`git push --force` 会重写 GitHub 上的历史，只在你明确知道自己要覆盖远程历史时使用。

