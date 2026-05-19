---
type: concept
aliases: [LibriTTS, LibriTTS-R]
---

# LibriTTS

## 定义
Zen et al. 2019 发布的多说话人英文 TTS 数据集，约 585 小时，从 LibriSpeech 派生但保留标点。

## 核心要点
1. 2456 个说话人，适合多说话人/零样本 TTS 训练
2. 采样率 24 kHz
3. TTS 评测标配（用 ASR 反转算 WER）

## 代表工作
- [[OmniFlatten]] / [[CosyVoice]] / [[VALL-E]] 等都在此训练或评测
- [[VibeVoice]]: acoustic tokenizer 在 test-clean / test-other 上做 PESQ/STOI/UTMOS 重建评测

## 相关概念
- [[OmniFlatten]]
- [[VibeVoice]]
- [[PESQ]] · [[STOI]] · [[UTMOS]]
