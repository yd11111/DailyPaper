---
title: TTS 技术路线图
type: 技术路线
domain: TTS
tags: [tech-route, tts, architecture]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 技术路线图

TTS 的技术演进可以按**中间表征**划分为四条主干路线，外加一条正在浮现的新路线。每条路线有其核心假设、代价和上限。

## 路线 1: 声学特征中介（Mel/Linear Spectrogram）

### 范式

```
Text → 文本编码器 → 声学模型(预测mel) → 声码器(mel→wav)
```

### 核心假设

mel-spectrogram 是一种足够好的中间表征——它保留了说话人音色、韵律、内容的大部分信息，同时维度远低于波形。

### 代表工作

| 模型 | 年份 | 声学模型 | 声码器 | 关键创新 |
|------|------|---------|--------|---------|
| [[TransformerTTS]] | 2018 | Transformer + autoregressive | Griffin-Lim / WaveRNN | 首次将 Transformer 用于 TTS |
| [[FastSpeech]] | 2019 | Transformer + 非自回归 | 任意 | Length Regulator 实现并行生成 |
| [[FastSpeech2]] | 2020 | Transformer + 显式 variance adaptor | HiFi-GAN | 直接预测 duration/pitch/energy |
| [[MegaTTS]] | 2023 | Transformer + GAN + prosody diffusion | HiFi-GAN | 解耦内容/韵律/音色为独立编码 |
| [[MegaTTS2]] | 2023 | Transformer + prosody LM | HiFi-GAN | 音素级韵律 latent + PLM |

### 优势与局限

- **优势**: 架构成熟、训练稳定、推理快（非自回归）、可解释性好（mel 可视化）
- **局限**: mel 到 wav 的声码器引入不可逆误差；mel 维度固定难以适配不同采样率；难做真正的端到端优化
- **现状**: 纯 mel 路线已不是 2024+ 的主流选择，但 mel 仍作为 tokenizer-free 路线的连续目标（见路线 5）

## 路线 2: Codec Token 路线（离散 token 序列）

### 范式

```
Text → LLM/AR模型 → 离散speech token序列 → Codec解码器(token→wav)
```

### 核心假设

语音可以被 [[RVQ]]（Residual Vector Quantization）压缩为离散 token 序列，从而复用 LLM 的序列建模能力。第一层 token 捕获语义/内容，深层 token 捕获声学细节。

### 代表工作

| 模型 | 年份 | 语言模型 | Codec | 关键创新 |
|------|------|---------|-------|---------|
| [[VALL-E]] | 2023 | AR(第1层) + NAR(2-8层) | [[EnCodec]] | 开创 Codec LM 范式 |
| [[SPEAR-TTS]] | 2023 | 语义LM → 声学LM | 自定义 | 两阶段：先预测语义 token 再预测声学 token |
| [[CosyVoice]] | 2024 | LLM(语义token) + Flow Matching | 自训练 | 将 LLM 与 Flow Matching 解码结合 |
| [[CosyVoice2]] | 2024 | LLM(FSQ token) + Flow Matching | [[FSQ]]-based | [[Finite Scalar Quantization]] 替代 RVQ |
| [[CosyVoice3]] | 2025 | LLM + Flow Matching | 同 v2 | 100万h 数据，工业极致 |
| [[IndexTTS2]] | 2025 | 100层 GPT + BigVGAN-v2 | 自训练 | 极深 LM + 高质量声码器 |
| [[GPT-SoVITS]] | 2024 | GPT(语义) + VITS(声学) | [[HuBERT]] token | 语义 token + VITS 联合 |
| [[SeedTTS]] | 2024 | AR LM + Diffusion | 自训练 | 自蒸馏 + 强化学习后训练 |
| [[GLM-TTS]] | 2024 | Streaming LM + Flow | 自训练 | 流式 chunk 级解码 |
| [[Qwen3-TTS]] | 2026 | Qwen3 LLM 直出 | multi-codebook | LLM-native，不额外训练声学模型 |
| [[FireRedTTS]] | 2024 | AR LM | 图像 tokenizer 跨模态 | 用图像 VQ 做语音 tokenization |

### Token 设计演进

```
EnCodec RVQ 8层    →    RVQ 简化(1-2层语义+NAR补全)    →    FSQ 连续量化
(2022, 信息分散)        (2023-24, VALL-E/SPEAR-TTS)        (2024, CosyVoice2)
                                                            ↓
                                                    Tokenizer-free
                                                    (2025, VoxCPM)
```

### 优势与局限

- **优势**: 复用 LLM scaling law；天然支持 in-context learning（零样本）；token 序列可做 text-speech 联合训练
- **局限**: RVQ 多层 token 带来序列长度爆炸（1s 语音 ≈ 600-1200 token）；离散化丢失细节；codec 质量是系统上限；对 codec 选择敏感

## 路线 3: 端到端路线（Text → Waveform）

### 范式

```
Text → 单一模型 → Waveform（无中间表征/显式中间表征在模型内部）
```

### 核心假设

消除级联系统的信息损失，让梯度从波形直接流回文本编码器。

### 代表工作

| 模型 | 年份 | 架构 | 关键创新 |
|------|------|------|---------|
| [[VITS]] | 2021 | VAE + Normalizing Flow + HiFi-GAN | 首个真正端到端的高质量 TTS |
| [[YourTTS]] | 2022 | VITS 多语言扩展 | 跨语言零样本 |
| [[NaturalSpeech]] | 2022 | VAE + Flow + 大规模预训练 | 首次在 LJSpeech 上达到人类水平 MOS |
| [[F5-TTS]] | 2024 | DiT + Flow Matching | 无需音素对齐、无需时长模型 |
| [[VoxCPM]] | 2025 | LLM 直接预测连续 mel → BigVGAN | 无 tokenizer 的端到端 |

### 优势与局限

- **优势**: 训练目标统一、无级联误差、架构简洁
- **局限**: 训练不稳定（波形空间高维）；调参难度大；大规模数据下训练成本高

## 路线 4: 连续 Latent 路线（Diffusion/Flow on Latent Space）

### 范式

```
Text → 条件编码 → Diffusion/Flow Matching → 连续latent → 解码器 → Waveform
```

### 核心假设

在连续 latent 空间做生成比离散 token 空间更自然——语音的韵律、音色本就是连续变化的，强制离散化会丢失信息。

### 代表工作

| 模型 | 年份 | 生成模型 | Latent 来源 | 关键创新 |
|------|------|---------|------------|---------|
| [[NaturalSpeech2]] | 2023 | Diffusion | 自训练 VAE latent | 连续 latent + diffusion，首个大规模零样本 |
| [[NaturalSpeech3]] | 2024 | Flow Matching | FACodec 分解 latent | 分解为 content/prosody/timbre/detail 四路 latent |
| [[SemaVoice]] | 2026 | Flow Matching | 语义 latent | SFM 对齐 + patch-wise diffusion head |
| [[MaskGCT]] | 2024 | Masked Generative | Codec latent | 非自回归 masked prediction |

### 优势与局限

- **优势**: 避免离散化信息瓶颈；生成质量理论上限更高；Flow Matching 训练比 Diffusion 稳定
- **局限**: 推理速度受 diffusion steps 限制（需 ODE solver）；latent 空间设计需要精心调参

## 路线 5（快速增长中）: Tokenizer-free / 连续 AR

### 范式

```
Text → LLM/AR模型 → 连续mel帧或连续latent序列 → 声码器 → Waveform
```

### 核心假设

语音的连续特性不应被强制离散化。跳过 codec tokenization 这个"人为瓶颈"，直接在连续空间做自回归建模，同时保留 LM 的 scaling 优势。

### 代表工作

| 模型 | 年份 | 连续目标 | 生成方式 | 关键创新 |
|------|------|---------|---------|---------|
| [[MELLE]] | 2024 | 连续 mel | AR + latent sampling module | 首个无 VQ 的 AR TTS |
| [[LatentLM]] | 2024 | 连续 latent | AR + diffusion loss | 用 diffusion loss 替代 cross-entropy |
| [[VoxCPM]] | 2025 | 连续 mel 帧 | LLM AR 连续预测 | 无 tokenizer，180 万小时数据 |
| [[FELLE]] | 2025 | 连续 mel | AR + token-wise Flow Matching | 在 MELLE 基础上加逐 token Flow Matching 精修 |
| [[CLEAR]] | 2025 | 连续 latent | AR 连续预测 | 专注低延迟，港中大/华为 |
| [[SemaVoice]] | 2026 | 语义 latent | 连续 AR + patch-wise diffusion | SFM 对齐 + diffusion head |

### 优势与局限

- **优势**: 避开 codec 设计这个"开放战场"；理论信息保真度更高；连续空间天然适配语音的渐变特性
- **局限**: 训练稳定性不如离散 token（连续值回归的方差更大）；scaling 能力是否匹配 Codec LM 路线尚无定论；缺乏公认的连续表征评测标准
- **现状**: 从 2024 年的单点探索（MELLE）发展为 2026 年的活跃方向（5+ 篇工作），但尚未出现百万小时级以上的成功案例（VoxCPM 180 万 h 是目前最大）

## 路线选择决策树

```
你的场景需要什么？
│
├─ 需要零样本说话人克隆？
│  ├─ 有 >10万h 数据 → 路线2 (Codec LM): CosyVoice/SeedTTS/IndexTTS2
│  ├─ 数据有限 (<1万h) → 路线4 (Latent Diffusion): NaturalSpeech2
│  └─ 社区/开源优先 → 路线3 (E2E): F5-TTS
│
├─ 需要流式/低延迟？
│  ├─ 首包 <200ms → 路线2 (Streaming LM): CosyVoice2/GLM-TTS
│  └─ 可接受 500ms+ → 路线4 (Diffusion): 需多步推理
│
├─ 需要多语言？
│  └─ 路线2/3: YourTTS / CosyVoice3 / Qwen3-TTS
│
└─ 需要最高质量（不考虑延迟）？
   └─ 路线4: NaturalSpeech3 / SemaVoice
```

## 路线融合趋势

2024-2026 的趋势是**路线融合**而非路线替代：

- **Codec LM + Flow Matching**（路线 2+4）: [[CosyVoice]] 系列——LM 预测语义 token，Flow Matching 生成波形
- **Codec LM + Diffusion**（路线 2+4）: [[SeedTTS]]——AR 预测 token + diffusion 精修
- **LM + 连续目标**（路线 2+5）: [[VoxCPM]]——LM 架构但预测连续值
- **连续 AR + Flow**（路线 5+4）: [[FELLE]]——连续 AR 主体 + token-wise Flow Matching 精修；[[SemaVoice]]——连续 AR + patch-wise diffusion head
- **E2E + Flow**（路线 3+4）: [[F5-TTS]]——DiT 端到端但用 Flow Matching 训练
- **LLM-native + 指令控制**: [[Qwen3-TTS]]——通用 LLM 直接做 TTS + 自然语言韵律理解；[[FlexiVoice]]——NL 指令控制风格

纯单一路线的系统越来越少，融合是主旋律。2026 年新增的趋势是 **LLM-native**（TTS 成为 LLM 的模态能力）和 **自然语言指令可控**（不再需要参考音频或显式条件）。

---

*最后更新: 2026-05-25*
