---
type: concept
aliases: [Mega-TTS2, MegaTTS2]
---

# Mega-TTS 2

## 定义

大规模零样本 TTS 系统，通过多尺度韵律建模和大规模数据训练实现高质量的零样本语音合成。

## 核心要点
1. 多尺度韵律建模（全局/局部/帧级）
2. 支持零样本多说话人合成
3. 在 RAVDESS 上的情感保持表现不错（MCD-Acc 0.39）

## 评测/常见数字
- LibriSpeech test-clean: WER 2.32, Sim-O 0.53, CMOS -0.20（相对 NaturalSpeech 3）
- RAVDESS: MCD 4.44, MCD-Acc 0.39

## 相关概念
- [[Zero-shot TTS]]
- [[Duration Predictor]]
- [[Prosody Transfer]]
