---
type: concept
aliases: [ELBO, Evidence Lower Bound, 变分下界]
---

# ELBO

## 定义
Evidence Lower Bound，变分推断中对不可计算的边际对数似然 $\log p(x)$ 的下界估计。最大化 ELBO 等价于最小化近似后验与真实后验之间的 KL 散度。

## 数学形式

$$
\text{ELBO} = \mathbb{E}_{q_\phi(z|x)}\Big[\log p_\theta(x|z)\Big] - D_\text{KL}\big(q_\phi(z|x) \| p_\theta(z)\big)
$$

第一项为重建项，第二项为正则项。

## 核心要点
1. 是 [[VAE]] 训练的理论基础
2. 可分解为重建损失 + [[KL Divergence]] 正则
3. ELBO 越大意味着模型越好地拟合了数据分布

## 代表工作
- [[VITS]]: 将 ELBO 与对抗训练结合用于端到端 TTS
- [[VAE]]: Kingma & Welling 2014，ELBO 的原始推导

## 相关概念
- [[VAE]]
- [[KL Divergence]]
- [[Normalizing Flow]]
