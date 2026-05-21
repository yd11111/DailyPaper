---
type: concept
aliases: [Centered Kernel Alignment]
---

# CKA

## 定义
Centered Kernel Alignment：量化两组神经表征相似度的指标。对正交变换与各向同性缩放不变，因此能跨初始化、跨架构甚至跨模型比较 hidden state。常用于神经网络可解释性、表示学习分析、跨模型对齐研究。

## 数学形式

线性核版本（最常用）：

$$
\mathrm{CKA}(X, Y) = \frac{\lVert Y^{\top} X \rVert_F^2}{\lVert X^{\top} X \rVert_F \, \lVert Y^{\top} Y \rVert_F}
$$

其中 $X, Y \in \mathbb{R}^{d \times n}$ 为两组在 $n$ 个样本上的激活矩阵（列向量为单样本表示）。实际使用时对 $X, Y$ 先做行中心化。输出范围 $[0, 1]$。

也有 RBF 核版本（更敏感但更慢），通过把 $X^{\top} X$ 换成核矩阵 $K$ 实现。

## 核心要点
1. **不变性**: 对正交变换 + 各向同性缩放不变 —— 因此适合跨模型/跨初始化比较
2. **比 CCA 更稳定**: 不需要做矩阵求逆，对小样本更鲁棒
3. **可时间化**: 把样本维换成时间维就能做时间序列同步分析（见 [[Synchronization-Turn-Taking]]）
4. **常见 baseline 数字**: 随机/无关表示约 $0.05-0.15$；中度相关约 $0.3-0.5$；高度相关 $0.7+$

## 代表工作
- 原论文：Kornblith et al. 2019, "Similarity of Neural Network Representations Revisited"（ICML）
- [[Synchronization-Turn-Taking]]: 把 CKA 跨时间 lag 应用到两个 [[Moshi]] 实例的 hidden state 序列上，测量对话双方内部表示的同步度

## 相关概念
- [[Mutual Information]]（替代度量，趋势一致但更难估计）
- [[Probing]]
