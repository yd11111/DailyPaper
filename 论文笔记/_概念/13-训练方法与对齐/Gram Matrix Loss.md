---
type: concept
aliases: [Gram Loss, Style Loss]
---

# Gram Matrix Loss

## 定义

把特征图各通道之间的内积矩阵（Gram 矩阵）作为风格描述子，对齐两组特征的 Gram 矩阵的损失。原本由 Gatys et al. (2016) 在 neural style transfer 中提出，用于约束生成图像与参考图像的"风格"一致。在音频领域被 [[LoSATok]] (2026) 借鉴成 [[Time-Relation Loss]]，用于跨维度的语义结构对齐。

## 数学形式

对特征 $F \in \mathbb{R}^{C \times N}$：

$$
G = F F^\top \in \mathbb{R}^{C \times C}
$$

Gram loss：$\mathcal{L} = \| G_{\mathrm{gen}} - G_{\mathrm{ref}} \|^2$。

[[Time-Relation Loss]] 是其"时间版本"：把 Gram 算在时间维度而非通道维度上，得到 $T \times T$ 帧关系矩阵。

## 核心要点

1. **不要求逐元素相等**：通过二阶统计（内积）匹配，捕捉的是"结构相关性"。
2. **跨维度对齐**：原图像应用中，Gram 矩阵的大小只依赖通道数；在音频中被巧妙改用为帧数 $T$，使得高/低维特征也能在相同 $T \times T$ 空间对齐。
3. **Jing et al. (2019) 综述**：被 [[LoSATok]] 引用作为 Gram 损失的概念背景文献。

## 代表工作

- Gatys et al. (2016)：原始 neural style transfer。
- Jing et al. (2019)：综述。
- [[LoSATok]]：在音频 SSL 蒸馏中创新性使用。

## 相关概念

- [[Time-Relation Loss]]：本损失在时间维度上的特殊化。
- [[Knowledge Distillation]]：结构化蒸馏的一类。
