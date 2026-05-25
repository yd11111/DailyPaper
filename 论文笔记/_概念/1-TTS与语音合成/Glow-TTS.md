---
type: concept
aliases: [Glow-TTS]
---

# Glow-TTS

## 定义
基于 [[Normalizing Flow]] 的并行 TTS 模型，提出 [[Monotonic Alignment Search]] (MAS) 算法实现无需外部对齐器的训练。Kim et al. 2020。

## 核心要点
1. 首次在 TTS 中用 MAS 动态规划搜索最优单调对齐，去掉了对外部 forced aligner 的依赖
2. 使用确定性时长预测器，推理时只能产生固定韵律
3. 是 [[VITS]] 的直接前作，VITS 在此基础上加入 VAE + GAN + 随机时长

## 代表工作
- [[VITS]]: 直接继承 MAS 和 Prior Encoder 设计

## 评测/常见数字
- LJSpeech 上需配合 [[HiFi-GAN]] 使用，MOS ~4.14-4.32

## 相关概念
- [[Monotonic Alignment Search]]
- [[Normalizing Flow]]
- [[VITS]]
- [[HiFi-GAN]]
