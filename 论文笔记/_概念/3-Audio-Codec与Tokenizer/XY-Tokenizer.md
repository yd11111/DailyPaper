---
type: concept
aliases: [XY Tokenizer]
---

# XY-Tokenizer

## 定义

由 Gong et al. (2025) 提出的低比特率离散语音 codec，目标是缓解"语义-声学冲突"（semantic-acoustic conflict in low-bitrate speech codecs），通过特殊设计的 dual-branch 在低帧率（12.5 Hz）下同时保留语义可懂度与声学重建。

## 核心要点

1. **极低帧率 12.5 Hz**：远低于 [[DAC]] (50 Hz) / [[DashengTokenizer]] (25 Hz)，bps 极低。
2. **语义-声学双分支**：显式分离两类信号，避免低比特率下两者互相伤害。
3. **离散 tokenizer**：与 [[LoSATok]] / [[DashengTokenizer]] 的连续 latent 路线不同，更适合自回归 LLM 接入。

## 代表工作

- Gong et al. (2025): "XY-Tokenizer: mitigating the semantic-acoustic conflict in low-bitrate speech codecs" (arXiv:2506.23325)。

## 评测/常见数字

[[LoSATok]] Table 4 中 SeedTTS-EN PESQ 2.173 / STOI 0.901 —— 重建明显弱于 50 Hz 的 [[DAC]] / [[EnCodec]]，但其优势在 LLM-friendly 的低 token 率。

## 相关概念

- [[DAC]] / [[SNAC]]：同为离散 NAC，但帧率更高。
- [[Acoustic Token]] / [[Semantic Token]]：双分支分别处理。
- [[Speech Tokenizer]]：通用范式。
