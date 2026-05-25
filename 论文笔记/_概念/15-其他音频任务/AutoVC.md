---
type: concept
aliases: [AutoVC, Auto Voice Conversion]
---

# AutoVC

## 定义
基于自编码器的零样本语音转换方法，仅用自编码器重建损失（无 GAN/对抗训练），通过信息瓶颈（bottleneck）实现说话人与内容的解耦。

## 核心要点
1. 关键设计：瓶颈维度精心调节，使其刚好编码内容信息而滤除说话人信息
2. 推理时替换说话人 embedding 即可实现零样本 VC
3. ICML 2019 (Qian et al.)

## 评测/常见数字
- VCTK ZS-VC: MOS ~3.54, Sim-MOS ~1.91（被 YourTTS 大幅超越）

## 代表工作
- [[YourTTS]]: 作为 ZS-VC baseline 被对比

## 相关概念
- [[NoiseVC]]
- [[Speaker Encoder]]
