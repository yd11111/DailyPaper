---
type: concept
aliases: [MeanFlow, CFG-aware MeanFlow, 平均流蒸馏]
---

# MeanFlow Distillation

## 定义
一种 flow-matching 模型的蒸馏方法。冻结多步 teacher 模型，训练 student 预测给定时间区间 $[t_a, t_b]$ 内的平均速度（mean velocity），使 student 能用更少的步数（如 NFE=2~4）达到接近 teacher 质量。

## 数学形式

$$
\mathcal{L}_{\text{mv}} = \mathbb{E}\left[ w_{\text{mv}} \cdot \left\| v_\phi(x_{t_a}, t_a, \Delta t, c) - \bar{v}_{t_a \to t_b}^{T} \right\|^2 \right]
$$

其中 $\bar{v}_{t_a \to t_b}^T = (x_{t_b}^T - x_{t_a}^T) / (t_b - t_a)$ 为 teacher 轨迹的平均速度。

## 核心要点
1. Student 继承 teacher 参数，新增 interval-duration embedder 处理 $\Delta t$
2. **CFG-aware 变体**：将 [[Classifier-Free Guidance|CFG]] 融入 teacher 轨迹，推理时 student 只需单次条件前向（省一半计算）
3. 自适应 per-sample 权重 $w_{\text{mv}} = (\text{sg}(\ell) + \epsilon)^{-1/2}$ 稳定训练

## 代表工作
- [[dots-tts]]: CFG-aware MeanFlow 蒸馏，NFE=4 下保持接近全精度质量
- MeanFlow 原始论文

## 相关概念
- [[Flow Matching]]
- [[Classifier-Free Guidance]]
- [[Knowledge Distillation]]
