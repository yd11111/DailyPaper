---
type: concept
aliases: [VITS, Conditional Variational Autoencoder with Adversarial Learning for End-to-End TTS]
---

# VITS

## 定义
端到端 TTS 模型，结合条件 VAE、Normalizing Flow 和对抗训练，直接从文本生成波形，无需两阶段 Mel 中间表示。Kim et al. 2021。

## 数学形式

$$
\mathcal{L}_{\text{VITS}} = \mathcal{L}_{\text{recon}} + D_{\text{KL}}(q_\phi(z|x_{\text{spec}}) \| p_\theta(z|x_{\text{text}})) + \mathcal{L}_{\text{adv}} + \mathcal{L}_{\text{fm}}
$$

## 核心要点
1. Posterior Encoder（WaveNet）从频谱编码后验分布 → 采样 z → HiFi-GAN 解码为波形
2. Prior Encoder（Transformer）从文本编码先验分布，通过 Normalizing Flow 与后验对齐
3. 随机时长预测器（Stochastic Duration Predictor）使韵律更自然
4. 端到端训练避免了 Mel-Vocoder 两阶段的误差累积

## 代表工作
- [[GPT-SoVITS]]: 使用 VITS 作为第二阶段声学解码器
- [[CosyVoice]]: 继承 VITS 的 Flow-based 思路

## 评测/常见数字
- LJSpeech MOS 4.43（GT 4.46），VCTK MOS 4.38（GT 4.38）
- 合成速度 x67 real-time（NVIDIA V100），是 Glow-TTS+HiFi-GAN 的 2.4 倍

## 相关概念
- [[Normalizing Flow]]
- [[HiFi-GAN]]
- [[KL Divergence]]
- [[VAE]]
