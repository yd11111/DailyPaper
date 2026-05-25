---
title: TTS 领域总览
type: 领域地图
domain: TTS
tags: [overview, tts, domain-map]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 领域总览

## 领域定义与范围

文本到语音（Text-to-Speech, TTS）将文本序列转化为自然语音波形。现代 TTS 的核心追求已从"能听"演进到"像人"——要求 **零样本说话人克隆**、**情感/韵律可控**、**流式低延迟**、**多语言泛化**四个维度同时达到高水平。

本笔记库覆盖 TTS 及其紧密上下游，包括：

| 层级 | 覆盖范围 | 代表笔记 |
|------|---------|---------|
| **前端** | 文本分析、G2P、韵律预测 | [[FastSpeech]]、[[FastSpeech2]] |
| **声学模型** | mel 生成、token 预测、latent 生成 | [[VALL-E]]、[[NaturalSpeech2]]、[[CosyVoice]] |
| **声码器/解码器** | mel→wav、token→wav、latent→wav | [[WaveNet]]、[[VITS]]、[[HierSpeech++]] |
| **端到端** | text→wav 一步到位 | [[VITS]]、[[F5-TTS]]、[[VoxCPM]] |
| **Codec 前置** | 语音 tokenization | [[SoundStream]]、[[DAC]]、[[SNAC]] |
| **评测** | 指标、benchmark、评测工具 | [[SUPERB]]、[[Emilia]]、[[TTSDS2]] |

## 四大技术路线

详见 [[TTS-技术路线图]]，此处仅给鸟瞰：

```
┌──────────────────────────────────────────────────────────┐
│                     TTS 技术路线                          │
├──────────────┬───────────────┬──────────────┬────────────┤
│  声学特征中介  │  Codec Token  │  端到端 E2E  │  Latent    │
│  (mel/linear) │  (RVQ codes)  │  (text→wav)  │  (连续表征) │
├──────────────┼───────────────┼──────────────┼────────────┤
│ FastSpeech2  │ VALL-E        │ VITS         │ NatSpeech2 │
│ TransformerTTS│ SPEAR-TTS    │ F5-TTS       │ NatSpeech3 │
│ Tacotron2    │ SeedTTS       │ VoxCPM       │ SemaVoice  │
│ MegaTTS      │ CosyVoice     │              │ MaskGCT    │
│              │ IndexTTS2     │              │            │
│              │ GLM-TTS       │              │            │
│              │ GPT-SoVITS    │              │            │
└──────────────┴───────────────┴──────────────┴────────────┘
```

## 关键演进节点

| 年份 | 里程碑 | 意义 |
|------|--------|------|
| 2016 | [[WaveNet]] | 自回归神经声码器，首次达到接近人类的语音质量 |
| 2019 | [[FastSpeech]] | 非自回归并行生成，解决推理速度瓶颈 |
| 2020 | [[FastSpeech2]] | 显式时长/音高/能量预测，mel 路线成熟 |
| 2021 | [[VITS]] | VAE + Flow + HiFi-GAN 端到端，消除级联误差 |
| 2023.01 | [[VALL-E]] | Codec LM 范式开创，in-context learning 做零样本 TTS |
| 2023.02 | [[SPEAR-TTS]] | 多阶段 token 预测（语义→声学），探索 text-only 预训练 |
| 2023.04 | [[NaturalSpeech2]] | 连续 latent + diffusion，避开离散 token 信息瓶颈 |
| 2023.06 | [[MegaTTS]] / [[MegaTTS2]] | 解耦韵律/音色/内容，mel 路线的巅峰 |
| 2024.06 | [[CosyVoice]] | LLM + Flow Matching 两阶段，开源可用 |
| 2024.09 | [[F5-TTS]] | DiT + Flow Matching 端到端，无需时长模型/音素对齐 |
| 2024.10 | [[FireRedTTS]] | 图像 tokenizer 跨模态做 TTS |
| 2024.12 | [[CosyVoice2]] | Finite Scalar Quantization 替代 RVQ，流式 chunk-aware |
| 2025.03 | [[IndexTTS2]] | 100 层 LM + BigVGAN-v2 声码器，开源可用 |
| 2025.05 | [[CosyVoice3]] | 100 万小时训练数据，Codec LM 路线的工业极致 |
| 2025.05 | [[Qwen3-TTS]] | 500 万小时私有数据，LLM-native TTS |
| 2025.05 | [[VoxCPM]] | 无 tokenizer 直接在连续 mel 上做 LLM |

## 当前格局

### 开源生态

| 系统 | 路线 | 训练数据 | 特点 |
|------|------|---------|------|
| [[CosyVoice3]] | Codec LM + Flow | 100万h | 阿里通义，工业级完整度 |
| [[F5-TTS]] | DiT Flow Matching | 10万h Emilia | 社区驱动，架构简洁 |
| [[GPT-SoVITS]] | VITS + GPT | 社区数据 | 低门槛，社区活跃 |
| [[Fish-Speech]] | Codec LM | 70万h | 开源社区，多语言 |
| [[IndexTTS2]] | AR LM + BigVGAN | 5.5万h Emilia | 100层深度 LM |

### 闭源/半开源

| 系统 | 路线 | 训练数据 | 特点 |
|------|------|---------|------|
| [[Qwen3-TTS]] | LLM-native | 500万h | 阿里通义，LLM 直出 speech token |
| [[SeedTTS]] | Codec LM + Diffusion | 未公开（推测>100万h） | 字节，强零样本 |
| [[GLM-TTS]] | Streaming LM + Flow | 10万h | 智谱，严格数据筛选 |
| [[VoxCPM]] | Tokenizer-free LM | 180万h | 面壁，连续 mel 建模 |

### 研究前沿（2025 热点）

1. **Tokenizer-free**: [[VoxCPM]] 证明 LLM 可直接建模连续 mel，跳过 codec 这一"信息瓶颈"
2. **流式低延迟**: [[CosyVoice2]] chunk-aware causal、[[GLM-TTS]] streaming decoder，目标首包延迟 < 200ms
3. **超大规模数据**: 从 10 万小时级（2024）跃升到 100-500 万小时级（2025），数据质量 > 数据量
4. **LLM 原生 TTS**: [[Qwen3-TTS]] 直接用 LLM 做 TTS，不额外训练声学模型
5. **评测标准化**: [[Emilia]]、[[TTSDS2]]、[[SpeechJudge]] 推动可复现评测

## 与相邻领域的关系

```
Speech LLM（5-Speech-LLM）
    ↕ 语音理解+生成统一
TTS（1-TTS）←→ Audio Codec（3-Codec）
    ↕ 端到端对话           ↕ token 设计
全双工（6-全双工）    语音 SSL（9-SSL）
    ↕ 多模态融合
Omni（7-Omni）
```

- **TTS → Speech LLM**: [[AudioLM]]、[[StepAudio2.5]] 等将 TTS 能力整合到统一语音 LLM 中
- **TTS ← Audio Codec**: codec 质量直接决定 Codec LM 路线的上限（[[SoundStream]]→[[DAC]]→[[SNAC]]）
- **TTS → 全双工**: 流式 TTS 是全双工对话系统的关键组件（[[Moshi]]、[[OmniFlatten]]）
- **TTS ← 语音 SSL**: [[HuBERT]]、[[WavLM]] 的语义 token 被广泛用作 TTS 的中间表征

## 导航

- 技术路线详解 → [[TTS-技术路线图]]
- 核心开放问题 → [[TTS-核心挑战]]
- 评测指标体系 → [[TTS-评测体系]]
- 模型族谱 → [[TTS-代表模型谱系]]
- 趋势判断 → [[TTS-趋势判断]]

---

*最后更新: 2026-05-25*
