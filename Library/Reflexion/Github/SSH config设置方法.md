# SSH config 设置方法

这份指南只保留本次排查后确认最简单、最稳定的 GitHub SSH 配置方式。

结论先说：

**不要优先给 GitHub SSH 配 v2rayN 的 ProxyCommand。**

本次最稳的方案是：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

它的含义是：

- Git remote 仍然使用正常 SSH 地址：`git@github.com:用户名/仓库名.git`
- SSH 实际连接 GitHub 时走 `ssh.github.com`
- 端口走 `443`
- 不经过 `ProxyCommand`
- 不使用 v2rayN mixed port

---
# 将 Git Bash 的 SSH 统一设置为 Windows 的 OpenSSH

## 全局性设置

在 **Git Bash** 里执行：

```bash
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

这样 Git for Windows 以后执行 `git clone/pull/push` 走 SSH 时，会统一使用 Windows 自带的 OpenSSH。

可验证：

```bash
git config --global --get core.sshCommand
```

如果以后想取消这个设置：

```bash
git config --global --unset core.sshCommand
```

## 仅设置为对当前仓库生效

在当前仓库目录里执行，不加 `--global`：

```bash
git config core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

验证：

```bash
git config --get core.sshCommand
```

取消当前仓库设置：

```bash
git config --unset core.sshCommand
```

这个设置会写入当前仓库的 `.git/config`，只对这个仓库生效。

## 关于 Config 配置文件的补充说明

执行：

```
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

只是让 Git 使用 **Windows 自带的 `ssh.exe`**。这个 `ssh.exe` 仍然会读取你用户目录下的 SSH 配置：

```
C:\Users\Administrator\.ssh\config
```

也就是你已有的这段配置仍然有效：

```
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

你不应该把 `config` 放到：

```
C:\Windows\System32\OpenSSH\
```

那个目录是 OpenSSH 程序目录，不是用户配置目录。

可以这样测试是否生效：

```
ssh -T git@github.com
```

如果你想确认 Git 调用的也是 Windows OpenSSH，可以执行：

```
git config --global --get core.sshCommand
GIT_SSH_COMMAND="C:/Windows/System32/OpenSSH/ssh.exe -v" git ls-remote git@github.com:你的用户名/你的仓库.git
```

看到连接到 `ssh.github.com`、端口 `443`，就说明你的 `C:\Users\Administrator\.ssh\config` 被正确读取了。

---
# 总结性反思

## 一、本次踩过的坑

### 1. 不要一开始就配置 ProxyCommand

之前尝试过：

```sshconfig
ProxyCommand /mingw64/bin/connect.exe -H 127.0.0.1:10888 %h %p
```

或：

```sshconfig
ProxyCommand /mingw64/bin/connect.exe -S 127.0.0.1:10888 %h %p
```

结果出现：

```text
Connection closed by UNKNOWN port 65535
```

原因是 v2rayN 的“本地混合监听端口”虽然存在，但 mixed port 和 `connect.exe + SSH ProxyCommand` 不一定兼容。

### 2. 不要把 v2rayN.exe 的 PID 当成代理端口监听进程

`tasklist | findstr v2rayN` 看到的是 v2rayN 图形界面进程。

真正监听端口的通常是：

```text
xray.exe
```

所以看到 `v2rayN.exe` 的 PID，并不代表它就是端口监听进程。

### 3. 不要把 HTTPS 代理设置当成 SSH 代理设置

这些命令主要影响 HTTPS remote：

```bash
git config --global http.proxy
git config --global https.proxy
```

如果 remote 是：

```text
git@github.com:用户名/仓库名.git
```

那走的是 SSH，不是 HTTPS。

### 4. 不要忘记 `ssh -T` 里的横杠

错误写法：

```bash
ssh T git@github.com
```

正确写法：

```bash
ssh -T git@github.com
```

## 二、最正确、最简便的设置方法

## 1. 打开 SSH config 文件

在 Git Bash 中执行：

```bash
notepad ~/.ssh/config
```

或者在 PowerShell 中执行：

```powershell
notepad $env:USERPROFILE\.ssh\config
```

如果文件不存在，记事本会提示是否创建，选择创建即可。

注意：

- 文件名必须是 `config`
- 不能是 `config.txt`
- 路径是 `C:\Users\Administrator\.ssh\config`

## 2. 写入最稳配置

把文件内容设置为：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

如果你有其他 SSH 配置，可以保留；但 GitHub 这一段建议不要加 `ProxyCommand`。

## 3. 保存文件

保存后关闭记事本。

## 4. 测试 SSH 连接

执行：

```bash
ssh -T git@github.com
```

成功时会看到类似：

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

这表示 GitHub SSH 已经连通。

## 5. 确认 Git remote 使用 SSH

进入你的仓库目录，执行：

```bash
git remote -v
```

正确的 SSH remote 类似：

```text
origin  git@github.com:你的用户名/你的仓库名.git (fetch)
origin  git@github.com:你的用户名/你的仓库名.git (push)
```

如果现在还是 HTTPS：

```text
https://github.com/你的用户名/你的仓库名.git
```

改成 SSH：

```bash
git remote set-url origin git@github.com:你的用户名/你的仓库名.git
```

例如：

```bash
git remote set-url origin git@github.com:tsyecommercialdesigner-cloud/claude-obsidian.git
```

如果有 upstream，也可以改：

```bash
git remote set-url upstream git@github.com:上游用户名/仓库名.git
```

## 6. 测试 Git 拉取

执行：

```bash
git fetch --progress origin
```

如果成功，说明 Git 已经能通过 SSH 连接 GitHub。

再检查：

```bash
git status
```

如果看到：

```text
Your branch is up to date with 'origin/main'.
```

说明本地和 GitHub 状态正常。

## 三、最终推荐配置

日常只保留这个：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_rsa
```

不要加：

```sshconfig
ProxyCommand ...
```

除非以后你明确知道：

- 代理端口类型是 HTTP 还是 SOCKS5
- `connect.exe` 路径正确
- mixed port 不会导致 SSH 断连

## 四、以后遇到问题时的最短排查顺序

### 1. 先测 SSH

```bash
ssh -T git@github.com
```

### 2. 再测 Git remote

```bash
git remote -v
```

### 3. 再测 fetch

```bash
git fetch --progress origin
```

### 4. 最后看状态

```bash
git status
```

不要一上来就改代理，也不要一上来就改 v2rayN。

## 五、一句话记忆

**GitHub SSH 卡住时，先用 `ssh.github.com:443`，不要先折腾 v2rayN mixed port。**

