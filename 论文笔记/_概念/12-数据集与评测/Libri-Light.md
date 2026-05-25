---
type: concept
aliases: [Libri-Light, LibriLight]
---

# Libri-Light

## 定义
基于 LibriVox 有声书的大规模无标注英文语音数据集，常用于自监督语音预训练。

## 核心要点
1. 总量约 60k 小时（unlab-60k 子集）
2. 无转录标注，仅音频
3. 包含多说话人、多录音条件（含噪声）
4. 比 LibriSpeech 大 60 倍，是语音 SSL 模型的标准训练集

## 代表工作
- [[AudioLM]]: 全部组件在 Libri-Light unlab-60k 上训练
- [[HuBERT]]: 自监督预训练使用此数据集

## 相关概念
- [[LibriSpeech]]
- [[LibriTTS]]
