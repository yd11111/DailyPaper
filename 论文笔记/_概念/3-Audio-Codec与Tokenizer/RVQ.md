---
type: concept
aliases: [Residual Vector Quantization, 残差向量量化]
---

# RVQ

## 定义

Residual Vector Quantization 残差向量量化：把连续向量逐层量化成离散码本索引，每层量化前一层的残差，从而在低比特码本下逼近精细量化。

## 数学形式

设有 $N_q$ 个码本 $\{C_k\}_{k=1}^{N_q}$，输入向量 $x$：
$$
\begin{aligned}
r_0 &= x \\
i_k &= \arg\min_{j} \|r_{k-1} - C_k[j]\|_2 \\
r_k &= r_{k-1} - C_k[i_k]
\end{aligned}
$$
重建：$\hat{x} = \sum_{k=1}^{N_q} C_k[i_k]$。总码率 $\propto N_q$。

## 核心要点

1. **几乎所有现代 neural codec 的基石**：[[EnCodec]] / [[SoundStream]] / [[DAC]] / [[SpeechTokenizer]] 都基于 RVQ
2. 多码本带来"分层"特性：可只保留前 $k$ 层做 coarse-to-fine 生成（[[VALL-E]]、AudioLM）
3. **VibeVoice 路线选择**：放弃 RVQ，转用单维连续 σ-VAE，避免多码本展开造成的序列爆炸

## 代表工作

- SoundStream / EnCodec 引入到神经 codec
- [[VibeVoice]] 显式不使用 RVQ

## 相关概念

- [[Audio Codec]]
- [[EnCodec]]
- [[DAC]]
- [[Acoustic Tokenizer]]
