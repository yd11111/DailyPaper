---
type: concept
aliases: [Feature-wise Linear Modulation, FiLM Layer, FiLM Conditioning]
---

# FiLM

## 定义

Feature-wise Linear Modulation（特征级线性调制），一种条件注入机制。通过学习条件相关的 scale ($\gamma$) 和 bias ($\beta$) 参数，对中间特征做 affine transform，将条件信息融入网络。

## 数学形式

$$
\text{FiLM}(h \mid c) = \gamma(c) \odot h + \beta(c)
$$

- $h$: 中间层特征（如 [[WaveNet]] hidden）
- $c$: 条件信息（如 speaker embedding、attention output）
- $\gamma(c), \beta(c)$: 由条件通过线性层/MLP 生成的 scale 和 bias
- $\odot$: element-wise 乘法

## 核心要点

1. 轻量级条件注入：只需 2 个线性层生成 $\gamma, \beta$，不改变主网络结构
2. 常用于扩散模型、声码器、TTS 中的说话人/风格条件注入
3. 比 cross-attention 更高效，但表达能力略弱
4. 最初由 Perez et al. (2018) 提出用于视觉问答

## 代表工作

- [[NaturalSpeech 2]]: 在 [[WaveNet]] diffusion model 中用 FiLM 层注入 speech prompt 信息
- Adaptive Instance Normalization (AdaIN): FiLM 的特例（先做 instance norm 再做 affine）
- DiT: 用 AdaLN（Adaptive Layer Norm）注入时间步和类别条件，本质也是 FiLM 思路

## 相关概念

- [[Adaptive Layer Normalization]]
- [[Cross-Attention]]
- [[Layer Normalization]]
- [[Classifier-Free Guidance]]
