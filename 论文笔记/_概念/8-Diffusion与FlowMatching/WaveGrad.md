---
type: concept
aliases: [Wave Grad]
---

# WaveGrad

## 定义

Google 2020 年 (Chen et al., ICLR 2021) 的扩散式神经声码器，直接在**波形空间**做 DDPM 采样，给定 Mel 谱条件生成 24 kHz 波形。是把 [[DDPM]] 思路引入音频生成的早期代表，开创了"audio in waveform space via diffusion"路线。

## 数学形式

forward：$q(x_n | x_0) = \mathcal{N}(\sqrt{\bar\alpha_n} x_0, (1-\bar\alpha_n) I)$
reverse：训练网络 $\epsilon_\theta(x_n, n, c)$ 预测噪声，条件 $c$ 为 Mel 谱。

## 核心要点

1. **波形域**：不在 latent / spectrogram，而是直接在 24kHz 波形采样点
2. **iterative refinement**：从噪声逐步去噪到波形，6–50 步可达 [[HiFi-GAN]] 级质量
3. **MOS 接近原始**：WaveGrad-50 在 LJSpeech 上 MOS ≈ 4.47
4. **后继**：[[DiffWave]] / [[BDDM]] / [[FastDiff]] 都受其启发
5. **教训**：波形域扩散计算成本高，被后来 [[Stable Audio]] 这类 latent 扩散反超

## 代表工作

- 原论文 ICLR 2021 (arXiv 2009.00713)
- [[WavFlow]]：把 flow matching 用到波形域，本质是 WaveGrad 思路在 FM 时代的复活

## 评测/常见数字

- LJSpeech 50 步采样 MOS 4.47（基本与 ground-truth 4.58 持平）

## 相关概念

- [[DDPM]]
- [[DiffWave]]
- [[Flow Matching]]
- [[HiFi-GAN]]
