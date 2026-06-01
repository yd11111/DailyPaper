---
type: concept
aliases: [MSD, multi-resolution discriminator, 多尺度判别器]
---

# Multi-scale Discriminator

## 定义

对输入波形在不同分辨率（原始 + 2x/4x 下采样）上分别用结构相同的卷积判别器做真假判别。捕获不同时间尺度的音频特征——高分辨率关注细节/高频，低分辨率关注整体/低频结构。

## 核心要点

1. 首先由 MelGAN 提出，后被 [[HiFi-GAN]] 和 [[SoundStream]] 等广泛采用
2. 与 [[Multi-Period Discriminator]] 互补使用——MSD 捕获多尺度时域特征，MPD 捕获周期性结构
3. SoundStream 使用 3 个尺度（1x, 2x, 4x downsampled）
4. 判别器的中间层特征被复用为 Feature Loss（perceptual loss 的免费来源）

## 代表工作

- MelGAN: 首次提出多尺度波形判别器
- [[HiFi-GAN]]: 结合 MSD + [[Multi-Period Discriminator]]
- [[SoundStream]]: 用于 neural codec 训练

## 相关概念

- [[Multi-Period Discriminator]]
- [[HiFi-GAN]]
- [[Adversarial Training]]
