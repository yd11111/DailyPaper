---
type: concept
aliases: [LibriTTS-R, LibriTTS Restored]
domain: TTS
tags: [dataset, tts, english, multi-speaker, libritts]
created: 2026-05-29
last_updated: 2026-05-29
---

# LibriTTS-R

## 定义

Koizumi et al. (2023) 在 [[LibriTTS]] 基础上用音频修复（restoration）技术清洗的版本：去噪、去混响、采样率统一，整体音质更接近 studio quality，但保留 LibriTTS 的多说话人 / 多书目 / 多语速分布。是当前 zero-shot TTS / controllable TTS 训练与评测的常用数据集。

## 规模

- 约 585 小时英文有声书音频（与 LibriTTS 同源切分）
- 多说话人（2,000+ speakers）
- 子集：train-clean-100 / train-clean-360 / train-other-500 / dev / test
- 24 kHz 采样率（restored）

## 用途

- **训练**：多说话人 zero-shot TTS（如 [[VALL-E]] / [[NaturalSpeech 2]] / [[NaturalSpeech 3]] 等开源对照）
- **评测**：从 test set 抽取文本作为 prompt-based / controllable TTS 实验输入（如 [[StyleSelfReferencing]] 2026 用 test 400 句做插值与过渡实验）

## 相关概念

- [[LibriTTS]]
- [[LibriSpeech]]
- [[Emilia]]
