---
type: concept
aliases: [Permutation Invariant Training, 排列不变训练, uPIT]
---

# PIT (Permutation Invariant Training)

## 定义

解决多说话人分离中输出排列歧义的训练策略。由于分离模型的多个输出通道没有固定的说话人对应关系，PIT 在所有可能的排列中选择损失最小的那个来计算梯度。

## 数学形式

$$
\pi^* = \arg\min_{\pi \in S_N} \sum_{s=1}^{N} \mathcal{L}(\hat{\mathbf{y}}_s, \mathbf{y}_{\pi(s)})
$$

其中 $S_N$ 为 N 元素的全排列集合，$\mathcal{L}$ 为任意损失函数（常用 SI-SNR / MSE / L1）。

对于 N=2，只有 2 种排列；N=3 有 6 种。N 较大时需用 Hungarian 算法近似。

## 核心要点

1. 首次由 Yu et al. (2017) 提出，解决了 deep clustering 之后语音分离领域的核心训练难题
2. utterance-level PIT (uPIT) 是最常用的变体——在整个 utterance 上选排列，而非逐帧选
3. 适用于任何多输出生成任务中存在排列歧义的场景（分离、多乐器转录等）
4. 计算开销随说话人数阶乘增长，N > 3 时通常需要近似算法

## 代表工作

- Yu et al. 2017: 提出 PIT 用于语音分离
- [[Conv-TasNet]]: 结合 PIT 实现端到端时域分离
- [[DialogueSidon]]: 在 VAE 隐空间中使用 PIT 解决对话分离的排列歧义

## 相关概念

- [[Source Separation]]
- [[Conv-TasNet]]
- [[SI-SNR]]
