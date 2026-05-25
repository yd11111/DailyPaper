---
type: concept
aliases: [FullSubNet, Full-band Sub-band Network]
---

# FullSubNet

## 定义
实时单通道语音增强模型，融合全频带（full-band）和子频带（sub-band）信息，在频域进行语音去噪。

## 核心要点
1. 全频带模型捕获全局频谱模式，子频带模型关注局部频率细节
2. 实时推理，适合作为 TTS 数据预处理工具
3. ICASSP 2021 (Hao et al.)

## 代表工作
- [[YourTTS]]: 用 FullSubNet 对葡萄牙语训练数据去噪

## 相关概念
- [[HiFi-GAN]]
