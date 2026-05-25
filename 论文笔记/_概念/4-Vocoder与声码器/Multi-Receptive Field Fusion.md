---
type: concept
aliases: [MRF, Multi-Receptive Field Fusion]
---

# Multi-Receptive Field Fusion

## 定义
[[HiFi-GAN]] V1 中提出的模块，将多个具有不同感受野大小的残差块的输出相加，使生成器能同时捕获不同尺度的音频特征模式。

## 核心要点
1. 每个 MRF 模块包含多个残差块，每个残差块有不同的 kernel size 和 dilation pattern
2. 各残差块输出直接求和（而非拼接），保持通道数不变
3. 被 [[VITS]] 的 Decoder 直接复用

## 代表工作
- [[HiFi-GAN]]: 原始提出
- [[VITS]]: 在端到端 TTS 中复用

## 相关概念
- [[HiFi-GAN]]
- [[Vocoder]]
- [[VITS]]
