---
type: concept
aliases: [伪自回归, PAR, Pseudo-Autoregressive Modeling]
domain: TTS
tags: [pseudo-autoregressive, codec-lm-tts, generation-paradigm]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
---

# Pseudo-Autoregressive (PAR)

## 定义
一种离散 token 生成范式：在**固定时间步**上**并行预测所有位置**，但每步只采纳**最左侧动态长度 span**，逐步向右推进直至生成完。由 [[PALLE]]（ACM MM 2025）提出，目的是统一 [[Autoregressive Model|AR]]（固定步 + 每步 1 token）与 [[Non-Autoregressive Model|NAR]]（单步 + 全长）。

## 数学形式
训练时随机掩码 span，只监督最左 $k=\lfloor rT\rfloor$ 个 token：

$$
\arg\max_{\theta}\ p\big([\mathbf{m}\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m})\odot\mathbf{c},\ \mathbf{x};\theta\big)
$$

推理时每步保留最左 $k'=\min(\lfloor r'T^{gen}\rfloor, N_{left})$ 个 token，迭代更新（$N_{left}$ = 剩余待生成 token 数）。

## 核心要点
1. 用"时间步 × 每步长度"二维统一三范式：AR=固定步/固定长，NAR=单步/全长，PAR=固定步/**动态长**。
2. 保留从左到右的时序顺序（软时序归纳偏置），同时并行解码逼近 O(1) 步级延迟。
3. 范式层面与模态融合解耦：可用时间维拼接（[[MaskGCT]] 式）或特征维融合（[[E2 TTS]] 式）。
4. 本质是 [[Masked Generative Transformer]] 解码调度的"强制有序"变体，相对 MaskGIT 的置信度无序解码。

## 代表工作
- [[PALLE]]: 首次提出 PAR + 两阶段（PAR 生成 + NAR 精修），LibriTTS 上同骨干对比 AR −36% WER/4× 快、NAR −43% WER/2× 快。

## 评测/常见数字
[[PALLE]] LibriTTS：cross-sentence WER-W 2.23、SIM-o 0.716、RTF 0.06。

## 相关概念
- [[Autoregressive Model]]
- [[Non-Autoregressive Model]]
- [[Masked Generative Transformer]]
- [[Confidence-based Iterative Decoding]]
