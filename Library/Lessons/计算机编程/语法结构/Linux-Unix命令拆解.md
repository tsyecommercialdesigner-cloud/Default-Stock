# Linux/Unix 命令拆解

> 本文根据本次会话整理，说明脚本执行权限、`chmod +x`、`ls -l` 权限字段以及 `./setup.sh` 的含义。

# 一、给脚本添加执行权限

正确命令：

```bash
chmod +x deploy.sh
```

不要写成：

```bash
chmod + x
```

原因是：

- `+x` 是一个完整的权限参数；
- 后面必须提供要修改的文件名。

基本语法：

```bash
chmod [权限修改] 文件名
```

示例：

```bash
chmod +x setup.sh
```

含义：

> 为 `setup.sh` 增加执行权限。

---

# 二、Linux/Unix 的三种基本权限

| 字符 | 英文 | 含义 |
|---|---|---|
| `r` | read | 读取 |
| `w` | write | 写入、修改 |
| `x` | execute | 执行 |

例如：

```text
rwx
```

表示同时具有：

- 读取权限
- 写入权限
- 执行权限

而：

```text
r-x
```

表示：

- 可以读取
- 不可以写入
- 可以执行

其中 `-` 表示对应权限未授予。

---

# 三、`chmod +x`、`u+x` 与 `a+x`

## 3.1 `chmod +x`

```bash
chmod +x deploy.sh
```

通常表示为该文件增加执行权限。

## 3.2 只给文件所有者增加执行权限

```bash
chmod u+x deploy.sh
```

其中：

```text
u = user / owner
```

## 3.3 给所属组增加执行权限

```bash
chmod g+x deploy.sh
```

其中：

```text
g = group
```

## 3.4 给其他用户增加执行权限

```bash
chmod o+x deploy.sh
```

其中：

```text
o = others
```

## 3.5 明确给所有人增加执行权限

```bash
chmod a+x deploy.sh
```

其中：

```text
a = all
```

常用对象缩写：

| 字符 | 含义 |
|---|---|
| `u` | 文件所有者 |
| `g` | 文件所属组 |
| `o` | 其他用户 |
| `a` | 所有人 |

---

# 四、拆解 `-rwxr-xr-x`

示例输出：

```text
-rwxr-xr-x 1 user user 1234 Aug 1 deploy.sh
```

这通常来自：

```bash
ls -l deploy.sh
```

将其拆开：

```text
- rwx r-x r-x
│ │   │   │
│ │   │   └─ 其他用户权限
│ │   └───── 所属组权限
│ └───────── 文件所有者权限
└─────────── 文件类型
```

---

## 4.1 第一个字符：文件类型

```text
-
```

表示普通文件。

常见文件类型：

| 字符 | 类型 |
|---|---|
| `-` | 普通文件 |
| `d` | 目录 |
| `l` | 符号链接 |
| `c` | 字符设备 |
| `b` | 块设备 |

因此：

```text
-rwxr-xr-x
```

开头的 `-` 表示它是普通文件，而不是目录。

---

## 4.2 文件所有者权限：`rwx`

```text
rwx
```

表示文件所有者可以：

- 读取文件
- 修改文件
- 执行文件

例如，所有者可以执行：

```bash
./deploy.sh
```

也可以编辑：

```bash
vim deploy.sh
```

---

## 4.3 所属组权限：`r-x`

```text
r-x
```

表示同组用户可以：

- 读取
- 执行

但不能：

- 修改

---

## 4.4 其他用户权限：`r-x`

最后一组：

```text
r-x
```

表示既不是文件所有者、也不属于文件所属组的其他用户可以：

- 读取
- 执行

但不能修改文件。

---

# 五、拆解完整的 `ls -l` 输出

原始示例：

```text
-rwxr-xr-x 1 user user 1234 Aug 1 deploy.sh
```

各字段含义如下：

| 字段 | 示例 | 含义 |
|---|---|---|
| 文件类型与权限 | `-rwxr-xr-x` | 普通文件及三组权限 |
| 硬链接数量 | `1` | 指向该 inode 的硬链接数量 |
| 文件所有者 | `user` | owner |
| 文件所属组 | `user` | group |
| 文件大小 | `1234` | 1234 字节 |
| 修改日期 | `Aug 1` | 最后修改时间 |
| 文件名 | `deploy.sh` | 文件名称 |

可翻译为：

> `deploy.sh` 是一个普通文件，大小为 1234 字节，属于 `user` 用户和 `user` 用户组。所有者可以读取、修改和执行；组用户与其他用户可以读取和执行，但不能修改。

---

# 六、执行前后权限变化

执行前：

```text
-rw-r--r-- 1 user user 1234 Aug 1 deploy.sh
```

权限含义：

```text
所有者：读、写
所属组：只读
其他人：只读
```

运行：

```bash
chmod +x deploy.sh
```

执行后可能变为：

```text
-rwxr-xr-x 1 user user 1234 Aug 1 deploy.sh
```

新增的 `x` 就是执行权限。

---

# 七、`./setup.sh` 的含义

命令：

```bash
./setup.sh
```

可以拆成：

```text
.  /  setup.sh
│     │
│     └─ 文件名
└─────── 当前目录
```

其中：

```text
.
```

表示当前目录。

因此：

```bash
./setup.sh
```

含义是：

> 执行当前目录中的 `setup.sh` 文件。

假设当前目录为：

```text
/home/user/project
```

那么：

```bash
./setup.sh
```

实际指向：

```text
/home/user/project/setup.sh
```

---

# 八、为什么不能总是直接输入 `setup.sh`

输入：

```bash
setup.sh
```

时，Shell 通常只会到 `PATH` 环境变量列出的目录中查找命令，例如：

```text
/usr/local/bin
/usr/bin
/bin
```

当前目录通常不会默认包含在 `PATH` 中。

因此，即使当前目录里存在 `setup.sh`，直接输入：

```bash
setup.sh
```

也可能得到：

```text
command not found
```

使用：

```bash
./setup.sh
```

等于明确告诉 Shell：

> 不要去 `PATH` 中查找，请执行当前目录里的这个文件。

这也是一种安全设计，可以防止当前目录中的同名恶意程序冒充系统命令。

---

# 九、直接执行脚本需要满足的条件

要成功执行：

```bash
./setup.sh
```

通常需要：

## 9.1 文件存在

```bash
ls -l setup.sh
```

## 9.2 文件具有执行权限

```bash
chmod +x setup.sh
```

## 9.3 脚本第一行指定解释器

常见 Bash 脚本：

```bash
#!/bin/bash
```

或更具可移植性的：

```bash
#!/usr/bin/env bash
```

这一行称为：

```text
shebang
```

它告诉系统使用哪个解释器执行脚本。

---

# 十、`./setup.sh` 与 `bash setup.sh` 的区别

## 10.1 直接执行

```bash
./setup.sh
```

特点：

- 文件本身必须具有执行权限；
- 系统读取 shebang 来决定解释器；
- 更像执行一个普通程序。

## 10.2 显式交给 Bash

```bash
bash setup.sh
```

特点：

- 由 `bash` 命令读取脚本；
- 脚本文件本身通常不需要 `x` 权限；
- 只需要 Bash 对该文件拥有读取权限；
- 会忽略脚本 shebang 对其他解释器的选择。

因此，即使：

```bash
./setup.sh
```

提示：

```text
Permission denied
```

仍可能可以运行：

```bash
bash setup.sh
```

但这不代表脚本已经具有可执行权限。

---

# 十一、典型安装流程

很多 Linux、WSL、MCP Server、Agent Skill 或开发工具文档会给出：

```bash
chmod +x install.sh
./install.sh
```

含义依次是：

1. 给 `install.sh` 添加执行权限；
2. 执行当前目录中的 `install.sh`。

另一个常见流程：

```bash
chmod +x setup.sh
./setup.sh
```

---

# 十二、Windows 11 中的注意事项

## 12.1 原生 PowerShell 与 CMD

Windows 原生 NTFS 权限模型不同于 Unix execute bit，因此：

```bash
chmod +x script.sh
```

不是标准的 PowerShell 权限管理方式。

## 12.2 可使用 `chmod` 的常见环境

在 Windows 11 中，以下环境经常支持 Unix 风格命令：

- WSL / WSL2
- Git Bash
- MSYS2
- Cygwin
- Linux Docker 容器
- SSH 登录到 Linux 服务器

例如在 WSL 中：

```bash
chmod +x setup.sh
./setup.sh
```

## 12.3 Git Bash 的注意事项

Git Bash 可以识别许多 Unix 命令，但底层仍可能位于 NTFS 文件系统上。权限位的保存和表现还可能受到 Git 配置、挂载方式及执行环境影响。

---

# 十三、常见报错排查

## 13.1 `Permission denied`

```text
bash: ./setup.sh: Permission denied
```

检查：

```bash
ls -l setup.sh
chmod +x setup.sh
```

## 13.2 `No such file or directory`

```text
bash: ./setup.sh: No such file or directory
```

可能原因：

- 当前目录没有该文件；
- 文件名大小写错误；
- shebang 指向不存在的解释器；
- Windows 换行符导致解释器路径异常。

检查：

```bash
pwd
ls -la
```

## 13.3 `command not found`

直接运行：

```bash
setup.sh
```

出现错误时，尝试：

```bash
./setup.sh
```

## 13.4 Windows 换行符问题

脚本在 Windows 中编辑后，可能使用 CRLF 换行符。在 Linux 中可能出现：

```text
/usr/bin/env: ‘bash\r’: No such file or directory
```

可以转换为 LF：

```bash
dos2unix setup.sh
```

或使用编辑器将行尾改为：

```text
LF
```

---

# 十四、速查表

| 命令或字段 | 含义 |
|---|---|
| `chmod +x file.sh` | 增加执行权限 |
| `chmod u+x file.sh` | 只给所有者增加执行权限 |
| `chmod a+x file.sh` | 给所有人增加执行权限 |
| `ls -l file.sh` | 查看详细权限 |
| `-` | 普通文件 |
| `d` | 目录 |
| `r` | 读取 |
| `w` | 写入 |
| `x` | 执行 |
| `.` | 当前目录 |
| `./setup.sh` | 执行当前目录中的脚本 |
| `bash setup.sh` | 使用 Bash 读取并执行脚本 |
| `pwd` | 显示当前目录 |
| `ls -la` | 显示目录中的全部文件及详细信息 |

---

# 十五、核心记忆

```bash
chmod +x setup.sh
./setup.sh
```

可以记成：

> 先允许脚本被直接执行，再运行当前目录中的脚本。

而：

```text
-rwxr-xr-x
```

可以记成：

```text
普通文件
所有者：读、写、执行
所属组：读、执行
其他人：读、执行
```
