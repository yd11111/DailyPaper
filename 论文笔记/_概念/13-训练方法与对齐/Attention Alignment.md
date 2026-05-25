---
type: concept
aliases: [Attention Alignment, 注意力对齐, Encoder-Decoder Attention]
---

# Attention Alignment

## 定义
在 encoder-decoder 架构中，attention 机制学到的源序列与目标序列之间的软对齐关系。在 TTS 中，指 phoneme 序列与 mel-spectrogram 帧之间的对应关系。

## 核心要点
1. 理想的 TTS attention 应该是近似对角线的（单调对齐），表示语音从左到右依次生成
2. AR TTS 模型（如 Tacotron 2）中 attention 不稳定会导致跳词（skipping）和重复（repeating）
3. FastSpeech 利用教师模型的 attention alignment 提取 ground-truth 音素时长
4. 通过 Focus Rate 指标选择最"对角"的 attention head
5. 后续工作如 MFA (Montreal Forced Aligner) 提供了不依赖 AR 模型的替代对齐方案

## 代表工作
- [[FastSpeech]]: 从教师模型 attention 中提取时长
- [[Tacotron 2]]: AR TTS 中 attention 不稳定的典型案例

## 相关概念
- [[Focus Rate]]
- [[Duration Predictor]]
- [[Forced Alignment]]
- [[Self-Attention]]
