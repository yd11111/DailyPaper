---
type: concept
aliases: [ScaledAdam, Scaled Adam Optimizer]
---

# ScaledAdam

## 定义
一种 Adam 优化器变体，对参数按尺度进行自适应缩放，配合梯度裁剪（clipping_scale）使用，常用于语音模型训练。来自 k2/Icefall 项目。

## 核心要点
1. 相比标准 Adam，根据参数的 RMS 值动态缩放学习率
2. clipping_scale 参数控制梯度裁剪阈值
3. 在 Zipformer / GPT-SoVITS 等语音模型中被采用

## 代表工作
- [[GPT-SoVITS]]: GPT 阶段使用 ScaledAdam (lr=0.01, betas=(0.9, 0.95), clipping_scale=2.0)

## 相关概念
- [[Cosine Scheduler]]
