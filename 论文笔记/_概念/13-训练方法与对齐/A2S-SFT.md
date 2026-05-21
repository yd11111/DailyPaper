---
type: concept
aliases: [Acoustic-to-Semantic Progressive SFT, Acoustic-to-Semantic SFT]
---

# A2S-SFT

## 定义

**Acoustic-to-Semantic Progressive SFT**：[[MegaASR]] 提出的鲁棒 ASR 监督微调方案，按 [[WER]] 难度分级（<30% → <50% → <70%）+ 模块按序解冻（speech encoder + aligner → LLM → joint）的三阶段 [[Curriculum Learning|课程学习]]。

## 核心要点

1. 三个 phase：
   - Phase 1：只训 encoder + aligner，按 WER<30%, <50%, <70% 三段课程
   - Phase 2：LLM 微调，用 WER<70% 全集
   - Phase 3：encoder + aligner + LLM 联合微调
2. 设计动机：先让声学模块"听清楚"再让语义模块"恢复"，避免一开始 LLM 被噪声特征带偏。
3. 比朴素 SFT 在鲁棒 benchmark 上多约 0.7+ WER 降幅（[[VOiCES]] 8.31 → 7.59）。

## 代表工作

- [[MegaASR]]：原始提出

## 相关概念

- [[DG-WGPO]]
- [[Curriculum Learning]]
- [[Voices-in-the-Wild-2M]]
- [[Qwen3-ASR]]
