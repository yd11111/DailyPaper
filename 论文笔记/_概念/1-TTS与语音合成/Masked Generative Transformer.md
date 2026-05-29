---
type: concept
aliases: [掩码生成式 Transformer, MaskGIT, Masked Generative Modeling]
domain: TTS
tags: [masked-generative, maskgit, non-autoregressive, generation-paradigm]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
---

# Masked Generative Transformer

## 定义
一类基于双向（bidirectional）注意力的离散 token 生成模型：训练时随机掩码一部分 token、监督模型还原；推理时从全掩码开始，多轮迭代地并行预测并按置信度逐步确定 token。源自图像生成的 MaskGIT（Chang et al., CVPR 2022），后被引入语音 token 生成。

## 数学形式
推理为迭代式：第 $t$ 轮在已确定 token 条件下并行预测全部掩码位置，保留高置信子集，重复直到全部确定。属 [[Non-Autoregressive Model|NAR]] 范式（无固定从左到右顺序）。

## 核心要点
1. 双向上下文——每个位置可看到左右两侧已确定 token。
2. 推理步数远少于 AR（不需逐 token），靠置信度调度决定每轮采纳哪些。
3. 在语音上由 [[MaskGCT]] 代表化；[[PALLE]] 在此骨干上加"强制从左到右"约束得到 [[Pseudo-Autoregressive|PAR]]。

## 代表工作
- [[MaskGCT]]: 语音 token 上的 masked generative TTS。
- [[PALLE]]: 复用该骨干（12 层 / 16 头 / dim 1024）作为 PAR 与精修两阶段的共享架构。

## 评测/常见数字
[[MaskGCT]]（同骨干 LibriTTS，[[PALLE]] Tab.3）：cross-sentence WER-W 4.52、SIM-o 0.703、RTF 0.10。

## 相关概念
- [[Non-Autoregressive Model]]
- [[Pseudo-Autoregressive]]
- [[Confidence-based Iterative Decoding]]
- [[MaskGCT]]
