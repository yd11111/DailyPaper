---
type: concept
aliases: [PE, 位置编码, Positional Embedding, Sinusoidal Positional Encoding]
---

# Positional Encoding

## 定义
为缺乏序列顺序感知能力的 [[Transformer]] 注入位置信息的机制。由于 [[Self-Attention]] 是置换不变的（permutation-invariant），没有位置编码的 Transformer 无法区分不同顺序的输入。

## 数学形式

### 正弦/余弦位置编码（Vaswani et al. 2017）

$$
PE(pos, 2i) = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

- $pos$: 序列中的位置索引
- $i$: 维度索引
- $d_{model}$: 模型隐藏维度
- 输出维度与输入 embedding 相同，直接相加

### Scaled Positional Encoding（Li et al. 2018, TransformerTTS）

$$
x_i = \text{prenet}(x_i) + \alpha \cdot PE(i)
$$

- $\alpha$: 可训练标量，适配不同域（文本 vs 声学）的尺度差异

## 核心要点
1. **正弦/余弦 PE**: 固定不可学习，支持外推到训练未见长度（理论上），无额外参数
2. **可学习绝对 PE**: 每个位置一个可训练向量（BERT / GPT-2），受限于最大长度
3. **相对位置编码**: [[RoPE]]（旋转位置编码）、ALiBi 等，直接编码相对距离而非绝对位置，外推性更好
4. **TTS 场景特殊性**: 源端（文本）和目标端（mel）尺度差异大，TransformerTTS 提出用可训练 $\alpha$ 解决

## 代表工作
- [[Transformer]]: 原始正弦/余弦 PE（Vaswani et al. 2017）
- [[TransformerTTS]]: Scaled PE with trainable $\alpha$（Li et al. 2018）
- [[RoPE]]: 旋转位置编码（Su et al. 2021），现代 LLM 主流方案

## 相关概念
- [[Transformer]]
- [[Self-Attention]]
- [[RoPE]]
