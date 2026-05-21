---
type: concept
aliases: [音频 VAE, Audio Variational Autoencoder]
---

# Audio VAE

## 定义

把音频信号压到**连续**潜空间的变分自编码器；区别于 [[EnCodec]] / [[DAC]] 这类**离散** RVQ 编码器。常作为 [[AudioLDM]] / [[Stable Audio]] / 各类 audio diffusion 模型的 latent 层，把 waveform 或 STFT 压成低维连续 latent 后再让扩散模型在 latent 域生成。

## 数学形式

ELBO（去掉常数）：
$$\mathcal{L} = \mathbb{E}_{x\sim D}\bigl[ \mathbb{E}_{z\sim q_\phi(z|x)}\log p_\theta(x|z) - \lambda \cdot D_{\mathrm{KL}}(q_\phi(z|x) \,\|\, p_\psi(z)) \bigr]$$

$\lambda = 1$ 退化为标准 ELBO；$\lambda < 1$ → β-VAE 偏重建；$\lambda > 1$ → 偏码率压缩。

## 核心要点

1. **连续 latent**：相比 RVQ codec，扩散模型在连续空间更自然
2. **KL 与重建的取舍**：β 太大 → posterior collapse；β 太小 → latent 信息冗余
3. **常见瓶颈**：β 手调代价高；[[Target-KL-VAE]] 提出"固定 KL 预算"来缓解
4. 应用：text-to-audio 生成、TTS 后端 latent diffusion、speech enhancement

## 代表工作

- [[Stable Audio]] / [[AudioLDM]]：latent diffusion 的 audio VAE backbone
- [[Target-KL-VAE]]：通过显式 KL 预算约束让码率—质量曲线更稳

## 评测/常见数字

- 典型 16 kHz audio VAE 压缩 ratio：waveform 长度 → latent 长度约 64×–256×

## 相关概念

- [[VAE]]
- [[Audio Codec]]
- [[Flow Matching]]
- [[AudioLDM]]
