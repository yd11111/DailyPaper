---
type: concept
aliases: [FDBench, FullDuplexBench 1.0, FullDuplexBench 1.5]
domain: Evaluation
tags: [benchmark, full-duplex, evaluation]
created: 2026-07-09
---

# FullDuplexBench

## 定义

全双工语音交互评测基准，衡量模型在同时听说场景下的交互能力。包含多个版本（1.0, 1.5），覆盖打断响应、反馈生成、延迟等维度。

## 核心指标

| 指标 | 含义 | 方向 |
|------|------|------|
| SRR | Speech Response Rate | ↑ |
| SIR | Successful Interruption Rate | ↑ |
| EIR | Erroneous Interruption Rate | ↓ |
| SRIR | Successful Response to Interruption Rate | ↑ |
| FSED | First Speech End Delay (首包延迟) | ↓ |
| IRD | Interruption Response Delay | ↓ |
| IRR | Interruption Recognition Rate | ↑ |
| BRR | Backchannel Recognition Rate | ↑ |

## 代表工作

- [[Lychee-FD]]: 10/11 指标最优
- [[Moshi]]: 早期基线

## 相关概念

- [[全双工]]
- [[Turn-Taking]]
- [[Barge-in]]
