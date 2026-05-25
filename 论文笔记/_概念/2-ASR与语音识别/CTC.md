---
type: concept
aliases: [Connectionist Temporal Classification, CTC Loss]
---

# CTC

## 定义
Connectionist Temporal Classification，一种用于序列到序列任务的损失函数/解码方法，无需输入输出的逐帧对齐。通过引入 blank token 和多对一映射，允许模型在不知道精确对齐的情况下训练。

## 数学形式

$$
\mathcal{L}_{\text{CTC}} = -\log \sum_{\pi \in \mathcal{B}^{-1}(y)} p(\pi \mid x)
$$

其中 $\mathcal{B}^{-1}(y)$ 是所有可以映射到目标序列 $y$ 的路径集合。

## 核心要点
1. 解决了输入序列和输出序列长度不等时的训练问题
2. 引入 blank token 处理重复字符和静音
3. 常用于 ASR（Conformer-CTC）、OCR 等任务
4. 可与注意力机制结合（CTC/Attention 混合解码）
5. 推理时用贪心解码或 beam search + CTC prefix score

## 代表工作
- [[Conformer]]: Conformer-CTC 是主流 ASR 架构
- [[VALL-E-X]]: SpeechUT 微调时用 CTC 做辅助损失

## 相关概念
- [[ASR]]
- [[Conformer]]
- [[Forced Alignment]]
