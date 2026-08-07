---
title: sed 常用指令速查表
created: 2026-08-07
tags:
  - sed
  - Stream_Editor
---
---

*`sed`（Stream Editor）是 Linux/Unix 环境下功能极其强大的流编辑器，常用于文本过滤、替换、删除及批量处理。本文档整理了 `sed` 的常用指令与核心技巧，提供简明的语法拆解与实战示例。*

---

# 一、 sed 基础语法与常用参数

### 基础命令格式
```bash
sed [选项] '编辑指令' 文件名
```

### 常用命令行选项（Options）
| 参数 | 说明 |
| :--- | :--- |
| `-n`, `--quiet`, `--silent` | 取消默认输出。默认情况下 `sed` 会打印处理后的每一行；结合 `-n` 与 `p` 指令可只打印匹配行。 |
| `-i[SUFFIX]` | **直接修改文件内容**（In-place）。如果不提供后缀，则直接覆盖原文件；若提供后缀（如 `-i.bak`），则会在修改前备份原文件。 |
| `-e script` | 拼接多个编辑脚本/指令。 |
| `-E`, `-r` | 使用扩展正则表达式（ERE），免去对 `+`、`?`、`()` 等特殊字符进行反斜杠转义。 |

---

# 二、 文本替换指令（s 命令）

替换操作是 `sed` 中最频繁使用的功能。

### 1. 全局文本替换
* **功能介绍**：将文本中匹配到的所有指定字符串替换为新的字符串。
* **语法结构**：
  ```bash
  sed 's/原字符串/目标字符串/g' filename
  ```
  * `s`：替换命令（substitute）。
  * `/.../.../`：分隔符（支持使用 `#`、`@` 等符号替代 `/`，避免路径中的斜杠冲突）。
  * `g`：全局替换标记（global）。若不加 `g`，默认仅替换每一行中**首次**出现的匹配项。
* **用例说明**：
  ```bash
  # 将 config.txt 中所有的 http 替换为 https
  sed 's/http/https/g' config.txt
  
  # 使用 @ 作为分隔符替换文件路径（避免斜杠转义）
  sed 's#/usr/local/bin#/opt/bin#g' path.txt
  ```

---

### 2. 定向行/指定次数替换
* **功能介绍**：仅对特定行数，或每行中第 N 次出现的匹配项进行替换。
* **语法结构**：
  ```bash
  sed 'N s/原字符串/目标字符串/' filename      # 仅替换第 N 行
  sed 's/原字符串/目标字符串/N' filename      # 仅替换每行中第 N 次匹配项
  ```
* **用例说明**：
  ```bash
  # 仅把第 5 行中的 debug 改为 release
  sed '5 s/debug/release/' app.conf
  
  # 把每行中第二次出现的 "error" 替换为 "warning"
  sed 's/error/warning/2' log.txt
  ```

---

### 3. 正则捕获组反向引用（Backreference）
* **功能介绍**：利用括号匹配并捕获特定文本段，在替换目标中使用 ``、`` 等进行引用。
* **语法结构**：
  ```bash
  sed -E 's/(正则1)(正则2)/ /' filename
  ```
* **用例说明**：
  ```bash
  # 将 "key=value" 格式调整为 "value: key"
  echo "PORT=8080" | sed -E 's/([A-Z]+)=(.*)/: /'
  # 输出: 8080: PORT
  ```

---

# 三、 行删除与过滤指令（d 命令）

### 1. 按行号删除
* **功能介绍**：删除指定行、多行区间或末尾行。
* **语法结构**：
  ```bash
  sed 'Nd' filename          # 删除第 N 行
  sed 'N,Md' filename        # 删除第 N 到 M 行
  sed '$d' filename          # 删除最后一行
  ```
* **用例说明**：
  ```bash
  # 删除首行（通常用于去掉 CSV/表头）
  sed '1d' data.csv
  
  # 删除第 10 行到第 20 行
  sed '10,20d' server.log
  ```

---

### 2. 按模式（Pattern）正则匹配删除
* **功能介绍**：删除所有包含特定模式或匹配正则的行。
* **语法结构**：
  ```bash
  sed '/匹配模式/d' filename
  ```
  * `/pattern/`：需要匹配的模式。
  * `d`：删除指令（delete）。
* **用例说明**：
  ```bash
  # 删除所有的空白行
  sed '/^$/d' file.txt
  
  # 删除所有以 # 开头的注释行
  sed '/^[[:space:]]*#/d' nginx.conf
  
  # 删除所有包含 "DEBUG" 的日志行
  sed '/DEBUG/d' app.log
  ```

---

# 四、 文本打印与检索指令（p 命令）

### 1. 按模式检索并打印
* **功能介绍**：提取并打印特定条件匹配到的文本行。
* **语法结构**：
  ```bash
  sed -n '/匹配模式/p' filename
  ```
  * `-n`：关闭默认输出（非常重要，否则未匹配行也会被打印）。
  * `p`：打印指令（print）。
* **用例说明**：
  ```bash
  # 打印包含 ERROR 的日志行
  sed -n '/ERROR/p' sys.log
  
  # 打印第 15 到第 25 行
  sed -n '15,25p' sys.log
  ```

---

### 2. 提取两个模式之间的文本块（地址范围）
* **功能介绍**：匹配从起始模式到结束模式之间的所有行（闭区间）。
* **语法结构**：
  ```bash
  sed -n '/起始模式/,/结束模式/p' filename
  ```
* **用例说明**：
  ```bash
  # 提取日志文件中从 "BEGIN TRANSACTION" 到 "END TRANSACTION" 之间的内容
  sed -n '/BEGIN TRANSACTION/,/END TRANSACTION/p' db.log
  ```

---

# 五、 文本插入、追加与替换（a, i, c 命令）

### 1. 行前插入与行后追加
* **功能介绍**：在指定行或匹配行的上方（insert）或下方（append）插入新行。
* **语法结构**：
  ```bash
  sed 'N i\追加内容' filename     # 在第 N 行前插入
  sed 'N a\追加内容' filename     # 在第 N 行后追加
  sed '/模式/ a\追加内容' filename # 在匹配行后追加
  ```
* **用例说明**：
  ```bash
  # 在第一行开头插入配置文件头信息
  sed -i '1 i\# Auto-generated config file' settings.ini
  
  # 在包含 "server_name" 的行之后追加一行配置
  sed '/server_name/ a\    listen 8080;' nginx.conf
  ```

---

### 2. 整行替换（c 命令）
* **功能介绍**：将指定行或匹配到的整行直接替换为新文本。
* **语法结构**：
  ```bash
  sed 'N c\新内容' filename
  sed '/匹配模式/ c\新内容' filename
  ```
* **用例说明**：
  ```bash
  # 将第 3 行整个替换为指定内容
  sed '3 c\ENVIRONMENT=production' .env
  ```

---

# 六、 组合多条编辑指令

### 1. 使用分号 `;` 链接多个命令
* **功能介绍**：在一个 `sed` 表达式中按顺序执行多项操作。
* **语法结构**：
  ```bash
  sed '指令1; 指令2; 指令3' filename
  ```
* **用例说明**：
  ```bash
  # 先删除注释行，再删除空行
  sed '/^#/d; /^$/d' config.conf
  ```

---

### 2. 使用 `-e` 选项链式处理
* **用例说明**：
  ```bash
  sed -e 's/foo/bar/g' -e 's/temp/prod/g' file.txt
  ```

---

# 七、 生产环境高频实战场景总结

| 场景需求            | 推荐命令示例                                                       |
| :-------------- | :----------------------------------------------------------- |
| **就地修改并备份原文件**  | `sed -i.bak 's/old/new/g' file.txt`                          |
| **删除空白行与注释行**   | `sed -i -E '/^[[:space:]]*($\|#)/d' config.conf`             |
| **在文件末尾追加一行**   | `sed -i '$ a\export PATH=$PATH:/usr/local/bin' ~/.bashrc`    |
| **提取指定时间段的日志**  | `sed -n '/2026-08-06 10:00/,/2026-08-06 11:00/p' access.log` |
| **批量修改网络配置/IP** | `sed -i 's/192.168.1.100/10.0.0.5/g' *.sh`                   |

---