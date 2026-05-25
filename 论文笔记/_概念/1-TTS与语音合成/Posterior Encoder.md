---
type: concept
aliases: [Posterior Encoder, 后验编码器]
---

# Posterior Encoder

## 定义
VITS 架构中将频谱图编码为隐变量后验分布 $q(z|x_{\text{spec}})$ 的模块，通常使用 WaveNet 风格的膨胀卷积。训练时从后验采样 $z$，推理时从先验采样。

## 数学形式

$$
q_\phi(z \mid x) = \mathcal{N}(z; \mu_\phi(x), \sigma_\phi(x))
$$

## 核心要点
1. 仅在训练时使用（提供 ground truth 的编码），推理时不需要
2. 输出均值和对数方差，通过重参数化技巧采样
3. 采样的 $z$ 送入 HiFi-GAN 解码器生成波形
4. 与 Prior Encoder 之间通过 KL Divergence + Normalizing Flow 对齐

## 代表工作
- [[VITS]]: 原始提出
- [[GPT-SoVITS]]: 继承 VITS 的后验编码器设计

## 相关概念
- [[VAE]]
- [[KL Divergence]]
- [[Normalizing Flow]]
