---
title: Windows 11 终端快捷键
created: 2026-08-02
source: Cherry Studio
tags:
  - Shortcuts
  - Windows_Terminal
---
---
# 原生方式

Windows 11 **默认没有类似 Linux 那种单一步骤的全局快捷键**（如 `Ctrl + Alt + T`），但提供了几种非常快速的快捷键组合来打开终端：

## 最常用的快捷启动方式

### 1. 高级用户菜单（最快捷的官方组合键）

- **打开普通终端：** 按 **`Win + X`**，然后按下字母 **`I`**
- **打开管理员终端：** 按 **`Win + X`**，然后按下字母 **`A`**

---

### 2. “运行”窗口方式

1. 按 **`Win + R`** 打开“运行”窗口。
2. 输入 **`wt`**（Windows Terminal 的缩写）并回车。
   * 如果想以**管理员身份**打开，输入 `wt` 后按下 **`Ctrl + Shift + Enter`** 即可。

---

### 3. Quake 模式（下拉式终端）

如果 Windows 终端已经在后台运行，按下：
- **`Win + ~`**（Tab 键上方、Esc 下方的波浪线键）

这会从屏幕顶部拉出一个顶置的终端窗口（再按一次隐藏）。

---

# 自定义方式

## 自定义像 Linux 那样的 `Ctrl + Alt + T` 快捷键

如果你习惯了使用 `Ctrl + Alt + T` 打开终端，可以手动设置一个：

1. 按 **`Win + R`**，输入 `%LocalAppData%\Microsoft\WindowsApps` 并回车。
2. 在打开的文件夹中找到 **`wt.exe`**，右键点击它选择 **“显示更多选项” → “创建快捷方式”**（或直接右键发送到桌面快捷方式）。
3. 右键刚生成的快捷方式，点击 **属性**。
4. 将光标移到 **“快捷键”** 框中，直接按下 **`Ctrl + Alt + T`**。
5. 点击 **确定** 保存。

之后在任意界面按下 `Ctrl + Alt + T` 即可直接召唤 Windows Terminal。
[1] [csdn.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHdkfy3JMZ7Yf3noxGWaYXdKi0K0EeHJi1wDeEsoNck_ugZ_K3aoYLvJ6SKC7v93PmSJ7IQn8T8KE-LIVMKwjXBQbkmsw4cyPk_83lulVjNr-1sWQhO4i5bwlLlwFCkMV-Ry4SVrZuOWao7feQ42w==)

[2] [51cto.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHXfmDT6HJF7lZXJwLcHZCWvmafJWVRlm6ghDNb9YOVvHAg5PlQQwqeQd09VXbTKiZRD7lK7kRTXdmGOkoOmFUu9qXEsnh2-XZucJ8T48qkXonCS5KUJ3gFr76iMtPcHpe_6Q==)

[3] [aomeitech.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGcar1WGpXMYKGe2nx2qbqPwfgzq-00EJT1qHKM_pRV933Y-sCzhdSOzEVL7wkkqRJrzZ63hY6mVI-0QouH0BWt388GoEMML4T6JueawAen3BYs-F0HczZHefNYZqeVt1wesdv-BOquvOierEGyvd1SeV-QQpx3nXHJeheMId8y0cCw8_0=)

[4] [sikich.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHtw0a8ZFIjXGvl36lV_Nk5Jh7yLFZe6-KjEmOKW47lsowJgFG5sWI_7bQqTFQbQgWqkV4DCwTcMSzN--w3d61PaqdoeRQnqNxzbASzOEcuGP6KPgnBG4J_YXhcruIdKdmKezwFt0O4UUluOBx9APAPhfsZdLvM2f5UnwhQisEWDrGDUhRuikPdaG1TUJ47wgqHZ7eZKWg93UsmkwQwrPo=)

[5] [csdn.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEFQqiq6uNberTDUA_-ZW2IkT22kJsYFMwByzbiK51AmoF8pV-RAmVSU3ZjBSKgx_gRrEN1a8s0WeYIO4DTYrElhsBFxcFG3kauLlb3XoSZak0TlvSvJszBWlpGkelkKcD0czxLKca95HLCBTzs_vYoPVA=)

[6] [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFuq0ZkDyhGOhOj45VJTI17g_xVn5FREW7EqfpDmMXYj3MRcmaxOUpuwXCqlhMCpeCR6zzgcCWJjdLervCk1oZV3qRVbDxsLGUOQ9xhQcOoJU4w78PI0DeB1Qxnffo3736FllQ5eatnLGC9kwdMvxRdrcrRbPPOd_FrE5ZLR10DQ-rNjxgJ45uv-jUVUGcsVPTRCwsqidXfutOnPjMbyUI=)