---
type: concept
aliases: [KL Divergence, KL 散度, Kullback-Leibler Divergence]
---

# KL Divergence

## 定义
衡量两个概率分布差异的非对称度量。在 VAE 中用于约束后验分布接近先验分布。

## 数学形式

$$
D_{\text{KL}}(q \| p) = \mathbb{E}_{x \sim q}\left[\log \frac{q(x)}{p(x)}\right]
$$

对高斯分布有解析解：

$$
D_{\text{KL}}(\mathcal{N}(\mu_1, \sigma_1^2) \| \mathcal{N}(\mu_2, \sigma_2^2)) = \log \frac{\sigma_2}{\sigma_1} + \frac{\sigma_1^2 + (\mu_1 - \mu_2)^2}{2\sigma_2^2} - \frac{1}{2}
$$

## 核心要点
1. 非对称：$D_{\text{KL}}(q \| p) \neq D_{\text{KL}}(p \| q)$
2. 在 VAE/VITS 中：$q$ 是后验，$p$ 是先验，KL 项防止后验偏离先验太远
3. KL 权重（$\beta$-VAE）可调节重建质量 vs 正则化的平衡

## 代表工作
- [[VITS]]: 后验编码器与先验编码器之间的 KL 对齐
- [[GPT-SoVITS]]: 继承 VITS 的 KL 训练

## 相关概念
- [[VAE]]
- [[Normalizing Flow]]
- [[Cross Entropy]]
