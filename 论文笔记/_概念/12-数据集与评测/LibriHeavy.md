---
type: concept
aliases: [Libri-Heavy]
---

# LibriHeavy

## 定义

**LibriVox 有声书**衍生的大规模英文 ASR / TTS 数据集，**~50K 小时**（[[Raon-OpenTTS]] 统计中是 42K h、10.8M 段），是 [[LibriSpeech]] / [[LibriTTS]] 同源但规模大得多的扩展。Public Domain 授权。

## 核心要点

1. **来源**: 与 [[LibriSpeech]] 同样的 LibriVox 有声书，但保留了更多 speaker 和更长片段
2. **数据特性**: audiobook 朗读，[[DNSMOS]] 3.22（高质量），[[Whisper]] WER 0.11（极低）
3. **过滤友好度**: [[Raon-OpenTTS-Core]] 中保留率 94.4%
4. **License**: Public Domain（最宽松）
5. **典型用途**: ASR 训练（k2/icefall）、TTS pretraining

## 代表工作

- LibriHeavy 数据集论文（Kang et al. 2023, k2 团队）
- [[Raon-OpenTTS]]: 作为 [[Raon-OpenTTS-Pool]] 的核心来源

## 相关概念

- [[LibriSpeech]] / [[LibriTTS]]: 同源前辈
- [[HiFiTTS2]]: 同为高质量 audiobook TTS 数据
