---
type: concept
aliases: [GAN, Generative Adversarial Network, 生成对抗网络]
---

# GAN

## 定义
生成对抗网络，通过生成器与判别器的对抗博弈来训练生成模型。生成器学习生成逼真样本，判别器学习区分真假样本。

## 数学形式

$$
\min_G \max_D \mathbb{E}_{x \sim p_\text{data}}[\log D(x)] + \mathbb{E}_{z \sim p_z}[\log(1 - D(G(z)))]
$$

## 核心要点
1. 对抗训练可以生成高保真细节，特别适合波形/图像生成
2. TTS 中常用 least-squares GAN 损失替代原始 GAN 损失以稳定训练
3. 常与 [[Feature Matching Loss]] 配合使用

## 代表工作
- [[HiFi-GAN]]: Multi-Period/Multi-Scale Discriminator 用于语音波形生成
- [[VITS]]: 将 GAN 对抗训练与 [[VAE]] 统一
- [[BigVGAN]]: 大规模 GAN vocoder

## 相关概念
- [[Feature Matching Loss]]
- [[HiFi-GAN]]
- [[Vocoder]]
