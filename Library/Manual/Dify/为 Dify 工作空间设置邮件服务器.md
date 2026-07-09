---
title: 为 Dify 工作空间设置邮件服务器
created: 2026-07-09
tags:
  - Dify
  - Dify_workspace
---
Dify 的邀请邮件要通过 `.env` 配置邮件服务。你现在是 Docker Compose 部署，所以改这里：

```
/opt/dify/docker/.env
```

Dify 官方说明：邮件用于成员邀请、密码重置、登录验证码、Human Input 节点通知等；并且 **邮件里的链接依赖 `CONSOLE_WEB_URL`**。参考：[Dify Environment Variables - Mail Configuration](https://docs.dify.ai/en/self-host/deploy/configuration/environments)。

**推荐配置方式：SMTP**

进入目录：

```
cd /opt/dify/docker
sudo nano .env
```

找到或新增这些配置：

```
MAIL_TYPE=smtp
MAIL_DEFAULT_SEND_FROM=你的发件邮箱

SMTP_SERVER=你的SMTP服务器地址
SMTP_PORT=465
SMTP_USERNAME=你的发件邮箱账号
SMTP_PASSWORD=你的SMTP授权码或密码
SMTP_USE_TLS=true
SMTP_OPPORTUNISTIC_TLS=false
SMTP_LOCAL_HOSTNAME=
```

如果你用 Gmail，SMTP 配置通常是固定的：

```
MAIL_TYPE=smtp
MAIL_DEFAULT_SEND_FROM=yourname@gmail.com

SMTP_SERVER=smtp.gmail.com
SMTP_PORT=465
SMTP_USERNAME=你的 Gmail 地址
SMTP_PASSWORD=你的 Google 应用专用密码
SMTP_USE_TLS=true
SMTP_OPPORTUNISTIC_TLS=false
```

注意：SMTP_PASSWORD 不是 Gmail 登录密码

现在 Gmail 一般不能直接用账号登录密码作为 SMTP 密码。你需要生成：

```
应用专用密码 / App Password
```

步骤：

1. 打开 Google 账号安全页：

```
https://myaccount.google.com/security
```

2. 确认已经开启：

```
两步验证 / 2-Step Verification
```

3. 开启后，进入：

```
应用专用密码 / App passwords
```

可以直接访问：

```
https://myaccount.google.com/apppasswords
```

4. 创建一个应用专用密码，比如名称填：

```
Dify SMTP
```

5. Google 会生成一串 16 位左右的密码。

把这串密码填到：

```
SMTP_PASSWORD=生成的应用专用密码
```

注意不要填普通 Gmail 登录密码。

如果你找不到应用专用密码，常见原因：

```
没有开启两步验证
账号是公司/学校 Workspace，管理员禁用了应用专用密码
账号启用了某些高级安全策略
```

这种情况下你需要换一个允许 SMTP 的邮箱，或者用 SendGrid / Resend 等邮件服务。

**Dify 最终示例**

```
MAIL_TYPE=smtp
MAIL_DEFAULT_SEND_FROM=yourname@gmail.com

SMTP_SERVER=smtp.gmail.com
SMTP_PORT=465
SMTP_USERNAME=yourname@gmail.com
SMTP_PASSWORD=你的Google应用专用密码
SMTP_USE_TLS=true
SMTP_OPPORTUNISTIC_TLS=false
```

如果你用的是 `465` 端口，通常这样：

```
SMTP_PORT=465
SMTP_USE_TLS=true
SMTP_OPPORTUNISTIC_TLS=false
```

如果你用的是 `587` 端口，通常这样：

```
SMTP_PORT=587
SMTP_USE_TLS=true
SMTP_OPPORTUNISTIC_TLS=true
```

不要优先用 `25` 端口。云服务器上 `25` 经常被云厂商限制，且更容易被邮件服务商拒收。

**还要确认 URL 配置**

你现在 Dify 地址是：

```
http://49.234.14.72:7670
```

所以 `.env` 里建议确认：

```
CONSOLE_WEB_URL=http://49.234.14.72:7670
CONSOLE_API_URL=http://49.234.14.72:7670
APP_WEB_URL=http://49.234.14.72:7670
APP_API_URL=http://49.234.14.72:7670
FILES_URL=http://49.234.14.72:7670
```

尤其是：

```
CONSOLE_WEB_URL=http://49.234.14.72:7670
```

否则邀请邮件里的链接可能是空的、localhost，或者跳转不对。

确认都改完以后重启：

```
cd /opt/dify/docker
docker compose -p dify up -d --force-recreate api worker
```

如果你想保险一点，也可以：

```
docker compose -p dify up -d --force-recreate
```

**测试**

回到 Dify 页面：

```
工作区设置 -> 成员 -> 邀请成员
```

发一封邀请邮件。

如果失败，看日志：

```
cd /opt/dify/docker
docker compose -p dify logs --tail=100 api
docker compose -p dify logs --tail=100 worker
```

