---
title: 赋予Codex调用贞贞API生图的技能
created: 2026-07-09
tags:
  - Skills
  - Codex
---
---
# 添加API Key到系统环境变量

## 方式一：通过 Powershell 添加

以管理员身份运行 PowerShell，然后输入：

```powershell
[Environment]::SetEnvironmentVariable("ZHENZHEN_API_KEY", "sk-xxxxxxxxxxxxxxxxxxx", "Machine")
```

请务必记得将其中的`sk-xxxxxxxxxxxxxxxxxxx`换成你自己的API Key。

说明：

> - `"Machine"` 表示添加为系统环境变量。
> - 设置后需要重启 PowerShell、Codex、IDE 或相关应用，新的环境变量才会被这些进程读取到。
> - 如果只想添加为当前用户变量，把 `"Machine"` 改成 `"User"`。

## 方式二：通过我的电脑添加

通过右键我的电脑->属性->高级系统设置->环境变量，
在环境变量对话框里面手动添加系统级/用户级的环境变量，
变量名就是ZHENZHEN_API_KEY，变量值就是你的API Key，
设置完成后记得点击确定提交更改，
同样需要重启Codex或相关应用，新的环境变量才会被这些进程读取到。

---
# 在Codex中创建技能的方法

## 创建技能前需要先指定存放Skill文件夹的目录

请务必记得在创建技能之前，需要先指定一个本机文件夹用来新建项目。具体方法是：

> 1. 在Codex的左侧边栏找到"项目"；
> 2. 点击位于"项目"右侧的新建图标；
> 3. 在弹出的对话框中，选择"使用现有文件夹"；
> 4. 在资源管理器中指定一个你能够找到的文件夹用来存放项目相关的文件；
> 5. 点击"选择文件夹"以完成项目的创建。

## 使用"创建技能的技能"来创建技能

在Codex的对话框中输入"/Skill"，在弹出的上下文菜单中选择"/Skill Creator"，然后使用提示词来描述你的技能，即可让Codex帮你创建技能。

## 使用"安装技能的技能"来从仓库文件中安装技能

在Codex的对话框中输入"/Skill"，在弹出的上下文菜单中选择"/Skill Installer"，然后就可以从官方提供的技能库或其它来源（比如Github仓库）中安装指定的技能。

## 调用已创建或安装的技能来执行任务

在Codex的对话框中输入"/技能名"来搜索技能，接着在弹出的上下文菜单中选择你已经创建或安装好的技能，然后使用提示词来引导Codex如何使用这项技能，即可让Codex使用技能来执行任务。

---
# 创建文生图的技能

这个技能只能通过纯文本描述来生成图像。

以下是安装"通过调用贞贞API进行GPT Image 2文生图"技能的提示词：

````markdown
[$skill-creator](C:\\Users\\Administrator\\.codex\\skills\\.system\\skill-creator\\SKILL.md) help me create a skill,named it "zhenzhen_image_generations",{user_prompt} is the prompt I need to enter.
```python
import requests
import json

url = "https://ai.t8star.org/v1/chat/completions"

payload = json.dumps({
   "model": "gpt-image-2-all",
   "stream": False,
   "messages": [
      {
         "role": "user",
         "content": "{user_prompt}"
      }
   ]
})
headers = {
   'Accept': 'application/json',
   'Authorization': f'Bearer ${env:ZHENZHEN_API_KEY}',
   'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)
```
````

其中，"model": "`gpt-image-2-all`", 这里的字段你可以换成别的模型，例如：

> - gemini-3.1-flash-lite-image
> - gemini-3.1-flash-image
> - gemini-3-pro-image

然后把提示词里的named it "`zhenzhen_image_generations`", 技能名称改一下，就可以通过Skill Creator来创建另一个技能了，区别只是调用的绘图模型不一样。

---
# 创建单图编辑的技能

这个技能单次只能提交一张参考图，主要用来进行单图修改，也可以把咱们自己的产品图作为唯一的参考图，然后将经过LLM优化/反推过的提示词提交给它来进行生图操作。

以下是安装"通过调用贞贞API进行GPT Image 2单图编辑"技能的提示词：

````markdown
[$skill-creator](C:\\Users\\Administrator\\.codex\\skills\\.system\\skill-creator\\SKILL.md) help me create a skill,named it "zhenzhen_image_edit",{image_url} is the image file I need to upload.
```python
import requests
import json

url = "https://ai.t8star.org/v1/chat/completions"

payload = json.dumps({
   "model": "gpt-image-2-all",
   "stream": False,
   "messages": [
      {
         "role": "user",
         "content": [
            {
               "type": "text",
               "text": "{user_prompt}"
            },
            {
               "type": "image_url",
               "image_url": {
                  "url": "{image_url}"
               }
            }
         ]
      }
   ]
})
headers = {
   'Accept': 'application/json',
   'Authorization': f'Bearer ${env:ZHENZHEN_API_KEY}',
   'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)
```
````

---
# 创建多图编辑的技能

这个技能可以一次提交多张参考图。

以下是安装"通过调用贞贞API进行GPT Image 2多图编辑"技能的提示词：

````markdown
[$skill-creator](C:\\Users\\Administrator\\.codex\\skills\\.system\\skill-creator\\SKILL.md) help me create a skill,name it "zhenzhen_image_multi_edit",{user_prompt} is the prompt I need to enter,{image_url} is the image file I need to upload,and it supports_uploading_multiple_images.
```python
import requests
import json

url = "https://ai.t8star.org/v1/chat/completions"

payload = json.dumps({
   "model": "gpt-image-2-all",
   "stream": False,
   "messages": [
      {
         "role": "user",
         "content": [
            {
               "type": "text",
               "text": "{user_prompt}"
            },
            {
               "type": "image_url",
               "image_url": {
                  "url": "{image_url}"
               }
            },
            {
               "type": "image_url",
               "image_url": {
                  "url": "{supports_uploading_multiple_images_url}"
               }
            }
         ]
      }
   ]
})
headers = {
   'Accept': 'application/json',
   'Authorization': f'Bearer ${env:ZHENZHEN_API_KEY}',
   'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)
```
````

---