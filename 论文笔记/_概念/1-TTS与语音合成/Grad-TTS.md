---
type: concept
aliases: [GradTTS, Grad TTS]
---

# Grad-TTS

## 定义

基于 score-based diffusion 的非自回归 TTS 模型（Popov et al., 2021）。将 TTS 建模为从噪声到 Mel-Spectrogram 的逐步去噪过程，使用 Monotonic Alignment Search (MAS) 做 duration 对齐。

## 数学形式

前向扩散过程：

$$
dx_t = -\frac{1}{2} \beta(t) x_t \, dt + \sqrt{\beta(t)} \, dW_t
$$

逆过程通过学到的 score function $s_\theta(x_t, \mu, t) \approx \nabla_{x_t} \log p_t(x_t | \mu)$ 生成 Mel 帧，$\mu$ 为对齐后的 phoneme hidden。

推理需指定 NFE（Number of Function Evaluations），步数越多质量越好但越慢（1000 步 RTF=4.12，10 步 RTF=0.082）。

## 核心要点

1. 首个将 diffusion 引入 TTS 的代表工作之一
2. 使用 [[Monotonic Alignment Search]] 做无监督时长对齐
3. 生成质量随推理步数增加而提升，但推理速度是主要瓶颈
4. 需要外接 [[Vocoder]]（如 [[HiFi-GAN]]）将 Mel 转波形

## 代表工作

- [[NaturalSpeech]]: 作为基线对比，-0.23 CMOS vs 录音
- [[Grad-TTS]]: Popov et al., "Grad-TTS: A Diffusion Probabilistic Model for Text-to-Speech", ICML 2021

## 评测/常见数字

- LJSpeech MOS ~4.37（+ HiFi-GAN），与录音有统计显著差异
- RTF: 0.082 (10 steps) ~ 4.12 (1000 steps)

## 相关概念

- [[DDPM]]
- [[Normalizing Flow]]
- [[Monotonic Alignment Search]]
- [[HiFi-GAN]]
- [[FastSpeech 2]]
- [[Glow-TTS]]
