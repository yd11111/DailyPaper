---
type: concept
aliases: [Finite Scalar Quantization, 有限标量量化]
---

# FSQ (Finite Scalar Quantization)

## 定义

将连续向量的每一维独立量化到有限整数集 $[-K, K]$，通过标量量化替代传统 VQ 的码本查找，避免码本坍塌等训练不稳定问题。

## 数学形式

$$
\bar{h}_j = \operatorname{ROUND}(\operatorname{Proj}_{down}(h)_j), \quad \bar{h}_j \in [-K, K]
$$

$$
\mu = \sum_{j=0}^{D-1} \bar{h}_j (2K+1)^j
$$

隐式码本大小为 $(2K+1)^D$，无需显式码本存储。训练时用直通估计（Straight-Through Estimation）近似梯度。

## 核心要点

1. 无需码本 → 无码本坍塌、无 EMA 更新
2. 码本利用率天然 100%（所有 token 都被使用）
3. 可与任何编码器架构组合（插入 encoder 中间层即可）
4. 在 VQ-VAE 类任务中与传统 VQ 性能相当甚至更优

## 代表工作

- [[CosyVoice3]]: 在 MinMo voice encoder 中插入 FSQ 做 speech tokenizer，25 Hz
- [[CosyVoice 2]]: 在 SenseVoice-Large 中插入 FSQ（前作方案）
- Mentzer et al. 2024 (ICLR): 提出 FSQ，在图像/视频 VQ-VAE 中验证

## 评测/常见数字

- CosyVoice 3 中 FSQ 量化后 ASR WER 比 VQ 低 40%+（Table 10 对比 VQ-SenseVoice vs FSQ-SenseVoice）
- 情感识别准确率反而提升（62.4→68.4），体现量化瓶颈的正则化效果

## 相关概念

- [[RVQ]]
- [[Vector Quantization]]
- [[Speech Tokenizer]]
- [[Semantic Token]]
