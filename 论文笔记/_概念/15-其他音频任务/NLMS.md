---
type: concept
aliases: [Normalized Least Mean Squares, 归一化最小均方]
---

# NLMS (Normalized Least Mean Squares)

## 定义
归一化最小均方算法，一种自适应滤波算法，用于在线估计线性系统的脉冲响应。在 [[AEC]] 中用于估计线性回声路径，是 LAEC（Linear AEC）模块的核心算法。

## 数学形式

$$
\mathbf{w}(n+1) = \mathbf{w}(n) + \frac{\mu}{\|\mathbf{x}(n)\|^2 + \delta} e(n) \mathbf{x}(n)
$$

其中 $\mathbf{w}$ 为滤波器权重，$\mathbf{x}$ 为输入向量（远端信号），$e(n)$ 为误差信号，$\mu$ 为步长，$\delta$ 为正则化项。

## 核心要点
1. 相比 LMS，NLMS 对输入信号功率做了归一化，收敛更稳定
2. 只能建模线性回声路径，非线性部分（设备失真）需要后续神经网络处理
3. 是两阶段混合 AEC 方法中 LAEC 模块的标准选择

## 代表工作
- [[LMPAN]]: 使用 NLMS 做 LAEC 前处理

## 相关概念
- [[AEC]]
- [[ERLE]]
