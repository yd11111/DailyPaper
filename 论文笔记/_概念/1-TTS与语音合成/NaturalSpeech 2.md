---
type: concept
aliases: [NaturalSpeech2, NaturalSpeech 2, NS2]
---

# NaturalSpeech 2

## 定义
微软推出的零样本 TTS 系统，使用 latent diffusion 模型在连续语音 latent 空间中生成，结合 RVQ 离散化的语音 codec token 和 duration/pitch 预测器，实现高质量零样本语音合成。

## 核心要点
1. 在连续 latent 空间（而非离散 token）上做 diffusion 建模
2. 使用 neural audio codec 提取多层 RVQ token 再映射回连续 latent
3. 引入 speech prompting 实现零样本音色克隆
4. 在 LibriSpeech test-clean 上达到 SOTA 水平

## 数学形式

$$
p_\theta(x_0 | c) = \int p(x_T) \prod_{t=1}^{T} p_\theta(x_{t-1} | x_t, c) dx_{1:T}
$$

其中 $c$ 为文本和说话人条件。

## 代表工作
- [[NaturalSpeech2]]: 435M params, 44K hours MLS, WaveNet-based latent diffusion, CMOS 与 GT 持平
- [[NaturalSpeech 3]]: 继任者，使用 DiT + [[Conditional Flow Matching]] 替代 WaveNet + diffusion

## 评测/常见数字
- LibriSpeech test-clean WER: ~2%
- Speaker Similarity: 高于 VALL-E

## 相关概念
- [[VALL-E]]
- [[DDPM]]
- [[RVQ]]
- [[Zero-shot TTS]]
- [[NaturalSpeech]]
