---
type: concept
aliases: [Finite Scalar Quantization, 有限标量量化]
---

# FSQ

## 定义
Finite Scalar Quantization（有限标量量化），将每个维度独立量化到有限整数集合 $[-K, K]$，取代传统 VQ 的码本查找机制。天然实现 100% 码本利用率，消除 dead codes 问题。

## 数学形式

$$
\bar{H} = \text{ROUND}(\text{Proj}_{down}(H))
$$

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j}(2K+1)^j
$$

- 码本大小: $(2K+1)^D$
- 典型配置: $K{=}4, D{=}4 \to 6561$ 个码字

## 核心要点
1. 不需要 codebook embeddings 和 commitment loss，训练更简单
2. 每个码本 entry 天然被使用，无需 EMA 更新或 reset 策略
3. 使用 [[Straight-Through Estimator]] 进行梯度近似
4. 在语音 tokenizer 中使用时，可有效解耦说话人信息与语义内容

## 评测/常见数字
- CosyVoice 2: VQ 利用率 23% -> FSQ 利用率 100%，ASR 错误率下降 40%+

## 代表工作
- [[CosyVoice2]]: 在语音 tokenizer 中用 FSQ 替代 VQ，实现 100% 码本利用率
- [[VoxCPM]]: 256 维 × 9 级 FSQ 作为 TTS 中间正则化瓶颈（不做预测目标），产生 semi-discrete 语义-韵律骨架
- Google (Mentzer et al. 2023): FSQ 原始论文，用于图像量化

## 相关概念
- [[RVQ]]
- [[VQ-VAE]]
- [[Straight-Through Estimator]]
