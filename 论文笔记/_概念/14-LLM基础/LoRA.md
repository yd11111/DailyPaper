---
type: concept
aliases: [Low-Rank Adaptation, LoRA Adapter]
---

# LoRA

## 定义

**Low-Rank Adaptation**：在 LLM / Transformer 的关键线性层旁挂一对低秩矩阵 $A \in \mathbb{R}^{d \times r}, B \in \mathbb{R}^{r \times d}$（$r \ll d$），微调时只更新 $A, B$，原始权重 $W$ 冻结。推理时把 $\Delta W = BA$ 合并回 $W$。

## 数学形式

$$
h = Wx + \Delta W x = Wx + B A x
$$

可训练参数从 $O(d^2)$ 降到 $O(dr)$，常见 $r \in \{4, 8, 16, 64\}$。

## 核心要点

1. 显存/计算开销小，适合在消费级 GPU 上微调大模型。
2. 支持多任务"插拔"：不同任务训不同 LoRA delta，推理时切换。
3. 在语音方向常用于：speech LLM 微调、router / 分类头训练、跨任务适配（如 [[MegaASR]] 的 environment-aware router）。

## 代表工作

- Hu et al. 2021, LoRA 原始论文
- 在语音应用：[[MegaASR]] router 训练

## 相关概念

- [[QLoRA]]
- [[PEFT]]
- [[Speech LLM]]
