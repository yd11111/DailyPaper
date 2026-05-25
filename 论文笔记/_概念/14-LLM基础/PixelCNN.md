---
type: concept
aliases: [Pixel CNN, PixelCNN++]
---

# PixelCNN

## 定义
van den Oord et al. (2016a) 提出的自回归图像生成模型，使用掩码卷积逐像素生成图像，每个像素条件于之前所有已生成的像素。是 [[WaveNet]] 的直接灵感来源。

## 核心要点
1. 将图像联合分布分解为逐像素条件概率之积
2. 使用掩码卷积保证自回归性
3. 训练时所有像素可并行计算（优于 PixelRNN）
4. 后续发展出 [[Gated PixelCNN]]、PixelCNN++、PixelSNAIL

## 代表工作
- [[WaveNet]]: 将 PixelCNN 范式从 2D 图像迁移到 1D 音频波形

## 相关概念
- [[PixelRNN]]
- [[Gated PixelCNN]]
- [[Autoregressive Model]]
