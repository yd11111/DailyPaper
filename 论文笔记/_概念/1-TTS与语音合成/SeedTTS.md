---
type: concept
aliases: [Seed-TTS, Seed TTS]
---

# SeedTTS

## 定义
字节跳动推出的大规模零样本 TTS 系统，在 Seed-TTS-eval 评测集上设立了 SOTA 基线。闭源商用系统。

## 核心要点
1. 包含自回归和非自回归（扩散）两种架构变体
2. Seed-TTS-eval 评测集（test-zh / test-en / test-hard）已成为零样本 TTS 的标准 benchmark
3. 闭源，社区无法复现，但评测集公开

## 评测/常见数字
- test-zh CER: 1.12%（最低）
- test-en WER: 2.25%
- test-hard WER: 7.59%
- SS: 0.796 (test-zh)

## 代表工作
- [[CosyVoice2]]: 在 SEED 评测上与 Seed-TTS 对比，test-zh 接近其性能

## 相关概念
- [[VALL-E]]
- [[CosyVoice]]
- [[F5-TTS]]
