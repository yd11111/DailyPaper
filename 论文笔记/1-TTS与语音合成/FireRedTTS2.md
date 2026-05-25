---
title: "FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot"
method_name: "FireRedTTS2"
authors: [Kun Xie, Feiyu Shen, Junjie Li, Fenglong Xie, Xu Tang, Yao Hu]
year: 2025
venue: arXiv
tags: [tts, dialogue-tts, streaming-tts, speech-tokenizer, rvq, dual-transformer, podcast-generation, multi-speaker]
arxiv_id: "2509.02020"
pdf_path: "assets/papers/FireRedTTS2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
image_source: pdf
created: 2026-05-22
---

# 论文笔记：FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Xiaohongshu (小红书) |
| 日期 | September 2025 |
| 项目主页 | https://fireredteam.github.io/demos/firered_tts_2 |
| 前作 | [[FireRedTTS]], FireRedTTS-1S |
| 对比基线 | [[Seed-TTS]], [[F5-TTS]], [[MaskedGCT]], [[SparkTTS]], [[CosyVoice]] 3-1.5B, MoonCast, ZipVoice-Dialog, MOSS-TTSD |
| 链接 | [arXiv](https://arxiv.org/abs/2509.02020) / [Demo](https://fireredteam.github.io/demos/firered_tts_2) |

---

## 一句话总结

> 小红书提出 FireRedTTS-2，一个面向**长对话语音生成**的流式 TTS 系统，通过 12.5Hz 低帧率语义增强 speech tokenizer + text-speech interleaved format + dual-transformer 架构，实现了 podcast / chatbot 场景下稳定的多说话人合成、可靠的说话人切换和上下文一致的韵律。

---

## 核心贡献

1. **12.5Hz 流式 speech tokenizer**: 帧率仅为主流方案的一半，结合 Whisper 语义注入和 RVQ 16 层量化，在压缩序列长度的同时编码更丰富的语义信息，支持流式解码
2. **Text-speech interleaved format**: 将说话人标签文本与对齐语音 token 按时间顺序交替拼接，天然支持逐句生成，适配交互式对话和离线 podcast 两种场景
3. **Dual-transformer 架构**: 大 backbone transformer 处理 interleaved 序列并预测第 1 层 token，小 decoder transformer 补全剩余层，减少自回归步数并降低首包延迟
4. **三阶段课程训练**: pretraining (1.1M h 独白) -> post-training (300k h 对话) -> SFT，逐步注入对话能力

---

## 问题背景

### 要解决的问题
现有对话 TTS 系统存在三大痛点：(1) 需要完整对话文本才能合成，无法用于实时交互；(2) 生成不可分离的单轨混合音频，无法分别编辑各说话人；(3) 合成不稳定、说话人切换不准确、韵律不连贯。

### 现有方法的局限
对话 TTS 按 text-speech 组织方式可分为三类：

| 方案 | 代表工作 | 局限 |
|------|---------|------|
| 双通道并行 | Covomix, Covomix2 | 需完整对话文本；输出单轨混合音频 |
| 文本拼接+说话人标签 | MoonCast, ZipVoice-Dialog, MOSS-TTSD | 需完整对话文本；输出不可分离 |
| Text-speech 交织 | Moshi/Sesame | 支持逐句生成，但稳定性和韵律一致性仍有不足 |

### 本文的动机
采用 text-speech interleaved 策略（方案 3），并通过低帧率语义增强 tokenizer + dual-transformer 架构解决稳定性与上下文建模问题，同时支持交互式聊天和 podcast 生成。

---

## 方法详解

### 整体架构

FireRedTTS-2 由两大模块组成：
- **Speech Tokenizer**: 12.5Hz 流式语音分词器，融合语义和声学信息
- **Text-to-Speech Model**: 基于 dual-transformer 的自回归生成模型，处理 text-speech interleaved 序列

![[assets/papers/FireRedTTS2.pdf#page=2&rect=figure1|Figure 1: FireRedTTS-2 整体架构。(a) 12.5Hz speech tokenizer；(b) Dual-transformer text-to-speech model]]

### 模块 1: Speech Tokenizer (12.5Hz)

**设计目标**: 降低帧率以延长有效对话上下文、注入语义信息以稳定 text-to-token 建模、支持流式解码。

**编码器端**:
- 使用预训练 [[Whisper]] encoder 提取 16kHz 输入语音的语义特征
- 语义特征经 Adapter 编码后，与一个结构相同的可训练 acoustic encoder 的输出拼接
- 合并特征经 **4x downsample** 从 50Hz 降至 **12.5Hz**
- 经 **16 层 [[RVQ]]** 离散化，每层 codebook 大小 2048

$$\text{Frame rate: } \frac{50\text{Hz}}{4} = 12.5\text{Hz}, \quad \text{Codebook: } 16 \times 2048$$

**解码器端**:
- 量化特征 4x upsample 回 50Hz
- **Semantic decoder**: 上采样特征预测原始 Whisper 语义特征（语义监督）
- **Acoustic decoder**: 基于 [[Vocos]] 架构重建波形，支持流式/非流式两种模式

**两阶段训练**:
1. **Stage 1**: Acoustic decoder 非流式模式，预测 16kHz 语音，500k h 数据，32x H800 GPU 训练 320k steps（最后 35k steps 加入感知损失）
2. **Stage 2**: 冻结编码器，将 acoustic decoder 替换为全流式变体，预测 24kHz 语音，60k h 高保真数据训练 80k steps

### 模块 2: Text-to-Speech Model (Dual-Transformer)

**输入格式 — Text-speech interleaved**:

每段对话文本以说话人标签为前缀（如 `[S1]`），与对应的语音 token 拼接，按时间顺序组合：

```
[S1]<text><audio>[S2]<text><audio>[S3]<text><audio>...
```

这种格式天然支持**逐句生成**（sentence-by-sentence），降低首包延迟，适配实时对话场景。

**Dual-transformer 架构**:

传统 delay pattern 方案的缺陷：
- 对于 $N$ 层 token，第 $i$ 层向右移 $i-1$ 步，$N$ 个预测头并行预测
- 每个 timestep 只能部分访问前序语音 token，上下文条件弱化
- 首个 timestep 需 $N$ 步自回归才能获得完整 $N$ 层 token，延迟高

FireRedTTS-2 的解决方案：

| 组件 | 角色 | 基座 |
|------|------|------|
| **Backbone Transformer** (大) | 处理 text-speech interleaved 序列，预测第 1 层 token | [[Qwen2.5]] |
| **Decoder Transformer** (小) | 消费第 1 层 token + backbone hidden states，补全 2~16 层 | [[Qwen2.5]] |

每个 timestep，decoder 同时获得预测的第 1 层 token 和 backbone 的 hidden states，确保完整的上下文信息。相比 delay pattern：
- Backbone 仅需 **1 步**自回归（而非 $N$ 步）
- Decoder 需 $N-1$ 步补全剩余层
- 首包延迟显著降低

**训练效率优化**: Decoder transformer 仅在 interleaved 序列中 1/8 的语音段上优化，节省计算。

---

## 关键公式

### 总损失函数

$$\mathcal{L}_{loss} = 2 \times \big((1 - \lambda_{decoder})\mathcal{L}_{backbone} + \lambda_{decoder}\mathcal{L}_{decoder}\big) + \lambda_{text}\mathcal{L}_{text} \tag{1}$$

其中：
- $\mathcal{L}_{backbone}$: backbone transformer 的 cross-entropy loss（预测第 1 层 token）
- $\mathcal{L}_{decoder}$: decoder transformer 的 cross-entropy loss（预测 2~16 层 token）
- $\mathcal{L}_{text}$: 文本部分的 cross-entropy loss，用于稳定训练
- 超参设定: $\lambda_{text} = 0.01$, $\lambda_{decoder} = 0.6$

### RVQ 量化

对于输入特征 $\mathbf{z}$，16 层 RVQ 逐层量化残差：

$$\hat{\mathbf{z}}_l = Q_l(\mathbf{r}_{l-1}), \quad \mathbf{r}_l = \mathbf{r}_{l-1} - \hat{\mathbf{z}}_l, \quad l = 1, \dots, 16$$

其中 $\mathbf{r}_0 = \mathbf{z}$，每层 codebook $Q_l$ 包含 2048 个码字。

### 帧率与序列长度

低帧率对长对话的优势：

$$\text{Tokens per second} = 12.5 \times 16 = 200 \text{ (vs. 主流 25Hz} \times 8 = 200 \text{)}$$

但由于 backbone 只处理第 1 层 token，实际 backbone 序列长度仅 12.5 token/s，是 25Hz 方案的一半，有效上下文窗口翻倍。

---

## 训练策略

### 三阶段课程学习

| 阶段 | 数据 | 规模 | 轮次 | 目标 |
|------|------|------|------|------|
| **Pretraining** | 独白语音 | 1.1M h | 2 epochs | 建立基础 text-to-speech 能力 |
| **Post-training** | 多说话人对话 | 300k h | 5 epochs | 学习对话生成、说话人切换 |
| **SFT** | 特定场景数据 | 少量 | - | 定制特定说话人/场景 |

对话训练数据每段包含 2~5 个说话人，确保模型学到多说话人交互模式。

---

## 下游应用

### 1. Voice Cloning (零样本语音克隆)
- 拼接 prompt 语音转录 + 目标文本 + prompt speech tokens
- 自回归生成新 speech tokens，由 tokenizer decoder 还原波形
- 适用于视频配音等独白场景

### 2. Interactive Chat (交互式聊天)
- 无缝接入现有 chat 框架，无需修改其他模块
- 通过 SFT 学习从隐式上下文线索推断情感和韵律
- SFT 数据：15 小时女声语料，覆盖 6 种情绪（惊讶/悲伤/开心/关切/道歉/愤怒）
- 模型根据对话历史动态调整情感和语调

### 3. Podcast Generation (播客生成)
- 以两轮对话作为 prompt context，逐句生成后续对话
- 当前支持 4 个说话人、3 分钟对话
- 可通过扩展训练语料支持更多说话人和更长对话
- SFT 定制：50 h 对话数据（一男一女 podcast 主持人），fine-tune 15 epochs

---

## 实验

### 4.1 Speech Tokenizer 对比

在 LibriSpeech test-clean (2620 条, 16kHz) 上评估：

| 模型 | BPS | 帧率 | WER $\downarrow$ | SPK-SIM $\uparrow$ | STOI $\uparrow$ | PESQ-WB $\uparrow$ | PESQ-NB $\uparrow$ | UTMOS $\uparrow$ |
|------|-----|------|-----|---------|------|---------|---------|-------|
| Ground Truth | - | - | 1.96 | - | 1.00 | 4.64 | 4.55 | 4.09 |
| Xcodec2 | 800 | 50 | 2.46 | 0.82 | 0.92 | 2.43 | 3.04 | **4.13** |
| XY-Tokenizer | 1000 | 12.5 | - | 0.83 | 0.91 | 2.41 | 3.00 | - |
| SpeechTokenizer | 2000 | 50 | 2.86 | 0.66 | 0.88 | 1.92 | 2.38 | 3.56 |
| [[Mimi]] | 2200 | 12.5 | 2.26 | **0.87** | **0.94** | **2.88** | **3.42** | 3.87 |
| **FireRedTTS-2** | 2200 | 12.5 | **2.16** | **0.87** | **0.94** | 2.73 | 3.28 | 3.88 |

**关键发现**: FireRedTTS-2 tokenizer 在 12.5Hz 低帧率下取得最优 WER（2.16%），语义注入 + 显式监督功不可没。SPK-SIM 和 STOI 并列第一，PESQ 略低于 Mimi（后者训练数据为纯英文，与测试集更匹配）。

### 4.2 Voice Cloning (Seed-TTS-eval)

| 系统 | 帧率 | CER $\downarrow$ (中) | SIM $\uparrow$ (中) | WER $\downarrow$ (英) | SIM $\uparrow$ (英) |
|------|------|------|------|------|------|
| Human | - | 1.26 | 0.755 | 2.14 | 0.734 |
| [[Seed-TTS]] | - | 1.12 | **0.796** | 2.25 | **0.762** |
| [[F5-TTS]] | - | 1.56 | 0.741 | **1.83** | 0.647 |
| [[MaskedGCT]] | 50 | 2.27 | 0.774 | 2.62 | 0.714 |
| [[SparkTTS]] | 50 | 1.20 | 0.672 | 1.98 | 0.584 |
| [[CosyVoice]] 3-1.5B | 25 | 1.12 | 0.781 | 2.21 | 0.720 |
| FireRedTTS-1S | 25 | **1.00** | 0.753 | 2.20 | 0.663 |
| **FireRedTTS-2** | 12.5 | 1.14 | 0.736 | 1.95 | 0.665 |

**关键发现**: CER 1.14% / WER 1.95% 接近最优水平（接近 Seed-TTS 的 1.12% / CosyVoice 3 的 2.21%），归因于语义增强 token 稳定了 text-to-token 建模。Speaker similarity 在中文上与真人录音对齐，英文略低（英文训练数据多样性有限）。帧率减半但指标无明显退化。

### 4.3 Interactive Chat 情感控制

通过 Qwen3 生成 30 条/情绪的 query-response 对，FireRedTTS-2 从隐式上下文推断情感：

| 情绪 | 惊讶 | 悲伤 | 开心 | 关切 | 道歉 | 愤怒 |
|------|------|------|------|------|------|------|
| 准确率 | 83.3% | 86.7% | 90.0% | 86.7% | **93.3%** | 76.7% |

平均情感准确率约 **86.1%**，验证了模型可以从对话上下文隐式推断情感，无需显式情感标签。

### 4.4 Podcast Generation

在自建 podcast 评测集（中文 100 段 / 英文 115 段，每段 4~10 轮）上对比：

| 模型 | 中文 CER $\downarrow$ | 中文 SIM $\uparrow$ | 中文 MCD $\downarrow$ | 中文 CMOS | 英文 WER $\downarrow$ | 英文 SIM $\uparrow$ | 英文 MCD $\downarrow$ | 英文 CMOS |
|------|------|------|------|------|------|------|------|------|
| MoonCast | 3.81 | 0.658 | 11.37 | -0.21 | 3.81 | 0.620 | 10.96 | -0.21 |
| ZipVoice-Dialog | 2.93 | 0.736 | 9.29 | -0.18 | 11.71 | 0.701 | 9.88 | -0.31 |
| MOSS-TTSD | 3.99 | 0.659 | 8.32 | -0.16 | 5.43 | 0.550 | 9.25 | -0.13 |
| **FireRedTTS-2** | **2.08** | **0.753** | **7.99** | **0.0** | **3.16** | **0.703** | **9.06** | **0.0** |

**关键发现**:
- **可懂度最佳**: 中文 CER 2.08%、英文 WER 3.16% 均为最低，低帧率 + 语义增强 tokenizer 在长序列上优势明显
- **Speaker similarity 最高**: 跨轮 voice cloning 一致性最强，说话人切换可靠
- **MCD 最低**: 与 ground truth 偏差最小
- **CMOS 基准**: 作为参考基准（0.0），其他系统均为负值

### 主观评测 (Fine-tuned Podcast)

对两位 podcast 主持人定制 fine-tune 后的主观偏好测试：
- **Win 28%** (FireRedTTS-2 比真人更自然) + **Even 28%** (难以区分) = **56% 匹配或超越真人**
- **Fail 44%** (真人更自然)
- Fine-tuned CER 降至 **1.66%**（低于 zero-shot 的 2.08%）

---

## 局限性与讨论

1. **说话人数量**: 当前支持最多 4 个说话人 / 3 分钟对话，需更多训练数据扩展
2. **英文 speaker similarity**: 受限于英文训练数据多样性，低于 Seed-TTS 等专攻英文的系统
3. **PESQ/UTMOS**: 12.5Hz 低帧率在客观音质指标上略低于 50Hz 方案（如 Xcodec2），但主观评测更优
4. **情感控制**: 愤怒情绪准确率最低（76.7%），细粒度情感控制仍有提升空间
5. **重叠语音**: 当前 interleaved 格式不原生支持 overlapping speech，与双通道方案相比有局限

---

## 与前作对比

| 特性 | FireRedTTS | FireRedTTS-1S | **FireRedTTS-2** |
|------|-----------|---------------|-----------------|
| 定位 | 工业级基础 TTS | 可流式基础 TTS | **对话/Podcast TTS** |
| 帧率 | 25 Hz | 25 Hz | **12.5 Hz** |
| 对话支持 | 无 | 无 | **多说话人对话** |
| 流式 | 否 | 是 | **是** |
| 架构 | - | - | **Dual-transformer** |
| Seed-TTS CER (中) | - | 1.00 | 1.14 |
| Seed-TTS WER (英) | - | 2.20 | **1.95** |

---

## 借鉴意义

1. **12.5Hz 是对话 TTS 的甜点帧率**: 帧率减半使 backbone 有效上下文翻倍，对长对话至关重要；配合语义注入可弥补信息损失
2. **Dual-transformer 优于 delay pattern**: 在多层 codec token 预测中，大模型预测第 1 层 + 小模型补全剩余层的范式，兼顾上下文质量和延迟
3. **Text-speech interleaved 是对话 TTS 的正确格式**: 天然支持逐句生成和流式输出，比全文拼接或双通道更灵活
4. **语义注入是 speech tokenizer 的关键差异点**: Whisper 语义特征的显式监督使低帧率 tokenizer 在 WER 上反超高帧率方案
5. **隐式情感推断的可行性**: 无需显式情感标签，仅通过对话上下文即可推断情感并调整韵律，降低了交互式 TTS 的使用门槛

---

## 🔗 链接

- arXiv: https://arxiv.org/abs/2509.02020
- Demo: https://fireredteam.github.io/demos/firered_tts_2
- PDF: [[assets/papers/FireRedTTS2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/fireredtts2.pdf`
