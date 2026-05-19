---
type: concept
aliases: [Acoustic Token, 声学 Token]
---

# Acoustic Token

## 定义
承载完整声学信息的语音离散 token，通常来自 [[Codec]]（如 [[EnCodec]]、[[SoundStream]]、[[DAC]]）的 RVQ 输出。

## 核心要点
1. 可直接重建高质量 waveform
2. 信息密度高、序列长，但 LLM 端建模困难
3. 通常用多层 RVQ，第一层语义信息多

## 代表工作
- [[VALL-E]]、[[AudioLM]] 等用 acoustic token；[[OmniFlatten]] 故意不用以简化建模

## 相关概念
- [[OmniFlatten]]
