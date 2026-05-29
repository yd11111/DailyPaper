---
title: "Qwen3-TTS Technical Report"
method_name: "Qwen3-TTS"
authors: [Hangrui Hu, Xinfa Zhu, Ting He, Dake Guo, Bin Zhang, Xiong Wang, Zhifang Guo, Ziyue Jiang, Hongkun Hao, Zishan Guo, Xinyu Zhang, Pei Zhang, Baosong Yang, Jin Xu, Jingren Zhou, Junyang Lin]
year: 2026
venue: arXiv
arxiv_id: "2601.15621"
tags: [zero-shot-tts, streaming-tts, multilingual-tts, voice-cloning, speech-tokenizer, controllable-tts, voice-design]

# === 论文核心技术元数据（三层 verify，2026-05-26 dogfood 完整核对，详见 SKILL.md §2.5 + no-hallucination-rules.md §11）===
# Layer 1: 论文原文（arXiv HTML 2601.15621v1）= §X / Tab.X / Fig.X
# Layer 2: GitHub 源码（github.com/QwenLM/Qwen3-TTS）= [GitHub: <path>]
# Layer 3: 第三方实现 / 复现报告 = 不需要
lm_init: "paper claims warm-start: 'Qwen3-TTS leverages the Qwen3 LM family' [§3.1] / 'built upon the Qwen3 text model foundation' [§3.3]; 但 GitHub 开源代码显示 talker 是 custom standalone transformer：不继承 Qwen3PreTrainedModel/Qwen3ForCausalLM，hidden_size=1024 不匹配任何已知 Qwen3 LLM 尺寸 (Qwen3-1.7B=2048)，_init_weights 用 standard normal init [GitHub: qwen_tts/core/models/{modeling_qwen3_tts.py, configuration_qwen3_tts.py}]；text_hidden_size=2048 暗示可能外部输入 Qwen3-1.7B text features；**pre-training init 代码未开源，literal Qwen3 LLM weight warm-start 实未 verify**"
training_loss: "speech-token-only CE on text channel + 多码本 intra-speech loss balancing (NOT text-speech balancing); finetuning 公式 loss = outputs.loss + 0.3 * sub_talker_loss [GitHub: finetuning/sft_12hz.py]; outputs.loss 是 codec_0 (第 0 层码本) 的 CE，sub_talker_loss 是残差 codebook 1-15 的 CE [GitHub: qwen_tts/core/models/modeling_qwen3_tts.py forward()]; text positions 全 -100 mask (codec_0_labels = torch.full((b,t), -100)) [GitHub: finetuning/dataset.py]; **无 text loss、无 KL 约束、无 distillation loss**; 论文 §3.2 仅定性描述 pre-training 三阶段，无显式公式 [§3.2]"
tokenizer_arch: "text+speech SEPARATE (NOT unified-token-space, NOT interleaved): 'Text is processed using the standard Qwen tokenizer, while speech is encoded using the Qwen-TTS-Tokenizer' [§3.1]; dual-channel input_ids shape (b, t, 2)，channel-0=text 用 Qwen text vocab + tts_bos/eos/pad，channel-1=codec 用 vocab_size=3072 + codec_bos/eos/pad [GitHub: finetuning/dataset.py + qwen_tts/core/models/configuration_qwen3_tts.py]"
multitask: 'finetune 阶段是的：codec_0 主 loss + 残差 codebook sub_talker 0.3 权重 (intra-speech 多码本) [GitHub: finetuning/sft_12hz.py]；pre-training 阶段多任务范围未开源；论文 §3.2 仅说明 ChatML 统一格式 + 三阶段预训练但未列具体多任务 loss 项'
training_data: "5M+ hours speech, 10 languages (zh/en/de/it/pt/es/ja/ko/fr/ru); 数据来源、清洗规则、配对方式 paper/code 均未公开 [§abstract / §3.2]"
post_training: "三阶段 [§3.2]: (1) Speech-DPO 人类偏好对齐 (Rafailov 2023 公式); (2) GSPO group sampling policy optimization with rule-based reward; (3) speaker fine-tuning 单说话人轻量微调; **后训练代码未开源，仅 finetuning/sft_12hz.py 是 SFT 阶段**"
codec_detail: "Tokenizer-12Hz: 16-layer RVQ, 码本 2048, 12.5 Hz/80ms, layer-1 语义蒸馏自 WavLM, layers 2-16 声学, GAN+多尺度 mel 重建 [§3.1, Tab.4]; Tokenizer-25Hz: 1-layer VQ, 码本 32768, 25 Hz/40ms, 基于 Qwen2-Audio 中间层 + DiT detokenizer + BigVGAN, 滑动窗口 block attention [§3.1, Tab.3]"

# === 知识地图联动（R6 强制要求，2026-05-26 dogfood verify 修订）===
domain: TTS
subdomain: llm-native-tts
routes: [speech-llm-tts, codec-lm-tts, controllable-tts, instruction-tts, streaming-tts, voice-cloning]
problems: [zero-shot-cloning, prosody-control, multilinguality, data-scale, codec-design, instruction-following, latency, long-form-stability, evaluation]
representations: [acoustic-token, semantic-token, mixed-token]
# === 元数据修订记录 ===
# 2026-05-26 (dogfood verify) 关键修正：
#   - tokenizer_arch: 已 verify SEPARATE [§3.1 + finetuning/dataset.py]，确认 NOT unified-token-space
#   - training_loss: 修正之前"无 loss balancing"误判 — 实际有 0.3 sub_talker 多码本 loss balancing (但仍是 intra-speech，不是 text-speech)
#   - lm_init: paper 暗示 warm-start，但 GitHub 显示标准 init + 定制小型架构，literal warm-start 实未 verify (pre-training 代码未开源)
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-表示层地图]]"
  - "[[TTS-代表模型谱系]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-评测体系]]"
  - "[[TTS-趋势判断]]"
  - "[[TTS-SpeechLM-Dialogue关系]]"
  - "[[SpeechLM-领域总览]]"
related_surveys:
  - "[[ControllableTTS-Survey]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-26

# === 回流状态 ===
map_backfilled: true
backfilled_at: 2026-05-26  # 8 个下游 cell 修订完成（路线图 ×2 + 谱系 ×4 + 趋势判断 ×2），详见 [[待回填地图]] §Qwen3-TTS 二次 verify 条目

# === 资源本地化路径（cache_paper_resources.py 自动填充，2026-05-26 smoke test 第一篇 demo）===
pdf_local: "~/DailyPaper/.cache/papers/2601.15621/paper.pdf"           # 8 MB，arXiv PDF 原文
html_local: "~/DailyPaper/.cache/papers/2601.15621/paper.html"         # 225 KB，arXiv HTML 备份（离线 grep / 章节定位）
figures_dir: "_resources/2601.15621/figures"                            # vault 内相对路径（Obsidian wikilink 用 `![[_resources/2601.15621/figures/fig-000.png]]`）
github_local: "~/DailyPaper/.cache/papers/2601.15621/github/QwenLM_Qwen3-TTS"  # 完整 clone，关键文件 sft_12hz.py / modeling_qwen3_tts.py 已就位
cached_at: 2026-05-26

# === 通用元数据 ===
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

## 关键公式

> [!note] 本论文为工程导向的 Tech Report，核心训练目标（DPO/GSPO/Flow Matching/GAN）均引用外部文献而未在正文显式给出。以下整理论文中可推导的关键量化关系和引用的核心损失函数。

### 码率计算

两套 tokenizer 的信息码率可从配置参数直接推导：

**12Hz 多码本**（16 层 [[RVQ]]，码本大小 2048，帧率 12.5 Hz）：

$$\text{Bitrate}_{12\text{Hz}} = f_s \times N_q \times \log_2 C = 12.5 \times 16 \times \log_2(2048) = 12.5 \times 16 \times 11 = 2200 \;\text{bps}$$

**25Hz 单码本**（1 层 VQ，码本大小 32768，帧率 25 Hz）：

$$\text{Bitrate}_{25\text{Hz}} = f_s \times 1 \times \log_2 C = 25 \times \log_2(32768) = 25 \times 15 = 375 \;\text{bps}$$

其中 $f_s$ 为帧率，$N_q$ 为码本层数，$C$ 为码本大小。25Hz 方案码率极低（375 bps），但依赖下游 [[DiT]] + [[BigVGAN]] 补充声学细节。

### 首包延迟分解

$$\text{First\text{-}Packet Latency} = T_{\text{LM-TTFP}} + T_{\text{Tokenizer-Decode}}$$

对于 12Hz-0.6B：$93\,\text{ms} + 4\,\text{ms} = 97\,\text{ms}$；对于 25Hz-1.7B：$125\,\text{ms} + 25\,\text{ms} = 150\,\text{ms}$。12Hz 方案因因果 ConvNet 解码器无需前看，tokenizer 解码仅需 4 ms。

### Token 时间映射

$$\Delta t = \frac{1}{f_s}$$

25Hz：$\Delta t_{25} = 40\;\text{ms/token}$；12Hz：$\Delta t_{12} = 80\;\text{ms/token}$。

25Hz 流式出包策略：chunk 大小 $C=8$ token，接收域 = 当前块 + 3 块回看 + 1 块前看，首包需等待 $2C = 16$ token（即 $16 \times 40 = 640\;\text{ms}$ 语音内容）。

### 引用的核心损失函数

**[[Speech DPO]] 损失**（后训练第 1 阶段，引用 Rafailov et al. 2023）：

$$\mathcal{L}_{\text{DPO}} = -\mathbb{E}_{(x, y_w, y_l)} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(y_w | x)}{\pi_{\text{ref}}(y_w | x)} - \beta \log \frac{\pi_\theta(y_l | x)}{\pi_{\text{ref}}(y_l | x)} \right) \right]$$

其中 $y_w, y_l$ 分别为人类偏好的优选和劣选语音，$\pi_\theta$ 为当前策略，$\pi_{\text{ref}}$ 为参考策略，$\beta$ 为温度系数。

**[[Flow Matching]] 损失**（25Hz detokenizer [[DiT]] 训练，引用 Lipman et al. 2023）：

$$\mathcal{L}_{\text{FM}} = \mathbb{E}_{t, x_0, x_1} \left\| v_\theta(x_t, t) - (x_1 - x_0) \right\|^2$$

其中 $x_t = (1-t)x_0 + t x_1$ 为插值路径，$v_\theta$ 为模型预测的速度场，$x_0 \sim \mathcal{N}(0, I)$，$x_1$ 为目标 mel 谱图。

**12Hz Tokenizer 训练损失**（GAN 框架 + 重建损失）：

$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{GAN}} + \lambda_{\text{mel}} \mathcal{L}_{\text{mel}} + \lambda_{\text{sem}} \mathcal{L}_{\text{sem}}$$

其中 $\mathcal{L}_{\text{mel}}$ 为多尺度 mel 谱图重建损失，$\mathcal{L}_{\text{sem}}$ 为以 [[WavLM]] 为教师的语义蒸馏损失（约束第 1 层码本），$\mathcal{L}_{\text{GAN}}$ 为对抗损失。

### 说话人相似度

$$\text{SIM}(s_{\text{ref}}, s_{\text{gen}}) = \frac{\mathbf{e}_{\text{ref}} \cdot \mathbf{e}_{\text{gen}}}{\|\mathbf{e}_{\text{ref}}\| \cdot \|\mathbf{e}_{\text{gen}}\|}$$

其中 $\mathbf{e}$ 为基于 [[WavLM]] 的说话人验证模型提取的 speaker embedding。

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

> ✅ **R6/R7 三层 verify 完成注脚**（2026-05-26 dogfood 完整 verify 替换原警告 banner）
>
> 以下 🗺️/🔄 区块的具体技术对照判断已通过 §11 三层 verify 体系核对：
> - **L1 论文原文**：arXiv HTML 2601.15621v1 §3.1 (Architectures) / §3.2 (Training) / §3.3 (Features) + Tab.3 + Tab.4
> - **L2 GitHub 源码**：`finetuning/sft_12hz.py`、`finetuning/dataset.py`、`qwen_tts/core/models/modeling_qwen3_tts.py`、`qwen_tts/core/models/configuration_qwen3_tts.py`、`qwen_tts/inference/qwen3_tts_model.py`
> - **L3 第三方**：不需要
>
> **关键修正记录**（与原 dogfood 错误对照）：
> 1. ✅ **Tokenizer 分离已 verify**：§3.1 原文 "Text is processed using the standard Qwen tokenizer, while speech is encoded using the Qwen-TTS-Tokenizer" + GitHub `finetuning/dataset.py` 显示 `input_ids` shape `(b, t, 2)` 双通道结构（channel-0 text，channel-1 codec）+ 各自独立 special tokens (`tts_bos/eos/pad` vs `codec_bos/eos/pad`)。**确认 NOT unified-token-space**。
> 2. ⚠️ **LM 初始化部分修正**：paper §3.1/§3.3 声称 "leverages the Qwen3 LM family" / "built upon the Qwen3 text model foundation"（暗示 warm-start），**但 GitHub 开源代码显示 talker 实为 custom standalone Qwen3-style transformer**：不继承 `Qwen3PreTrainedModel/Qwen3ForCausalLM`，`hidden_size=1024` 不匹配任何已知 Qwen3 LLM 尺寸 (Qwen3-1.7B 为 2048)，`_init_weights` 用 standard normal init。`text_hidden_size=2048` 暗示外部输入 Qwen3-1.7B text features 的可能。**Pre-training init 代码未开源，literal Qwen3 LLM weight warm-start 仍未 verify** —— 之前笔记把"leverages Qwen3"直接当作"warm-start 加载 Qwen3 LLM checkpoint"是过度解读。
> 3. ✅ **Training loss 修正**：之前笔记说"无 loss balancing"是错的。GitHub `finetuning/sft_12hz.py` 实际公式为 `loss = outputs.loss + 0.3 * sub_talker_loss`，其中 `outputs.loss` 是 codec_0 第 0 层码本的 CE，`sub_talker_loss` 是残差 codebook 1-15 的 CE —— **存在多码本 intra-speech loss balancing (0.3 权重)**。但 text 通道仍全 -100 mask（`codec_0_labels = torch.full((b,t), -100)`），因此**无 text loss / 无 KL / 无 catastrophic-forgetting prevention**这一点之前笔记说对了；只是误判了"是否有任何 loss balancing"。
>
> **frontmatter R7 元数据**（lm_init / training_loss / tokenizer_arch / multitask / training_data / post_training / codec_detail）已按本次 verify 结果填入，每条带 §X / GitHub 来源标注。详见 [[方法论复盘-2026-05-26-知识地图建设]]、`feedback_no_hallucination_on_paper_details.md`、`feedback_second_order_analysis_hallucination.md`。

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]（核心 TTS 系统）+ 跨域到 [[SpeechLM-领域总览]]（用通用 Qwen3 LLM 做基座）
- **技术路线**：
  - [[TTS-技术路线图]] §路线 2 Codec LM（采用 speech token，沿用 LM 序列建模）
  - **新兴路线** "LLM-native TTS" —— 通用 LLM 直出 speech token，不再单独训练专用声学模型（与 CosyVoice 系列的"Codec LM 专用模型"路线形成对照）
  - [[TTS-技术路线图]] §控制范式 §策略 4 Instruction-Guided Synthesis（LLM 直接理解自然语言指令推断韵律）
  - [[TTS-技术路线图]] §路线融合：Codec LM + 通用 LLM 基座 + 双 tokenizer + 流式
- **核心问题**：
  - [[TTS-核心挑战]] §挑战 1 零样本克隆（500 万 h 数据 + LLM in-context learning）
  - [[TTS-核心挑战]] §挑战 2 韵律与表达性（**LLM 语义理解推断韵律**，不需要显式 prosody 标注）
  - [[TTS-核心挑战]] §挑战 4 训练数据（500 万 h 是目前已知最大规模，超过 CosyVoice3 100 万 h、VoxCPM 180 万 h）
  - [[TTS-核心挑战]] §挑战 5 Codec 设计（**双 tokenizer**：25Hz 单码本语义 + 12Hz 多码本语义-声学联合，是该方向的新探索）
  - [[TTS-核心挑战]] §挑战 3 流式低延迟
- **表示层位置**：
  - [[TTS-表示层地图]] §2 量化维度 SVQ（25Hz 单码本是 SVQ 大词表的代表）+ 多码本（12Hz）
  - [[TTS-表示层地图]] §5.1 混合 token（12Hz tokenizer 是语义+声学联合）
  - [[TTS-表示层地图]] §4.1 LM scaling 友好性（**LM init 机制语焉**：paper §3.1/§3.3 声称 "leverages Qwen3 LM family / built upon Qwen3 text model foundation" [已 verify §3.1, §3.3 原文措辞]，但 GitHub 开源代码显示 talker 是 custom standalone Qwen3-style transformer (hidden_size=1024 ≠ 任何已知 Qwen3 LLM size, 不继承 Qwen3PreTrainedModel) [已 verify GitHub: qwen_tts/core/models/{modeling_qwen3_tts.py, configuration_qwen3_tts.py}]，pre-training init 代码未开源 → **literal Qwen3 LLM weight warm-start 仍未 verify**。与 CosyVoice3 (cold-start codec LM) 的差异更准确表述为"采用 Qwen3-style 架构 + 可能的部分 warm-start" vs "完全独立 codec LM"，而不是"完整 warm-start vs cold-start"二元对比）
- **在 SpeechLM/对话框架内的位置**：
  - [[TTS-SpeechLM-Dialogue关系]] **位置 ① 独立产品**（作为 API/SDK 提供 voice cloning + voice design）
  - 同时 **接近位置 ②**：Qwen3 LLM 直出 speech token 的设计模糊了 TTS 与 SpeechLM 的边界（论文标题"TTS"但底座是通用 LLM）
- **相邻工作**：[[CosyVoice3]] / [[StepAudio2.5]] / [[SeedTTS]] / [[VoxCPM]] / [[F5-TTS]] / [[IndexTTS2]] / [[Fish-Speech]]
- **趋势位置**：[[TTS-趋势判断]] **趋势 3 LLM-native TTS / 统一 Speech LLM 的标志性代表系统**（与 [[StepAudio2.5]] 并列）；趋势 4 自然语言指令可控（LLM 语义理解驱动韵律）；趋势 6 Tokenizer 开放战场（双 tokenizer 是新探索）；趋势 9 开源闭源差距（500 万 h 闭源数据是当前已知最大）

---

## 🔄 后续重估

- **2026-05-25**：初读（首次精读 501 行笔记完成）。核心定位为 LLM-native TTS 路线代表：用 Qwen3 通用 LLM 做基座，双 tokenizer 设计，500 万 h 训练数据。
- **2026-05-26**：基于 11 篇综述综合的重新定位。新增以下判断：
  - **LLM-native 路线的本质（精确表述，2026-05-26 二次修正）**：text 和 speech tokenizer 仍然是分离的（与 CosyVoice3 一样），**真正的差异在 LLM 参数初始化策略——Qwen3-TTS 用 Qwen3 通用 LLM 初始化 (warm start)，CosyVoice3 是从头训练专用 LM (cold start)**。
    - **warm start 是"借用初始化"而非"保留通用能力"**：Qwen3-TTS 训练时只对 speech token 做 loss，没有文本 loss，文本能力会被 catastrophic forget 掉。最终它**只是一个专用 TTS**，不是"通用 LLM + TTS 模态"
    - **warm start 的真实价值**：通用 LLM 预训练学到的语义表征作为更好的初始化点 → speech token 学习时的 inductive bias 受益（即便文本能力被 forget，初始化时的语义对齐残影仍影响最终模型）
    - **指令跟随能力来源的精确表述**：不是"训练时保留语言模型能力"产生的，是"warm start 初始化的语义表征残影 + speech-instruction 配对训练数据"共同作用的结果
    - **不是** "unified token space" 架构（那是 Spirit-LM 的 interleaved 或 Moshi 的并行 token 设计——text 与 speech token 真正混在同一序列；Qwen3-TTS 的 token 仍然是分离的）
    - **与 StepAudio2.5 的进一步对比**：StepAudio2.5 是真的多任务训练（ASR+TTS+Realtime 共享基座，多任务 loss），有 "保留多任务能力" 的诉求；Qwen3-TTS 没有这个诉求——它只做 TTS
  - **数据规模优势** vs 评估困难：500 万 h 是当前已知最大规模，但缺乏与 CosyVoice3 (100 万 h) / VoxCPM (180 万 h) 等的公平第三方对比；论文自报指标受 Position #11 (arXiv:2510.06927) 警示影响——"500 万 h" 不直接转化为更好的 SECS / WER（数据规模 diminishing returns）
  - **双 tokenizer 设计的重要性**被低估：25Hz 单码本服务 ASR / 语义任务，12Hz 多码本服务高保真生成。这是 [[TTS-表示层地图]] §2 量化维度 SVQ vs 多 codebook 设计权衡的少见双轨实现
  - **证据强度** = medium：技术报告非同行评审；500 万 h 数据闭源；与 CosyVoice3 / SeedTTS / StepAudio2.5 等同期工业级系统缺乏第三方独立大规模评测对比
  - **成熟度** = emerging：LLM-native 路线整体仍 exploratory~emerging，Qwen3-TTS 是早期标志性工作但路线尚未充分验证（[[TTS-趋势判断]] 趋势 3 自评"LLM-native 全面取代专用系统尚早"）
  - **与 [[CosyVoice3]] 的关键路线分歧**：CosyVoice3 维持"独立 TTS 系统 + 专用 Codec LM"；Qwen3-TTS 走"通用 LLM 直出"。两者同属阿里通义但代表不同技术押注，[[TTS-代表模型谱系]] 应明示这一分化
  - **与 [[StepAudio2.5]] 的关键差异**：StepAudio2.5 强调统一基座 + ASR/TTS/Realtime 分支特化（位置 ②+④）；Qwen3-TTS 仍以"独立 TTS API"为产品形态（位置 ①）
- **2026-05-26 (dogfood 二次 verify)**：按新工作流 §2.5 + no-hallucination-rules.md §11 三层 verify，重读论文 §3 + GitHub 源码，对前一日推断做以下精确化（保留原条目作为审计轨迹，本条为最新可信版本）：
  - **Tokenizer 分离 [已 verify §3.1 + GitHub: finetuning/dataset.py]**：原文 "Text is processed using the standard Qwen tokenizer, while speech is encoded using the Qwen-TTS-Tokenizer"。GitHub 显示 `input_ids` 双通道 `(b, t, 2)`，channel-0 text + channel-1 codec 各有独立 special tokens 和 vocab。**前判断"text/speech tokenizer 分离 (NOT unified-token-space)"成立**。
  - **LM 初始化精确表述 [部分 verify, 部分仍未 verify]**：
    - 论文措辞 "leverages Qwen3 LM family" [§3.1] / "built upon Qwen3 text model foundation" [§3.3] 是 paper claim，**已 verify**。
    - GitHub 开源代码反向 verify：talker 实为 custom standalone transformer，**不继承 Qwen3PreTrainedModel/Qwen3ForCausalLM**，hidden_size=1024 不匹配 Qwen3-1.7B (2048) [已 verify GitHub: configuration_qwen3_tts.py + modeling_qwen3_tts.py]。`_init_weights` 用 standard normal init，无 Qwen3 LLM weight loading 调用 [已 verify GitHub: modeling_qwen3_tts.py]。
    - **Pre-training init 代码未开源** → literal "warm-start from Qwen3 LLM checkpoint" 仍未 verify [L2 不可用]。
    - **修正前判断**：之前笔记把 "leverages Qwen3" 直接当作 "literal weight warm-start" 是过度解读。更准确表述：**采用 Qwen3-style 架构（RoPE / GQA / RMSNorm / SwiGLU / sliding window）**，是否真的 warm-start 自 Qwen3 LLM checkpoint 没有公开证据。`text_hidden_size=2048` 提示可能存在外部 Qwen3-1.7B text features 输入路径，但代码未直接展示。
    - **对 CosyVoice3 对照判断的精确化**：原"warm vs cold start"二元对比过于简化；更准确是**架构同源（Qwen3-style block）vs 独立 codec LM**。
  - **Training loss 关键修正 [已 verify GitHub: finetuning/sft_12hz.py + dataset.py + modeling_qwen3_tts.py]**：
    - 之前笔记两处错判：
      - ❌ 之前隐含"warm start 必须搭配 loss balancing 防 catastrophic forgetting" → 实际无 text loss、无 forgetting prevention 设计
      - ❌ 之前明示"无 loss balancing" → 实际 finetuning loss 公式为 `loss = outputs.loss + 0.3 * sub_talker_loss`（codec_0 主 + 残差 codebook 1-15 副，**有多码本 intra-speech loss balancing**）
    - ✅ 正确表述：**finetuning 阶段是 speech-token-only CE on text channel + 多码本 intra-speech 0.3 权重 loss balancing；text 通道全 -100 mask；无 text loss、无 KL 约束**。pre-training loss 代码未开源，但 model.forward 只对 codec logits 算 loss [已 verify GitHub: modeling_qwen3_tts.py forward()]，强烈暗示 pre-training 也是 speech-token-only CE。
  - **二阶幻觉根源复盘**：前两次错误（unified-token-space + 必有 loss balancing）都是基于"已有笔记摘要 + ML 直觉补全"，未读原文 + 未查 GitHub。本次按 §11 三层 verify 后修正了一个直觉假阳性（loss balancing 实际有，但是 intra-speech 不是 text-speech），同时确认了一个直觉假阴性（unified-token-space 假说）。**结论：直觉对二阶判断的覆盖率永远不够，必须强制 L1+L2 双层 verify 才能下"X 是 Y"式的具体技术决策断言**。

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
