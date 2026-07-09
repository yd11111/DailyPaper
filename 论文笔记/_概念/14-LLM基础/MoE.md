---
type: concept
aliases: [Mixture of Experts, 混合专家模型, Sparse MoE]
---

# MoE (Mixture of Experts)

## 定义

Mixture of Experts 是一种稀疏激活架构：模型总参数量大，但每次推理只激活其中一部分"专家"子网络，由路由器（Router）决定激活哪些专家。兼具大参数量的知识容量和小激活量的推理效率。

## 数学形式

$$
y = \sum_{i=1}^{N} g_i(x) \cdot E_i(x)
$$

其中 Router 输出 $g_i(x)$ 只对 Top-K 个专家非零（稀疏激活）。

## 核心要点
1. 总参数大但每 token 计算成本低（如 30B 总参/3B 激活）
2. Router 通常用线性层 + Top-K softmax
3. 需要 load balancing loss 防止专家利用不均
4. 训练需要 Expert Parallelism (EP) 分布式策略

## 代表工作
- [[Audex]]: Nemotron-Cascade-2-30B-A3B（128 experts, 6 activated）
- Switch Transformer (Google, 2021)
- Mixtral 8x7B (Mistral, 2024)
- DeepSeek-V2/V3 (DeepSeek, 2024-2025)

## 相关概念
- [[Decoder-only Transformer]]
- [[Cross-Entropy Loss]]
