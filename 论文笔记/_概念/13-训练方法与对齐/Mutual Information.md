---
type: concept
aliases: [Mutual Information, MI, 互信息]
---

# Mutual Information

## 定义

信息论中度量两个随机变量 $X, Y$ 间统计依赖性的量；零当且仅当二者独立。在表示学习/对话分析中被用作"两个表示之间共享了多少信息"的非线性度量，是 [[CKA]] 的常见对照指标。

## 数学形式

$$
I(X; Y) = \mathbb{E}_{p(x, y)}\!\left[\log \frac{p(x, y)}{p(x)\, p(y)}\right]
$$

等价于 $H(X) - H(X \mid Y) = H(Y) - H(Y \mid X)$。

高维连续变量上很难精确估计，常用方法：MINE（神经网络下界）、InfoNCE、k-NN 估计、binning。

## 核心要点
1. **比 CKA / 线性相关更强**：能捕捉非线性依赖；但估计方差大、对样本数敏感
2. **trends 一致**：在 [[Synchronization-Turn-Taking]] 中作者报告 MI 与 [[CKA]] 趋势一致，因此正文只展示 CKA
3. **常被用作 SSL 目标**：InfoNCE 等对比学习损失实质是 MI 下界
4. **lag 化版本**：把 $X_t, Y_{t+\Delta}$ 当作输入，可做时间错位下的信息共享分析（类似 CKA-vs-lag）

## 代表工作
- 信息论经典（Shannon 1948）
- MINE: Belghazi et al. 2018, "Mutual Information Neural Estimation"
- [[Synchronization-Turn-Taking]]: 作为 [[CKA]] 的旁证（趋势一致，未在正文展示）

## 相关概念
- [[CKA]]
- [[Probing]]
