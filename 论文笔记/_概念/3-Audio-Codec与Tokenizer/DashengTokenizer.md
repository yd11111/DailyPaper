---
type: concept
aliases: [Dasheng Tokenizer]
---

# DashengTokenizer

## 定义

由 Mispeech / Xiaomi 团队 (Dinkel et al., 2026) 提出的 **连续高维 (1280-d) 统一音频 tokenizer**。在冻结的 [[MiDashengLM]] 语义编码器之上注入声学信息（用 non-overlapping mel patch embedding 与语义对齐），构造一个能同时支持理解与生成的 25 Hz 连续 latent。是 [[LoSATok]] 直接前作。

## 模型规模 / 关键配置

- 帧率 25 Hz，输入 16 kHz waveform
- latent 维度：1280
- decoder 基于 [[Vocos]]
- HuggingFace：`mispeech/dashengtokenizer`

## 核心要点

1. **"One layer is enough"**：用一层 acoustic 注入即可让冻结的 MiDashengLM 特征同时保留语义和重建声学。
2. **理解能力强**：XARES 15 任务平均 74.67，接近原始 MiDashengLM 的 75.48。
3. **重建质量优**：SeedTTS-EN PESQ 4.122，是同类 unified tokenizer 中最高。
4. **DiT 友好性差**：因为 latent 是 1280 维太宽，下游 DiT 必须扩到 1536-d / 975M 参数才能在 TTS 上达到 LoSATok 用 208M 即可达到的水平 ([[LoSATok]] Table 3)。

## 代表工作

- Dinkel et al. (2026): "DashengTokenizer: one layer is enough for unified audio understanding and generation" (arXiv:2602.23765)。
- 被 [[LoSATok]] 直接对比并提出低维改进。

## 相关概念

- [[MiDashengLM]]：semantic encoder backbone。
- [[LoSATok]]：在其基础上压缩到 128 维的改进版。
- [[Vocos]]：decoder backbone。
- [[Mixed Token]]：连续 mixed semantic+acoustic latent。
- [[Audio VAE]]：连续 latent codec 范式。
