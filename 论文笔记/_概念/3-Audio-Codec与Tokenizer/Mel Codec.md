---
type: concept
aliases: [Mel Codec, Mel-Spectrogram Codec]
---

# Mel Codec

## 定义
将 Mel-Spectrogram 压缩为多流离散 token 序列的编解码器，区别于直接对 waveform 编码的 Audio Codec（如 EnCodec/DAC）。通常基于 CNN-GAN 架构，支持流式（streamable）编解码。

## 核心要点
1. 输入/输出为 Mel-Spectrogram 而非原始 waveform，压缩比更温和
2. 通常采用多流（multi-stream）RVQ 结构，每流独立码本
3. 基于 CNN 的因果卷积设计支持流式推理
4. 与下游 vocoder（如 BigVGAN）配合，先解码到 Mel 再到 waveform

## 代表工作
- [[FireRedTTS]]: 4 流 Mel Codec，20ms 帧率，每流 16384 codewords，用于 Streamable Decoder

## 相关概念
- [[EnCodec]]
- [[DAC]]
- [[RVQ]]
- [[Mel-Spectrogram]]
- [[BigVGAN]]
