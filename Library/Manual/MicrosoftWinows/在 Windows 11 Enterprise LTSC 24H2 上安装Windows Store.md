---
title: 在 Windows 11 Enterprise LTSC 24H2 上安装Windows Store
created: 2026-07-28
tags:
  - Microsoft_Winows
  - Windows_Store
---
---
# 必要提示

## 系统版本说明

Windows 11 Enterprise LTSC 24H2 的内核机制相比旧版 LTSC 更严苛：

- 抽离了基础依赖
- 锁定核心服务
- 更新了 AppX 校验  

## 核心原则与避雷点

> 1. **不要直接运行 wsreset.exe \-i**：24H2 下极易中断并导致“版本受限/链接无效”报错。  
> 2. **不要下载旧版 Win10 的脚本**：会导致依赖库不匹配。  
> 3. **不能只装商店主体 .msixbundle**：系统缺少 3 个基础依赖包，直接安装必定报 0x80073CF9 / 0x80004005。  
> 4. **必须带上强制参数**：24H2 需要带上 \-ForceUpdateFromAnyVersion 绕过底层校验。

# 操作流程

## 第一步：解锁系统部署服务（解决服务锁死/拒绝访问）

按 Win \+ R 输入 regedit 打开注册表，将以下两个路径中的 **Start** 值修改为 **3**（手动启动），然后**重启一次电脑**：

> 1. 计算机\\HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\AppXSvc \-\> 双击 Start 改为 **3**  
> 2. 计算机\\HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\InstallService \-\> 双击 Start 改为 **3**

## 第二步：提取并下载正确的文件包

> 1. 打开应用包提取网站：[https://store.rg-adguard.net](https://store.rg-adguard.net)  
> 2. 选择 URL(link)，输入：`https://apps.microsoft.com/detail/9WZDNCRFJBMP`，通道选择 RP 或 Retail，点击对勾。  
> 3. 在生成的下载列表中，必须下载以下 4 个文件（均选择 **x64** 版本）：  
   * 依赖 1：`Microsoft.VCLibs.140.00\_...\_x64.Appx`  
   * 依赖 2：`Microsoft.UI.Xaml.2.8\_...\_x64.Appx`  
   * 依赖 3：`Microsoft.NET.Native.Runtime.2.2\_...\_x64.Appx`  
   * **商店主体**：`Microsoft.WindowsStore\_...\_neutral\_\~\_...\_x64.Msixbundle`（版本号需大于 22400+）

## 第三步：按顺序在 PowerShell 中一键部署

以管理员身份打开 PowerShell 终端，依次复制并运行以下命令（将路径替换为文件的实际路径）：

```PowerShell  
\# 1\. 强行清理残留  
Get-AppxPackage *WindowsStore* -AllUsers | Remove-AppxPackage -AllUsers -ErrorAction SilentlyContinue

\# 2\. 依次安装 3 个核心基础依赖  
Add-AppxPackage -Path "C:\\Downloads\\Microsoft.VCLibs.140.00\_x64.Appx"  
Add-AppxPackage -Path "C:\\Downloads\\Microsoft.UI.Xaml.2.8\_x64.Appx"  
Add-AppxPackage -Path "C:\\Downloads\\Microsoft.NET.Native.Runtime.2.2\_x64.Appx"

\# 3\. 带有强制覆盖参数安装商店主体（核心解决 0x80073CF9 报错）  
Add-AppxPackage -Path "C:\\Downloads\\Microsoft.WindowsStore\\_x64.Msixbundle" -ForceUpdateFromAnyVersion -ForceApplicationShutdown
```

## 完成

安装完成后，无需重启，按下快捷键 `Win + R` 并输入 ：

```plain
ms-windows-store:
```

全新的微软应用商店即可直接无缝打开使用！
免登录任何微软账户，也能随意下载使用 VS Code、Windows Terminal 等各类免费软件。