---
type: concept
aliases: [Montreal Forced Aligner, 蒙特利尔强制对齐器]
---

# MFA (Montreal Forced Aligner)

## 定义

基于 Kaldi 的开源强制对齐工具，将文本转录与音频在时间上对齐，输出每个音素/词的起止时间。广泛用于 TTS 训练数据准备和语音学研究。

## 核心要点

1. 支持多语种对齐（需提供发音词典）
2. 输出 TextGrid 格式的时间对齐信息
3. 在 TTS 中用于获取 phoneme duration（训练 Duration Predictor）
4. CosyVoice 3 数据管线中用于词间停顿检测（标点调整）

## 代表工作

- McAuliffe et al. 2017: "Montreal Forced Aligner" (Interspeech)
- [[CosyVoice3]]: 数据管线中用 MFA 获取词间停顿，调整标点

## 相关概念

- [[Duration Predictor]]
- [[ASR]]
