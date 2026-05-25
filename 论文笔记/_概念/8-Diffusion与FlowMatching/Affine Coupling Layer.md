---
type: concept
aliases: [Affine Coupling, 仿射耦合层]
---

# Affine Coupling Layer

## 定义
Normalizing Flow 中的基础可逆变换层。将输入分成两部分，一部分保持不变并用于预测另一部分的 scale 和 shift 参数，实现可逆且可计算雅可比行列式的变换。

## 数学形式

$$
\begin{aligned}
y_{1:d} &= x_{1:d} \\
y_{d+1:D} &= x_{d+1:D} \odot \exp(s(x_{1:d})) + t(x_{1:d})
\end{aligned}
$$

其中 $s(\cdot)$ 和 $t(\cdot)$ 是任意神经网络（scale 和 translation 函数）。

## 核心要点
1. 可逆性保证训练和推理使用同一模型
2. 雅可比行列式为对角矩阵，计算高效
3. 源自 Real NVP (Dinh et al., 2017)

## 代表工作
- [[VITS]]: Flow-based decoder 使用 affine coupling layers
- [[YourTTS]]: 4 层 affine coupling layers，每层含 4 个 WaveNet 残差块
- [[SC-GlowTTS]]: Flow-based ZS-TTS

## 相关概念
- [[VAE]]
- [[WaveNet]]
