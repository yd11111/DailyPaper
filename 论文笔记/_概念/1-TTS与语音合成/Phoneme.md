---
type: concept
aliases: [音素, Phone]
---

# Phoneme

## 定义
语言中最小的语音区分单位。传统 TTS 系统以音素序列为输入，需要外部 phonemizer（如 G2P 工具）将文本转换为音素。

## 核心要点
1. 常用音素集：IPA（国际音标）、CMUDict（美式英语）
2. 传统 TTS（FastSpeech、VITS）依赖音素输入
3. CosyVoice 用 BPE 替代音素，避免了 phonemizer 依赖
4. 音素 vs BPE：音素更接近发音但需额外工具，BPE 端到端但语义更粗

## 代表工作
- [[CosyVoice]]: 证明 BPE 可替代 phoneme（WER 5.05% → 3.93%）
- [[VALL-E]]: 使用 phoneme 作为文本输入

## 相关概念
- [[BPE]]
- [[Forced Alignment]]
- [[Duration Predictor]]
