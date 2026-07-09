---
title: 导出适配Blender版Octane渲染器的纹理贴图
created: 2026-05-20
tags:
  - Octane_render
  - Octane_material_conversion
  - Export_textures
---
# 一句话总结

最省事也最稳定的做法就是：

> **导入选择 PBR Metallic Roughness 模板，
> 导出选择 PBR Metallic Roughness 贴图，
> Blender 里用 Octane 节点手动搭材质，
> 黑白贴图全部设为 Non-Color，
> 文件格式选择PNG 16-bit 。**

# 导入模板选择

在 Adobe Substance 3D Painter 里新建项目，为来自Blender的3D文件选择项目模板时：

可以选择默认的 **ASM - PBR Metallic Roughness (starter_assets)** ；

如果材质需要做成半透明的，选择 **PBR - Metallic Roughness Alpha-blend(starter_assets)** ；

如果材质需要做出全透明的，选择 **PBR - Metallic Roughness Alpha-test(starter_assets)** 。

# 导出通道设置

在 Adobe Substance 3D Painter 里做好贴图，准备导出给Blender的Octane渲染时：

==先把模型烘焙和贴图绘制完成，然后在 Export Textures 里选一个适合 Octane 的 PBR 预设。==

如果没有现成的 Octane 预设，就选 **Metallic/Roughness** 路线，导出常用通道：

> - **Base Color**
> - **Roughness**
> - **Metallic**
> - **Normal**
> - **Height**
> - **Ambient Occlusion**
> - **Opacity（可选）**
> - **Emissive（可选）**

输出通道和位深都可以在插件/导出设置里调整，Height 还可以单独启用位移输出。

# 文件格式设置

推荐的文件格式：

>- **Base Color**：PNG 或 TIFF。
>- **Roughness / Metallic / AO / Height / Normal**：建议 PNG 16-bit 或 TIFF，避免精度损失。
>- **最终要给高质量渲染用**：也可以用 EXR，但常规贴图多数 PNG/TIFF 就够。
>- **Normal** 一般是彩色图，但在 Blender/Octane 里要按“Non-Color”处理。
