---
type: concept
aliases: [FD, Frechet Distance, Wasserstein-2 Distance, FID, FAD]
---

# Fréchet Distance

## 定义

两个多元高斯分布之间的 Wasserstein-2 距离的闭合形式解。在生成模型评测中广泛使用：比较生成样本与真实样本在预训练特征空间中的分布差异。图像领域称 FID (Fréchet Inception Distance)，音频领域称 FAD (Fréchet Audio Distance)。

## 数学形式

$$
\mathrm{FD}(\mathcal{N}(\boldsymbol{\mu}_g, \boldsymbol{\Sigma}_g), \mathcal{N}(\boldsymbol{\mu}_r, \boldsymbol{\Sigma}_r)) = \|\boldsymbol{\mu}_g - \boldsymbol{\mu}_r\|_2^2 + \mathrm{Tr}(\boldsymbol{\Sigma}_g + \boldsymbol{\Sigma}_r) - 2\mathrm{Tr}\left[(\boldsymbol{\Sigma}_r^{1/2}\boldsymbol{\Sigma}_g\boldsymbol{\Sigma}_r^{1/2})^{1/2}\right]
$$

- 输入：两组样本在冻结特征空间中的均值和协方差
- 输出：标量距离（越小表示分布越接近）

## 核心要点

1. 假设特征分布为高斯——对非高斯分布可能不准确
2. 需要足够多的样本才能可靠估计协方差矩阵（高维时容易 rank-deficient）
3. 对特征空间的选择敏感：FID 用 Inception-v3，FAD 用 VGGish，SR-FD 用 Whisper/CTC
4. 传统上只用于评测（离线计算），SR-FD 首次将其转为可微分训练损失

## 代表工作

- FID (Heusel et al. 2017): 图像生成评测标准
- FAD (Kilgour et al. 2019): 音频生成评测
- KAD (Chung et al. 2025): 改进的音频评测距离
- [[SR-FD]]: 首次将 FD 从评测指标转为 TTS 训练损失

## 评测/常见数字

- FID: ImageNet 生成模型典型 1-10（越低越好）
- FAD: 音频生成典型 1-5（依特征空间而异）
- SR-FD 中的 empirical FD（CTC 空间）: 0.005-0.008 量级

## 相关概念

- [[Flow Matching]]
- [[UTMOS]]
- [[DNSMOS]]
- [[Feature Matching]]
