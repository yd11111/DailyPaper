---
type: concept
aliases: [Variational Autoencoder, σ-VAE, sigma-VAE, 变分自编码器]
---

# VAE

## 定义

变分自编码器（Kingma & Welling 2014）：把输入编码到一个隐变量分布 $q_\phi(z|x)$，再从中采样并解码回 $\hat{x}$；目标是 ELBO（重建 + KL 散度正则）。

**σ-VAE**（来自 [[LatentLM]]）：把 $\sigma$ 固定为预定义先验分布 $\mathcal{N}(0, C_\sigma)$ 而非可学，以避免 AR 建模时 latent 方差坍缩——[[VibeVoice]] 的 acoustic tokenizer 即采用此设计。

## 数学形式

标准 VAE：
$$
\mathcal{L}_{\mathrm{ELBO}} = \mathbb{E}_{q_\phi(z|x)}[\log p_\theta(x|z)] - \mathrm{KL}(q_\phi(z|x) \,\|\, p(z))
$$
σ-VAE 重参数化：
$$
z = \mu + \sigma \odot \epsilon,\quad \epsilon \sim \mathcal{N}(0,1),\ \sigma \sim \mathcal{N}(0, C_\sigma)
$$

## 核心要点

1. 标准 VAE：$\mu, \sigma$ 都来自 encoder，可学
2. σ-VAE：$\sigma$ 固定为先验采样（不是 learnable），保证 latent 方差不坍缩
3. 在 AR + diffusion head 范式下，σ-VAE 是当前主流选择

## 代表工作

- 原论文 (Kingma & Welling 2014)
- [[LatentLM]]: 引入 σ-VAE 处理 AR 建模
- [[VibeVoice]]: acoustic tokenizer 用 σ-VAE

## 相关概念

- [[LatentLM]]
- [[Acoustic Tokenizer]]
- [[Next-Token Diffusion]]
