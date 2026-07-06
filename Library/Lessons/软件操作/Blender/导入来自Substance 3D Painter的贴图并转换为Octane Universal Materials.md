---
title: 导入来自Substance 3D Painter的贴图并转换为Octane Universal Materials
created: 2026-05-20
tags:
  - Octane_render
  - Octane_material_conversion
---
# 节点桥接方式

在 Blender 版 Octane 渲染器的Universal Materials材质里，通常这样接：

> - **Base Color** → Diffuse / Albedo
> - **Roughness** → Roughness
> - **Metallic** → Metallic
> - **Normal** → 先过 Normal Map 节点，再进材质的 Normal 输入
> - **Height** → 进 Bump 或 Displacement
> - **AO** → 可与 Base Color 混合，或保留后期合成
> - **Opacity** → 进 Alpha / Opacity
> - **Emissive** → 进Emissive
  
如果是粗糙度、金属度、AO、Height 这类黑白贴图，要记得在 Blender 里把纹理色彩空间设成 **Non-Color**，这是标准做法。 

> [!warning] Octane 里要特别注意的点
> - Base Color 不要被当作Linear数据
> - Roughness、Metallic、AO、Height、Normal 都要按Non-Color数据处理
> - Normal 贴图的方向如果不对，可能需要翻转绿色通道
> - 如果你导出的是旧式 Specular/Glossiness 工作流，也能接，但现在更推荐 Metallic/Roughness

> [!info] 在Blender中直接使用.sbsar材质的可行性
> 可以直接在 Blender 里通过 Substance 3D 面板加载 `.sbsar` 材质，
> 并在输出区域调整文件格式和位深。
> 但对 Octane 来说，实际工作里更常见的还是**导出贴图后手动接到 Octane 材质节点**，
> 这样可控性更高，也更容易排错。

# 最实际的工作流程

> 1. 在 Substance Painter 里确认模型法线、UV、烘焙都正确。
> 2. 导出贴图时选 Metallic/Roughness 预设。
> 3. 把 Normal、Roughness、Metallic、AO、Height 设为Non-Color输出或16-bit输出。
> 4. 回到 Blender，创建 Octane 材质。
> 5. 按贴图类型逐个接线。
> 6. 检查 Roughness 和 Normal 的方向、强度。
> 7. 先做低采样预览，再微调。
