---
type: concept
aliases: [LPC, 线性预测编码, Linear Prediction]
---

# Linear Predictive Coding

## 定义
一种经典语音信号分析方法，假设语音信号可由过去 $P$ 个采样点的线性组合加高斯噪声近似。是传统语音处理的基石，但有线性/高斯/固定窗三个强假设。

## 数学形式

$$
x_t = \sum_{p=1}^{P} a_p x_{t-p} + \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, G^2)
$$

- $a_p$: 第 $p$ 阶线性预测系数
- $P$: 预测阶数（典型 10-16）
- $G^2$: 预测残差方差

## 核心要点
1. 假设语音是线性自回归高斯过程——三个假设均与真实语音有偏差
2. 需要 20-30 ms 固定分析窗（假设短时平稳），但某些音素（如爆破音）远短于此
3. WaveNet 可视为 LPC 的非线性深度学习推广，无需上述三个假设
4. LPC 系数可导出线谱对（LSP）、倒谱等参数化表示

## 代表工作
- [[WaveNet]]: 论文在 Appendix 中将 WaveNet 定位为 LPC 的非线性推广

## 相关概念
- [[Mel-Spectrogram]]
- [[Statistical Parametric Speech Synthesis]]
