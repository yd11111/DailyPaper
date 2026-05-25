---
type: concept
aliases: [F5 TTS]
---

# F5-TTS

## 定义

上海交大 2024 年的零样本 TTS 模型，特点是**纯 [[Flow Matching]]、无 phoneme、无 [[Duration Predictor]]、无 forced alignment**——直接把字符序列 + 参考音频塞进 DiT，靠 mask-and-infill 范式还原目标波形 Mel。

## 数学形式

把 mask 部分的 Mel 当作目标，CFM 训练：
$$
\mathcal{L}_{\mathrm{CFM}} = \mathbb{E}_{t,(x_0, x_1)} \big\| v_\theta(x_t, t, c) - (x_1 - x_0) \big\|^2
$$
其中 $c$ 是字符 + reference Mel 拼接的 condition。

## 核心要点

1. 无显式 duration：依赖 reference 音频的总长 + 文字提供时长隐式信息
2. DiT 架构（Diffusion Transformer 风格）
3. 训练稳定、合成质量高，常作为现代 TTS 强基线

## 代表工作

- F5-TTS 原论文 (arXiv 2410.06885, Chen et al. 2024): 本概念的原始论文，详见 [[F5-TTS]]
- [[VibeVoice]]: 把 F5-TTS 当作"短句零样本 TTS"对比基线之一

## 评测/常见数字

- LibriSpeech-PC test-clean WER: 2.42% (32 NFE) / 2.53% (16 NFE)
- SIM-o: 0.66
- RTF: 0.15 (16 NFE Euler)
- Seed-TTS test-en WER: 1.83%, CMOS: +0.31
- Seed-TTS test-zh WER: 1.56%, CMOS: +0.21

## 相关概念

- [[Flow Matching]]
- [[DiT]]
- [[ConvNeXt]]
- [[E2 TTS]]
- [[CosyVoice]]
- [[Duration Predictor]]
- [[Vocos]]
- [[Emilia]]
