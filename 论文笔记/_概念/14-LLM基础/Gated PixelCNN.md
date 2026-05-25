---
type: concept
aliases: [Conditional Image Generation with PixelCNN Decoders]
---

# Gated PixelCNN

## 定义
van den Oord et al. (2016b) 提出的条件图像生成模型，在 PixelCNN 基础上引入门控激活单元和条件机制。其门控激活设计被 [[WaveNet]] 直接借鉴用于音频生成。

## 核心要点
1. 引入 [[Gated Activation Unit]]：$\tanh$ 和 $\sigma$ 双分支逐元素相乘
2. 支持条件生成（类别标签等）
3. 解决了原始 PixelCNN 的盲点问题
4. 为 WaveNet 提供了核心架构灵感

## 代表工作
- [[WaveNet]]: 将门控激活从图像生成迁移到音频生成

## 相关概念
- [[PixelCNN]]
- [[Gated Activation Unit]]
