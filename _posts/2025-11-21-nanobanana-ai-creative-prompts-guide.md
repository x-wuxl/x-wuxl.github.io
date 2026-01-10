---
layout: post
title: "Nano-Banana AI 创意玩法与提示词完全指南"
date: 2025-11-21 09:41:00 +0800
categories: [AI, 图像生成, 视频生成]
tags: [Nano-Banana, Veo, AI 绘画, Prompt, 提示词, 图像生成, 视频生成, AI 工具]
description: "完整收录 Nano-Banana 和 Veo AI 模型的创意玩法与提示词大合集。包含手办模型化、人像风格转换、视频生成、产品设计等 30+ 种实用技巧，帮助你快速掌握 AI 图像和视频创作。"
keywords: "Nano-Banana, Veo, AI 图像生成, AI 视频生成, Prompt 提示词, 手办生成, 风格转换, 人像摄影, 工业设计, AI 创作"
author: "wuxl"
image: 
  path: /assets/images/nano-banana-pro.png
  alt: "Nano-Banana AI 创意玩法"
seo:
  type: Article
  date_modified: 2025-11-21 09:41:00 +0800
canonical_url: https://xwuxl.com/2025/11/21/nanobanana-ai-creative-prompts-guide/
---

## 概述

**Nano-Banana** 是一款强大的 AI 图像生成模型，结合 **Veo** 视频生成技术，可以实现从图像风格化、人像摄影到产品设计的多样化创意应用。

本文整理自 GitHub 开源项目 [ZHO-nano-banana-Creation](https://github.com/ZHO-ZHO-ZHO/ZHO-nano-banana-Creation)，汇总了 **30+ 种实用 Prompt 提示词**，涵盖：

- 🎨 图像风格转换（手办化、玩具化）
- 📸 人像摄影与风格化处理
- 🎬 视频生成与运镜控制
- 🎯 产品设计与商业应用
- ✨ 特殊效果与多步编辑

无论你是 AI 创作新手还是专业设计师，都能从中找到适合的玩法！

[11个 Nano Banana Pro 免费使用渠道汇总](https://xwuxl.com/2025/11/24/nano-banana-pro-free-channels/)

---

## 1. 手办、模型与玩具化

将真实照片或插画转化为各种风格的实体模型或玩具包装效果。

### 🖼️ 图片变手办 (Figure Style)

将任意角色照片转换为精美手办模型展示效果。

**核心 Prompt：**

```text
turn this photo into a character figure. Behind it, place a box with the character's image printed on it, and a computer showing the Blender modeling process on its screen. In front of the box, add a round plastic base with the character figure standing on it. set the scene indoors if possible
```

**适用场景：** 动漫角色、游戏人物、IP 形象设计

---

### 🏠 建筑图转模型 (Architecture Model)

将建筑照片转换为逼真的建筑模型展示效果。

**核心 Prompt：**

```text
convert this photo into a architecture model. Behind the model, there should be a cardboard box with an image of the architecture from the photo on it. There should also be a computer, with the content on the computer screen showing the Blender modeling process of the figurine. In front of the cardboard box, place a cardstock and put the architecture model from the photo I provided on it. I hope the PVC material can be clearly presented. It would be even better if the background is indoors.
```

**适用场景：** 建筑设计展示、房地产营销、模型制作预览

---

### 🧸 图片转 Funko Pop 公仔

将人物照片转换为流行的 Funko Pop 收藏玩具风格。

**核心 Prompt：**

```text
Transform the person in the photo into the style of a Funko Pop figure packaging box, presented in an isometric perspective. Label the packaging with the title 'ZHOGUE'. Inside the box, showcase the figure based on the person in the photo, accompanied by their essential items (such as cosmetics, bags, or others). Next to the box, also display the actual figure itself outside of the packaging, rendered in a realistic and lifelike style.
```

> **💡 扩展玩法：** 替换关键词即可生成其他风格
> - **LEGO 乐高风格：** 将 `Funko Pop` 替换为 `LEGO`
> - **Barbie 芭比风格：** 将 `Funko Pop` 替换为 `Barbie`
> - **Gundam 高达风格：** 将 `Funko Pop` 替换为 `Gundam`

---

### 🧶 图片转针织玩偶

将角色转换为温馨可爱的手工针织玩偶效果。

**核心 Prompt：**

```text
A close-up, professionally composed photograph showing a handmade crocheted yarn doll being gently held in both hands. The doll has a rounded shape and an adorable chibi-style appearance, with vivid color contrasts and rich details. The hands holding the doll appear natural and tender, with soft diffused lighting highlighting the yarn texture and craftsmanship.
```

**适用场景：** 儿童产品设计、手工艺品展示、温馨风格创作

---

## 2. 人像摄影与风格转换

提升照片质感、进行风格跨越或特定人像生成的专业技巧。

### 📸 名人/指定人物超写实照片级生成

生成极具质感的专业人像摄影作品。

**核心 Prompt 要素：**

需包含详细的摄影术语，确保生成效果专业：

- **构图：** `Medium shot`, `intimate distance`, `slight excessive perspective`
- **光影：** `Direct flash`, `hard highlights`, `deep shadows`, `slightly overexposed face`
- **质感：** `Film grain`, `high ISO texture`, `90s street snap style`
- **细节：** 描述衣物材质（velvet, silk）、配饰（sunglasses reflection）及背景杂物

**示例组合：**
```text
A medium shot portrait of [人物名称], intimate distance with slight excessive perspective. Direct flash creates hard highlights and deep shadows on the face, slightly overexposed. Film grain texture with high ISO, 90s street snap style. She wears velvet jacket with silk scarf, sunglasses reflecting street lights. Background shows urban street details.
```

---

### 🎎 动漫转真人 (1:1 还原)

将动漫插画还原为真人 Cosplay 照片，完美保留原作姿态。

**核心 Prompt：**

```text
Generate a highly detailed photo of a girl cosplaying this illustration, at Comiket. Exactly replicate the same pose, body posture, hand gestures, facial expression, and camera framing as in the original illustration. Keep the same angle, perspective, and composition, without any deviation
```

**适用场景：** Cosplay 参考、动漫角色真人化、二次元文化创作

---

### 👶 赛博生娃 (两张人脸生成孩子)

趣味功能：根据两张人物照片生成"孩子"的样貌。

**核心 Prompt：**

```text
生成图中两人物所生孩子的样子，专业摄影
```

**适用场景：** 趣味创作、家庭纪念、创意设计

---

### 💅 随手拍变专业大片 (废片拯救)

将普通照片升级为高质量时尚大片。

**核心 Prompt：**

```text
Transform the person in the photo into highly stylized ultra-realistic portrait, with sharp facial features and flawless fair skin, standing confidently against a bold green gradient background. Dramatic, cinematic lighting highlights her facial structure, evoking the look of a luxury fashion magazine cover. Editorial photography style, high-detail, 4K resolution.
```

**适用场景：** 人像修图、时尚摄影、社交媒体内容

---

## 3. 视频生成与运镜控制

利用 **Veo 模型** 进行视频动态生成的专业玩法。

### 🎥 手办变视频 (把玩视角)

让静态手办动起来，展示把玩细节。

**Prompt (Veo3)：**

```text
A pair of hands picks up the figurine and examines it closely
```

**适用场景：** 手办开箱视频、产品展示、收藏品介绍

---

### 🔄 指定人物切换视角 (特征保持)

在保持人物特征的前提下，切换摄影机角度。

**核心 Prompt：**

```text
change the Camera angle to a high-angled selfie perspective looking down at the woman, while preserving her exact facial features, expression, and clothing. Maintain the same living room interior background with the sofa, natural lighting, and overall photographic composition and style.
```

**适用场景：** 影视分镜、多角度人像、视频创作

---

### 🛍️ 商品广告短片

快速生成产品展示广告视频。

**Image Prompt：**

```text
The model is holding a perfume
```

**Video Prompt：**

```text
The model is showcasing the perfume with beautiful music
```

**适用场景：** 电商推广、品牌广告、产品发布

---

### 🤡 精准替换视频人物

在视频中精准替换指定人物。

**Img Prompt：**
```text
把左边第二位人物换成希斯莱杰小丑/上传照片人物
```

**Video Prompt：**
```text
小丑左右看了看，走向镜头前
```

**适用场景：** 影视特效、创意视频、角色替换

---

## 4. 产品设计与商业应用

设计师实用的草图渲染与展示工具。

### ✏️ 工业设计：手绘转实景

将手绘草图转换为逼真的产品渲染图。

**核心 Prompt：**

```text
turn this photo into realistic version, with light brown leather, put into a Minimalism museum
```

**适用场景：** 工业设计、产品原型、概念可视化

---

### 🧴 产品包装贴合

将设计图贴合到实际产品上进行预览。

**核心 Prompt：**

```text
把图一贴在图二易拉罐上，并放在极简设计的布景中，专业摄影
```

**适用场景：** 包装设计、品牌视觉、营销物料

---

### 📺 曲面屏/裸眼3D内容生成

为大屏幕生成炫酷的裸眼 3D 展示内容。

**核心 Prompt：**

```text
为大屏幕上换上裸眼 3D 猫猫
```

或：
```text
把图一放在图二大屏幕上，撑满整个屏幕
```

**适用场景：** 数字广告、展览展示、商场大屏

---

## 5. 多步编辑与特殊效果

需要分步骤进行的复杂视觉处理技巧。

### 🖌️ 连续编辑 (组合+背景+视频)

通过多步骤打造完整的创意场景。

**Step 1：** 添加元素
```text
图一人物背上图二 logo 的斜挎包
```

**Step 2：** 更换背景
```text
换上 Y2K 风格的背景
```

**Step 3 (Veo3)：** 生成视频
```text
女孩转过身来摆出一个可爱帅气的pose，背景音乐也是y2k风格
```

---

### 🎨 生成绘画过程 (四宫格)

展示从草图到完成的绘画过程。

**核心 Prompt：**

```text
为人物生成绘画过程四宫格，第一步：线稿，第二步平铺颜色，第三步：增加阴影，第四步：细化成型。不要文字
```

**适用场景：** 绘画教程、创作过程展示、艺术教育

---

### 🔍 人群中分离指定人物 + 高清化

从群体照片中提取单个人物并高清化。

**核心 Prompt：**

```text
Separate the person inside the green box and turn it into a high-definition single-person photo
```

**适用场景：** 照片修复、人物提取、证件照制作

---

### ✨ 一句咒语：风格变写实

将任意风格图片转换为写实风格。

**核心 Prompt：**

```text
turn this illustration into realistic version
```

**适用场景：** 插画转照片、风格迁移、概念设计

---

## 总结

Nano-Banana 和 Veo 为 AI 创作者提供了强大的图像和视频生成能力。通过本文收录的 30+ 种 Prompt 玩法，你可以：

- ✅ 快速掌握 AI 图像生成技巧
- ✅ 创作专业级的人像和产品照片
- ✅ 制作创意视频内容
- ✅ 提升设计工作效率

**开始你的 AI 创作之旅吧！** 🚀

---

> **声明：** 本文内容整理自开源项目，仅供学习交流使用。