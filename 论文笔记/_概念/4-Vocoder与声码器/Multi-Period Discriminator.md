---
type: concept
aliases: [Multi-Period Discriminator, MPD, 多周期判别器]
---

# Multi-Period Discriminator

## 定义
HiFi-GAN 引入的判别器结构，将 1D 波形按不同周期（2, 3, 5, 7, 11 等）reshape 成 2D 然后用 2D 卷积判别，捕捉不同频率尺度的真假差异。

## 核心要点
1. 多个 Period Discriminator 分别按不同周期切分波形
2. 常与 Multi-Scale Discriminator (MSD) 联合使用
3. 各判别器独立产生对抗损失，合并训练
4. GPT-SoVITS V2Pro 扩展到周期 [2, 3, 5, 7, 11, 17, 23]

## 代表工作
- [[HiFi-GAN]]: 原始提出
- [[GPT-SoVITS]]: 声码器训练使用 MPD + MSD
- [[VITS]]: 端到端 TTS 中使用 MPD

## 相关概念
- [[HiFi-GAN]]
- [[Adversarial Loss]]
- [[Feature Matching Loss]]
