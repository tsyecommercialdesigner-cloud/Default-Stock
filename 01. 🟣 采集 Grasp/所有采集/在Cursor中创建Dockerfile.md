# 环境准备与基础镜像构建

开发MCP插件首先需要准备适当的环境。

首先需要构建增强基础镜像，由于地方平台默认镜像缺少node.js和npm，需要构建包含这些组件的增强镜像：[镜像构建命令]。此外用户也可直接使用预构建镜像，避免重复工作。

基础镜像是插件运行的环境基础，需要包含node.js和npm等组件。

使用Dockerfile，打开Cursor编辑器，创建mingweiDockerfile的新文件。

在文件中输入以下代码：

```Dockerfile

FROM langgenius/dify-plugin-daemon:0.0.6-local

RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash -

RUN apt-get install nodejs -y

RUN curl -qL https://www.npmjs.com/install.sh | sh

RUN npm config set registry https://registry.npmmirror.com/

```

该代码以Dify基础镜像为基础，安装node.js和npm等组件。

保存文件后，在终端中执行docker build命令来完成镜像构建。

这种方式便于了解镜像的构建过程，适合需要自定义镜像内容的场景。