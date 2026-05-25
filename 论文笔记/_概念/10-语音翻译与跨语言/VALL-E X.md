---
type: concept
aliases: [VALLEX, VALL-EX]
---

# VALL-E X

## 定义

微软提出的跨语言零样本语音合成模型，将 VALL-E 的 codec language modeling 范式扩展到跨语言 TTS 和 speech-to-speech 翻译（S2ST）。可以用源语言的 3 秒参考音频合成目标语言的语音，保持说话人音色。

## 核心要点

1. 在 VALL-E 基础上增加多语言支持
2. 支持 zero-shot 跨语言 TTS：输入英文 prompt，输出中文语音（保持音色）
3. 可用于 speech-to-speech translation

## 代表工作

- [[VALL-E]]: 前作（单语言）
- VALL-E X (Zhang et al., 2023): 跨语言扩展

## 相关概念

- [[VALL-E]]
- [[Zero-shot TTS]]
- [[SeamlessM4T]]
