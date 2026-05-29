---
type: concept
aliases: [Mi-DashengLM, MispeechMiDasheng]
---

# MiDashengLM

## 定义

由 Mispeech / Xiaomi 团队开发的通用音频理解大模型（"Mi-Dasheng + LLM"），用大规模通用音频 caption 数据训练得到，支持 speech / music / general audio 跨域理解。其内部的 audio encoder 输出 1280-d 高维语义特征，被 [[DashengTokenizer]] 和 [[LoSATok]] 等后续工作作为冻结的 semantic encoder 复用。

## 模型规模

- 主 release：MiDashengLM-7B（基于 7B 通用 LLM backbone）
- HuggingFace 路径：`mispeech/midashenglm-7b-1021-fp32`

## 核心要点

1. **多任务音频字幕**：训练数据以"audio + caption"为主，覆盖 speech / music / sound event 三大域。
2. **音频 encoder 是关键产出**：encoder 部分（不含 LLM head）被广泛用作 SSL 风格的音频 backbone，在 XARES 等理解 benchmark 上接近或超过 [[Whisper]] / [[HuBERT]] / [[WavLM]]。
3. **1280-d 高维**：直接用做 supervision 时，会增加下游生成模型的负担 → 催生了 [[LoSATok]] 的 [[Semantic Bottleneck]] 压缩到 128 维。

## 代表工作

- Dinkel et al. (2025): "MiDashengLM: efficient audio understanding with general audio captions" (arXiv:2508.03983)。
- [[DashengTokenizer]]: 把 MiDashengLM 编码器 + 声学注入做成 1280-d unified tokenizer。
- [[LoSATok]]: 在 MiDashengLM encoder 之后接 SemBo 压到 128 维。

## 评测/常见数字

在 XARES 15 任务上平均 75.48（1280-d），是 LoSATok 笔记 Table 2 中的上界。

## 相关概念

- [[DashengTokenizer]]
- [[LoSATok]]
- [[HuBERT]] / [[WavLM]] / [[Whisper]]：同类 audio backbone 对照。
