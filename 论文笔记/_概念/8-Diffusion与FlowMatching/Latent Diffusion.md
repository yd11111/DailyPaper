---
type: concept
aliases: [LDM, Latent Diffusion Model, 潜空间扩散]
---

# Latent Diffusion

## 定义

在压缩后的潜空间（latent space）而非原始高维数据空间（如 waveform 或图像像素）上执行扩散过程的生成模型。先用 encoder 将数据映射到低维 latent，再在 latent 上做 forward/reverse diffusion，最后用 decoder 还原到数据空间。

## 数学形式

$$
z = f_{enc}(x), \quad \hat{z}_0 = \text{Diffusion}(z_T, c), \quad \hat{x} = f_{dec}(\hat{z}_0)
$$

- 输入：原始数据 $x$（waveform / 图像）
- latent 维度远小于原始数据维度
- 在语音场景中，encoder 常为 [[Audio Codec]]（如 [[EnCodec]]），latent 为 [[RVQ]] 连续向量

## 核心要点

1. 计算效率高：在低维 latent 上做 diffusion 比在原始波形上快得多
2. 灵活的条件注入：可通过 cross-attention、[[FiLM]] 等方式注入文本/说话人条件
3. 需要预训练好的 encoder-decoder（codec / VAE），质量依赖于 latent 的信息完整性
4. 图像领域由 Stable Diffusion (Rombach et al. 2022) 推广，语音领域由 [[NaturalSpeech 2]] 首次系统化应用

## 代表工作

- [[NaturalSpeech 2]]: 在 [[RVQ]] codec latent 上做 [[WaveNet]]-based diffusion，435M params，44K hours
- [[NaturalSpeech 3]]: 升级为 DiT + [[Conditional Flow Matching]]
- Stable Diffusion (Rombach et al. 2022): 图像领域经典 LDM

## 评测/常见数字

- NaturalSpeech 2: 150 diffusion steps 推理（歌声 1000 steps），CMOS 与 GT 持平
- 相比直接在 waveform 上做 diffusion（如 [[WaveGrad]]），latent diffusion 在语音上显著降低计算成本

## 相关概念

- [[DDPM]]
- [[Conditional Flow Matching]]
- [[Audio Codec]]
- [[RVQ]]
- [[VAE]]
