---
type: concept
aliases: [FVQ, 分解向量量化, Factorized VQ]
---

# Factorized Vector Quantization

## 定义

将向量量化分解为多个独立的子空间（如 content / prosody / acoustic detail），每个子空间用独立的 VQ 码本量化，使不同属性被编码到不同的离散 token 序列中。与 RVQ 的层级残差分解不同，FVQ 实现的是属性维度的分解。

## 数学形式

$$
\mathbf{h} \rightarrow \{\mathbf{z}_p, \mathbf{z}_c, \mathbf{z}_d\} = \{\text{FVQ}_p(\mathbf{h}), \text{FVQ}_c(\mathbf{h}), \text{FVQ}_d(\mathbf{h})\}
$$

每个 FVQ 包含信息瓶颈（投影到低维再量化）：
$$
\mathbf{z}_i = \text{VQ}(\text{Proj}_{\text{down}}(\mathbf{h}_i)), \quad \text{dim}(\text{Proj}_{\text{down}}) = 8
$$

## 核心要点
1. 不同于 RVQ 的"粗到细"层级分解，FVQ 按语义属性分解
2. 需要额外的解耦技术（信息瓶颈、监督损失、梯度反转）确保属性分离
3. 每个子空间可独立生成，实现属性级别的控制

## 代表工作
- [[NaturalSpeech3]]: FACodec 中使用 FVQ 将语音分解为 prosody / content / detail 三个子空间
- [[SpeechTokenizer]]: 类似思路但通过 HuBERT 蒸馏分离语义层

## 相关概念
- [[RVQ]]
- [[Vector Quantization]]
- [[EnCodec]]
- [[Information Bottleneck]]
