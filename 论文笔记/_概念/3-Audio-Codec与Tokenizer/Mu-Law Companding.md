---
type: concept
aliases: [μ-law 压扩, mu-law companding, μ-law encoding, G.711]
---

# Mu-Law Companding

## 定义
ITU-T G.711 标准定义的非线性音频量化方法，通过对数压缩将 16-bit 线性 PCM 信号压缩为 8-bit（256 级），在低幅值区域保留更多精度，适合语音信号的动态范围特性。

## 数学形式

$$
f(x_t) = \text{sign}(x_t) \frac{\ln(1 + \mu |x_t|)}{\ln(1 + \mu)}
$$

- $x_t \in (-1, 1)$: 归一化音频采样值
- $\mu = 255$: 压扩系数
- 输出量化为 256 级后用 softmax 建模

## 核心要点
1. 非线性量化显著优于等间距线性量化，重建语音质量接近原始
2. WaveNet 中将 16-bit（65,536 级）压缩为 256 级，使 softmax 输出可行
3. 广泛用于电话通信（PCM 编码）
4. 后续 WaveRNN 等模型转向混合逻辑分布替代 softmax

## 代表工作
- [[WaveNet]]: 使用 μ-law 256 级量化 + softmax 输出分布

## 相关概念
- [[Softmax]]
- [[Linear Predictive Coding]]
