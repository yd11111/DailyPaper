---
type: concept
aliases: [PerceiverResampler, Perceiver Resampler, Perceiver-Resampler]
---

# PerceiverResampler

## 定义
一种基于 cross-attention 的降采样模块，使用固定数量的 learnable query vectors 从变长输入序列中提取固定数量的输出 token。源自 Perceiver 架构思想，广泛用于多模态模型中将不同模态的变长特征压缩为固定长度的条件表示。

## 数学形式

$$
\text{latents} = \text{CrossAttn}(Q=\text{learnable\_queries}, KV=\text{input\_sequence}) + \text{learnable\_queries}
$$

输入：变长序列（如 w2v-BERT 音频特征）
输出：固定 $N$ 个 latent token（如 PilotTTS 中 $N=32$）

## 核心要点
1. 通过 learnable query vectors 和 cross-attention 实现变长到定长的映射
2. 计算量与输出 latent 数量成正比，与输入长度解耦
3. 常用于 Q-Former、Flamingo 等视觉-语言模型，PilotTTS 将其迁移到语音条件编码

## 代表工作
- [[PilotTTS]]: 用 PerceiverResampler（depth=2, 32 latents）从 w2v-BERT 2.0 特征提取风格条件 token
- Flamingo: 视觉-语言模型中首次大规模使用

## 相关概念
- [[Q-Former]]
- [[Cross-Attention]]
- [[w2v-BERT 2.0]]
