---
type: concept
aliases: [Stochastic Duration Predictor, SDP, 随机时长预测器]
---

# Stochastic Duration Predictor

## 定义
基于 flow 的生成模型，学习给定文本条件下音素时长的概率分布（而非确定性点估计），推理时通过采样产生多样化的韵律节奏。由 [[VITS]] 首次提出。

## 数学形式

$$
\log p_\theta(d|c_\text{text}) \geq \mathbb{E}_{q_\phi(u,\nu|d,c_\text{text})}\Big[\log\frac{p_\theta(d-u,\nu|c_\text{text})}{q_\phi(u,\nu|d,c_\text{text})}\Big]
$$

- $d$: 离散整数时长
- $u \in [0,1)$: 变分反量化噪声
- $\nu$: 变分数据增强变量

## 核心要点
1. 用 variational dequantization 将离散整数时长连续化
2. 用 variational data augmentation 解决标量维度过低无法做 flow 变换的问题
3. 使用 Neural Spline Flow（单调有理二次样条）作为 coupling layer
4. 训练时用 stop gradient 隔离梯度，不影响其他模块
5. 推理时通过调节噪声标准差控制韵律变化幅度

## 代表工作
- [[VITS]]: 首次提出，实现了多样化韵律的并行 TTS

## 相关概念
- [[VITS]]
- [[Normalizing Flow]]
- [[Duration Predictor]]
- [[Monotonic Alignment Search]]
