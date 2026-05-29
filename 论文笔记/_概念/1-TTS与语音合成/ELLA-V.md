---
type: concept
aliases: [ELLA-V]
domain: TTS
tags: [zero-shot-tts, codec-language-model, robustness, alignment]
created: 2026-05-29
last_updated: 2026-05-29
---

# ELLA-V

## 定义

一种增强鲁棒性的零样本 codec 语言模型 TTS（Song et al., 2024, arXiv:2401.07333），通过**交错（interleave）声学 token 与 phoneme token** 的序列，在 phoneme 级别实现细粒度文本可控，缓解 [[VALL-E]] 的漏读/重复问题。

## 核心要点

1. 用 alignment-guided sequence reordering，把 phoneme 标记插入声学 token 序列。
2. 代价：交错使自回归序列显著拉长（10s 语音约多 2×105 个 phoneme token），推理变慢。
3. 在 [[VALL-E R]] 的对比中：continuation WER 2.10、cross-sentence WER 7.15、推理 15.76s（慢于 VALL-E）。

## 代表工作

- ELLA-V (Song et al., 2024)：本词条本体
- 常作为 [[VALL-E R]] / [[RALL-E]] 的鲁棒性对比基线

## 相关概念

- [[VALL-E]] / [[VALL-E R]] / [[RALL-E]] / [[VALL-T]]
- [[Monotonic Alignment]]
