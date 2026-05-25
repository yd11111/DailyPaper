---
title: "Qwen3-TTS Technical Report"
method_name: "Qwen3-TTS"
authors: [Hangrui Hu, Xinfa Zhu, Ting He, Dake Guo, Bin Zhang, Xiong Wang, Zhifang Guo, Ziyue Jiang, Hongkun Hao, Zishan Guo, Xinyu Zhang, Pei Zhang, Baosong Yang, Jin Xu, Jingren Zhou, Junyang Lin]
year: 2026
venue: arXiv
tags: [zero-shot-tts, streaming-tts, multilingual-tts, voice-cloning, speech-tokenizer, controllable-tts, voice-design]
image_source: online
arxiv_html: https://arxiv.org/html/2601.15621v1
created: 2026-05-25
---

# 论文笔记：Qwen3-TTS Technical Report

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Alibaba Group (通义实验室) |
| 日期 | January 2026 |
| 项目主页 | — |
| 对比基线 | [[CosyVoice 3]] / [[F5-TTS]] / [[MaskGCT]] / [[Seed-TTS]] / [[Spark-TTS]] / MiniMax-Speech / ElevenLabs / GPT-4o |
| 链接 | [arXiv](https://arxiv.org/abs/2601.15621) / [Code](https://github.com/QwenLM/Qwen3-TTS) |

---

## 一句话总结

> Qwen 系列首个 TTS 模型，基于双轨 LM 架构和两套语音 tokenizer（25Hz 单码本 + 12Hz 多码本），在 500 万小时多语种数据上训练，实现 SOTA 零样本克隆、可控语音设计和 97ms 首包延迟的流式合成。

---

## 核心贡献

1. **双轨 Tokenizer 设计**: 提出 Qwen-TTS-Tokenizer-25Hz（单码本，语义为主，基于 [[Qwen-Audio]]）和 Qwen-TTS-Tokenizer-12Hz（16 层 [[RVQ]] 多码本，语义-声学联合建模，基于 [[Mimi]] 架构），分别针对长序列稳定性和低延迟流式场景
2. **大规模工业级训练流水线**: 500 万小时多语种（10 语种）语音数据，三阶段预训练 + 三阶段后训练（[[Speech DPO]] + GSPO + 说话人微调），将 TTS 后训练推到工业 RLHF 级别
3. **可控语音设计**: 继承 Qwen3 文本理解能力，通过概率激活 "thinking pattern" 实现自然语言描述驱动的声音创建，在 InstructTTSEval 上超越 GPT-4o-mini-tts

---

## 问题背景

### 要解决的问题
构建一个统一的多语种、可控、鲁棒、支持流式的 TTS 系统，同时支持声音克隆、声音设计、细粒度风格控制等多种任务。

### 现有方法的局限
- 纯 [[Semantic Token]] 方案（如 [[CosyVoice]]）语义保真好但表现力受限，细粒度声学控制能力不足
- 纯 [[Acoustic Token]] 方案会注入过多底层声学细节，导致 LM 建模困难、上下文窗口浪费
- 现有零样本 TTS 模型在多语种、长文本、可控生成等维度难以同时做好
- 流式 TTS 的首包延迟与生成质量之间存在 trade-off

### 本文的动机
通过设计两套互补的 tokenizer（25Hz 语义主导 + 12Hz 语义-声学联合），各自搭配不同的 LM 解码策略，在同一框架下覆盖不同部署场景的需求。

---

## 方法详解

### 模型架构

Qwen3-TTS 采用 **自回归 LM** 架构，核心组件包括：

- **Backbone**: [[Qwen3]] 系列 LM（0.6B / 1.7B 参数）
- **文本编码**: Qwen 标准 tokenizer
- **语音 tokenizer**: Qwen-TTS-Tokenizer-25Hz 或 Qwen-TTS-Tokenizer-12Hz（二选一）
- **说话人编码**: 可学习的 Speaker Encoder，与 backbone 联合训练
- **解码**: 25Hz 版本用 block-wise [[DiT]] + [[BigVGAN]]；12Hz 版本用因果 ConvNet 直接出波形
- **输出格式**: 文本 token 与语音 token 沿 channel 维度拼接的双轨表示

### 核心模块

#### 模块 1: Qwen-TTS-Tokenizer-25Hz（单码本语义 tokenizer）

**设计动机**: 利用 [[Qwen-Audio]]（Qwen2-Audio）的强语义理解能力，在其中间层注入 [[Vector Quantization]] 模块，使单个码本同时编码语义和声学信息。

**具体实现**:
- **Stage 1**: 在 Qwen2-Audio 基础上继续预训练 ASR 任务，在中间位置插入 resampling 层 + VQ 层，输出 25 Hz 单码本 token（码本大小 32768）
- **Stage 2**: 加入卷积 mel 谱图解码器进行重建微调，将声学信息注入 token 表示
- **Detokenizer**: 使用滑动窗口 block attention 的 [[Diffusion Transformer]]，通过 [[Flow Matching]] 将 token 序列映射为 mel 谱图，再由改进版 [[BigVGAN]] 重建波形
- **流式策略**: 每 block 8 个 token，receptive field = 当前块 + 3 块回看 + 1 块前看；首包需等 16 token（640 ms 内容 → 实际约 150 ms 首包延迟）

#### 模块 2: Qwen-TTS-Tokenizer-12Hz（多码本语义-声学联合 tokenizer）

**设计动机**: 基于 [[Mimi]] 的语义-声学解耦量化策略，进一步压低帧率至 12.5 Hz，实现极低码率和超低首包延迟。

**具体实现**:
- 第 1 层码本编码语义信息，以 [[WavLM]] 特征为蒸馏目标
- 第 2-16 层使用 15 层 [[RVQ]] 编码声学细节（韵律、音色等）
- 码本大小 2048，帧率 12.5 Hz（80 ms/token）
- 训练使用 GAN 框架 + 多尺度 mel 重建损失
- 全因果编解码器，无前看，支持即时流式出波形
- **首包延迟**: 97 ms（0.6B）/ 101 ms（1.7B），业界最低

#### 模块 3: 双轨 LM 解码

**25Hz 版本**:
- 单层 token 序列，backbone 融合文本特征与前序语音 token，通过线性头预测当前 token
- Chunk-wise [[DiT]] + [[BigVGAN]] 完成流式波形重建

**12Hz 版本**:
- Backbone 从聚合的多层码本特征中预测第 0 层（语义）码本
- [[Multi-Token Prediction]] (MTP) 模块并行生成剩余 15 层残差码本
- 因果 ConvNet decoder 即时出波形，无需额外 vocoder

### 训练流程

**数据格式**: 所有数据统一为 ChatML 格式

**预训练（3 阶段）**:
1. **S1 通用阶段**: 500 万+ 小时多语种语音，建立单调 text→speech 映射
2. **S2 高质量阶段**: 用分层筛选的高质量数据继续预训练，减少幻觉
3. **S3 长上下文阶段**: 最大 token 长度从 8192 扩展到 32768，上采样长语音样本

**后训练（3 阶段）**:
1. **[[Speech DPO]]**: 使用人类反馈偏好对进行直接偏好优化
2. **GSPO**: 基于规则奖励的 Group Sampling Policy Optimization，全面提升能力
3. **说话人微调**: 轻量级特定说话人微调，提升自然度和表达力

### 特色功能

- **声音克隆**: 两种方式 —（i）speaker embedding 提取；（ii）text-speech pair in-context learning
- **声音设计**: 利用 Qwen3 的文本理解能力，训练时概率激活 "thinking pattern"，提升自然语言描述→声音属性的指令遵循
- **预定义声音细粒度控制**: 对预设声音进行风格/情感/语速等属性调节

---

## 关键图表

### Figure 1: Overview / 系统概览

![Figure 1](https://arxiv.org/html/2601.15621v1/figures/Qwen3TTS-pr-0121.png)

**说明**: Qwen3-TTS 的功能全景图。展示多语种支持（10 语种）、声音克隆（3 秒参考音频）、声音创建（自然语言描述）、细粒度控制以及复杂文本处理能力。

### Figure 2: Tokenizer Architecture / Tokenizer 架构

![Figure 2](https://arxiv.org/html/2601.15621v1/figures/Tokenizer.png)

**说明**: 两套 Qwen-TTS-Tokenizer 的架构对比。左侧为 25Hz 单码本方案（基于 [[Qwen-Audio]] + VQ + [[DiT]] detokenizer），右侧为 12Hz 多码本方案（语义+声学 [[RVQ]] + 因果 ConvNet decoder）。

### Figure 3: Model Architecture / 模型架构

![Figure 3](https://arxiv.org/html/2601.15621v1/figures/Qwen3TTS_1104.png)

**说明**: Qwen3-TTS 完整模型架构。虚线表示可选组件。展示文本 token 与语音 token 的双轨融合、speaker encoder 条件注入、以及 25Hz/12Hz 两种解码路径。

### Table 1: Qwen3-TTS 模型家族概览

| 模型名称 | 流式 | 多语种 | 克隆 | 指令遵循 |
|----------|:----:|:------:|:----:|:--------:|
| Qwen3-TTS-12Hz-1.7B-Base | ✓ | ✓ | ✓ | |
| Qwen3-TTS-12Hz-1.7B-VoiceDesign | ✓ | ✓ | | ✓ |
| Qwen3-TTS-12Hz-1.7B-CustomVoice | ✓ | ✓ | ✓ | |
| Qwen3-TTS-12Hz-0.6B-Base | ✓ | ✓ | ✓ | |
| Qwen3-TTS-12Hz-0.6B-CustomVoice | ✓ | ✓ | | |
| Qwen3-TTS-25Hz-1.7B-Base | ✓ | ✓ | ✓ | |
| Qwen3-TTS-25Hz-1.7B-VoiceEditing | ✓ | ✓ | ✓ | ✓ |
| Qwen3-TTS-25Hz-1.7B-CustomVoice | ✓ | ✓ | ✓ | |
| Qwen3-TTS-25Hz-0.6B-Base | ✓ | ✓ | ✓ | |
| Qwen3-TTS-25Hz-0.6B-CustomVoice | ✓ | ✓ | | |

**说明**: 共 10 个模型变体，覆盖两种 tokenizer、两种参数规模、三种功能定位（Base / VoiceDesign / CustomVoice）。

### Table 2: 流式效率（不同并发数）

| 模型 | 并发 | LM TTFP | Tokenizer TPP | 首包延迟 | LM TPP | RTF |
|------|:----:|--------:|--------------:|---------:|-------:|----:|
| 25Hz-1.7B | 1 | 125 ms | 25 ms | 150 ms | 56 ms | 0.253 |
| 25Hz-1.7B | 3 | 222 ms | 62 ms | 284 ms | 64 ms | 0.394 |
| 25Hz-1.7B | 6 | 376 ms | 147 ms | 523 ms | 85 ms | 0.725 |
| 25Hz-0.6B | 1 | 113 ms | 25 ms | 138 ms | 50 ms | 0.234 |
| 25Hz-0.6B | 3 | 198 ms | 62 ms | 260 ms | 59 ms | 0.378 |
| 25Hz-0.6B | 6 | 334 ms | 147 ms | 481 ms | 80 ms | 0.709 |
| 12Hz-1.7B | 1 | 97 ms | 4 ms | 101 ms | 21 ms | 0.313 |
| 12Hz-1.7B | 3 | 190 ms | 5 ms | 195 ms | 24 ms | 0.363 |
| 12Hz-1.7B | 6 | 328 ms | 5 ms | 333 ms | 32 ms | 0.463 |
| 12Hz-0.6B | 1 | 93 ms | 4 ms | **97 ms** | 19 ms | 0.288 |
| 12Hz-0.6B | 3 | 174 ms | 5 ms | 179 ms | 22 ms | 0.338 |
| 12Hz-0.6B | 6 | 294 ms | 5 ms | 299 ms | 30 ms | 0.434 |

**说明**: 12Hz tokenizer 因无需 DiT 解码，tokenizer decode TPP 仅 4-5 ms（vs 25Hz 的 25-147 ms），首包延迟显著更低。12Hz-0.6B 单并发首包延迟仅 97 ms，是目前已知最低的流式 TTS 首包延迟之一。[[RTF]] 均 < 1，满足实时要求。

### Table 3: 25Hz Tokenizer ASR 对比（[[WER]] ↓）

| 模型 | 码本大小 | FPS | C.V. EN | C.V. CN | Fleurs EN | Fleurs CN |
|------|:--------:|:---:|--------:|--------:|----------:|----------:|
| S3 Tokenizer(VQ) | 4096 | 50 | 12.06 | 15.38 | - | - |
| S3 Tokenizer(VQ) | 4096 | 25 | 11.56 | 18.26 | 7.65 | 5.03 |
| S3 Tokenizer(FSQ) | 6561 | 25 | 10.67 | 7.29 | 6.58 | 4.43 |
| **Qwen-TTS-Tokenizer-25Hz (S1)** | 32768 | 25 | **7.51** | 10.73 | **3.07** | **4.23** |
| Qwen-TTS-Tokenizer-25Hz (S2) | 32768 | 25 | 10.40 | 14.99 | 4.14 | 4.67 |

**说明**: S1 阶段的 25Hz tokenizer 在 ASR 任务上显著优于 S3 Tokenizer。S2 阶段因注入声学信息导致 ASR 性能略有下降，但这是语义保真与声学丰富度之间的有意 trade-off。

### Table 4: 12Hz Tokenizer 重建质量对比

| 模型 | NQ | 码本 | FPS | [[PESQ]]_WB | [[PESQ]]_NB | [[STOI]] | [[UTMOS]] | SIM |
|------|:--:|:----:|:---:|:----------:|:----------:|:-------:|:-------:|:---:|
| [[SpeechTokenizer]] | 8 | 1024 | 50 | 2.60 | 3.05 | 0.92 | 3.90 | 0.85 |
| X-Codec | 2 | 1024 | 50 | 2.68 | 3.27 | 0.86 | 4.11 | 0.84 |
| X-Codec 2 | 1 | 65536 | 50 | 2.43 | 3.04 | 0.92 | 4.13 | 0.82 |
| XY-Tokenizer | 8 | 1024 | 12.5 | 2.41 | 3.00 | 0.91 | 3.98 | 0.83 |
| [[Mimi]] | 16 | 2048 | 12.5 | 2.88 | 3.42 | 0.94 | 3.87 | 0.87 |
| [[FireRedTTS-2]] | 16 | 2048 | 12.5 | 2.73 | 3.28 | 0.94 | 3.88 | 0.87 |
| **Qwen-TTS-Tokenizer-12Hz** | **16** | **2048** | **12.5** | **3.21** | **3.68** | **0.96** | **4.16** | **0.95** |

**说明**: 在 [[LibriSpeech]] test-clean（2620 条）上评测，Qwen-TTS-Tokenizer-12Hz 在所有关键指标上全面 SOTA。尤其 SIM 达 0.95（vs [[Mimi]] 0.87），说话人保真度大幅领先。

### Table 5: 零样本语音生成（[[Seed-TTS-eval]]，[[WER]] ↓）

| 模型 | test-zh | test-en |
|------|--------:|--------:|
| [[Seed-TTS]] | 1.12 | 2.25 |
| [[MaskGCT]] | 2.27 | 2.62 |
| E2 TTS | 1.97 | 2.19 |
| [[F5-TTS]] | 1.56 | 1.83 |
| [[Spark-TTS]] | 1.20 | 1.98 |
| Llasa-8B | 1.59 | 2.97 |
| KALL-E | 0.96 | 1.94 |
| [[FireRedTTS-2]] | 1.14 | 1.95 |
| [[CosyVoice 3]] | 0.71 | 1.45 |
| MiniMax-Speech | 0.83 | 1.65 |
| Qwen3-TTS-25Hz-0.6B | 1.18 | 1.64 |
| Qwen3-TTS-25Hz-1.7B | 1.10 | 1.49 |
| Qwen3-TTS-12Hz-0.6B | 0.92 | 1.32 |
| **Qwen3-TTS-12Hz-1.7B** | **0.77** | **1.24** |

**说明**: 12Hz-1.7B 在 test-en 上达 1.24 WER，SOTA。test-zh 上 0.77 仅次于 [[CosyVoice 3]] 的 0.71，差距极小。12Hz 系列一致优于 25Hz，说明多码本的语义-声学联合建模对内容准确度更有利。

### Table 6: 多语种语音生成（WER ↓ / SIM ↑）

**内容一致性 WER ↓**:

| 语种 | 25Hz-0.6B | 25Hz-1.7B | 12Hz-0.6B | 12Hz-1.7B | MiniMax | ElevenLabs |
|------|----------:|----------:|----------:|----------:|--------:|-----------:|
| 中文 | 1.108 | 0.777 | 1.145 | **0.928** | 2.252 | 16.026 |
| 英文 | 1.048 | 1.014 | **0.836** | 0.934 | 2.164 | 2.339 |
| 德语 | 1.501 | 0.960 | 1.089 | 1.235 | 1.906 | **0.572** |
| 意大利语 | 1.169 | 1.105 | 1.534 | **0.948** | 1.543 | 1.743 |
| 葡萄牙语 | 2.046 | 1.778 | 2.254 | 1.526 | 1.877 | **1.331** |
| 西班牙语 | 2.031 | 1.491 | 1.491 | 1.126 | **1.029** | 1.084 |
| 日语 | 4.189 | 5.121 | 6.404 | 3.823 | **3.519** | 10.646 |
| 韩语 | 2.458 | 2.695 | **1.741** | 1.755 | 1.747 | 1.865 |
| 法语 | 2.852 | 2.631 | 2.931 | **2.858** | 4.099 | 5.216 |
| 俄语 | 5.957 | 4.535 | 4.458 | **3.212** | 4.281 | 3.878 |

**说话人相似度 SIM ↑**:

| 语种 | 25Hz-0.6B | 25Hz-1.7B | 12Hz-0.6B | 12Hz-1.7B | MiniMax | ElevenLabs |
|------|:---------:|:---------:|:---------:|:---------:|:-------:|:----------:|
| 中文 | 0.797 | 0.796 | **0.811** | 0.799 | 0.780 | 0.677 |
| 英文 | 0.811 | 0.815 | **0.829** | 0.775 | 0.756 | 0.613 |
| 德语 | 0.749 | 0.737 | 0.769 | **0.775** | 0.733 | 0.614 |
| 意大利语 | 0.722 | 0.718 | 0.792 | **0.817** | 0.699 | 0.579 |
| 葡萄牙语 | 0.790 | 0.783 | 0.794 | **0.817** | 0.805 | 0.711 |
| 西班牙语 | 0.732 | 0.731 | 0.812 | **0.814** | 0.762 | 0.615 |
| 日语 | **0.810** | 0.807 | 0.798 | 0.788 | 0.776 | 0.738 |
| 韩语 | **0.824** | 0.814 | 0.812 | 0.799 | 0.776 | 0.700 |
| 法语 | 0.698 | 0.703 | 0.700 | **0.714** | 0.628 | 0.535 |
| 俄语 | 0.734 | 0.744 | 0.781 | **0.792** | 0.761 | 0.676 |

**说明**: Qwen3-TTS 在 10 个语种的说话人相似度上全面领先 MiniMax-Speech 和 ElevenLabs。WER 方面在中/英/法/俄等语种上占优，但日/西/葡语上 ElevenLabs 或 MiniMax 更强，说明多语种能力仍有提升空间。

### Table 7: 跨语言生成（CV3-Eval，Mixed Error Rate ↓）

| 任务 | 25Hz-1.7B | 12Hz-1.7B | [[CosyVoice 3]] | CosyVoice2 |
|------|----------:|----------:|---------:|----------:|
| en→zh | 5.66 | **4.77** | 5.09 | 13.5 |
| ja→zh | 3.92 | 3.43 | **3.05** | 48.1 |
| ko→zh | 1.14 | 1.08 | **1.06** | 7.70 |
| zh→en | 2.91 | **2.77** | 2.98 | 6.47 |
| ja→en | 3.95 | **3.04** | 4.20 | 17.1 |
| ko→en | 3.48 | **3.09** | 4.19 | 11.2 |
| zh→ja | 9.29 | 8.40 | **7.08** | 13.1 |
| en→ja | 7.74 | 7.21 | **6.80** | 14.9 |
| ko→ja | 4.17 | **3.67** | 3.93 | 5.86 |
| zh→ko | 8.12 | **4.82** | 14.4 | 24.8 |
| en→ko | 6.83 | **5.14** | 5.87 | 21.9 |
| ja→ko | 6.86 | **5.59** | 7.92 | 21.5 |

**说明**: 12Hz-1.7B 在 12 个跨语言任务中赢了 8 个（vs [[CosyVoice 3]] 赢 4 个）。zh→ko 任务上 Qwen3-TTS 将错误率从 CosyVoice3 的 14.4 降至 4.82（-66%），优势极为显著。

### Table 8: 可控语音生成（InstructTTSEval）

**目标说话人控制**:

| 模型 | ZH APS↑ | ZH DSD↑ | ZH RP↑ | EN APS↑ | EN DSD↑ | EN RP↑ |
|------|--------:|--------:|-------:|--------:|--------:|-------:|
| Gemini-flash | **88.2** | **90.9** | **77.3** | **92.3** | **93.8** | **80.1** |
| Gemini-pro | 89.0 | 90.1 | 75.5 | 87.6 | 86.0 | 67.2 |
| Qwen3TTS-25Hz-1.7B-CV | 83.1 | 75.0 | 63.0 | 79.0 | 82.8 | **69.3** |
| Qwen3TTS-12Hz-1.7B-CV | 83.0 | 77.8 | 61.2 | 77.3 | 77.1 | 63.7 |
| GPT-4o-mini-tts | 54.9 | 52.3 | 46.0 | 76.4 | 74.3 | 54.8 |

**声音设计**:

| 模型 | ZH APS↑ | ZH DSD↑ | ZH RP↑ | EN APS↑ | EN DSD↑ | EN RP↑ |
|------|--------:|--------:|-------:|--------:|--------:|-------:|
| **Qwen3TTS-12Hz-1.7B-VD** | **85.2** | **81.1** | **65.1** | **82.9** | **82.4** | **68.4** |
| Mimo-Audio-7B | 75.7 | 74.3 | 61.5 | 80.6 | 77.6 | 59.5 |
| VoiceSculptor | 75.7 | 64.7 | 61.5 | - | - | - |
| Hume | - | - | - | 83.0 | 75.3 | 54.3 |
| VoxInstruct | 47.5 | 52.3 | 42.6 | 54.9 | 57.0 | 39.3 |
| Parler-tts-mini | - | - | - | 63.4 | 48.7 | 28.6 |
| Parler-tts-large | - | - | - | 60.0 | 45.9 | 31.2 |
| PromptTTS | - | - | - | 64.3 | 47.2 | 31.4 |
| PromptStyle | - | - | - | 57.4 | 46.4 | 30.9 |

**说明**: 目标说话人控制方面，Qwen3-TTS 在中文 APS 上超越 GPT-4o-mini-tts 达 +28.1 分（83.1 vs 54.9），但仍逊于 Gemini。声音设计方面，Qwen3-TTS-12Hz-1.7B-VD 在中/英双语上全面 SOTA，超越 Mimo-Audio-7B 和 VoiceSculptor。

### Table 9: 目标说话人多语种生成（WER ↓）

| 语种 | 25Hz-0.6B-CV | 25Hz-1.7B-CV | 12Hz-0.6B-CV | 12Hz-1.7B-CV | GPT-4o |
|------|:-----------:|:-----------:|:-----------:|:-----------:|:------:|
| 中文 | 0.874 | **0.708** | 0.944 | 0.903 | 3.519 |
| 英文 | 1.332 | 0.936 | 1.188 | **0.899** | 2.197 |
| 德语 | 0.990 | **0.634** | 2.722 | 1.057 | 1.161 |
| 意大利语 | 1.861 | 1.271 | 2.545 | 1.362 | **1.194** |
| 葡萄牙语 | 1.728 | 1.854 | 3.219 | 2.681 | **1.504** |
| 西班牙语 | 1.309 | 1.284 | **1.154** | 1.330 | 4.000 |
| 日语 | **3.875** | 4.518 | 6.877 | 4.924 | 5.001 |
| 韩语 | 2.202 | 2.274 | 3.053 | **1.741** | 2.763 |
| 法语 | 3.865 | **3.080** | 3.841 | 3.781 | 3.605 |
| 俄语 | 6.529 | **4.444** | 5.809 | 4.734 | 5.250 |

**说明**: Qwen3-TTS 在 10 语种中 7 个优于 GPT-4o Audio Preview，尽管仅在单语数据上做说话人微调，展现出强跨语言泛化能力。

### Table 10: 长文本语音生成（WER ↓）

| 模型 | long-zh | long-en |
|------|--------:|--------:|
| [[HiggsAudio-v2]] (chunk) | 5.505 | 6.917 |
| [[VibeVoice]] | 22.619 | 1.780 |
| [[VoxCPM]] | 4.835 | 7.474 |
| **Qwen3-TTS-25Hz-1.7B-CV** | **1.517** | **1.225** |
| Qwen3-TTS-12Hz-1.7B-CV | 2.356 | 2.812 |

**说明**: 在 200-2000 字（>10 分钟语音）的长文本生成上，25Hz 版本大幅优于 12Hz（zh: 1.517 vs 2.356，en: 1.225 vs 2.812），证实语义主导的 token 在长序列稳定性上更优。这与短文本任务中 12Hz 占优形成有趣对比。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 内部多语种语音数据 | 500 万+ 小时 | 10 语种 | 预训练 |
| [[Seed-TTS-eval]] | test-zh + test-en | 零样本 TTS 标准评测 | 测试 |
| [[Common Voice]] | 多语种 | CV3-Eval 跨语言子集 | 测试 |
| [[LibriSpeech]] test-clean | 2620 条 | 英文 ASR / Codec 评测 | Tokenizer 测试 |
| InstructTTSEval | 中英双语 | 可控 TTS 评测 | 测试 |
| 内部长文本集 | 100 条×2 语种 | 200-2000 字 | 测试 |

### 实现细节

- **Backbone**: Qwen3 LM 系列（0.6B / 1.7B）
- **训练数据**: 500 万+ 小时多语种语音
- **预训练 3 阶段**: S1 通用 → S2 高质量 → S3 长上下文（8192→32768 token）
- **后训练 3 阶段**: DPO → GSPO → 说话人微调
- **推理硬件**: A100（效率测试基准）
- **开源协议**: Apache 2.0

### 关键发现

- **12Hz vs 25Hz trade-off**: 12Hz 在短文本零样本克隆和流式延迟上占优，25Hz 在长文本稳定性上占优
- **Scaling**: 0.6B → 1.7B 一致带来 WER 和 SIM 提升
- **后训练的价值**: DPO + GSPO 显著提升指令遵循和可控性

---

## 批判性思考

### 优点
1. **系统工程完成度极高**: 双 tokenizer 设计精准覆盖不同场景（低延迟 vs 长序列），10 个模型变体形成完整产品矩阵
2. **训练规模业界领先**: 500 万小时数据 + 三阶段预训练 + 三阶段后训练（含 DPO/GSPO），是目前公开报告中最大规模的 TTS 训练
3. **评测全面且与强基线对比**: 覆盖零样本/多语种/跨语言/可控/长文本 5 大维度，对比 [[CosyVoice 3]]、GPT-4o、MiniMax、ElevenLabs 等工业级系统
4. **开源承诺**: tokenizer + 模型全部 Apache 2.0 开源，工程价值极大

### 局限性
1. **无主观评测**: 全文没有 [[MOS]] / [[CMOS]] 主观评测，仅依赖 [[WER]] + SIM 客观指标，无法判断合成语音的自然度和听感质量
2. **训练数据不可复现**: 500 万小时内部数据完全不公开，对学术界而言无法复现训练
3. **12Hz vs 25Hz 选择缺乏统一指导**: 两套 tokenizer 在不同任务上各有优劣，论文未给出明确的场景选择建议
4. **日语/西语等非中英语种仍有差距**: 多语种 WER 在某些语种上不如 MiniMax 或 ElevenLabs，10 语种覆盖可能仍有不足
5. **无消融实验**: 后训练三阶段（DPO / GSPO / 说话人微调）各自的独立贡献缺乏消融

### 潜在改进方向
1. 统一两套 tokenizer 为一个自适应方案，根据场景自动选择帧率和码本配置
2. 扩展到更多语种（目前仅 10 语种，vs MMS 等覆盖 1000+ 语种）
3. 加入主观评测和用户偏好评测（MOS / A-B test）
4. 发布训练数据清洗 pipeline 或使用公开数据的复现方案

### 可复现性评估
- [x] 代码开源（GitHub）
- [x] 预训练模型（Apache 2.0）
- [ ] 训练细节完整（数据不公开）
- [ ] 数据集可获取（内部 500 万小时）

---

## 关联笔记

### 基于
- [[Qwen-Audio]]: 25Hz tokenizer 基于 Qwen2-Audio 构建
- [[Mimi]]: 12Hz tokenizer 参考 Mimi 的语义-声学解耦量化策略
- [[SpeechTokenizer]]: 12Hz 设计灵感之一，第一层语义 + 后续层声学
- [[WavLM]]: 12Hz tokenizer 第一层语义码本的蒸馏目标

### 对比
- [[CosyVoice 3]]: 同为阿里系，跨语言和零样本最主要对手
- [[F5-TTS]]: 纯 [[Flow Matching]] 方案的代表
- [[MaskGCT]]: NAR mask-and-predict 方案
- [[Seed-TTS]]: 字节跳动零样本 TTS 基线

### 方法相关
- [[RVQ]]: 12Hz tokenizer 的核心量化方法
- [[Flow Matching]]: 25Hz detokenizer 的核心生成方法
- [[DiT]]: 25Hz 版本的 mel 谱图生成模块
- [[BigVGAN]]: 25Hz 版本的流式波形重建声码器
- [[Multi-Token Prediction]]: 12Hz 版本并行解码残差码本
- [[Speech DPO]]: 后训练偏好优化
- [[Zero-shot TTS]]: 核心任务范式

### 数据相关
- [[Seed-TTS-eval]]: 主要零样本评测集
- [[LibriSpeech]]: Tokenizer 重建质量评测集
- [[Common Voice]]: 多语种评测集

---

## 速查卡片

> [!summary] Qwen3-TTS Technical Report
> - **核心**: Qwen 系列首个 TTS，双轨 tokenizer（25Hz 单码本 / 12Hz 多码本）+ LM 架构
> - **方法**: 500 万小时数据三阶段预训练 + DPO/GSPO 后训练，支持克隆/设计/控制
> - **结果**: Seed-TTS-eval WER SOTA（1.24 en），首包 97 ms，10 语种覆盖
> - **代码**: [github.com/QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)

---

*笔记创建时间: 2026-05-25*
