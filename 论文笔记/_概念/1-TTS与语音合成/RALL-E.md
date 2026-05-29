---
type: concept
aliases: [RALL-E]
domain: TTS
tags: [zero-shot-tts, codec-language-model, robustness, chain-of-thought]
created: 2026-05-29
last_updated: 2026-05-29
---

# RALL-E

## 定义

一种用 **chain-of-thought (CoT) prompting** 增强鲁棒性的零样本 codec 语言模型 TTS（Xin et al., 2024, arXiv:2404.03204），在自回归过程中显式建模 duration 和 pitch 等控制信息，提升 [[VALL-E]] 类系统的稳定性。

## 核心要点

1. 通过 CoT 把时长/音高作为中间推理步骤先生成，再生成声学 token。
2. 代价：CoT 引入额外控制 token，拉长自回归序列、增加推理时间。
3. 在 [[VALL-E R]] 的效率对比中：自回归约 ~105+750 步，推理 12.28s（慢于 VALL-E 10.27s）。

## 代表工作

- RALL-E (Xin et al., 2024)：本词条本体
- 与 [[VALL-E R]] 同期、同为"给 codec LM 补鲁棒性"的工作

## 相关概念

- [[VALL-E]] / [[VALL-E R]] / [[ELLA-V]] / [[VALL-T]]
