---
type: concept
aliases: [Megatron, Megatron-Core]
---

# Megatron-LM

## 定义

NVIDIA 开发的大规模 LLM 训练框架，支持 Tensor Parallelism (TP)、Expert Parallelism (EP)、Context Parallelism (CP)、Sequence Parallelism (SP) 等多维并行策略。是训练百亿参数级 MoE/dense 模型的工业标准框架之一。

## 核心要点
1. 支持多维并行：TP + EP + CP + SP + Data Parallelism
2. 与 Transformer Engine 配合实现 FP8/BF16 混合精度训练
3. Megatron-Energon 提供多模态 dataloader + sequence packing
4. 开源：https://github.com/NVIDIA/Megatron-LM

## 代表工作
- [[Audex]]: 用 Megatron-LM + 512 H100 训练（4-TP, 32-EP, 8-CP）
- Nemotron 系列
- Shoeybi et al., 2019

## 相关概念
- [[MoE]]
- [[Decoder-only Transformer]]
