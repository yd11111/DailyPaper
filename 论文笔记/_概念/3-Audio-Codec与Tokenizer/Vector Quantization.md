---
type: concept
aliases: [VQ, Vector Quantization, 向量量化]
---

# Vector Quantization

## 定义
把连续向量映射到离散码本中最近 codeword 的量化方法。

## 核心要点
1. 训练用 commitment loss + EMA 更新码本
2. VAE-VQ (van den Oord 2017) 是源头
3. RVQ 把 VQ 残差再量化，多层堆叠提升码率/质量比

## 代表工作
- [[CosyVoice]] / [[EnCodec]] / [[SoundStream]] 等 codec 都基于 VQ

## 相关概念
- [[OmniFlatten]]
