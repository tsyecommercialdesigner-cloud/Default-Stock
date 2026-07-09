---
title: obsidian-git 插件的 ssh-agent 配置
created: 2026-07-08
tags:
  - obsidian-git
  - git
---
# 一次搞定流程

前提：Windows 桌面版 Obsidian，插件是 `Vinzent03/obsidian-git`，私钥是：

```
C:\Users\Administrator\.ssh\id_rsa
```

1. 启用 Windows OpenSSH Agent

用管理员 PowerShell：

```
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

2. 把 `id_rsa` 加入 agent

用普通 PowerShell：

```
ssh-add $env:USERPROFILE\.ssh\id_rsa
ssh-add -l
```

3. 配置 Git 使用 Windows 本机 OpenSSH

```
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
git config --global ssh.variant ssh
```

4. 配置 `C:\Users\Administrator\.ssh\config`

因为你要绕过 22 端口，使用 GitHub 的 SSH over HTTPS 端口：

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile C:/Users/Administrator/.ssh/id_rsa
  IdentitiesOnly yes
```

5. 远程仓库地址仍然用 `github.com`

在 Obsidian 仓库目录里检查：

```
git remote -v
```

应类似：

```
origin  git@github.com:用户名/仓库.git (fetch)
origin  git@github.com:用户名/仓库.git (push)
```

如果不是，改成：

```
git remote set-url origin git@github.com:用户名/仓库.git
```

不要写成 `git@ssh.github.com:用户名/仓库.git`，因为 `ssh.github.com` 和 `Port 443` 已经由 `ssh config` 接管了。

6. 测试 SSH 和 Git

```
ssh -T git@github.com
```

成功时会看到类似：

```
Hi 用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

再在仓库目录测试：

```
git pull
git push
```

终端成功后，Obsidian Git 插件里也应该能同步。

7. Obsidian Git 插件设置

如果插件找不到 Git，到：

```
Settings -> Git -> Advanced -> Custom Git binary path
```

填：

```
C:\Program Files\Git\cmd\git.exe
```

或者用这个命令查实际路径：

```
where git
```

`Additional Environment Variables` 通常不需要填。尤其不要乱填：

```
SSH_ASKPASS=...
GIT_ASKPASS=...
```

---
# 错题反思

这次关键误区有三个：

1. 私钥文件名搞错了  
    你的机器上没有 `id_ed25519`，实际私钥是 `id_rsa`，所以应该执行：

```
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

2. Obsidian Git 后台没有交互终端  
    所以如果私钥没有提前加入 `ssh-agent`，SSH 会尝试调用 `ssh_askpass` 弹窗要密码。一旦 askpass 在 Windows/Electron 环境里启动失败，就会出现：

```
CreateProcessW failed error:193
ssh_askpass: posix_spawnp: Unknown error
Permission denied (publickey)
```

正确做法是提前让 `ssh-agent` 持有已解锁的私钥。

3. 绕过 22 端口时，remote 不要改乱  
    正确结构是：

```
Git remote: git@github.com:用户名/仓库.git
SSH config: HostName ssh.github.com + Port 443
```

也就是 remote 仍写 `github.com`，让 `~\.ssh\config` 把它转发到 `ssh.github.com:443`。