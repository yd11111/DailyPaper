---
type: concept
aliases: [分组codec语言建模, Grouped Codec Language Modeling]
---

# Grouped Code Modeling

## 定义

[[VALL-E 2]] 提出的序列组织方法：把 codec code 序列的**第一码本沿时间轴的连续 $G$ 帧**打包成一个 AR 帧来建模（组间自回归、组内并行）。注意是**沿时间维分组**，**不是**跨 RVQ 码本层堆叠。作用是摆脱 off-the-shelf codec 的固定帧率约束，把帧率按整数倍降低，序列长度缩短为 $1/G$。

## 数学形式

把 code 序列 $\mathbf{C}^{T\times J}$ 沿时间切成大小 $G$ 的组，训练目标：

$$
\mathcal{L} = -\sum_{t=0}^{T/G-1}\log p\big(\mathbf{C}_{t\cdot G:(t+1)\cdot G}\mid \mathbf{C}_{<t\cdot G},\mathbf{x};\theta\big)
$$

AR 侧把组内第一码本嵌入在隐维度拼接，过 group embedding 矩阵 $\mathbf{W}^g$ 得组嵌入。

## 核心要点

1. 帧率/序列长度从 codec 设计问题解耦为 LM 侧的序列重组
2. 同时提速（序列减半/减 4 倍）与缓解长上下文建模问题
3. $G=2$ 几乎白拿 2× 加速且 WER/DNSMOS 略有提升；$G\ge4$ 后 SIM/WER 退化（细粒度依赖被削弱）
4. 长 prompt（如 VCTK 10s）下收益更明显

## 代表工作

- [[VALL-E 2]] (Chen et al., 2024): 提出该方法

## 评测/常见数字

- LibriSpeech: $G=2$ WER/DNSMOS 优于 $G=1$；$G=8$ ref utt SIM 降到 0.566（vs $G=1$ 的 0.643）

## 相关概念

- [[VALL-E 2]]
- [[RVQ]]
- [[Autoregressive Model]]
