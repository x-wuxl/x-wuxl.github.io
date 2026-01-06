---
layout: post
title: "使用 Nano Banana Pro 进行老照片修复 - 高真实感人像修复完整指南"
description: "详细讲解如何使用 Nano Banana Pro 进行老照片修复，包含完整的结构化配置提示词，实现不改脸、不美颜、不换脸的高真实感人像修复增强，保留自然肤质和细节。"
date: 2025-12-17 09:20:00 +0800
categories: [AI工具, 图像处理]
tags: [老照片修复, AI修复, 人像修复, Nano Banana Pro, 图像增强, 照片还原, AI工具, 提示词工程]
author: wuxl
image: 
  path: /assets/images/nano-banana.png
  alt: "老照片修复效果对比图"
keywords: "老照片修复, AI照片修复, 人像修复, Nano Banana Pro, 照片增强, 图像还原, AI修复工具, 真实感修复, 人像增强, 提示词配置"
lang: zh-CN
permalink: /posts/nano-banana-pro-photo-restoration-guide/
canonical_url: https://x-wuxl.github.io/posts/nano-banana-pro-photo-restoration-guide/

# SEO优化标签
seo:
  type: BlogPosting
  headline: "使用 Nano Banana Pro 进行老照片修复 - 高真实感人像修复完整指南"
  description: "详细讲解如何使用 Nano Banana Pro 进行老照片修复，包含完整的结构化配置提示词，实现不改脸、不美颜、不换脸的高真实感人像修复增强。"
  keywords: "老照片修复, AI照片修复, 人像修复, Nano Banana Pro, 照片增强, 图像还原, AI修复工具"
  author: "wuxl"
  datePublished: "2025-12-17T09:20:00+08:00"
  dateModified: "2025-12-17T09:20:00+08:00"

# Open Graph 标签
og:
  title: "使用 Nano Banana Pro 进行老照片修复 - 高真实感人像修复完整指南"
  description: "详细讲解使用 Nano Banana Pro 进行老照片修复的完整配置，不改脸、不美颜、不换脸的高真实感修复方案。"
  type: article
  url: https://x-wuxl.github.io/posts/nano-banana-pro-photo-restoration-guide/
  image: https://x-wuxl.github.io/assets/images/nano-banana.png
  locale: zh_CN

# Twitter Card 标签
twitter:
  card: summary_large_image
  title: "使用 Nano Banana Pro 进行老照片修复"
  description: "详细讲解使用 Nano Banana Pro 进行老照片修复的完整配置指南"
  image: https://x-wuxl.github.io/assets/images/nano-banana.png

# 结构化数据 Schema.org
schema:
  "@context": "https://schema.org"
  "@type": "TechArticle"
  headline: "使用 Nano Banana Pro 进行老照片修复 - 高真实感人像修复完整指南"
  description: "详细讲解如何使用 Nano Banana Pro 进行老照片修复，包含完整的结构化配置提示词。"
  author:
    "@type": "Person"
    name: "wuxl"
  datePublished: "2025-12-17T09:20:00+08:00"
  dateModified: "2025-12-17T09:20:00+08:00"
  image: "https://x-wuxl.github.io/assets/images/nano-banana.png"
  publisher:
    "@type": "Organization"
    name: "x-wuxl Blog"
    logo:
      "@type": "ImageObject"
      url: "https://x-wuxl.github.io/assets/og-image.png"
---

# 使用 Nano Banana Pro 进行老照片修复

## 概述

本文将详细介绍如何使用 **Nano Banana Pro** 对老照片进行高真实感修复。该方法的核心是在**不改脸、不美颜、不换脸**的前提下，实现照片的高清修复增强，包括：清晰度提升、噪点清除、柔光棚拍效果、85mm 浅景深、自然肤质保留和电影感调色。同时，我们会使用负向提示严格防止风格跑偏、磨皮、假脸、伪影和畸形等问题。

---

## 完整提示词配置
```
{
  "task": "portrait_restoration",
  "language": "zh-CN",
  "prompt": {
    "subject": {
      "type": "human_portrait",
      "identity_fidelity": "match_uploaded_face_100_percent",
      "no_facial_modification": true,
      "expression": "natural",
      "eye_detail": "sharp_clear",
      "skin_texture": "ultra_realistic",
      "hair_detail": "natural_individual_strands",
      "fabric_detail": "rich_high_frequency_detail"
    },
    "lighting": {
      "exposure": "bright_clear",
      "style": "soft_studio_light",
      "brightness_balance": "even",
      "specular_highlights": "natural_on_face_and_eyes",
      "shadow_transition": "smooth_gradual"
    },
    "image_quality": {
      "resolution": "8k",
      "clarity": "high",
      "noise": "clean_low",
      "artifacts": "none",
      "over_smoothing": "none"
    },
    "optics": {
      "camera_style": "full_frame_dslr",
      "lens": "85mm",
      "aperture": "f/1.8",
      "depth_of_field": "soft_shallow",
      "bokeh": "smooth_natural"
    },
    "background": {
      "style": "clean_elegant",
      "distraction_free": true,
      "tone": "neutral"
    },
    "color_grading": {
      "style": "cinematic",
      "saturation": "rich_but_natural",
      "white_balance": "accurate",
      "skin_tone": "natural_true_to_subject"
    },
    "style_constraints": {
      "no_cartoon": true,
      "no_beauty_filter": true,
      "no_plastic_skin": true,
      "no_face_reshaping": true,
      "no_ai_face_swap": true
    }
  },
  "negative_prompt": [
    "cartoon",
    "anime",
    "cgi",
    "painterly",
    "plastic skin",
    "over-smoothing",
    "over-sharpening halos",
    "heavy skin retouching",
    "face reshaping",
    "identity drift",
    "face swap",
    "beauty filter",
    "uncanny",
    "washed out",
    "color cast",
    "blown highlights",
    "crushed shadows",
    "banding",
    "jpeg artifacts",
    "extra fingers",
    "deformed eyes",
    "asymmetrical face",
    "warped features"
  ],
  "parameters": {
    "fidelity_priority": "identity",
    "detail_priority": "eyes_skin_hair_fabric",
    "realism_strength": 0.95,
    "sharpening": "micro_contrast_only",
    "skin_retention": "keep_pores_and_microtexture",
    "recommended_denoise": "low_to_medium"
  }
}
```

---

## 配置说明

以上是一个**人像修复/人像增强的结构化配置**，它将修复效果拆分为多个维度（人物、光线、画质、镜头、背景、调色、禁用项等），让 AI 模型严格按照这些规则处理你上传的照片。

---

## 字段详细解释

### 顶层字段

- **`task: "portrait_restoration"`**  
  任务类型：人像修复/还原（通常指提升清晰度、去噪、补细节、纠正曝光/色彩，但尽量不"整容"）。

- **`language: "zh-CN"`**  
  语言偏好（对结构化字段本身影响不大，更多是给系统/日志或某些可读提示用）。

---

### prompt（正向要求：你想要什么）

#### 1. subject（主体：人像该长什么样）

- **`type: "human_portrait"`**  
  主体是人像。

- **`identity_fidelity: "match_uploaded_face_100_percent"`**  
  身份一致性优先——尽量 100% 匹配你上传的人脸（不跑脸、不换人）。

- **`no_facial_modification: true`**  
  不做脸型/五官结构修改（避免"变好看但不像本人"）。

- **`expression: "natural"`**  
  表情自然。

- **`eye_detail: "sharp_clear"`**  
  眼睛细节清晰锐利。

- **`skin_texture: "ultra_realistic"`**  
  皮肤质感真实（毛孔/细纹保留，而不是磨皮）。

- **`hair_detail` / `fabric_detail`**  
  头发要有根根分明的自然细节；衣物纹理要细、信息量高。

#### 2. lighting（光照）

- **`bright_clear` / `soft_studio_light` / `even`**  
  明亮干净、柔和棚拍光、亮度均匀。

- **`specular_highlights: natural_on_face_and_eyes`**  
  面部和眼睛的高光要自然（不油、不假）。

- **`shadow_transition: smooth_gradual`**  
  阴影过渡平滑（不生硬、不脏）。

#### 3. image_quality（画质目标）

- **`resolution: "8k"`**  
  目标是超高分辨率（更多是"追求很高细节"的意图，未必真输出 8K）。

- **`clarity: high` / `noise: clean_low` / `artifacts: none`**  
  高通透、低噪点、无压缩块/伪影。

- **`over_smoothing: none`**  
  禁止过度平滑（防"塑料皮肤"）。

#### 4. optics（镜头/景深风格）

- **`full_frame_dslr` + `85mm` + `f/1.8`**  
  模拟全画幅 + 85mm 人像头 + 大光圈。

- **`soft_shallow DOF` + `smooth_natural bokeh`**  
  浅景深、背景虚化柔和自然。

#### 5. background（背景）

- **`clean_elegant` / `distraction_free` / `neutral`**  
  干净高级、不抢主体、中性背景色调。

#### 6. color_grading（调色）

- **`cinematic`**  
  偏电影感的整体色调。

- **`rich_but_natural saturation`**  
  饱和度丰富但不夸张。

- **`white_balance accurate` / `skin_tone natural`**  
  白平衡准确、肤色真实。

#### 7. style_constraints（风格约束：明确禁止）

- **`no_cartoon` / `no_beauty_filter` / `no_plastic_skin` / `no_face_reshaping` / `no_ai_face_swap`**  
  明确：不要二次元、不要美颜滤镜、不要塑料皮、不要改脸型、不要 AI 换脸。

---

### negative_prompt（负向提示：你不想要什么）

这是一长串"避雷清单"，用来强行压制常见问题，比如：

- **风格跑偏**：`cartoon`、`anime`、`cgi`、`painterly`
- **皮肤处理过度**：`plastic skin`、`over-smoothing`、`heavy retouching`、`beauty filter`
- **身份漂移**：`identity drift`、`face swap`
- **画面问题**：`washed out`、`color cast`、`blown highlights`、`crushed shadows`、`banding`、`jpeg artifacts`
- **畸形/瑕疵**：`extra fingers`、`deformed eyes`、`asymmetrical face`、`warped features` 等

---

### parameters（偏好/强度参数：怎么权衡）

- **`fidelity_priority: "identity"`**  
  优先保证"像本人"。

- **`detail_priority: "eyes_skin_hair_fabric"`**  
  细节优先级：眼睛/皮肤/头发/布料。

- **`realism_strength: 0.95`**  
  写实强度很高（越接近 1 越强调真实）。

- **`sharpening: "micro_contrast_only"`**  
  只做微对比锐化，避免锐化光晕。

- **`skin_retention: keep_pores_and_microtexture`**  
  保留毛孔与微纹理。

- **`recommended_denoise: low_to_medium`**  
  建议去噪强度低到中等（避免糊）。

---

## 总结

通过以上结构化配置，Nano Banana Pro 能够在严格保持人物原貌的前提下，对老照片进行高真实感修复。这种方法的优势在于：

1. **身份保真**：100% 保持原人物面部特征，不跑脸、不换人
2. **真实质感**：保留皮肤毛孔、细纹等自然细节，避免塑料感
3. **专业效果**：模拟专业棚拍的光影效果和电影感调色
4. **细节丰富**：8K 高清输出，眼睛、头发、衣物纹理细节清晰
5. **严格约束**：通过负向提示避免常见 AI 修复问题

该配置适用于各种老照片修复场景，特别是需要高度保真的珍贵照片修复。
