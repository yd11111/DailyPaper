---
type: concept
aliases: [强制对齐, FA, Force Alignment]
---

# Forced Alignment

## 定义
给定音频和对应文本，自动确定每个音素/词在音频中的起止时间。是传统 TTS 流水线中获取时长标注的关键步骤。

## 核心要点
1. 常用工具：Montreal Forced Aligner (MFA)
2. CosyVoice 的重要优势之一是不需要 forced alignment
3. 依赖外部 phonemizer 和对齐器是传统 TTS 的瓶颈

## 代表工作
- [[CosyVoice]]: 不依赖 forced alignment（对比优势）
- [[Voicebox]]: 需要 phoneme duration 和 forced alignment

## 相关概念
- [[Duration Predictor]]
- [[Phoneme]]
