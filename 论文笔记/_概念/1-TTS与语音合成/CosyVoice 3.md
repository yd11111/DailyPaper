---
type: concept
aliases: [CosyVoice3]
---

# CosyVoice 3

## 定义

阿里通义 [[CosyVoice 2]] 的后继版本，**0.5B / 1.5B 双尺寸**，训练数据约 **1M 小时**（私有）。继承 supervised semantic token + flow matching + vocoder 三段式架构，把 [[Zero-shot TTS]] 推到接近 [[Seed-TTS]] 的水平。

## 核心要点

1. **架构**: 沿用 CosyVoice 系列的 semantic token LM + [[Flow Matching]] + 声码器
2. **数据规模**: ~1M 小时，比 [[CosyVoice 2]] 的 170K 大一个数量级
3. **强项**: [[Raon-OpenTTS-Eval]] 上 Wild 场景 WER 8.31 是闭源数据中最鲁棒的，DNSMOS 3.98 最佳
4. **弱项**: [[Raon-OpenTTS]] Table 1 显示 1.5B 版本 Seed-TTS WER 2.21 略输 [[Qwen3-TTS]] 1.46

## 代表工作

- CosyVoice 3 技术报告（阿里）

## 相关概念

- [[CosyVoice]] / [[CosyVoice 2]]: 前置版本
- [[Qwen3-TTS]]: 同阿里出品但走纯 LLM TTS 路线
- [[Seed-TTS]]: 字节同档对手
