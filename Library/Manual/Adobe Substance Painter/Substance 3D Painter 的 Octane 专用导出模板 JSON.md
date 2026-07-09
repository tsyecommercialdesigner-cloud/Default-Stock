---
title: Substance 3D Painter 的 Octane 专用导出模板 JSON
created: 2026-05-22
tags:
  - Adobe_substance_painter
  - File_output_templates
---
这是完全匹配你当前工作流（**Substance → Blender 5.1.1 → Octane Universal Material**）的工程级模板，包含：

- 正确的贴图命名规范
    
- 正确的输出通道（BaseColor / Roughness / Metallic / NormalDX / Height / AO / Emissive）
    
- 正确的色彩空间（sRGB / Linear）
    
- 与自动化插件完全兼容的关键词
    

下面直接给你 **可用 JSON 模板**。

---

# 🎯 Substance → Octane 导出模板 JSON（可直接复制使用）

> 你可以在 **Export Textures → Output Templates → Import Preset** 中导入此 JSON。

```
{
  "name": "Octane_Universal_PBR",
  "default": false,
  "maps": [
    {
      "name": "BaseColor",
      "channels": [
        {
          "channel": "baseColor",
          "usage": "baseColor"
        }
      ],
      "suffix": "_BaseColor",
      "colorSpace": "sRGB"
    },
    {
      "name": "Roughness",
      "channels": [
        {
          "channel": "roughness",
          "usage": "roughness"
        }
      ],
      "suffix": "_Roughness",
      "colorSpace": "linear"
    },
    {
      "name": "Metallic",
      "channels": [
        {
          "channel": "metallic",
          "usage": "metallic"
        }
      ],
      "suffix": "_Metallic",
      "colorSpace": "linear"
    },
    {
      "name": "NormalDX",
      "channels": [
        {
          "channel": "normal",
          "usage": "normal"
        }
      ],
      "suffix": "_NormalDX",
      "colorSpace": "linear",
      "normalFormat": "DirectX"
    },
    {
      "name": "Height",
      "channels": [
        {
          "channel": "height",
          "usage": "height"
        }
      ],
      "suffix": "_Height",
      "colorSpace": "linear"
    },
    {
      "name": "AO",
      "channels": [
        {
          "channel": "ao",
          "usage": "ambientOcclusion"
        }
      ],
      "suffix": "_AO",
      "colorSpace": "linear"
    },
    {
      "name": "Emissive",
      "channels": [
        {
          "channel": "emissive",
          "usage": "emissive"
        }
      ],
      "suffix": "_Emissive",
      "colorSpace": "sRGB"
    }
  ],
  "resolution": "$resolution",
  "fileName": "$material$mapSuffix_$resolution",
  "fileFormat": "png"
}
```

---

# 🟦 导出后文件命名示例（自动生成）

```
Leather_BaseColor_4K.png
Leather_Roughness_4K.png
Leather_Metallic_4K.png
Leather_NormalDX_4K.png
Leather_Height_4K.png
Leather_AO_4K.png
Leather_Emissive_4K.png
```

完全符合自动化插件（如 **Blender PBR Auto‑Importer**、**OC‑AutoPBR**）的识别规则。

---

# 🟧 为什么这个 JSON 是最适合 Octane 的？

- Normal 使用 **DirectX**（Blender + Octane 标准）
    
- Roughness / Metallic / Height / AO 全部 **Linear**
    
- BaseColor / Emissive 使用 **sRGB**
    
- 命名后缀完全匹配自动化插件的关键词
    
- 输出文件名结构统一：`材质名_贴图类型_分辨率`
    
- 兼容 Universal Material 的所有输入槽
    

你导出后可以直接一键导入 Blender Octane，无需手动接节点。

---
