---
type: concept
aliases: [VALL-T]
domain: TTS
tags: [zero-shot-tts, codec-language-model, transducer, robustness]
created: 2026-05-29
last_updated: 2026-05-29
---

# VALL-T

## 定义

一种 decoder-only 的生成式 **transducer** 零样本 TTS（Du et al., 2024, arXiv:2401.14321），通过引入 transducer 结构与可调的相对位置编码，实现对文本的可控性与解码鲁棒性。

## 核心要点

1. 把 transducer 思想与 decoder-only Transformer 结合，提供显式的文本-声学对齐控制。
2. 通过调整相对位置编码增强文本可控性。
3. 在 [[VALL-E R]] 的对比中：cross-sentence WER 4.16（优于 VALL-E 5.48，劣于 VALL-E R 3.18）。

## 代表工作

- VALL-T (Du et al., 2024)：本词条本体
- 与 [[VALL-E R]] / [[ELLA-V]] / [[RALL-E]] 同属 VALL-E 系鲁棒性增强分支

## 相关概念

- [[VALL-E]] / [[VALL-E R]]
- [[Transducer]]
