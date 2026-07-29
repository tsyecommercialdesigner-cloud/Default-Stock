# 修改 SSH passphrase 并应用到 ssh-agent

这份指南用于处理 Git / GitHub 使用 SSH 时反复提示输入私钥密码的问题。

如果你看到：

```text
Enter passphrase for key '/c/Users/Administrator/.ssh/id_rsa':
```

这里要求输入的是 **SSH 私钥的 passphrase**，不是 GitHub 登录密码。

## 1. 确认私钥文件位置

常见私钥路径：

```text
C:\Users\Administrator\.ssh\id_rsa
```

在 Git Bash 中通常显示为：

```text
/c/Users/Administrator/.ssh/id_rsa
```

注意：

- `id_rsa` 是私钥，passphrase 设置在这个文件上
- `id_rsa.pub` 是公钥，不要修改它

## 2. 修改 SSH passphrase

在 Git Bash 中执行：

```bash
ssh-keygen -p -f ~/.ssh/id_rsa
```

在 PowerShell 中执行：

```powershell
ssh-keygen -p -f $env:USERPROFILE\.ssh\id_rsa
```

然后按提示输入：

```text
Enter old passphrase:
Enter new passphrase:
Enter same passphrase again:
```

## 3. 取消 SSH passphrase

如果你想以后不再输入 passphrase，可以在设置新密码时直接回车。

流程大概是：

```text
Enter old passphrase: 输入旧 passphrase
Enter new passphrase: 直接回车
Enter same passphrase again: 直接回车
```

这样私钥就不再有 passphrase。

注意：取消 passphrase 会更方便，但安全性会降低。如果电脑多人共用，或者私钥可能被复制，不建议取消。

## 4. 启用 ssh-agent

`ssh-agent` 可以记住你的 SSH 私钥。你输入一次 passphrase 后，后续 GitHub Desktop、Git Bash、PowerShell 的 SSH 操作通常就不需要反复输入。

在 PowerShell 中执行：

```powershell
Get-Service ssh-agent
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
```

如果 `Start-Service ssh-agent` 提示服务已经在运行，可以忽略。

## 5. 把私钥添加到 ssh-agent

在 PowerShell 中执行：

```powershell
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

如果你的私钥仍有 passphrase，这里会要求你输入一次。

添加成功后，后续通常不需要反复输入。

## 6. 如果修改过 passphrase，重新加载私钥

如果你刚改过 passphrase，建议先清空 agent 中旧的缓存，再重新添加：

```powershell
ssh-add -D
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

## 7. 测试 GitHub SSH 是否正常

执行：

```bash
ssh -T git@github.com
```

成功时通常会看到类似：

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

这表示 SSH key 已经能正常连接 GitHub。

## 8. 常见问题

### Permission denied publickey

说明 GitHub 没有识别你的 SSH key，可能是：

- 公钥没有添加到 GitHub
- 当前使用的不是你添加到 GitHub 的那把私钥
- `~/.ssh/config` 指向了错误的 IdentityFile

### 仍然反复要求输入 passphrase

可以检查 agent 中是否已经有 key：

```powershell
ssh-add -l
```

如果显示没有身份信息，再执行：

```powershell
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

### GitHub Desktop 仍然卡住

先用命令行测试：

```bash
ssh -T git@github.com
git fetch --progress origin
```

如果命令行正常，GitHub Desktop 多半是界面缓存或认证状态问题，可以重启 GitHub Desktop 后再试。

