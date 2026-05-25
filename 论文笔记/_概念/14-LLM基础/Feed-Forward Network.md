---
type: concept
aliases: [FFN, Position-wise Feed-Forward Network, 前馈网络, MLP层]
---

# Feed-Forward Network

## 定义
Transformer 中每个注意力子层之后的逐位置前馈网络，由两层线性变换和一个非线性激活组成。对序列中每个位置独立施加相同的变换。

## 数学形式

$$
\text{FFN}(x) = W_2 \cdot \sigma(W_1 x + b_1) + b_2
$$

- 输入/输出维度：$d_{model}$（通常 512 或 768）
- 内部维度：$d_{ff}$（通常 $4 \times d_{model}$，即 2048 或 3072）
- $\sigma$：激活函数（原始 Transformer 用 ReLU，后续模型用 GELU / SwiGLU 等）

## 核心要点
1. 与 [[Self-Attention]] 配合：注意力层负责跨位置信息交互，FFN 负责逐位置特征变换
2. 参数量占 Transformer 总参数的约 2/3（因内部维度远大于模型维度）
3. 后续研究发现 FFN 层充当"键值记忆"，存储事实知识（Geva et al. 2021）
4. GLU 变体（SwiGLU, GeGLU）在现代 LLM 中已成为主流替代

## 代表工作
- [[Transformer]]: 原始 FFN 设计（Vaswani et al. 2017）
- [[TransformerTTS]]: 在 TTS encoder/decoder 中使用 FFN

## 相关概念
- [[Self-Attention]]
- [[Layer Normalization]]
- [[Transformer]]
