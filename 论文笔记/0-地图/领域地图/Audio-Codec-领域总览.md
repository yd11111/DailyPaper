---
title: Audio Codec 领域总览（占位）
type: 领域地图
domain: Codec
tags: [overview, codec, tokenizer, domain-map, placeholder]
created: 2026-05-26
last_updated: 2026-05-26
status: placeholder
---

# Audio Codec / Tokenizer 领域总览

> 🚧 **占位文档**。本领域的核心 taxonomy 已在 [[TTS-表示层地图]] §2 完整建立（基于 Mousavi 2025 #7 的五维分类），本文先以 stub 形式存在，等需要从 codec 自身视角组织（而非 TTS 表示层视角）时再扩展。

## 领域定义与范围

神经音频 codec / tokenizer 把连续音频波形压缩成离散或半离散表示，**既是 TTS / ASR / SpeechLM 的底层基座**，也是独立的研究方向。在 2023+ TTS 领域，**codec 选择对系统质量的影响往往超过模型架构本身**。

## 与 TTS-表示层地图的分工

| 视角 | 文档 |
|------|------|
| **TTS 选型视角**（codec 怎么影响 TTS）| [[TTS-表示层地图]] §2 / §4.2 / §7 |
| **Codec 自身视角**（codec 如何设计、如何评估、跨域可迁移性）| 本文（待扩展）|

短期可以**只看 [[TTS-表示层地图]]**，长期 codec 演进足够多时再单独扩展本文。

## 当前主流 codec 速览

详见 [[TTS-表示层地图]] §2 表格。简表：

| Codec | 量化 | 帧率 | 流式 | 主要应用 |
|---|---|---|---|---|
| [[SoundStream]] | RVQ | — | 部分 | 通用 |
| [[EnCodec]] | RVQ | 75 fps | 部分 | 通用 / VALL-E 系列底座 |
| [[DAC]] | RVQ | 50 fps | — | 通用 / 高质量 |
| [[SNAC]] | MSRVQ（多尺度）| — | 部分 | 通用 |
| [[SpeechTokenizer]] | RVQ + SSL 蒸馏 | — | — | speech-only / TTS 友好 |
| [[Mimi]] | RVQ + SSL 蒸馏 | **12.5 fps** | ✓ 因果 T | 流式对话 |
| [[FACodec]] | GRVQ + 解耦 | — | — | NaturalSpeech 3 |
| [[FSQ]] | FSQ（CosyVoice 2 使用）| — | ✓ | LLM-native TTS 友好 |
| WavTokenizer | SVQ | — | — | 单码本大词表 |
| X-Codec | RVQ（双编码器融合）| — | — | speech-only |
| SemantiCodec | RVQ + diffusion decoder | — | — | speech-only |

## 核心挑战（与 [[TTS-核心挑战]] 挑战 5 共享）

1. **重建质量上限 → TTS 质量天花板**（详见 [[TTS-表示层地图]] §4.2）
2. **流式可行性 vs 非因果 SSL tokenizer**（详见 [[TTS-表示层地图]] §4.3）
3. **跨域可迁移性**（speech ↔ music ↔ general audio）
4. **公平比较问题**（Mousavi 2025 自承"trained under inconsistent conditions"）
5. **Tokenizer-free 路线的兴起是否使 codec 设计问题过时？**（见 [[TTS-技术路线图]] 路线 5）

## 评测方法（来自 Mousavi 2025）

详见 [[TTS-表示层地图]] §6 + [[TTS-评测体系]] §跨域 Codec 评测的补充。

- **重建质量**：PESQ / STOI / ViSQOL / Mel-distance / SI-SDR
- **跨域 benchmark**：Codec-SUPERB / VERSA / DASB / SALMon / Zero-resource

## 待完善清单

- [ ] 写 `0-代表模型谱系/Audio-Codec-代表模型谱系.md`（codec 自身的演进谱系，与 [[TTS-代表模型谱系]] 不同侧重）
- [ ] 用 11 篇综述方法读"Codec 专题综述"（已有 Mousavi 2025 作主线 + 其他补充）
- [ ] 把现有 `3-Audio-Codec与Tokenizer/` 下论文按 [[笔记frontmatter规范]] 加 routes/problems 标签

## 相关笔记

- [[TTS-表示层地图]] — Codec 在 TTS 语境下的完整地图（本领域主要参考）
- [[TTS-核心挑战]] 挑战 5 — Codec 设计作为 TTS 7 大挑战之一
- [[TTS-评测体系]] §跨域 Codec 评测的补充 — Codec 评测在 TTS 视角下的应用
- [[待回填地图]] — 新 codec 论文进入后的回流入口

---

*占位 v1，2026-05-26 — 短期可只看 [[TTS-表示层地图]]。*
