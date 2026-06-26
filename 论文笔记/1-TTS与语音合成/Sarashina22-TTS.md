---
title: "Sarashina2.2-TTS: Tackling Kanji Polyphony in Japanese Speech Generation via Data Scaling and Targeted Data Synthesis"
method_name: "Sarashina2.2-TTS"
authors: [Lianbo Liu, Shiao Zhu, Kai Washizaki, Reo Yoneyama, Haesung Jeon, Mengjie Zhao, Yusuke Fujita, Hao Shi, Nao Yoshida, Yuan Gao, Roman Koshkin, Yukiya Hono, Yui Sudo]
year: 2026
venue: arXiv
arxiv_id: "2606.25369"
tags: [tts, japanese-tts, kanji-polyphony, data-augmentation, speech-llm, zero-shot-cloning, evaluation]
zotero_collection:
note_tier: standard

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: false
post_training_type: none
streaming: false

# === 知识地图联动 ===
domain: TTS
subdomain: japanese-tts
routes: [speech-llm-tts, codec-lm-tts]
problems: [multilinguality, evaluation, data-scale, zero-shot-cloning]
representations: [semantic-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-评测体系]]"
  - "[[TTS-核心挑战]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-06-26

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.25369v1/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.25369v1/paper.html"
figures_dir: "_resources/2606.25369v1/figures"
github_local: "~/DailyPaper/.cache/papers/2606.25369v1/github/sbintuitions_sarashina2.2-tts"
cached_at: 2026-06-26

# === 通用元数据 ===
image_source: mixed
arxiv_html: https://arxiv.org/html/2606.25369v1
created: 2026-06-26
---

# 论文笔记：Sarashina2.2-TTS: Tackling Kanji Polyphony in Japanese Speech Generation via Data Scaling and Targeted Data Synthesis

> **笔记分级**：standard（方法清晰、数据策略有系统性、评测框架有独立贡献，值得精读）。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | SB Intuitions（SoftBank 子公司） |
| 日期 | June 2026 |
| 项目主页 | — |
| 对比基线 | [[T5Gemma-TTS]], [[Qwen3-TTS]], [[FishAudio S1-mini]], [[FireRedTTS2]] |
| 链接 | [arXiv](https://arxiv.org/abs/2606.25369) / [Code](https://github.com/sbintuitions/sarashina2.2-tts) / [Benchmark](https://github.com/sbintuitions/JoyoKanji-Yomi-Benchmark) / [Kana-ASR](https://huggingface.co/sbintuitions/kana-whisper) |

## 一句话总结

> 以日语为核心的 LLM-TTS 系统，通过 361k 小时大规模数据训练 + 覆盖全部 2136 个常用汉字的定向合成增强（PronSteering），配合专门设计的 Kana-CER 评测指标和 Joyo Kanji Yomi Benchmark，在汉字多音字读音准确率和跨语言提示鲁棒性上显著优于现有多语种 TTS 系统。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 必要图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源不写在这里，统一收到「附录·核验结论」表。

## 核心贡献

1. **数据策略**：~361k 小时训练数据（日语 194k + 英语 167k），日语占比 53.7%；设计 PronSteering 机制生成覆盖全部 2136 个常用汉字 4378 种读法的定向合成数据（~320 小时，28 万样本）
2. **评测框架**：提出 Kana-CER 指标（在假名空间比较而非正字法空间），构建 Joyo Kanji Yomi Benchmark（13095 条测试句，35 名母语标注者人工验证）
3. **跨提示鲁棒性**：唯一一个在非日语提示（英语口音）下日语发音不退化的系统（退化率 -0.2%，其余系统 18%~328%）

## 问题背景

### 要解决的问题

日语 TTS 面临汉字多音字（polyphony）挑战：2136 个常用汉字有 4378 种读法，同一个字在不同语境读音完全不同（如"生"有 12 种读法）。现有多语种 TTS 系统在日语汉字读音上表现不稳定。

### 现有方法的局限

1. **数据层面**：多语种系统（如 [[Qwen3-TTS]]）以英/中为主，日语占比低，罕见汉字读法训练不足
2. **评测层面**：标准 CER/WER 在正字法空间计算，日语的汉字/假名/罗马字正字法变体导致指标虚高——同一发音写法不同被误判为错误
3. **跨提示问题**：用非日语提示音频合成日语文本时，发音准确度严重下降

### 本文的动机

通过"数据策略 + 评测方法论"双管齐下：用大规模日语数据保底 + 定向合成稀有读法数据补短板；用假名空间评测消除正字法噪声。

## 方法详解

### 领域定位

Sarashina2.2-TTS 属于 **[[Speech LLM]] + [[Semantic Token]] TTS** 路线，与 [[CosyVoice2]]、[[Qwen3-TTS]] 同类——backbone 是预训练 LLM（生成语义 token），下游接 [[Flow Matching]] 解码器 + [[Vocoder]] 还原波形。核心差异不在模型架构（架构几乎完全复用 CosyVoice 2），而在**日语定向的数据策略和评测方法论**。

### 端到端数据流（先地图后街景）

Sarashina2.2-TTS 的完整流水线：**文本 + 参考音频** → **Stage 1: Backbone LLM**（AR 生成语义 token 序列）→ **Stage 2: Flow-Matching Decoder**（语义 token → Mel 频谱）→ **Stage 3: HiFi-GAN Vocoder**（Mel → 波形）。

![[_resources/2606.25369v1/figures/fig-000.png]]

> **Figure 1**：Sarashina2.2-TTS 架构。蓝色块为语义阶段——backbone LLM 自回归地将提示文本、目标文本、参考音频的语义 token 映射为目标语义 token。绿色块为声学阶段——flow-matching 解码器将语义 token（以提示语义 token、说话人嵌入、参考 Mel 为条件）重建为 Mel 频谱，最后 HiFi-GAN 解码为波形。

下面逐个放大每个关键模块。

### Speech Tokenizer（S3Tokenizer V2）

**为什么这样设计**：TTS 需要一种既紧凑又能保留发音信息的中间表示。S3Tokenizer V2（来自 [[CosyVoice2]]）将 [[FSQ]]（Finite Scalar Quantization）嵌入大规模 ASR 编码器内部，以 ASR 目标端到端训练，保证 token 主要编码音素内容而非声学细节。

**怎么做**：输入 16kHz 音频 → 计算 log-mel 频谱 → S3Tokenizer V2 编码为**单码本**、**25 Hz** 帧率的离散 token 序列。码本大小 6561（代码 `S3_VOCAB_SIZE = 6561`）。

### Backbone LLM

**为什么这样设计**：用预训练日语 LLM 初始化而非从头训练，利用 LLM 对日语文本（包括汉字上下文）的理解能力来辅助正确读音选择。这是解决汉字多音字的关键：LLM 的语言理解能力用于区分同一汉字在不同语境的读法。

**怎么做**：基于 Sarashina2.2-0.5B-Instruct（24 层 Transformer decoder，约 0.5B 参数，以日语为主预训练）。词表扩展 6561 个语义 token + 特殊 token（`<|speech_start|>`、`<|pron_start|>`/`<|pron_end|>` 等）。

**具体例子**：给定提示转录 $\mathbf{x}^p$、目标文本 $\mathbf{x}^t$、提示语义 token $\mathbf{s}^p$，模型自回归生成目标语义 token $\mathbf{s}^t$：

$$
\underbrace{\texttt{BOS},\;\mathbf{x}^{p},\;\mathbf{x}^{t},\;\texttt{<|speech\_start|>},\;\mathbf{s}^{p}}_{\text{Input}},\;\underbrace{\mathbf{s}^{t},\;\texttt{EOS}}_{\text{Predicted}}
$$

**含义**：将文本和参考音频 token 拼接为输入序列，LLM 以 teacher forcing 方式自回归预测目标语义 token。

### Flow-Matching Decoder + Vocoder

**为什么这样设计**：语义 token 只编码音素内容，缺乏声学细节。Flow-matching 解码器将语义 token 转换为 Mel 频谱，恢复声学细节；HiFi-GAN 再将 Mel 转为波形。整个声学阶段直接采用 [[CosyVoice2]] 的实现。

**怎么做**：
- **Flow-matching 解码器**：条件 Flow Matching（CFM），使用卷积 Transformer UNet 架构（`CausalMaskedDiffWithXvec`），学习将高斯噪声传输到目标 Mel 频谱的向量场。条件输入包括：语义 token 序列、参考 Mel 频谱、说话人嵌入（192 维，来自 CAM++ 模型）。CFG 率 0.7。
- **Vocoder**：HiFT-GAN（基于 HiFi-GAN 的变体），上采样率 8x5x3 x iSTFT hop 4 = 480，输出 24kHz 波形。

### PronSteering（发音引导机制）

**为什么这样设计**：标准训练数据中罕见汉字读法出现频率极低，模型倾向于回退到高频读法。PronSteering 通过在文本中显式标注汉字读音（假名 + 声调标记），让模型在推理时可以被引导到正确的罕见读法。

**怎么做**：添加两个特殊 token `<|pron_start|>` 和 `<|pron_end|>`，将目标汉字的读音（假名 + 声调标记 `[` `]`）包裹其中。

**具体例子**：
- 原文："今日はいい天気ですね。"
- PronSteering 标注："`<|pron_start|>`キョ]ー`<|pron_end|>`はいい天気ですね。"

声调标记采用韵律符号方式：`[` 表示音高上升，`]` 表示音高下降。

### 定向合成增强流水线（3 步）

**为什么这样设计**：覆盖全部 2136 个常用汉字的 4378 种读法需要系统性生成包含目标读法的句子 + 精确标注 + 质量过滤。

**怎么做**：
1. **生成句子**：LLM 生成包含目标汉字的句子；UniDic 形态分析验证；最多 2 轮修正
2. **标注并合成**：定位目标汉字形态素，提取读法 + 声调，构造 PronSteering 控制片段；处理连浊和促音；97.5% 成功标注；用多样化说话人提示合成
3. **质量过滤**：Kana-ASR 转录对比参考读法（汉字级 + 句子级）；超过错误阈值的样本丢弃；保留率 95.1%

**产出**：~28 万条合成样本（~320 小时）。

### 训练流程

- **Stage 1（预训练）**：全参数训练，361k 小时语料，恒定学习率 $1 \times 10^{-4}$
- **Stage 2（微调）**：线性衰减学习率 $1 \times 10^{-4} \to 1 \times 10^{-6}$，使用 Stage 1 数据的更高质量子集 + 定向合成数据（~320h）

声学阶段（flow-matching 解码器 + HiFi-GAN vocoder）直接沿用 CosyVoice 2，不重新训练。

### 推理流程

1. 参考音频 → S3Tokenizer 提取语义 token + CAM++ 提取 192 维说话人嵌入 + 提取 Mel 频谱（80 维，24kHz，hop 480）
2. 拼接提示文本 + 目标文本 + `<|speech_start|>` + 提示语义 token → LLM AR 解码（temperature 0.9, top_p 0.95）
3. 输出语义 token → Flow-matching 解码器（以参考 Mel + 说话人嵌入为条件）→ Mel 频谱
4. Mel → HiFT-GAN → 24kHz 波形
5. （可选）SilentCipher 音频水印嵌入

## 关键结果

> 只列支撑主结论的核心表/图。完整表格见附录。

**核心证据**：Table 2（Joyo Kanji Yomi Benchmark + JSUT 整体评测）是全文最强证据，直接展示 Stage 2 在汉字级读音准确率上的显著优势。

| System | Kana-CER_kanji ↓ | Kana-CER†_kanji ↓ | Kana-CER_sent ↓ (Joyo) | SIM ↑ (CV3-Ja) |
|--------|---------|---------|---------|---------|
| T5Gemma-TTS | 13.81 | 8.55 | 3.69 | 50.59 |
| Qwen3-TTS | 185.89 | 21.70 | 23.20 | 69.86 |
| FishAudio S1-mini | 33.43 | 20.19 | 5.46 | 61.38 |
| FireRedTTS-2 | 27.82 | 16.39 | 4.28 | 66.20 |
| Sarashina2.2-TTS (Stage 1) | 11.06 | 6.94 | 4.59 | **75.64** |
| **Sarashina2.2-TTS (Stage 2)** | **7.83** | **5.45** | **3.41** | 74.75 |

**结论**：Stage 2 在 Kana-CER†_kanji 上达到 5.45，比次优 T5Gemma-TTS 的 8.55 低 36%。同时在说话人相似度（SIM）上达到最高水平（Stage 1: 75.64）。但 Qwen3-TTS 表现异常不稳定（标准差极大），可能是其日语支持不成熟。

**跨语言鲁棒性**（最独特的结论）：

| System | 日语提示 CER | 非日语提示 CER | 退化率 |
|--------|---------|---------|---------|
| Qwen3-TTS | 63.11 | 270.12 | +328.1% |
| FireRedTTS-2 | 11.05 | 29.63 | +168.3% |
| FishAudio S1-mini | 9.44 | 20.14 | +113.4% |
| T5Gemma-TTS | 10.88 | 12.87 | +18.4% |
| **Sarashina2.2-TTS (Stage 2)** | **9.55** | **9.53** | **-0.2%** |

**结论**：Sarashina2.2-TTS 是唯一在使用非日语提示时不退化的系统。这归因于日语占训练数据的 53.7%，而其他系统以英语/中文为主。

## 可复用的设计模式

1. **PronSteering（显式发音引导）**：在文本序列中用特殊 token 包裹目标发音标注，让 LLM 在推理时可被引导到特定读法。适用于任何多音字/多读法语言（中文也有此问题）的 TTS 系统。来自本文的发音引导机制。
2. **假名空间评测（Kana-CER）**：将 ASR 输出和参考答案都转换到语音学空间（假名/拼音/IPA）而非正字法空间进行比较，消除书写系统变体导致的指标虚高。适用于任何有正字法多样性的语言（日语、中文繁简、阿拉伯语元音省略等）。来自本文的 Kana-CER 指标设计。
3. **定向稀有样本合成流水线**：对长尾分布中的稀有类别（罕见读法/罕见音素/罕见说话人风格），用 LLM 生成包含目标的句子 → 形态分析验证 → 合成 → ASR 质量过滤。适用于任何长尾分布问题的数据增强。来自本文的三步增强流水线。
4. **跨语言提示鲁棒性测试**：用非目标语言的提示音频测试目标语言的合成质量，检验模型是否过度依赖提示语言。适用于多语种 TTS 系统的鲁棒性评估。来自本文的 cross-prompt evaluation。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 📋 核验结论（技术元数据）

> 从 frontmatter relocate 来的"已 verify + [§X]"prose 版结论。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start：从 Sarashina2.2-0.5B-Instruct（日语主导预训练 LLM）初始化，扩展词表加入 6561 个语义 token + 特殊 token | [已 verify §2, GitHub: additional_tokens.py] |
| 训练 loss | 标准 CE loss，仅在语义 token 位置计算：$\mathcal{L} = -\sum_{t=1}^{T}\log p_\theta(\mathbf{s}_t \mid \mathbf{x}, \mathbf{s}_{<t})$ | [已 verify §2 Eq.2] |
| Tokenizer 架构 | text+speech 分离：文本用 LLM 原始 tokenizer，语音用 S3Tokenizer V2 编码为单码本离散 token（6561 码本，25Hz），二者拼接为序列输入 LLM | [已 verify §2, GitHub: additional_tokens.py, generate.py] |
| 多任务 | false。单任务 TTS（文本→语义 token），无 ASR 或其他辅助任务 loss | [已 verify §2 Eq.2, §5.1] |
| 训练数据 | 361k 小时（日语 194k / 英语 167k）；日语涵盖播客、有声书、客服、广播等 7 域；Stage 2 增加 ~320h 定向合成数据 | [已 verify §3 Tab.1] |
| 后训练 | 无 RLHF/DPO。两阶段训练：Stage 1 全参数预训练 + Stage 2 微调（高质量子集 + 合成数据） | [已 verify §5.1] |
| Codec 细节 | S3Tokenizer V2：单码本，FSQ，6561 词汇量，25 Hz 帧率。声学阶段直接采用 CosyVoice 2 的 CFM decoder + HiFT-GAN | [已 verify §2, GitHub: S3_VOCAB_SIZE=6561, decoder.py, flow.py] |
| 说话人嵌入 | CAM++ 模型，输出 192 维嵌入向量 | [已 verify GitHub: speech_encoder.py, campplus.py] |
| Vocoder | HiFT-GAN（HiFi-GAN 变体），上采样 8x5x3 x iSTFT hop 4 = 480，24kHz 输出 | [已 verify GitHub: decoder.py L24, hifigan.py] |
| 水印 | SilentCipher（Sony），推理时可选嵌入不可闻水印 | [已 verify GitHub: generate.py L316-325] |

## 完整公式

### 公式1: [[Language Modeling|序列生成目标]]

$$
\underbrace{\texttt{BOS},\;\mathbf{x}^{p},\;\mathbf{x}^{t},\;\texttt{<|speech\_start|>},\;\mathbf{s}^{p}}_{\text{Input}},\;\underbrace{\mathbf{s}^{t},\;\texttt{EOS}}_{\text{Predicted}}
$$

**含义**：输入序列由提示文本、目标文本、提示语义 token 拼接；模型自回归预测目标语义 token。

**符号说明**：
- $\mathbf{x}^p$: 提示音频的文本转录
- $\mathbf{x}^t$: 目标合成文本
- $\mathbf{s}^p$: 提示音频的语义 token 序列
- $\mathbf{s}^t$: 待生成的目标语义 token 序列

### 公式2: [[Cross-Entropy Loss|训练损失]]

$$
\mathcal{L} = -\sum_{t=1}^{T}\log p_{\theta}(\mathbf{s}_{t} \mid \mathbf{x}, \mathbf{s}_{<t})
$$

**含义**：标准因果语言模型交叉熵损失，仅在语义 token 位置计算。

**符号说明**：
- $p_\theta$: 模型预测的条件概率
- $\mathbf{s}_t$: 第 $t$ 个语义 token
- $\mathbf{x}$: 拼接后的文本+提示 token 序列
- $T$: 目标语义 token 序列长度

## 完整图表

### Figure 1: 系统架构

（已嵌入主文）

### Figure 2: 汉字读法错误分布累积曲线

![[_resources/2606.25369v1/figures/fig-002.png]]

> **Figure 2**：所有 4378 个汉字-读法对的逐读法错误计数累积分布。每个点表示错误计数 ≤ 阈值的读法百分比（15 次试验）。曲线越靠左上方表示汉字读音准确率越高。Sarashina2.2-TTS Stage 2 有 81.5% 的读法完全正确（零错误），显著领先其他系统。

### Table 1: 训练数据组成

| Domain | Japanese (h) | English (h) | Total (h) |
|---|---|---|---|
| Podcast | 58,304 | 106,927 | 165,231 |
| Audiobook / Narration | 66,839 | – | 66,839 |
| Customer Service | 27,864 | – | 27,864 |
| TV / Broadcast | 21,287 | – | 21,287 |
| Public Speech | 16,157 | – | 16,157 |
| Conversation | 2,565 | – | 2,565 |
| Language Learning | 1,035 | – | 1,035 |
| Uncategorized* | – | 60,226 | 60,226 |
| **Total** | **194,051** | **167,153** | **361,204** |

**说明**：日语占 53.7%，英语占 46.3%。所有音频来源均为正版授权或公共领域。

### Table 2: Joyo Kanji Yomi Benchmark + JSUT 综合结果

| System | Kana-CER_kanji ↓ | Kana-CER†_kanji ↓ | Kana-CER_sent ↓ (Joyo) | CER ↓ (Joyo) | Kana-CER_sent ↓ (JSUT) | CER ↓ (JSUT) |
|---|---|---|---|---|---|---|
| T5Gemma-TTS | 13.81±2.52 | 8.55±0.45 | 3.69±1.85 | 5.68±2.85 | 2.80±0.04 | 7.63±0.07 |
| Qwen3-TTS | 185.89±105.56 | 21.70±0.30 | 23.20±11.53 | 13.26±3.67 | 15.58±10.34 | 14.87±6.47 |
| FishAudio S1-mini | 33.43±1.41 | 20.19±0.20 | 5.46±0.29 | 6.15±0.70 | 5.16±0.05 | 9.03±0.09 |
| FireRedTTS-2 | 27.82±0.88 | 16.39±0.15 | 4.28±0.06 | 5.32±0.11 | 5.26±0.22 | 9.33±0.18 |
| Sarashina2.2-TTS (Stage 1) | 11.06±0.65 | 6.94±0.08 | 4.59±0.83 | 6.36±0.63 | 3.04±0.07 | 8.08±0.09 |
| **Sarashina2.2-TTS (Stage 2)** | **7.83±0.70** | **5.45±0.10** | **3.41±0.77** | **5.28±0.37** | **2.91±0.06** | **8.02±0.07** |

**说明**：Stage 2 在所有 Kana-CER 指标上最优。标准 CER 始终高于 Kana-CER_sent，证实正字法变体导致传统指标虚高。

### Table 3: Stage 1 逐读法错误分析（选摘）

| 汉字 | 目标读法 | 示例 | 错误数 (/15) | 产出读法 |
|---|---|---|---|---|
| 六 | ム (mu) | 六十路, 六三四 | 15 | ロクジューロ(5), ロクサンシ(4)... |
| 坂 | ハン (han) | 急坂, 坂路 | 15 | ザカ(5), サカ(4)... |
| 従 | ショウ (shou) | 従容 | 15 | ジュー(15) |
| 生 | ショウ (shou) | 生滅, 一生 | 5 | セー(4), セーミ(1) |

**说明**：15/15 全错 = 模型完全回退到高频读法；中间值 = 语境依赖性失败。

### Table 4: 跨系统逐读法对比（选摘）

| 汉字 | 读法 | 类型 | Stage1 | Stage2 | T5Gemma | FireRed2 | S1-mini | Qwen3 |
|---|---|---|---|---|---|---|---|---|
| 事 | ズ (zu) | 稀有 | 15 | 1 | 0 | 15 | 15 | 15 |
| 出 | スイ (sui) | 稀有 | 12 | 0 | 15 | 14 | 15 | 15 |
| 従 | ショウ (shou) | 稀有 | 15 | 2 | 15 | 15 | 15 | 15 |
| 生 | オウ (ou) | 稀有 | 14 | 14 | 14 | 15 | 14 | 15 |

**说明**：Stage 2 定向增强对多数稀有读法有效（如"事-ズ"从 15 降到 1），但极端稀有读法（如"生-オウ"）仍有困难。

### Table 5: 跨风格评测（9 种日语提示）

| System | CER Mean ↓ | Cross-Style STD ↓ |
|---|---|---|
| Qwen3-TTS | 63.11 | 125.00 |
| FireRedTTS-2 | 11.05 | 2.06 |
| FishAudio S1-mini | 9.44 | 0.71 |
| T5Gemma-TTS | 10.88 | 2.69 |
| Sarashina2.2-TTS (Stage 1) | 10.18 | 1.46 |
| **Sarashina2.2-TTS (Stage 2)** | **9.55** | **1.32** |

### Table 6: 跨语言评测

（已嵌入主文关键结果部分）

### Table 7: 零样本说话人相似度 (CV3-Ja)

| System | SIM ↑ |
|---|---|
| T5Gemma-TTS | 50.59 |
| Qwen3-TTS | 69.86 |
| FishAudio S1-mini | 61.38 |
| FireRedTTS-2 | 66.20 |
| **Sarashina2.2-TTS (Stage 1)** | **75.64** |
| Sarashina2.2-TTS (Stage 2) | 74.75 |

**说明**：SIM 使用 CAM++ 提取说话人嵌入后计算余弦相似度。Stage 1 略高于 Stage 2（75.64 vs 74.75），说明合成数据微调对说话人相似度有轻微负面影响但不显著。

### Table 8: 语音质量自动 MOS 评测 (CV3-Ja)

| System | UTMOS ↑ | UTMOS V2 ↑ | DNSMOS ↑ | DNSMOS P.835 ↑ |
|---|---|---|---|---|
| Prompt Speech | 2.576 | 2.455 | 3.495 | 2.940 |
| T5Gemma-TTS | 2.952 | 2.523 | 3.517 | 3.015 |
| Qwen3-TTS | 3.406 | 2.806 | 3.733 | 3.263 |
| FishAudio S1-mini | 3.294 | 2.748 | 3.773 | 3.244 |
| FireRedTTS-2 | 2.582 | 2.508 | 3.598 | 3.142 |
| Sarashina2.2-TTS (Stage 1) | 3.184 | **2.888** | 3.811 | 3.238 |
| Sarashina2.2-TTS (Stage 2) | 3.174 | 2.877 | **3.824** | 3.242 |

**说明**：Stages 1 和 2 在语音质量上表现相当——合成数据增强改善了发音准确率而未损害感知语音质量。DNSMOS 指标上 Sarashina2.2-TTS 最优。

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | Joyo Kanji Yomi Benchmark 上的 Kana-CER 结果 | 自建 benchmark 但已开源，13095 条测试集，35 名母语标注者人工验证，5 random seeds 报告均值±标准差，评测代码开源 |
| **高** | JSUT 上的 Kana-CER/CER 结果 | JSUT 是公开 benchmark，评测协议透明 |
| **中** | 跨提示鲁棒性评测 | 方法论新颖且合理，但提示集（12 条）规模较小，风格覆盖不全面 |
| **中** | 说话人相似度（SIM） | 使用 CAM++ 而非更常用的 WavLM-TDNN 或 ECAPA-TDNN，跨系统可比性有限 |
| **中** | 自动 MOS（UTMOS/DNSMOS） | 仅自动指标，无人工主观评测（MOS/CMOS） |
| **低** | 定向增强的泛化性声称 | 仅在自建 benchmark 上验证，缺乏外部独立评测 |

## 核心 Claim 审查

1. **Paper Claim**：Stage 2 实现 SOTA 汉字级读音准确率
   **My Assessment**：在作者自建的 Joyo Kanji Yomi Benchmark 上确实最优（Kana-CER†_kanji 5.45 vs 次优 8.55），JSUT 上也最优。但 benchmark 是自建的（虽已开源），且基线选择限于 4 个系统，不含其他日语专用 TTS（如 VOICEVOX、COEIROINK 等）。结论在所报告设置下成立。

2. **Paper Claim**：唯一在非日语提示下不退化的系统
   **My Assessment**：Table 6 的数据明确支持这一结论（-0.2% vs 其他系统 +18%~+328%）。但测试只用了 3 种非日语提示（美式/英式/印度口音英语），未包含中文/韩语等东亚语言提示。该结论在英语提示条件下成立。

3. **Paper Claim**：最高说话人相似度
   **My Assessment**：SIM 75.64（Stage 1）确实在 5 个对比系统中最高。但使用 CAM++ 作为说话人编码器，与领域内更常用的 WavLM-TDNN / ECAPA-TDNN 不同，跨论文不可比。此外 CAM++ 也是系统内部组件（flow-matching 条件的说话人嵌入来自同一模型），存在 encoder bias 风险。

4. **Paper Claim**：Kana-CER 比标准 CER 更准确反映日语 TTS 发音质量
   **My Assessment**：概念合理——正字法变体确实导致标准 CER 虚高。但 Kana-ASR 本身在 JSUT 实录上的 Kana-CER 为 0.979%（非零），说明 Kana-ASR 转写错误会引入系统性噪声。论文承认 Kana-ASR 在口语体上可能失准。

## 批判性思考

### 优点
1. **问题聚焦明确**：汉字多音字是日语 TTS 的真实痛点，论文对问题的分析（数据不足 + 评测不当）准确且有系统性
2. **评测框架贡献突出**：Joyo Kanji Yomi Benchmark + Kana-CER 指标为日语 TTS 评测提供了缺失的基础设施，且全部开源
3. **实验设计严谨**：5 random seeds 报告均值±标准差，跨风格/跨语言多维度评测，逐读法细粒度分析
4. **开源完整**：模型权重、benchmark、Kana-ASR、评测脚本全部开源

### 局限性
1. **架构无创新**：模型架构完全复用 [[CosyVoice2]]（semantic tokenizer + CFM decoder + HiFT-GAN），核心贡献在数据和评测而非模型设计
2. **无人工主观评测**：缺乏 MOS/CMOS 人工评分——对 TTS 系统这是重要缺失，自动指标（UTMOS/DNSMOS）与人工感知存在差距
3. **基线有限**：仅对比 4 个系统，缺少日语专用 TTS（VOICEVOX 等）和更多工业系统（Azure、Google Cloud TTS）
4. **训练数据不公开**：361k 小时数据"正版授权或公共领域"但不公开数据列表，可复现性受限
5. **PronSteering 未开源**：论文明确表示开源版不含 PronSteering 功能，但 Stage 2 的定向增强数据是由 PronSteering 生成的——开源模型的优势部分来自不可复现的增强流程
6. **非商用许可**：Sarashina Model NonCommercial License，商用需联系 SB Intuitions
7. **极端稀有读法仍失败**："生-オウ"Stage 2 仍 14/15 错误，说明定向增强对极端稀有语境效果有限

### 潜在改进方向
1. 将 PronSteering 开源，允许用户在推理时显式指定发音
2. 增加人工 MOS/CMOS 评测，特别是日语母语者对自然度的判断
3. 扩展 Kana-CER 到中文（拼音-CER）、韩语等有类似多音字问题的语言
4. 探索更大参数量的 backbone LLM（当前仅 0.5B）对罕见读法泛化的影响
5. 流式推理支持——当前架构不支持流式

### 可复现性评估
- [x] 代码开源（推理代码 + 评测脚本）
- [x] 预训练模型（HuggingFace 可下载）
- [ ] 训练代码（未开源）
- [ ] 训练细节完整（Stage 1/2 的具体 batch size、步数、硬件未报告）
- [ ] 数据集可获取（训练数据为正版授权/公共领域，但列表未公开）

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §codec-LM TTS / Speech-LLM TTS（架构复用 CosyVoice 2 流水线）
- **核心问题**：[[TTS-核心挑战]] §多语言能力（日语汉字多音字）；[[TTS-评测体系]] §发音准确率评测方法论
- **表示层位置**：[[TTS-表示层地图]] §语义 token（S3Tokenizer V2，单码本 FSQ，25Hz）
- **相邻工作**：[[CosyVoice2]]（架构来源）/ [[Qwen3-TTS]]（基线对比）/ [[FireRedTTS2]]（基线对比）/ T5Gemma-TTS

## 🔄 后续重估

- **2026-06-26**：初读。核心价值在数据策略和评测方法论而非模型架构创新。PronSteering + Joyo Kanji Yomi Benchmark 对日语 TTS 社区有基础设施意义。跨提示鲁棒性结论（日语数据占比 53.7% → 不退化）有启发性但需更大规模验证。SB Intuitions 作为 SoftBank 子公司有日语数据资源优势。限制：0.5B 参数量偏小、无流式支持、非商用许可。

---

## 关联笔记

### 基于
- [[CosyVoice2]]: 声学阶段（CFM decoder + HiFT-GAN + S3Tokenizer V2）直接采用
- [[Sarashina2.2-0.5B-Instruct]]: backbone LLM 初始化来源

### 对比
- [[Qwen3-TTS]]: 日语表现不稳定的多语种 LLM-TTS 基线
- [[FireRedTTS2]]: 小红书开源 TTS 基线
- [[Fish-Speech]]: FishAudio S1-mini 基线

### 方法相关
- [[S3Tokenizer]]: 语义 token 提取器（CosyVoice 系列的 speech tokenizer）
- [[Flow Matching]]: 声学阶段的生成范式
- [[CAM++]]: 说话人嵌入提取模型（3D-Speaker 项目）
- [[HiFi-GAN]]: HiFT-GAN vocoder 的基础

### 硬件/数据相关
- JSUT: 日语语音语料库评测集
- Joyo Kanji Yomi Benchmark: 本文提出的汉字读音评测基准

---

## 速查卡片

> [!summary] Sarashina2.2-TTS
> - **核心**: 日语中心 LLM-TTS，通过数据策略+评测方法论解决汉字多音字问题
> - **方法**: CosyVoice 2 架构 + 日语 LLM warm-start + 361k h 训练 + PronSteering 定向增强
> - **结果**: Kana-CER†_kanji 5.45（次优 8.55）；唯一跨语言提示不退化的系统；SIM 75.64 最高（限所报告基线）
> - **代码**: [GitHub](https://github.com/sbintuitions/sarashina2.2-tts) / [Benchmark](https://github.com/sbintuitions/JoyoKanji-Yomi-Benchmark)

---

*笔记创建时间: 2026-06-26*
