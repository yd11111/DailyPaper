---
type: concept
aliases: [GigaSpeech Corpus]
---

# GigaSpeech

## 定义

大规模多域英文 ASR 数据集，包含约 10,000 小时标注语音数据，来源涵盖有声书 (AudioBook)、播客 (Podcast)、YouTube 视频等多种域。

## 核心要点

1. 多域数据（audiobook / podcast / YouTube），比纯有声书数据（如 [[LibriSpeech]]）的声学条件更多样
2. 提供 XS (10h) / S (250h) / M (1000h) / L (2500h) / XL (10000h) 多种子集
3. 标注来自 forced alignment + 人工审核
4. 广泛用于 ASR 训练和 TTS 预训练（提供大规模多说话人数据）

## 代表工作

- [[MegaTTS]]: 与 [[WenetSpeech]] 合并组成 20K h 训练集
- [[Whisper]]: 大规模 ASR 预训练的对照数据集之一

## 评测/常见数字

- 总时长：~10,000 h
- 说话人数量：数万级
- 采样率：16 kHz

## 相关概念

- [[LibriSpeech]]
- [[WenetSpeech]]
- [[Common Voice]]
- [[LibriTTS]]
