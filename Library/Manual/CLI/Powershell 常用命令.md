---
title: Powershell 常用命令
created: 2026-08-02
tags:
  - Powershell
  - User_Account
  - Permission_Management
---
---
# 属性

## 查看属性

查看属性：

```powershell
attrib "$env:USERPROFILE\Desktop\desktop.ini"
```

返回结果：

```plain
A  SHR  D:\Music\desktop.ini
```

字段的含义：

- `S` = System，系统文件；
- `H` = Hidden，隐藏文件；
- `R` = Read-only，只读文件；
- `A` = Archive，存档属性。

---
# 权限

## 检查权限

检查权限：

```powershell
icacls "$env:USERPROFILE\Desktop\desktop.ini"
```

返回结果：

```plain
C:\Users\Administrator\Desktop\desktop.ini NT AUTHORITY\SYSTEM:(I)(F)
                                             BUILTIN\Administrators:(I)(F)
                                             DESKTOP-XXXX\Administrator:(I)(F)
                                             Successfully processed 1 files; Failed processing 0 files
```

字段的含义：

```plain
(F)     完全控制
(M)     修改
(RX)    读取和执行
(R)     读取
(W)     写入
(D)     删除
(I)     从父文件夹继承
(OI)    对象继承：对子文件生效
(CI)    容器继承：对子文件夹生效
(IO)    仅继承：规则本身不作用于当前对象
(NP)    不传播继承
(DENY)  显式拒绝
```

返回信息解读：

- `NT AUTHORITY\SYSTEM:(I)(F)`：`SYSTEM` 账户拥有完全控制。
- `BUILTIN\Administrators:(I)(F)`：本机管理员组拥有完全控制。
- `...\Administrator:(I)(F)`：当前用户拥有完全控制。
- `(I)`：权限从父目录**继承**而来（Inherited）。
- `(F)`：完全控制（Full control）。

## 显式拒绝

如果出现：

```plain
C:\Users\Administrator\Desktop\desktop.ini DESKTOP-XXXX\Administrator:(DENY)(W)
                                             DESKTOP-XXXX\Administrator:(I)(F)
                                             NT AUTHORITY\SYSTEM:(I)(F)
```

含义：

- `(DENY)(W)`：明确拒绝该用户写入。
- 即使下面同时存在 `(F)`，**拒绝规则通常优先于允许规则**。
- 所以该账户依然可能无法写入。

这类情况不常见，但一旦存在，就足以解释“访问被拒绝”。

## 提升权限

检查当前窗口是否提升到“管理员”权限：

```powershell
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()
).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

返回 `True` 表示 PowerShell 已提升为管理员权限。
返回 `False` 表示没有提升权限。需关闭该窗口，右键 PowerShell/终端选择“以管理员身份运行”。

---
# 所有者

## 检查文件所有者

```powershell
(Get-Acl "$env:USERPROFILE\Desktop\desktop.ini").Owner
```

---
# 重启资源管理器

在 Powershell 中运行：

```powershell
Stop-Process -Name explorer -Force
Start-Process explorer.exe
```