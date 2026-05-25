---
title: TTS 代表模型谱系
type: 领域地图
domain: TTS
tags: [genealogy, models, tts, timeline]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 代表模型谱系

## 谱系全景

TTS 模型可按**技术传承关系**组织为若干"家族"。同一家族的模型共享核心设计理念，后续工作在此基础上改进。

## 家族 1: Autoregressive Acoustic Model 系

**祖先**: Tacotron (2017) → Tacotron 2 (2018)

```
Tacotron 2 (2018)
├── [[TransformerTTS]] (2018) — Transformer 替代 RNN
├── [[FastSpeech]] (2019) — 非自回归 + Length Regulator
│   └── [[FastSpeech2]] (2020) — 显式 variance adaptor
│       └── AdaSpeech 系列 (2021-22) — 自适应微调
└── [[NaturalSpeech]] (2022) — VAE + Flow，LJSpeech 首次匹配人类 MOS
```

**核心特征**: 以 mel-spectrogram 为中间表征，需要外接声码器

### 关键转折

- FastSpeech 将 TTS 从自回归推向非自回归，解决了推理速度和 robustness 问题
- NaturalSpeech 在单说话人场景上把 mel 路线推到极致，但多说话人/零样本仍需更强的框架

## 家族 2: 端到端 (VITS 系)

**祖先**: [[VITS]] (2021)

```
[[VITS]] (2021) — VAE + Normalizing Flow + HiFi-GAN 端到端
├── [[YourTTS]] (2022) — 多语言 + 零样本扩展
├── [[HierSpeech++]] (2023) — 层次化 VAE，显式分离语义/声学
├── [[GPT-SoVITS]] (2024) — GPT 语义预测 + VITS 声学生成
├── [[MaskGCT]] (2025) — Masked Generative Codec Transformer
└── Piper / eSpeak-ng 集成 — 轻量端侧部署
```

**核心特征**: text → waveform 一步到位，VAE latent 空间统一建模

### 关键转折

- VITS 证明了端到端可以达到高质量，但 VAE 容量限制了其在大数据集上的 scaling
- GPT-SoVITS 通过引入 GPT 做语义预测，突破了 VITS 在零样本场景的局限

## 家族 3: Codec Language Model 系

**祖先**: [[AudioLM]] (2022) + [[VALL-E]] (2023)

```
[[AudioLM]] (2022) — 语音 token LM 的概念验证
│
[[VALL-E]] (2023) — 首个 Codec LM TTS
├── [[VALL-E-X]] (2023) — 跨语言扩展
├── [[SPEAR-TTS]] (2023) — 两阶段（语义LM → 声学LM），text-only 预训练
├── [[SeedTTS]] (2024) — 自蒸馏 + RL 后训练
├── [[CosyVoice]] (2024) — LLM + Flow Matching 两阶段
│   ├── [[CosyVoice2]] (2024) — FSQ + 流式
│   └── [[CosyVoice3]] (2025) — 100万h 数据规模
├── [[FireRedTTS]] (2024) — 图像 tokenizer 跨模态
│   └── [[FireRedTTS2]] (2025) — 改进版
├── [[IndexTTS2]] (2025) — 100层 GPT + BigVGAN-v2
├── [[GLM-TTS]] (2025) — 流式 LM + Flow Matching
├── [[Qwen3-TTS]] (2025) — LLM-native（Qwen3 直出 speech token）
├── [[Fish-Speech]] (2024) — 开源社区驱动
├── [[XTTS]] (2024) — Coqui 开源多语言
└── [[Tortoise]] (2023) — 社区先驱（质量高但速度慢）
```

**核心特征**: 将语音离散化为 token，用 LM 做序列到序列预测

### 关键分支

| 分支 | 代表 | 特点 |
|------|------|------|
| AR 全部层 | [[VALL-E]]、[[IndexTTS2]] | 简单直接但序列长 |
| AR 语义 + NAR 声学 | [[VALL-E]]（部分）、[[SPEAR-TTS]] | 分层预测降低复杂度 |
| AR token + Flow/Diffusion 波形 | [[CosyVoice]]、[[GLM-TTS]] | LM 只预测语义 token，波形交给连续模型 |
| LLM 直接出 | [[Qwen3-TTS]] | 不额外训练声学模型 |

### 关键转折

- VALL-E 开创范式，但 8 层 RVQ token 导致序列过长
- CosyVoice 找到 sweet spot：LM 只管语义 token，Flow Matching 做波形——兼顾 LM scaling 和生成质量
- Qwen3-TTS 走向极端：直接用通用 LLM 做 TTS，标志着 TTS 可能融入 LLM 基础设施

## 家族 4: Diffusion / Flow Matching 系

**祖先**: WaveGrad (2020) / Grad-TTS (2021)

```
Grad-TTS (2021) — 基于 score matching 的 mel 生成
├── [[NaturalSpeech2]] (2023) — 连续 latent + diffusion，大规模零样本
│   └── [[NaturalSpeech3]] (2024) — FACodec 四路分解 latent
├── [[F5-TTS]] (2024) — DiT + Flow Matching 端到端
├── [[SemaVoice]] (2025) — 语义 latent + shallow diffusion
└── [[VoxCPM]] (2025) — LLM 预测连续 mel（diffusion loss 替代 CE）
```

**核心特征**: 在连续空间做去噪/流匹配生成，避免离散化信息瓶颈

### 关键转折

- NaturalSpeech2 证明 diffusion 可以做大规模零样本 TTS
- F5-TTS 简化了架构（无需音素对齐、时长模型），降低了使用门槛
- Flow Matching 逐渐替代 Diffusion（训练更稳定、推理更快）

## 家族 5: Neural Vocoder 系

**祖先**: [[WaveNet]] (2016)

```
[[WaveNet]] (2016) — 自回归波形生成，每次预测一个样本点
├── WaveRNN (2018) — 轻量化 RNN 声码器
├── WaveGlow (2018) — Flow-based 并行声码器
├── MelGAN (2019) — GAN-based 快速声码器
├── HiFi-GAN (2020) — 多尺度 GAN，质量+速度平衡
│   └── BigVGAN (2023) — 大规模 HiFi-GAN 变体
│       └── BigVGAN-v2 (2024) — IndexTTS2 等采用
└── Vocos (2023) — 频域声码器
```

**核心特征**: mel → waveform 的最后一公里

### 现状

HiFi-GAN / BigVGAN 已成为事实标准声码器。多数现代 TTS 系统不再单独创新声码器，而是直接采用或微调 HiFi-GAN 变体。

## 家族 6: 解耦表征系 (Disentanglement)

```
[[MegaTTS]] (2023) — 内容/韵律/音色三路分离
├── [[MegaTTS2]] (2023) — 音素级韵律 latent + PLM
└── [[NaturalSpeech3]] (2024) — FACodec 四路（content/prosody/timbre/detail）

[[BaseTTS]] (2024) — 超大规模预训练 + 涌现韵律
```

**核心特征**: 显式或隐式地将语音表征分解为独立因子

## 工业系统族谱

工业级 TTS 系统通常不属于单一学术家族，而是融合多条路线：

| 系统 | 所属机构 | 混合路线 | 核心设计 |
|------|---------|---------|---------|
| [[CosyVoice]] 系列 | 阿里通义 | Codec LM + Flow | LLM 预测语义 token → Flow Matching 生成波形 |
| [[Qwen3-TTS]] | 阿里通义 | LLM-native | Qwen3 LLM 直接输出 multi-codebook token |
| [[SeedTTS]] | 字节 | Codec LM + Diffusion | AR LM + diffusion 精修 + RL 后训练 |
| [[GLM-TTS]] | 智谱 AI | Streaming LM + Flow | 流式 chunk 级 LM + Flow Matching |
| [[VoxCPM]] | 面壁智能 | Tokenizer-free LM | LLM 直接自回归预测连续 mel 帧 |
| [[StepAudio2.5]] | 阶跃星辰 | Speech LLM 统一 | 语音理解+生成统一在一个 LLM 中 |
| [[IndexTTS2]] | Bilibili | Deep Codec LM | 100 层 GPT-2 + BigVGAN-v2 |
| [[FireRedTTS]] | 小红书 | 跨模态 Codec LM | 图像 VQ tokenizer 用于语音 |

## 技术传承图（简化版）

```
WaveNet (2016)
    ↓ 声码器独立
Tacotron 2 (2018) ──→ FastSpeech (2019) ──→ FastSpeech2 (2020)
                                                    ↓
                                              mel 路线成熟
                                                    ↓
VITS (2021) ←─── 端到端思想 ←──── NaturalSpeech (2022)
    ↓                                      ↓
YourTTS / GPT-SoVITS              NaturalSpeech2 (2023)
                                           ↓
AudioLM (2022) + SoundStream              NaturalSpeech3 (2024)
    ↓
VALL-E (2023) ──→ CosyVoice (2024) ──→ CosyVoice2/3 (2024-25)
    ↓                                          ↑
SPEAR-TTS (2023)                    Flow Matching 引入
    ↓
SeedTTS (2024)
    ↓
Qwen3-TTS (2025) ← LLM-native 趋势
VoxCPM (2025) ← Tokenizer-free 趋势
```

## 如何使用本谱系

1. **定位新论文**: 读到一篇新 TTS 论文，先判断它属于哪个家族、解决了哪个家族的什么问题
2. **理解技术演进**: 顺着家族树看"从 A 到 B 改了什么、为什么"
3. **找灵感**: 看不同家族是否有可交叉的思路（如 VITS 的端到端 + Codec LM 的 scaling）
4. **写 related work**: 按家族组织 related work 比按时间线更清晰

---

*最后更新: 2026-05-25*
