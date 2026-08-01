# Cowart MCP 启用失败诊断 SOP

> 适用对象：在 Codex Desktop 中安装 Cowart 后，`cowart_mcp` 未成功启用，或同时未观察到 `node_repl` / Codex 内置 Node 运行时的计算机。  
> 适用系统：macOS、Windows。  
> 诊断重点：确认 Cowart npm 依赖是否完整、网络环境是否导致首次安装失败，以及该故障是否与 Codex 内置运行时未唤起有关。

---

## 1. 结论边界

Cowart 项目明确依赖以下 npm 包：

| 依赖 | Cowart 声明版本 |
|---|---|
| `@modelcontextprotocol/ext-apps` | `^1.7.4` |
| `@modelcontextprotocol/sdk` | `^1.29.0` |
| `tldraw` | `^5.1.1` |
| `vite` | `^7.0.0` |
| `react` | `^19.0.0` |

Cowart 的 `scripts/start-mcp.mjs` 会检查依赖目录。发现依赖缺失时，它会自动执行：

```bash
npm install
```

如果安装失败，启动脚本会退出，真正的 MCP 服务入口 `mcp/server.mjs` 不会被加载，`cowart_mcp` 因而无法完成 STDIO 握手。

可以确认的故障链：

```text
npm 依赖下载失败
→ Cowart 依赖不完整
→ start-mcp.mjs 退出
→ cowart_mcp 无法启动或握手
```

暂时不能仅凭 Cowart 源码确认的故障链：

```text
cowart_mcp 启动失败
→ Codex 必然不唤起 node_repl
```

Cowart 仓库只声明 `cowart_mcp`，没有声明或直接启动 `node_repl`。`node_repl` 属于 Codex Desktop 的独立能力。两者可能在 UI 或会话初始化过程中同时出现，但源码不能证明前者是后者启动的必要条件。

---

## 2. 诊断目标

完成本 SOP 后，应能将问题归入以下一种类型：

1. **Cowart npm 依赖下载失败**
2. **npm 镜像、代理、DNS 或 TLS 配置异常**
3. **依赖完整，但 Cowart MCP 本身启动失败**
4. **`cowart_mcp` 正常，Codex 会话或 MCP 状态未刷新**
5. **Codex 内置运行时缺失、被拦截或版本不一致**
6. **证据不足，需收集 Codex 日志进一步判断**

---

## 3. 准备信息

记录正常电脑与故障电脑的以下信息：

```text
操作系统：
系统版本：
CPU 架构：
Codex Desktop 版本：
Cowart 版本：
Node.js 版本：
npm 版本：
网络环境：
是否使用代理：
是否为企业管理设备：
安全软件或 EDR：
```

检查 Node.js 与 npm：

```bash
node --version
npm --version
```

若命令不存在，Cowart 的 `command = "node"` 无法执行，应先安装或修复 Node.js。

---

## 4. 找到 Cowart 安装目录

Cowart MCP 配置的启动方式是：

```text
node ./scripts/start-mcp.mjs
```

因此需要进入包含以下文件的 Cowart 根目录：

```text
package.json
package-lock.json
.mcp.json
scripts/start-mcp.mjs
mcp/server.mjs
```

在可能的 Codex 插件目录中查找。

### macOS / Linux

```bash
find ~/.codex -type f -path '*/Cowart*/scripts/start-mcp.mjs' 2>/dev/null
find ~/.codex -type f -name 'start-mcp.mjs' 2>/dev/null
```

### Windows PowerShell

```powershell
Get-ChildItem "$HOME\.codex" -Recurse -Filter "start-mcp.mjs" -ErrorAction SilentlyContinue
```

进入搜索结果对应的 Cowart 根目录。

---

## 5. 确认依赖声明

在 Cowart 根目录执行：

```bash
node -e "const p=require('./package.json'); for (const d of ['@modelcontextprotocol/ext-apps','@modelcontextprotocol/sdk','tldraw','vite','react']) console.log(d, p.dependencies?.[d] ?? 'NOT DECLARED')"
```

预期输出类似：

```text
@modelcontextprotocol/ext-apps ^1.7.4
@modelcontextprotocol/sdk ^1.29.0
tldraw ^5.1.1
vite ^7.0.0
react ^19.0.0
```

### 判定

- 全部有版本号：进入下一步。
- 出现 `NOT DECLARED`：插件文件可能不是当前官方仓库版本，或安装目录不正确。
- `package.json` 无法读取：安装不完整或目录错误。

---

## 6. 检查依赖是否完整

执行：

```bash
node -e "
const fs=require('fs');
const deps=[
  '@modelcontextprotocol/ext-apps',
  '@modelcontextprotocol/sdk',
  '@tldraw/assets',
  '@vitejs/plugin-react',
  'react',
  'react-dom',
  'tldraw',
  'vite',
  'zod'
];
for (const d of deps) {
  console.log(d.padEnd(38), fs.existsSync('node_modules/'+d) ? 'OK' : 'MISSING');
}
"
```

### 判定

#### 所有依赖均为 `OK`

依赖目录基本完整，跳到“第 9 节：直接启动 Cowart MCP”。

#### 存在一个或多个 `MISSING`

Cowart 首次启动会尝试执行 `npm install`。继续检查网络和 npm 配置。

---

## 7. 检查 npm 网络与配置

### 7.1 查看 registry

```bash
npm config get registry
```

常见有效值：

```text
https://registry.npmjs.org/
```

在中国大陆环境中，也可以使用组织批准的 npm 镜像，但应确保镜像完整、证书可信且没有过期。

### 7.2 基础连通性测试

```bash
npm ping
npm view @modelcontextprotocol/ext-apps version
npm view @modelcontextprotocol/sdk version
npm view tldraw version
npm view vite version
npm view react version
```

### 7.3 查看代理配置

```bash
npm config get proxy
npm config get https-proxy
```

### macOS / Linux 环境变量

```bash
env | grep -i proxy
```

### Windows PowerShell 环境变量

```powershell
Get-ChildItem Env: | Where-Object Name -Match 'proxy'
```

### 判定

以下错误高度支持网络、代理、DNS 或 TLS 问题：

```text
ETIMEDOUT
ECONNRESET
ENOTFOUND
EAI_AGAIN
ECONNREFUSED
SELF_SIGNED_CERT_IN_CHAIN
UNABLE_TO_VERIFY_LEAF_SIGNATURE
CERT_HAS_EXPIRED
403 Forbidden
Failed to connect
```

注意：终端中代理可用，不代表 Codex Desktop 启动的子进程一定继承相同代理变量。

---

## 8. 手动安装依赖并保留日志

在 Cowart 根目录执行：

### macOS / Linux

```bash
npm install --verbose 2>&1 | tee cowart-npm-install.log
```

### Windows PowerShell

```powershell
npm install --verbose *>&1 | Tee-Object -FilePath cowart-npm-install.log
```

安装完成后执行：

```bash
npm run build
```

再检查依赖：

```bash
node -e "
const fs=require('fs');
for (const d of ['@modelcontextprotocol/ext-apps','@modelcontextprotocol/sdk','tldraw','vite','react']) {
  console.log(d, fs.existsSync('node_modules/'+d) ? 'OK' : 'MISSING');
}
"
```

### 判定

#### `npm install` 失败，并出现网络类错误

可初步诊断为：

> 中国大陆网络、代理、DNS、TLS 检查或 npm 镜像问题导致 Cowart 依赖未安装完整，进而导致 `cowart_mcp` 未成功启用。

#### `npm install` 成功

进入下一步，不应继续把问题简单归因于网络。

#### 安装后依旧缺包

检查：

```bash
npm config list
npm cache verify
```

必要时清理后重新安装：

### macOS / Linux

```bash
rm -rf node_modules
npm ci --verbose
```

### Windows PowerShell

```powershell
Remove-Item node_modules -Recurse -Force
npm ci --verbose
```

优先使用 `npm ci`，因为仓库包含 `package-lock.json`。

---

## 9. 直接启动 Cowart MCP

在 Cowart 根目录执行：

```bash
node ./scripts/start-mcp.mjs
```

### 正常表现

STDIO MCP 服务成功启动后，进程可能看起来停在那里、没有普通输出并等待输入。这通常是正常现象。

使用 `Ctrl+C` 结束测试。

### 异常表现

若立即退出，记录完整错误。常见类型包括：

```text
Cannot find package
MODULE_NOT_FOUND
ERR_MODULE_NOT_FOUND
npm install failed while preparing Cowart MCP
permission denied
operation not permitted
ENOENT
spawn npm ENOENT
```

### 进一步探测

Cowart 项目提供探测脚本时，可执行：

```bash
npm run probe:mcp
```

也可执行完整质量检查：

```bash
npm run quality
```

### 判定

- 手动启动成功：Cowart 源码及依赖基本正常，问题更可能出在 Codex 的插件状态、启动环境或会话刷新。
- 手动启动失败：根据报错继续处理，暂时不诊断为 Codex 内置运行时问题。

---

## 10. 检查 Codex 中的 MCP 状态

执行：

```bash
codex mcp list
```

在 Codex TUI 中也可输入：

```text
/mcp
```

确认是否存在：

```text
cowart_mcp
node_repl
```

完整退出 Codex Desktop 后重新打开，并创建一个全新会话。不要只关闭窗口。

### 状态矩阵

| `cowart_mcp` | `node_repl` | 优先判断 |
|---|---|---|
| 失败 | 正常 | Cowart 路径、依赖、Node 或 npm 问题 |
| 正常 | 失败 | Codex 版本、内置运行时、安全策略或会话问题 |
| 失败 | 失败 | Codex 子进程启动、沙箱、安全策略，或两个独立问题并存 |
| 正常 | 正常 | UI、Widget 或当前会话工具发现问题 |
| 手动启动正常，Codex 中失败 | 任意 | Codex 启动环境与终端环境不同 |

---

## 11. 检查 Codex 内置运行时是否存在

本步骤用于判断 `node_repl` 缺失是否为 Codex 安装或系统拦截问题。

### macOS

```bash
find /Applications/Codex.app -iname '*node_repl*' -print 2>/dev/null
```

检查签名：

```bash
codesign --verify --deep --strict --verbose=2 /Applications/Codex.app
spctl --assess --type execute --verbose=4 /Applications/Codex.app
```

### Windows PowerShell

```powershell
Get-ChildItem "$env:LOCALAPPDATA" -Recurse -Filter "*node_repl*" -ErrorAction SilentlyContinue
Get-ChildItem "$env:PROGRAMFILES" -Recurse -Filter "*node_repl*" -ErrorAction SilentlyContinue
```

同时查看：

```text
Windows 安全中心
→ 病毒和威胁防护
→ 保护历史记录
```

企业设备还需检查：

```text
EDR 隔离记录
AppLocker
WDAC
MDM 安全策略
应用白名单
```

### 判定

- 正常电脑存在，故障电脑不存在：优先诊断 Codex 版本或安装完整性问题。
- 两台均存在，但故障电脑无法启动：优先诊断权限、安全软件、签名或沙箱问题。
- 文件存在且可执行，但 Codex 中不显示：优先诊断 Codex 会话、功能状态或工具发现问题。
- `cowart_mcp` 修复后 `node_repl` 随之出现：只能证明二者在当前 Codex 初始化流程中存在相关性，不能仅凭该现象断言 Cowart 直接启动了 `node_repl`。

---

## 12. 比较正常电脑与故障电脑

两台电脑分别收集：

```bash
node --version
npm --version
npm config get registry
npm config get proxy
npm config get https-proxy
codex mcp list
```

在 Cowart 根目录收集：

```bash
npm ls --depth=0
node ./scripts/start-mcp.mjs
```

重点比较：

```text
Codex Desktop 版本
Cowart 版本
Node.js 版本
npm registry
代理环境变量
node_modules 完整性
node_repl 文件是否存在
安全软件拦截记录
```

---

## 13. 最终诊断模板

### A. 确认为网络导致 Cowart MCP 失败

满足以下证据：

- Cowart 依赖存在 `MISSING`；
- `npm install --verbose` 失败；
- 日志包含超时、DNS、代理、TLS、403 或连接失败；
- 网络或 npm 配置修复后安装成功；
- `node ./scripts/start-mcp.mjs` 随后可以正常保持运行；
- 重启 Codex 后 `cowart_mcp` 成功启用。

诊断结论：

```text
Cowart 首次启动需要从 npm registry 安装依赖。故障电脑的网络、代理、
DNS、TLS 或 npm 镜像配置导致依赖未完整下载，start-mcp.mjs 因此退出，
cowart_mcp 未完成 STDIO 握手。
```

### B. 不能归因于网络

出现以下任一情况时，不应继续简单归因于中国大陆网络：

- `npm install` 成功；
- 所有依赖均为 `OK`；
- `node ./scripts/start-mcp.mjs` 可以正常运行；
- 只有 `node_repl` 缺失或失败；
- Codex 安装目录中缺少内置运行时；
- 安全软件明确拦截 Codex 子进程；
- 正常与故障电脑的 Codex Desktop 版本不同。

诊断方向：

```text
Codex Desktop 版本或安装完整性
Codex 内置运行时状态
系统权限或安全策略
Codex 会话缓存或工具发现
Desktop 子进程未继承终端代理环境
```

---

## 14. 修复建议

### npm 网络问题

优先采用组织允许、稳定且可信的网络和镜像方案。不要长期关闭 TLS 校验。

检查并修正 registry：

```bash
npm config set registry https://registry.npmjs.org/
```

若组织提供正式镜像，使用组织指定地址。

设置代理示例：

```bash
npm config set proxy http://127.0.0.1:PORT
npm config set https-proxy http://127.0.0.1:PORT
```

取消错误代理：

```bash
npm config delete proxy
npm config delete https-proxy
```

重新安装：

```bash
npm ci --verbose
npm run build
npm run probe:mcp
```

### Codex 状态问题

```text
完全退出 Codex Desktop
重新启动
创建全新会话
重新检查 MCP 状态
```

### Codex 安装或内置运行时问题

```text
确认两台机器使用相同版本
重新安装官方 Codex Desktop
检查安全软件隔离记录
检查 macOS 签名或 Windows 应用控制策略
```

---

## 15. 需要提交的诊断材料

向项目维护者或内部 IT 提交时，建议包含：

```text
操作系统及版本
Codex Desktop 版本
Cowart 版本
Node.js / npm 版本
npm registry
是否使用代理
codex mcp list 输出
依赖检查输出
cowart-npm-install.log
node ./scripts/start-mcp.mjs 的完整错误
安全软件拦截记录
正常电脑与故障电脑的差异
```

提交日志前，应删除用户名、目录中的个人信息、代理凭据、访问令牌及内部域名。

---

## 16. 诊断流程摘要

```text
找到 Cowart 根目录
        ↓
确认 package.json 包含所需依赖
        ↓
检查 node_modules 是否缺包
        ↓
缺包 → 测试 npm registry / DNS / 代理 / TLS
        ↓
运行 npm install --verbose
        ↓
运行 npm run build
        ↓
直接运行 node ./scripts/start-mcp.mjs
        ↓
成功 → 重启 Codex 并检查 cowart_mcp
失败 → 根据具体错误修复
        ↓
单独检查 node_repl 文件、版本及系统拦截
        ↓
区分 Cowart 网络故障与 Codex 内置运行时故障
```

---

## 参考源码

- Cowart `package.json`  
  `https://github.com/zhongerxin/Cowart/blob/main/package.json`
- Cowart `.mcp.json`  
  `https://github.com/zhongerxin/Cowart/blob/main/.mcp.json`
- Cowart MCP 启动脚本  
  `https://github.com/zhongerxin/Cowart/blob/main/scripts/start-mcp.mjs`
- Cowart MCP 服务入口  
  `https://github.com/zhongerxin/Cowart/blob/main/mcp/server.mjs`

---

**文档版本：** 1.0  
**编制日期：** 2026-08-01
