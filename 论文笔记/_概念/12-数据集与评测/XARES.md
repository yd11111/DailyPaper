---
type: concept
aliases: [X-ARES]
---

# XARES (X-ARES)

## 定义

由 Zhang et al. (2025) 提出的跨域音频 encoder 评估框架："A Comprehensive Framework for Assessing Audio Encoder Performance"。统一了 speech / music / general audio 三类共 15 个下游任务，用 linear probing 协议（典型是 MLP / Qwen2.5-0.5B + frozen encoder）评估表示能力，结果归一化到 0-1。

## 任务列表（15 项）

- **Speech (7)**: LS100h (ASR), CD (Crema-D emotion), FSC (intent), LibCnt (speaker counting), LSMF (gender), RAV (emotion), VocS (vocal sound)
- **Music (4)**: FMA, GTZAN (genre), MAESTRO/MT (transcription), NSynth (timbre)
- **Audio (4)**: Clo (captioning), DES (sound event detection), ESC (env classification), Urb8 (urban sound)

## 核心要点

1. **跨域综合性**：单一 benchmark 覆盖三大音频域，方便比较通用 audio encoder。
2. **统一协议**：所有任务都走 linear probing / 轻量 head + frozen backbone，避免不同 fine-tuning 配方的混淆。
3. **是评估 unified tokenizer 的标准平台**：[[DashengTokenizer]] / [[LoSATok]] / Ming-UniAudio 等近期工作都用 XARES 报理解能力。

## 代表工作

- Zhang et al. (2025): "X-ARES: A Comprehensive Framework for Assessing Audio Encoder Performance"。
- GitHub: jimbozhang/xares
- 被 [[LoSATok]] / [[DashengTokenizer]] 等多个 2025-2026 工作使用。

## 典型数字

- MiDashengLM 平均 75.48
- DashengTokenizer 74.67
- Whisper 62.43
- LoSATok (128-d) 59.30
- HuBERT 49.82
- WavLM 44.33

## 相关概念

- [[Audio Understanding]]
- [[Linear Probing]]
- [[Audio Encoder]]
