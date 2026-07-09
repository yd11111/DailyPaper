---
title: "Unified Audio Intelligence Without Regressing on Text Intelligence"
method_name: "Audex"
authors: [Zhifeng Kong, Sang-gil Lee, Jaehyeon Kim, Boxin Wang, Zihan Liu, Sungwon Kim, Yang Chen, Arushi Goel, Rajarshi Roy, Wenliang Dai, Zhuolin Yang, Yangyi Chen, Dongfu Jiang, Sreyan Ghosh, Tuomas Rintamaki, Andrew Tao, Jonathan Raiman, Mohammad Shoeybi, Bryan Catanzaro, Wei Ping]
year: 2026
venue: arXiv
arxiv_id: "2607.05196"
tags: [speech-llm, unified-audio-text, audio-understanding, tts, asr, audio-generation, moe, cascade-rl]
zotero_collection:
note_tier: heavy

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: true
post_training_type: custom
streaming: partial

# === 知识地图联动 ===
domain: SpeechLM
subdomain: unified-audio-text-llm
routes: [speech-llm-unified, codec-lm-tts, audio-understanding]
problems: [text-regression, unified-generation-understanding, audio-generation-quality]
representations: [acoustic-token, continuous-encoder]
related_maps:
  - "[[SpeechLM-技术路线图]]"
  - "[[TTS-技术路线图]]"
  - "[[SpeechLM-核心挑战]]"
related_surveys: []
evidence_level: high
maturity: emerging
last_repositioned: 2026-07-09

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2607.05196/paper.pdf"
html_local:
figures_dir: "_resources/2607.05196/figures"
github_local:
cached_at: 2026-07-09

# === 通用元数据 ===
image_source: online
arxiv_html:
created: 2026-07-09
---

# 论文笔记：Unified Audio Intelligence Without Regressing on Text Intelligence

> **笔记分级**：heavy（工业系统报告：NVIDIA 30B MoE 统一音频-文本 LLM，含完整训练管线 + Cascade RL + 多任务评测）。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | NVIDIA |
| 日期 | July 2026 |
| 项目主页 | — |
| 对比基线 | [[Step-Audio]], [[Qwen3-Omni]], [[MiMo]], [[Kimi-Audio]], [[UALM]] |
| 链接 | [arXiv](https://arxiv.org/abs/2607.05196) / [HuggingFace](https://huggingface.co/collections/nvidia/nemotron-labs-audex) |

## 一句话总结

> 基于 Nemotron-Cascade-2 MoE 骨架构建统一音频-文本 LLM，通过多阶段 SFT + 纯文本 Cascade RL，在音频理解/ASR/TTS/音频生成/S2S 全面达到开源最强水平，同时几乎不损失文本推理能力。

---

# 一、阅读层（主文）

## 核心贡献

1. **统一架构不退化文本**：单一 MoE Transformer decoder 同时处理音频输入/输出和文本，在文本推理 benchmark 上与纯文本骨架 Nemotron-Cascade-2 高度一致（AIME 2025 达 91.2|98.3，MMLU-Pro 78.9）
2. **全面音频 SOTA**：在 ASR (OpenASR 6.82 avg WER)、音频理解 (MMAU 75.6)、TTS (Seed-TTS-Eval WER 1.70)、音频生成 (AudioCaps FD 66.9)、语音交互 (BigBenchAudio 90.0) 多维度达到或超越专用模型
3. **训练方法论**：系统验证了"多阶段 SFT + 纯文本 Cascade RL"策略对保持文本能力的有效性，并消融了 audio warmup 冻结策略、数据混合比、CFG 超参等关键设计

## 问题背景

### 要解决的问题

现有多模态 LLM（尤其支持音频输出的）在获得音频能力后，文本推理/知识/对齐能力显著退化。例如 [[Qwen3-Omni]] 和 Qwen3.5-Omni 在推理 benchmark 上相比其纯文本骨架 Qwen3 / Qwen3.5 明显下降。

### 现有方法的局限

- 级联系统（ASR + LLM + TTS）延迟高、无法端到端优化
- 统一模型（如 Qwen3-Omni、Step-Audio）虽然支持音频输入输出，但文本推理能力退化严重
- 音频生成和音频理解在同一模型中统一仍然困难——生成需要精细声学细节，理解只需语义特征

### 本文的动机

NVIDIA 认为关键在于 (1) 正确的多阶段训练策略（先稳定文本再加音频），(2) 纯文本 Cascade RL 不破坏已有音频能力，(3) 适当的 audio warmup 冻结策略防止文本 embedding 被干扰。

## 方法详解

### 领域定位

Audex 属于 **统一音频-文本 LLM** 路线，与 [[UALM]]（NVIDIA 前作）、[[Qwen3-Omni]]、[[Step-Audio]] 同类。核心差异在于：(1) 使用比 Qwen/Step 更强的 MoE 文本骨架，(2) 通过多阶段 SFT + 纯文本 RL 系统性解决文本退化问题，(3) 分离 speech codec 和 audio codec 的双通道设计。

### 端到端数据流（先地图后街景）

Audex 的完整流水线：

**音频输入路径**：音频 waveform → **AF-Whisper 编码器**（提取 25Hz 连续特征）→ **2层 MLP 适配器**（投影到 LLM 维度）→ 与文本 token 一起送入 LLM

**文本/音频输出路径**：LLM 自回归生成 → 输出文本 token 或离散音频 token（`<speechcodec_{id}>` / `<audiocodec_{id}>`）→ **Speech Decoder**（X-Codec2 解码器或流式 ConvNeXt-Vocos）/ **Audio Decoder**（X-Codec 解码器 + Enhancement VAE）→ 最终波形

![[_resources/2607.05196/figures/fig-000.png]]

> **Figure 1**：Audex-30B-A3B 架构。底部音频编码器（AF-Whisper）+ MLP 将音频转为连续 embedding 送入 LLM；LLM 同时预测文本 token 和音频 token；顶部 Speech Decoder / Audio Decoder 将离散 token 解码为波形。核心设计：音频 token 与文本 token 在同一词表中统一自回归生成。

### LLM 骨架：Nemotron-Cascade-2-30B-A3B

**为什么这样设计**：选择 MoE 而非 dense 模型，因为 30B 总参数但仅 3B 激活参数，推理高效但知识容量大；Mamba-Transformer hybrid 进一步提升长序列效率。

**架构规格**：
- 52 层，模型维度 2688
- 128 个可路由专家，每次激活 6 个
- Mamba2-Transformer 混合架构
- 上下文长度 1M token
- 词表从 131,072 扩展到 205,312（加入 65,536 speech token + 8,192 audio token + 特殊控制 token）

初始化来自 Nemotron-Cascade-2 post-SFT checkpoint，新增的音频 token embedding 用均值 0、标准差 0.02 的高斯随机初始化。

### 音频编码器：AF-Whisper

**为什么选择 AF-Whisper**：它基于 [[Whisper]] Large-v3 架构并由 Audio Flamingo 3 微调，既有语音识别能力又扩展到通用音频理解。

**规格**：
- 输入：16kHz 音频，每 30 秒一窗
- 输出：25Hz 连续特征（30 秒 = 750 帧），维度 1280
- 通过 2 层 MLP 适配器投影到 LLM 模型维度 2688

### 音频 Codec：双通道设计

**为什么分两个 codec**：语音和非语音音频的性质不同——语音可以用单层低码率量化器高效表示（语言结构性强），而通用音频（音乐/环境音）声学结构更复杂，需要多层 RVQ 才能重建质量好。

**Speech Codec — X-Codec2**：
- 50Hz 帧率，单层 [[FSQ]]，码本大小 65,536
- 即 50 token/s，词表 65,536
- 专为语音设计：低码率但语义清晰

**Audio Codec — [[X-Codec]]**：
- 50Hz 帧率，8 层 [[RVQ]]，每层码本 1,024
- 实际使用前 4 层，展平为 $50 \times 4 = 200$ token/s，词表 $1024 \times 4 = 4096$
- 通用音频（音乐/音效）设计

**音频解码增强**：
- 语音：流式 ConvNeXt-Vocos 解码器替代 X-Codec2 非因果解码器，支持流式推理
- 通用音频：Enhancement VAE（基于 [[BigVGAN]]-v2）将采样率从 16kHz 提升到 48kHz 并减少 codec artifact

### 训练流程

训练分两大阶段：**多阶段 SFT** + **Cascade RL**。

#### 多阶段 SFT（推荐策略）

**Stage 1 — Text SFT**：直接复用 Nemotron-Cascade-2 的文本 SFT checkpoint（不重新训练）

**Stage 2 — Audio Warmup**：
- 扩展词表，加入音频编码器 + MLP 适配器 + 初始化音频 token embedding
- **关键设计**：冻结文本 token embedding，只训 MLP + 音频 token embedding
- 冻结 LLM 权重
- 目的：让音频 embedding 进入 LLM 能理解的空间，同时不干扰文本

**具体例子**：如果不冻结文本 embedding（消融实验），MMLU-Pro 从 78.9 骤降到 71.2，AIME 2025 从 93.2 降到 71.8——说明文本 embedding 对文本能力极度敏感。

**Stage 3 — Audio Generation SFT**：
- 解冻 LLM，联合训练音频/语音生成 + 文本任务
- 音频编码器和 MLP 不参与训练（生成不需要音频输入）
- 文本数据占比 59%

**Stage 4 — Audio Gen + Audio Understanding SFT**：
- 解冻 MLP 适配器 + LLM
- 加入 ASR、AST、音频理解任务
- 音频编码器始终冻结
- 文本数据占比 69%

#### Cascade RL（纯文本）

选择多阶段 SFT checkpoint 后，进行纯文本 Cascade RL：
1. Instruction-Following RL
2. Multi-domain RL
3. Multi-domain On-policy Distillation (MOPD)
4. RLHF
5. Long-context RL

**关键发现**：纯文本 RL 显著提升文本 benchmark（ArenaHard 从 59.4 → 81.6，tau2-Bench 从 52.4 → 57.2），而音频任务几乎不退化（ASR/TTS/AudioCaps/MMAU 基本稳定）。这验证了 Nemotron-Cascade 的核心洞察：文本 RL 对音频能力是安全的。

### 推理流程

- **音频理解/ASR/AST**：音频 QA 格式，音频编码 → LLM 生成文本回答，Top-p=0.9, temp=0.7
- **TTS**：用户发 `<|text to speech|> Generate speech for this transcription. {TEXT}`，LLM 输出 `<speechgen_start><speechcodec_X>...<speechgen_end>`，Top-k=80, temp=0.1, CFG lambda=1.5
- **TTA**：`<|text to audio|> Generate audio for this caption. {CAPTION}`，LLM 输出 `<audiogen_start><audiocodec_X>...<audiogen_end>`，Top-k=80, temp=1.0, CFG lambda=3
- **Speech-to-Speech**：级联 pipeline（ASR → 文本推理 → TTS），但全程用同一个 Audex 模型

[[Classifier-Free Guidance]] 实现方式：训练时随机选 10% 生成样本将 transcription/caption 替换为 padding token（等效于条件 dropout），推理时同 batch 生成条件/无条件两路并合并 logits。

## 关键结果

> 只列支撑主结论的核心表/图。完整数据见附录。

**核心证据——Table 1 (Main Results)**：Audex vs 文本骨架 + 多个 SOTA 模型

| Benchmark | Nemotron-Cascade-2 30B-A3B | Qwen3-Omni 30B-A3B Thinking | Qwen3.5-Omni-Flash | **Audex 30B-A3B** |
|---|---|---|---|---|
| AIME 2025 | 92.4\|98.6 | 91.9 | — | **91.2\|98.3** |
| MMLU-Pro | 79.8 | 80.4 | 79.9 | **78.9** |
| ArenaHard v2 | 83.5 | 65.4 | — | **81.6** |
| NIAH (256K\|1M) | 100\|99.0 | 0.0\|0.0 | — | **99.4\|83.4** |
| OpenASR (avg) | N/S | 8.00 | — | **6.82** |
| Seed-TTS-Eval WER | N/S | N/S | — | **1.70** |
| MMAU | N/S | 75.4 | 80.4 | **75.6** |
| BigBenchAudio | N/S | — | 59.0 | **90.0** |
| AudioCaps FD | N/S | N/S | N/S | **66.9** |

**结论**：Audex 在文本推理上与 Nemotron-Cascade-2 骨架几乎无差（AIME 差 1.2 点在方差内；MMLU-Pro 差 0.9），而 Qwen3-Omni 在 NIAH 长上下文上几乎完全崩溃（0.0 vs 原骨架 100）。音频任务全面达到或超越对标开源模型。

## 可复用的设计模式

1. **冻结文本 embedding 的 Audio Warmup**：加入新模态 token 时冻结原有文本 embedding，只训新 embedding + adapter。适用于任何在已有 LLM 上扩展新模态的场景。来自 Stage 2 设计（消融证实解冻会导致文本能力崩溃 7+ 点）。

2. **纯文本 RL 不损音频能力**：多模态 SFT 后执行纯文本 Cascade RL，文本能力显著提升而音频能力几乎无回归。适用于希望在统一模型中持续提升文本能力的场景。来自 §4.4.3 的 RL 阶段结论。

3. **分离语音/音频双 codec 设计**：语音用低码率单层 FSQ（50 token/s），通用音频用多层 RVQ 展平（200 token/s）。适用于需要同时处理语音和非语音音频生成的统一模型。来自 §3.3 的设计动机。

4. **多阶段 SFT 优于单阶段**：逐步加入任务（text → audio warmup → generation → understanding）比一次性混合所有数据更稳定，尤其在长上下文能力保持上。适用于多任务多模态训练。来自 §4.4.2 消融。

5. **Text 数据高占比防退化**：在音频相关 SFT stage 保持 59-75% 的文本数据占比，辅以 scaled loss 平衡长短序列。适用于任何多模态混合训练。来自 §4.3 + Appendix C.2。

---

# 二、研究/审计层（附录）

## 📋 核验结论（技术元数据）

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start from Nemotron-Cascade-2-30B-A3B post-SFT checkpoint | [已 verify §3.1] |
| 训练 loss | 标准 cross-entropy (next token prediction)，附 scaled loss (平方根长度归一化) + MoE auxiliary load balancing loss (系数 1e-6) | [已 verify §4.3, Eq.1-2] |
| Tokenizer 架构 | text + speech 分离词表（65536 speech + 8192 audio token 追加到原词表），生成时统一自回归预测 | [已 verify §3.3] |
| 多任务 | true：ASR + AST + TTS + TTA + 音频理解 + 文本 SFT，分阶段引入 | [已 verify §4.1, Tab.2] |
| 训练数据 | 157.4B audio tokens + 320.5B text tokens = 477.9B total；1,092K 音频小时；394M 样本 | [已 verify Tab.2] |
| 后训练 | Cascade RL（纯文本）：IF-RL → Multi-domain RL → MOPD → RLHF → Long-context RL | [已 verify §4.4.3] |
| Speech Codec | X-Codec2：50Hz，单层 FSQ，码本 65536 | [已 verify §3.3] |
| Audio Codec | X-Codec：50Hz，8 层 RVQ（实用 4 层），每层码本 1024，展平 200 token/s | [已 verify §3.3] |
| 推理 CFG | TTS: lambda=1.5, Top-k=80, temp=0.1; TTA: lambda=3, Top-k=80, temp=1.0 | [已 verify §5.2, Fig.4, Appendix C.3] |
| 基础设施 | 512 NVIDIA H100 GPU，4-way TP，32-way EP，8-way CP，BF16，Megatron-LM | [已 verify §4.5] |

## 完整公式

### 公式1: [[Cross-Entropy Loss|标准 LLM Loss]]

$$
\max_{\theta} \left\{ \frac{1}{\#\text{Trainable tokens}} \sum_{i:\ x_i\ \text{is trainable}} \log p_\theta(x_i | x_{<i}) \right\}
$$

**含义**：标准自回归交叉熵 loss，仅对 trainable token（文本/音频输出）计算。

**符号说明**：
- $x_i$: 序列中第 $i$ 个 token
- $p_\theta(x_i|x_{<i})$: 模型预测第 $i$ 个 token 的条件概率
- Trainable tokens: 排除 prompt/system 等不计算 loss 的位置

### 公式2: Scaled-Loss（平方根长度归一化）

$$
\text{Scaled-Loss}(\{L_1, \cdots, L_k\}) = \left( \sum_{j=1}^{k} \frac{\text{sum}(L_j)}{\sqrt{|L_j|}} \right) \cdot \left( \sum_{j=1}^{k} \sqrt{|L_j|} \right)^{-1}
$$

**含义**：在 packed sequence 训练中，短样本容易被长样本淹没，本公式用 $\sqrt{|L_j|}$ 归一化使不同长度样本贡献更均衡。

**符号说明**：
- $L_j$: 第 $j$ 个样本的逐 token loss 向量
- $|L_j|$: 第 $j$ 个样本的长度
- $k$: 一个 packed sequence 中的样本数

### 公式3: [[Classifier-Free Guidance|CFG 推理]]

$$
p_\theta^{\text{CFG}}(\cdot|x_{<i}) = p_\theta(\cdot|x_{<i}) + (\lambda - 1) \cdot [p_\theta(\cdot|x_{<i}) - p_\theta(\cdot|\emptyset)]
$$

**含义**：推理时将条件生成与无条件生成的 logits 差值放大 $\lambda-1$ 倍叠加，增强文本条件对生成的控制力。

**符号说明**：
- $\lambda$: CFG 强度（TTS 最优 1.5，TTA 最优 3）
- $\emptyset$: 无条件输入（将 transcription/caption 替换为等长 padding）
- $\lambda=1$ 时等价于无 CFG

## 完整图表

### Figure 3: 训练管线

![[_resources/2607.05196/figures/fig-001.png]]

> **说明**：上半部展示两种 SFT 策略——多阶段（逐步加任务）和单阶段合并（一次混合所有数据）。两者都先经过 Audio Warmup。下半部展示 RL 阶段：从 Audex SFT → Cascade-2 RL + MOPD → 最终 Audex。

### Table 2: 数据统计

| Task | # samples | audio hours | audio tokens | text tokens | total tokens |
|---|---|---|---|---|---|
| Text-to-Speech (TTS) | 147M | 421K | 75.8B | 5.2B | 81.0B |
| Text-to-Audio (TTA) | 12M | 34K | 24.3B | 0.4B | 24.7B |
| Audio Understanding | 49M | 308K | 27.7B | 2.2B | 29.9B |
| Speech Recognition (ASR) | 95M | 201K | 18.1B | 4.4B | 22.5B |
| Speech Translation (AST) | 58M | 128K | 11.5B | 5.3B | 16.8B |
| Text SFT | 33M | 0 | 0 | 303.0B | 303.0B |
| **Total** | **394M** | **1,092K** | **157.4B** | **320.5B** | **477.9B** |

### Table 3: 多阶段 SFT 数据混合 + 超参

**数据混合比 (epochs)**：

| Data Type | Audio Warmup | Audio Gen. | Audio Gen. + Und. |
|---|---|---|---|
| Text | 0.5 | 0.5 | 1.5 |
| ASR + AST | 0.0 | 0.0 | 1.0 |
| Audio understanding | 0.0 | 2.0 | 2.0 |
| TTS | 0.0 | 1.0 | 1.0 |
| TTA | 0.0 | 2.0 | 1.0 |
| Text weights (%) | 44 | 59 | 69 |

**超参**：Global BS=64; Max LR 2e-5 → 2e-3 (warmup) / 2e-5 (others); Min LR 1e-6; Cosine scheduler; AdamW (0.9, 0.999)

### Table 5: 文本 Benchmark 结果

| Benchmark | Step-Audio R1.1 33B | Qwen3-Omni Thinking | Qwen3.5-Omni-Flash | **Audex 30B-A3B** | **Audex 2B** |
|---|---|---|---|---|---|
| AIME 2025 | 44.8 | 73.7 | — | **91.2\|98.3** | 68.9\|68.3 |
| AIME 2026 | 57.8 | — | — | **89.4\|96.6** | — |
| HMMT Feb25 | 27.0 | 60.4 | 59.0 | **92.2\|93.8** | 68.5\|67.3 |
| LiveCodeBench v6 | 22.9 | 59.2 | 56.6 | **85.3\|86.2** | 64.5\|72.7 |
| MMLU-Pro | 75.3 | 80.4 | 79.9 | **78.9** | 57.0 |
| ArenaHard v2 | 44.3 | 55.1 | — | **81.6** | 11.9 |
| NIAH (256K\|1M) | N/S | 0.0\|0.0 | — | **99.4\|83.4** | — |

### Table 6: TTS 结果 (Seed-TTS-Eval English)

| Models | WER (prompting) | SIM | WER (fixed voice) |
|---|---|---|---|
| Seed-TTS | 2.25 | 76.2 | — |
| Step-Audio-TTS-3B | 2.31 | 66.0 | — |
| F5 TTS | 1.83 | 64.7 | — |
| Qwen3-Omni-30B-A3B-Instruct | — | — | 1.39 |
| **Audex 30B-A3B** | **2.07** | **45.3** | **1.70** |
| **Audex 2B** | — | — | 2.04 |

### Table 7: TTA 结果 (FD_openL3)

| Models | Model Type | AudioCaps | SongDescriber |
|---|---|---|---|
| UALM | LLM | 65.9 | 83.7 |
| UALM-Gen | Autoregressive | 75.1 | 74.4 |
| **Audex 30B-A3B** | LLM | **66.9** | **62.7** |
| **Audex 2B** | LLM | 79.3 | 78.4 |

### Table 8: OpenASR 结果 (WER)

| Models | LS-clean | LS-other | AMI | Earnings22 | GigaSpeech | SPGI | TED-LIUM | VoxPopuli | Average |
|---|---|---|---|---|---|---|---|---|---|
| Whisper-large-v3 | 2.01 | 3.91 | 15.95 | 11.29 | 10.02 | 2.94 | 3.86 | 9.54 | 7.44 |
| Qwen3-Omni-Instruct | 1.28 | 2.52 | 11.44 | 10.47 | 8.79 | 2.44 | 2.78 | 6.07 | 5.72 |
| **Audex 30B-A3B** | **1.34** | **3.06** | **17.24** | **11.92** | **9.90** | **1.76** | **3.50** | **5.83** | **6.82** |

### Table 13: BigBenchAudio

| Models | Score |
|---|---|
| Step-Audio R1.1 Realtime | 97.6 |
| **Audex 30B-A3B** | **90.0** |
| Audex 2B | 64.3 |
| Qwen3.5 Omni Flash Realtime | 59.0 |
| Moshi | 4.4 |

### Table 16: Audio Warmup 消融

| Warmup Method | MMLU-Pro | GPQA-Diamond | AIME 2025 |
|---|---|---|---|
| **Freeze text embeddings (Audex)** | **78.9** | **73.8** | **93.2** |
| Unfreeze all embeddings | 71.2 | 62.3 | 71.8 |
| Freeze then unfreeze | 65.1 | — | — |
| Unfreeze + higher text weight | 59.7 | — | — |

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | ASR (OpenASR leaderboard), 文本推理 (AIME/HMMT/MMLU)，BigBenchAudio | 公开 leaderboard / 标准化 benchmark / 可复现设置 / checkpoint 已开源 |
| **中** | TTS (Seed-TTS-Eval WER)，AudioCaps/SongDescriber FD | 标准评测但 SIM 较低（45.3）需关注 / FD metric 依赖参考分布 |
| **低** | VoiceBench，Speech-to-Speech | 级联 pipeline 评测（非端到端）/ 部分结果从 leaderboard 取而非标准化对比 |

## 核心 Claim 审查

1. **Paper Claim**：Audex achieves "marginal or no regression" on text intelligence
   **My Assessment**：在报告的 benchmark 集合上基本成立——AIME/MMLU-Pro/GPQA 差距在 1-2 点内（可能在方差内）。但 ArenaHard (81.6 vs 83.5) 有 1.9 点差距，NIAH@1M (83.4 vs 99.0) 有 15.6 点差距。"No regression" 对长上下文场景过于乐观。

2. **Paper Claim**：Audex is the only model supporting general audio generation beyond speech
   **My Assessment**：在对比的模型集合中确实如此——Qwen3-Omni/Step-Audio 不支持 TTA。但表述不含私有模型（GPT-4o 能力不透明），应理解为"开源模型中唯一"。

3. **Paper Claim**：Text-only Cascade RL does not degrade audio abilities
   **My Assessment**：Table 15 (Appendix B) 确认 ASR/TTS/音频理解在 RL 各阶段基本稳定。唯一异常是 MOPD 阶段 ASR 轻微退化后在后续 RL 恢复。总体可信。

4. **Paper Claim**：TTS WER 1.70 on Seed-TTS-Eval
   **My Assessment**：这是 fixed-voice TTS（固定音色，非 zero-shot cloning）。SIM 只有 45.3（vs Seed-TTS 76.2），说明 zero-shot 音色克隆能力弱。作者自己承认 codec 选择和训练 recipe 未优化 SIM。

## 批判性思考

### 优点
1. **工程完整度极高**：从骨架选择、codec 设计、多阶段训练、RL 策略到消融研究，每个环节都有系统论证，reproducibility 强（开源 checkpoint）
2. **文本退化问题得到有说服力的解决**：消融实验（Table 16, Figure 5-7）系统验证了冻结策略和数据比的关键性
3. **Cascade RL 洞察有广泛影响**：证明"文本 RL 不损音频"为所有多模态 LLM 提供了后训练路径

### 局限性
1. **Zero-shot TTS 能力不足**：SIM 仅 45.3 vs Seed-TTS 76.2，说明音色克隆远未达到专用系统水平；论文只报 fixed-voice WER 避开了这个弱点
2. **音频理解在部分细分上有差距**：MMAR (63.2 vs Audio-Flamingo 60.1) 和 MMSU (63.4 vs Qwen3.5 72.2) 并非全面占优
3. **全部是非流式/级联评测**：Speech-to-Speech 用 ASR→推理→TTS 级联，不支持端到端/流式对话，延迟数据完全未报告
4. **TTA 固定 10 秒输出**：无法生成变长音频，与 diffusion 基线（可控时长）相比是硬限制
5. **RL 仅纯文本**：作者明确承认 audio-text RL 是 future work——当前 RL 不能直接提升音频质量

### 潜在改进方向
1. 引入 speech/audio RL（如 reward model 基于 UTMOS/DNSMOS）直接优化生成质量
2. 加入 speaker similarity loss 或 speaker-conditioned training 提升 zero-shot cloning
3. 支持端到端 streaming / duplex 交互
4. 变长音频生成训练

### 可复现性评估
- [x] 代码开源（Megatron-LM 框架）
- [x] 预训练模型（HuggingFace: nemotron-labs-audex-30b-a3b + audex-2b）
- [x] 训练细节完整（多阶段超参、数据比、消融全部披露）
- [ ] 数据集可获取（部分公开语料 + permissive license，但具体筛选/处理未全部公开）

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[SpeechLM-领域总览]]
- **技术路线**：[[SpeechLM-技术路线图]] §统一理解+生成 LLM
- **核心问题**：[[SpeechLM-核心挑战]] §多模态能力不损文本推理
- **表示层位置**：[[SpeechLM-表示层地图]] §混合表示（连续编码器输入 + 离散 token 输出）
- **相邻工作**：[[UALM]]（前作，7B）/ [[Qwen3-Omni]]（对标）/ [[Step-Audio]]（对标）/ [[MiMo]]（对标）

## 🔄 后续重估

- **2026-07-09**：初读。当前开源统一音频 LLM 中文本能力保持最佳的方案，工程贡献大于方法创新。关键限制是 TTS 音色克隆弱（SIM 45.3）且不支持流式对话。后续需关注 audio RL 是否能弥补生成质量差距。

---

## 关联笔记

### 基于
- [[UALM]]: NVIDIA 前作（1.5B/7B），本文直接继承其 TTA 技术路线（BPE token + CFG）
- [[Nemotron-Cascade-2]]: 文本骨架 LLM（30B MoE，3B activated）

### 对比
- [[Qwen3-Omni]]: 阿里统一音频 LLM，文本退化更严重
- [[Step-Audio]]: 阶跃星辰 130B，不支持通用音频生成
- [[Kimi-Audio]]: 月之暗面音频 LLM

### 方法相关
- [[X-Codec]]: 通用音频 codec（8-layer RVQ）
- [[Classifier-Free Guidance]]: 音频生成质量控制的关键推理技巧
- [[Whisper]]: 音频编码器基于 Whisper Large-v3 架构

### 硬件/数据相关
- [[Megatron-LM]]: 训练框架
- [[Seed-TTS-Eval]]: TTS 评测集

---

## 速查卡片

> [!summary] Audex: Unified Audio Intelligence Without Regressing on Text Intelligence
> - **核心**: 30B MoE 统一音频-文本 LLM，多阶段 SFT + 纯文本 Cascade RL 保持文本推理能力
> - **方法**: AF-Whisper 编码 + 双 codec (X-Codec2 speech + X-Codec audio) 生成 + 统一自回归预测
> - **结果**: 文本推理与骨架几乎无差（AIME 91.2）；ASR/TTS/音频理解/TTA/S2S 全面开源最强（有条件限定）
> - **代码**: [HuggingFace](https://huggingface.co/collections/nvidia/nemotron-labs-audex)

---

*笔记创建时间: 2026-07-09*
