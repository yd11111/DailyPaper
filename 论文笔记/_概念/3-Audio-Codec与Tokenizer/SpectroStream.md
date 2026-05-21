---
type: concept
aliases: [Spectro Stream]
---

# SpectroStream

## 定义

Google 2024 年提出的低码率连续 audio codec（部分论文也将其纳入离散 codec 行列），核心特点是直接在 spectrogram 域做编码 + 流式生成，码率压到 < 1 kbps 仍保持中等可懂度，作为 [[VALL-E]] / [[AudioLM]] 类工作的低码率对照基线。

## 核心要点

1. **spectrogram-native**：不在 waveform 域做下采样，而是直接对 STFT 表示编码 / 解码
2. **流式**：支持低延迟 streaming，适合实时 TTS / 语音通话压缩
3. **超低码率**：≤ 1 kbps 下仍能保住可懂度
4. 与 [[EnCodec]] / [[DAC]] 的对比常见于 codec 评测论文

## 代表工作

- [[Target-KL-VAE]]：作为 audio VAE 比对的低码率代表
- 论文：SpectroStream tech report (Google)

## 评测/常见数字

- 1 kbps STOI ≈ 0.85，可懂但音质明显折损
- 推理延迟 < 20 ms（streaming）

## 相关概念

- [[EnCodec]]
- [[DAC]]
- [[Audio Codec]]
