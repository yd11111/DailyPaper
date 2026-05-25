---
type: concept
aliases: [Reconstruction Loss, 重建损失, Mel Loss]
---

# Reconstruction Loss

## 定义
衡量生成输出与目标之间差距的损失函数。在 TTS 中通常计算生成语音的 [[Mel-Spectrogram]] 与真实 mel 之间的 L1 或 L2 距离。

## 数学形式

$$
L_\text{recon} = \|x_\text{mel} - \hat{x}_\text{mel}\|_1
$$

L1 损失等价于假设 Laplace 分布下的最大似然估计。

## 核心要点
1. 是 [[VAE]] 中 [[ELBO]] 的重建项
2. 在 TTS 中通常在 mel 域计算，而非波形域（波形域噪声过大）
3. 常与对抗损失、[[Feature Matching Loss]] 联合使用

## 代表工作
- [[VITS]]: L1 mel reconstruction loss
- [[FastSpeech 2]]: L2 mel loss + pitch/energy loss

## 相关概念
- [[Mel-Spectrogram]]
- [[ELBO]]
- [[Feature Matching Loss]]
