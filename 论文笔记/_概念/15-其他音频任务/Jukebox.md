---
type: concept
aliases: [Jukebox]
---

# Jukebox

## 定义
OpenAI 提出的音乐生成模型，使用分层 VQ-VAE 将音频压缩为多分辨率离散 token，再用自回归 Transformer 逐层生成。

## 核心要点
1. 多分辨率 VQ-VAE tokenization（不同时间分辨率）
2. 分层自回归生成：先粗后细
3. 时域连贯性较好但存在明显伪影
4. 在 AudioLM 之前的分层音频生成代表工作

## 代表工作
- [[AudioLM]]: 指出 Jukebox 的分层思路有启发但伪影严重，AudioLM 用 semantic + acoustic 互补解决

## 相关概念
- [[VQ]]
- [[Autoregressive]]
- [[AudioLM]]
