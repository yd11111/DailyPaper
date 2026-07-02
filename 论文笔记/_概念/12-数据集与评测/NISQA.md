---
type: concept
aliases: [NISQA, Non-Intrusive Speech Quality Assessment]
---

# NISQA

## 定义

基于机器学习的非侵入式语音质量评估模型，预测感知 MOS 分数。与 [[DNSMOS]] 类似，但使用不同的训练数据和架构。不需要参考信号（non-intrusive）。

## 核心要点

1. 非侵入式：只需输入信号即可预测质量分，无需干净参考
2. 预测的是感知 MOS（1-5 分），与人工主观评测相关但不完全一致
3. 常与 [[DNSMOS]] 并用以交叉验证自动质量评估结果
4. 已知局限：对分离/恢复后的语音可能给出与人工 MOS 不一致的分数（如 DialogueSidon 中 NISQA 低于 GENESES 但人工 MOS 高于 GENESES）

## 代表工作

- [[DialogueSidon]]: 作为评测指标之一

## 相关概念

- [[DNSMOS]]
- [[MOS]]
- [[UTMOS]]
