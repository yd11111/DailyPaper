---
type: concept
aliases: [UniFlowAudio]
---

# UniFlow-Audio

## 定义

由 Xu et al. (2025) 提出的统一 Flow Matching DiT 框架，用单一 DiT 模型从 omni-modal 条件（文本 / 音频 / 视频）生成音频，覆盖 TTA / TTM / TTS 等任务。在 [[LoSATok]] 中被作为下游 DiT 框架使用 —— LoSATok 把 UniFlow-Audio 原始的 acoustic VAE 替换成 LoSATok tokenizer 以做对比。

## 核心要点

1. **多任务 Flow Matching 框架**：原本基于 acoustic VAE (128-d latent)，DiT 学条件分布 $p_\theta(z | c)$。
2. **作为 tokenizer benchmark 平台**：[[LoSATok]] / [[DashengTokenizer]] 等论文都在 UniFlow-Audio 里替换 VAE 做公平对比。
3. **作 acoustic baseline**：在 [[LoSATok]] 的对比中代表"纯 acoustic + DiT"路线，是统一 tokenizer 工作的标准对照组。

## 代表工作

- Xu et al. (2025): "UniFlow-Audio: unified flow matching for audio generation from omni-modalities"。
- 被 [[LoSATok]] 用作 DiT 后端 + acoustic tokenizer baseline。

## 评测/常见数字

LoSATok Table 3 中 UniFlow-Audio 在 LibriTTS TTS 上 WER=3.589, SIM=0.408, UTMOS=2.768（208M DiT）。

## 相关概念

- [[DiT]] / [[Flow Matching]]：底层范式。
- [[Audio VAE]]：原始 tokenizer 设计。
- [[LoSATok]] / [[DashengTokenizer]]：用同框架对比。
