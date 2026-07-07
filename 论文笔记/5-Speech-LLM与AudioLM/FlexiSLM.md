---
title: "FlexiSLM: A Dynamic and Controllable Frame Rate Spoken Language Model"
method_name: "FlexiSLM"
authors: [Jiaqi Li, Chaoren Wang, Xiaohai Tian, Mingjie Chen, Xinyu Liang, Xu Li, Yufan Lin, Junwen Qiu, Jun Zhang, Lu Lu, Haizhou Li, Zhizheng Wu]
year: 2026
venue: arXiv
arxiv_id: "2606.31247"
tags: [speech-llm, dynamic-frame-rate, frame-merging, flexicodec, fsq, spoken-language-model, thinker-talker]
zotero_collection:
note_tier: standard

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: true
post_training_type: none
streaming: false

# === 知识地图联动 ===
domain: SpeechLM
subdomain: dynamic-frame-rate-slm
routes: [speech-llm-tts, codec-lm-tts]
problems: [latency, codec-design, evaluation, dialogue-integration]
representations: [semantic-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-SpeechLM-Dialogue关系]]"
  - "[[TTS-表示层地图]]"
  - "[[TTS-核心挑战]]"
related_surveys: []
evidence_level: medium
maturity: exploratory
last_repositioned: 2026-07-07

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.31247/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.31247/paper.html"
figures_dir: "_resources/2606.31247/figures"
github_local:
cached_at: 2026-07-07

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2606.31247
created: 2026-07-07
---

# 论文笔记：FlexiSLM: A Dynamic and Controllable Frame Rate Spoken Language Model

> **笔记分级**：standard（方法清晰、值得精读的 SLM 框架论文，动态帧率是该领域首次系统性探索）。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | The Chinese University of Hong Kong, Shenzhen (CUHK-SZ) + ByteDance |
| 日期 | June 2026 |
| 项目主页 | [flexislm.github.io](https://flexislm.github.io) |
| 对比基线 | [[Qwen2.5-Omni]], [[Kimi-Audio]], Mimo-Audio, Qwen2-Audio, Fun-Audio-Chat, [[GLM-4-Voice]] |
| 链接 | [arXiv](https://arxiv.org/abs/2606.31247) / [Code (planned)](https://github.com/AmphionTeam/FlexiSLM) |

## 一句话总结

> 首个支持动态与可控帧率的 SLM 框架：通过 Frame Merging Module 自适应压缩语音帧，结合直接帧率条件化实现 4.0~12.5 Hz 连续可控，在高质量点优于 Qwen2.5-Omni 和 Kimi-Audio 等 7B 固定帧率基线。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 必要图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源 [§X] 不写在这里，统一收到「附录·核验结论」表。

## 核心贡献

1. **动态帧率 SLM 框架**：首次将动态帧率压缩同时应用于 SLM 的语音输入和输出侧，单一模型覆盖 4.0~12.5 Hz 帧率范围
2. **直接帧率条件化**：用户直接指定目标平均帧率（而非间接调节合并阈值），控制精度在 0.1 Hz 以内，实现可预测的推理加速
3. **FlexiSLM-Data 对话语料**：构建 1.4M 样本 / 9.9K 小时的 speech-to-speech 对话数据集，从 Qwen3-Omni 30B 蒸馏而成

## 问题背景

### 要解决的问题

现有 SLM（如 [[Qwen2.5-Omni]] 25 Hz、[[Kimi-Audio]] 12.5 Hz、[[GLM-4-Voice]] 12.5 Hz、[[Moshi]]）均使用固定帧率的语音 token 表示。这带来两个问题：（1）忽略语音信号时变的信息密度——静音段和信息稀疏段浪费了计算；（2）推理时无法在质量和速度之间灵活权衡。

### 现有方法的局限

动态帧率 codec 已有探索（[[FlexiCodec]]、CodecSlime、TFC、VARSTok），但此前仅在 0.3B TTS pipeline 中验证，从未被集成到端到端 SLM 框架中。此外，已有的帧率控制依赖合并阈值 $\tau$，存在结果帧率方差大、用户不直观等问题。

### 本文的动机

如果语音 token 序列能根据内容复杂度自适应地"变长变短"，SLM 就能在不重训的情况下覆盖多种帧率工作点——高帧率换高质量，低帧率换低延迟。这对异构部署场景（云端 vs 端侧）尤其实用。

## 方法详解

### 领域定位

FlexiSLM 属于 **Thinker-Talker SLM** 路线（与 [[Qwen2.5-Omni]] 同类架构范式），与 [[Kimi-Audio]]（并行 speech-text LM head）同属端到端 SLM。核心差异在于**首次在 SLM 中引入动态帧率**：输入和输出两侧均可自适应调节语音 token 序列长度。

### 端到端数据流（先地图后街景）

FlexiSLM 的完整流水线：用户语音输入 → **Audio Encoder**（Qwen2.5-Omni encoder，提取 25 Hz 连续特征）→ **Frame Merging Module**（自适应压缩至 <=12.5 Hz 动态帧率）→ **Thinker LLM Backbone**（Qwen2.5-7B-Instruct，理解语义+生成文本）→ **Talker Transformer**（解码 Thinker 隐状态为 FlexiCodec FSQ token + 帧长度）→ **Audio Decoder**（NAR Flow-Matching + [[Vocos]] 声码器，token→mel→24 kHz 波形）。

![[_resources/2606.31247/figures/fig-016.png]]

> **Figure 2**：FlexiSLM 整体架构。左侧 Thinker-Talker 主干：语音输入经 Audio Encoder（25 Hz）→ Frame Merging Module（动态压缩）→ Thinker LLM（文本理解+生成）→ Talker Transformer（生成 FlexiCodec token + 帧长度）→ Audio Decoder 输出波形。右侧 (a) 展示 FlexiCodec 语义编码分支的 Frame Merging + FSQ 量化流程；(b) 展示 Frame Merging Module 的逐帧余弦相似度判定+合并机制。红色虚线箭头为可选的 Talker-to-Thinker 反馈连接（Stage 3 激活）。

下面逐个放大每个关键模块。

### Frame Merging Module

**为什么这样设计**：语音信号的信息密度随时间变化——元音稳态段相邻帧高度相似，而辅音/转折处帧间差异大。固定帧率对所有段一视同仁，浪费了计算预算。Frame Merging 利用相邻帧余弦相似度来判断是否合并，实现"信息密度自适应压缩"。

**怎么做**：给定 $T$ 个固定帧率特征 $\mathbf{x}_1, \dots, \mathbf{x}_T$，计算相邻帧余弦相似度 $s_t$。若 $s_t > \tau$（合并阈值），则将 $\mathbf{x}_t$ 和 $\mathbf{x}_{t+1}$ 合并为均值。贪心从左到右执行，连续高相似帧被合并为单个平均表示。合并后每组产出一个平均特征 $\bar{\mathbf{x}}_k$ 和帧长度属性 $l_k$。原始帧与平均帧交错排列后，经一个**轻量级局部注意力 Transformer** 处理，最终取平均特征位置的输出作为合并序列。

**具体例子**：假设 4 帧输入 $[\mathbf{x}_1, \mathbf{x}_2, \mathbf{x}_3, \mathbf{x}_4]$，余弦相似度为 $[0.33, 0.95, 0.60]$，阈值 $\tau = 0.85$。则 $s_2 = 0.95 > \tau$，$\mathbf{x}_2$ 与 $\mathbf{x}_3$ 合并为 $\bar{\mathbf{x}}_{2,3}$（len=2）。输出 3 帧：$[\mathbf{x}_1(\text{len}=1), \bar{\mathbf{x}}_{2,3}(\text{len}=2), \mathbf{x}_4(\text{len}=1)]$，帧率从 12.5 Hz 降为约 9.4 Hz。

$$
s_t = \frac{\mathbf{x}_t \cdot \mathbf{x}_{t+1}}{\|\mathbf{x}_t\| \, \|\mathbf{x}_{t+1}\|}, \quad t = 1, \ldots, T{-}1
$$

**含义**：衡量相邻帧信息冗余度；**符号**：$s_t$ = 第 $t$ 帧与第 $t+1$ 帧的余弦相似度，$\tau$ = 合并阈值。

Frame Merging Module 出现两次：（1）输入侧，将 Audio Encoder 的 25 Hz 特征压缩至 <=12.5 Hz；（2）FlexiCodec 内部，对 12.5 Hz ASR 特征执行相同操作后再做 FSQ 量化。两处共享同一机制，每个 Merging Transformer 20M 参数。

### 直接帧率条件化（核心创新）

**为什么这样设计**：基线方法用合并阈值 $\tau$ 间接控制帧率，但同一 $\tau$ 在不同话语上产生的帧率方差很大（$\sigma \approx 0.40\text{--}0.78$），用户难以预测实际加速比。直接帧率条件化解决了这个一对多映射问题。

**怎么做**：训练时随机采样合并阈值 $\tau \sim \mathcal{U}(0.85, 1.0)$，计算每条话语的经验平均帧率 $r$，将 $r$ 作为条件信号注入。帧率 $r$ 经正弦位置编码为向量：

$$
\text{PE}(r) = [\sin(r\omega_1), \cos(r\omega_1), \ldots, \sin(r\omega_d), \cos(r\omega_d)]
$$

**含义**：将标量帧率编码为高维向量，注入 Talker Transformer 的每个位置；**符号**：$r$ = 目标帧率，$\omega_i = 10{,}000^{-2i/d}$ 为频率基。

推理时用户直接指定目标帧率（如 6.25 Hz），模型据此调节输出 token 数量和帧长度分布。控制精度：直接帧率控制在所有 benchmark 上误差 < 0.1 Hz，标准差 $\sigma \leq 0.06$（对比阈值控制的 $\sigma \approx 0.40$--$0.78$）。

### Talker Transformer

![[_resources/2606.31247/figures/fig-021.png]]

> **Figure 3**：Talker Transformer 输入-输出结构。输入包含四路：(1) Thinker LLM 最后一层隐状态；(2) 帧率正弦编码；(3) 历史文本+语音 token 嵌入；(4) 前一步 Talker 输出嵌入的 concat。输出两路并行 LM head：FlexiCodec FSQ code + 帧长度。FSQ token 流相对文本流延迟 5 个 token，帧长度再延迟 1 个 token。

Talker 架构与 Thinker LLM 相同（hidden 1280, 20 层, 8 头, intermediate 5120），共 630M 参数。两路输出 LM head 并行预测 FlexiCodec FSQ code 和帧长度，实现动态帧率输出。5 token 延迟提供文本→语音 lookahead，防止语音超前于文本。

### Talker-to-Thinker 反馈连接

可选的双向连接：将 Talker 已生成的 speech token 嵌入反馈回 Thinker LLM，使 Thinker 能感知已说出的内容。Stage 1/2 禁用（LoRA 容量不足以吸收双向信号），Stage 3 全参数微调时激活。消融显示 Stage 2 提前激活反而降性能（LoRA 吸收不了双向信号的新信息量，产生不稳定反馈）。

### 训练流程

三阶段训练：

**Stage 1 — Talker 预训练**：冻结 LLM backbone，仅训练随机初始化的 Talker。英文 TTS 数据（[[Emilia]] + [[MLS]]），约 100K 小时，300K steps。Talker-to-Thinker 连接禁用。

**Stage 2 — 多任务 [[LoRA]] 微调**：激活输入侧 Frame Merging Module + Thinker（LoRA rank=32, $\alpha$=64）+ Talker。Frame Merging Module 随机初始化。混合任务训练（dialog-s2s、TTS、ASR、audio understanding），训练数据见 Table 2。240K steps。训练时随机采样输入帧率 $\sim \mathcal{U}(4, 12.5)$ Hz 和合并阈值 $\tau \sim \mathcal{U}(0.85, 1.0)$。核心数据集 FlexiSLM-Data（1.4M 对话样本 / 9.9K 小时，从 Qwen3-Omni 30B 蒸馏）。

**Stage 3 — 全参数微调**：合并 LoRA 到 LLM，全参数训练。激活 Talker-to-Thinker 连接。同 Stage 2 数据，160K steps，梯度累积=2。

### 推理流程

推理时用户指定输入帧率 $r_{\text{in}}$ 和输出帧率 $r_{\text{out}}$（如 6.25/12.5 Hz）。输入侧 Frame Merging 根据 $r_{\text{in}}$ 确定合并阈值压缩输入序列。Thinker 自回归生成文本 token，Talker 并行解码 FSQ code + 帧长度。文本生成 END 后，Talker 继续生成直到语音也结束。Audio Decoder（Flow-Matching + Vocos）将 FSQ token 和帧长度解码为 24 kHz 波形。

### 训练 Loss

$$
\mathcal{L} = \lambda_{\text{text}} \mathcal{L}_{\text{text}} + \lambda_{\text{speech}} \mathcal{L}_{\text{speech}} + \lambda_{\text{speech\_len}} \mathcal{L}_{\text{speech\_len}}
$$

**含义**：加权三路交叉熵总 loss；**符号**：$\mathcal{L}_{\text{text}}$ = 文本 token CE，$\mathcal{L}_{\text{speech}}$ = FlexiCodec FSQ code CE，$\mathcal{L}_{\text{speech\_len}}$ = 帧长度 CE。权重 $\lambda_{\text{text}} = 2$，$\lambda_{\text{speech}} = \lambda_{\text{speech\_len}} = 1$。非语音序列中 $\mathcal{L}_{\text{speech}}$ 和 $\mathcal{L}_{\text{speech\_len}}$ 置零。

## 关键结果

> 只列支撑主结论的核心表/图（论点→证据→结论）。完整表格见附录。

**核心证据**：Table 3（Kimi-Audio-Evalkit 主评测）是全文最强证据，覆盖 OpenAudioBench + VoiceBench + ASR 三大维度。

| Model | In FR | Out FR | Overall s2t | Overall s2s | ASR WER (clean) |
|-------|-------|--------|-------------|-------------|-----------------|
| Qwen2.5-Omni-7B | 25 | 50 | 66.7/63.3 | — | 2.38 |
| Kimi-Audio-7B | 12.5 | 12.5 | 69.7/57.2 | — | 1.80 |
| Mimo-Audio-7B | 6.25 | 6.25 | 70.6/59.0 | — | — |
| **FlexiSLM Stage3** | **12.5** | **12.5** | **72.4/67.2** | — | **1.98** |
| FlexiSLM Stage3 | 12.5 | 6.25 | 72.3/66.2 | — | 1.98 |
| FlexiSLM Stage3 | 6.25 | 6.25 | 70.2/64.3 | — | 2.55 |
| FlexiSLM Stage3 | 5.0 | 5.0 | 69.0/60.4 | — | 3.34 |
| FlexiSLM Stage3 | 4.0 | 4.0 | 67.2/56.5 | — | 4.47 |

**结论**：在 12.5/12.5 Hz 设置下，FlexiSLM-Stage3 的 s2t/s2s Overall 分数为 72.4/67.2，在 7B SLM 中最优，领先 Qwen2.5-Omni（66.7/63.3）+5.7/+3.9 点，优于 Kimi-Audio（69.7/57.2）。降至 6.25 Hz 输出时几乎无损（72.3/66.2），推理时间减半（RTF 1.17→0.59）。进一步降至 5.0/4.0 Hz 质量平滑退化但仍可用。

**推理效率证据**（Table 5）：

| Model | In | Out | RTF | TFLOPs |
|-------|-----|------|-----|--------|
| Qwen2.5-Omni | 25 | 50 | 1.57 (1.3x) | 5.26 (1.2x) |
| FlexiSLM | 12.5 | 12.5 | 1.17 (1.0x) | 4.57 (1.0x) |
| FlexiSLM | 12.5 | 6.25 | 0.59 (0.5x) | 3.41 (0.7x) |
| FlexiSLM | 6.25 | 6.25 | 0.57 (0.5x) | 2.73 (0.6x) |

**结论**：输出帧率是推理加速的主要驱动力（12.5→6.25 Hz 近乎减半 RTF，因 Talker 自回归生成主导总耗时）。输入帧率降低的 RTF 收益较小（prefill 不是瓶颈）。

## 可复用的设计模式

1. **信息密度自适应压缩**：用相邻帧余弦相似度做贪心合并，以信息密度而非固定步长决定压缩率。适用于任何序列数据的变长压缩（语音、视频帧、时间序列）。来自本文 Frame Merging Module。

2. **直接条件化替代间接控制**：用目标量（帧率）而非中间变量（合并阈值）做条件化，消除一对多映射的歧义。适用于任何需要精确控制生成属性的场景。来自本文直接帧率条件化 vs 阈值控制的对比。

3. **正弦编码注入连续标量条件**：将连续控制变量（帧率 $r$）通过正弦位置编码映射到高维向量，注入模型每个位置。适用于需要将连续值条件（如说话速度、情感强度）注入 Transformer 的场景。来自本文帧率编码设计。

4. **双向反馈连接的分阶段激活**：Talker-to-Thinker 连接在 LoRA 阶段禁用、全参数阶段才激活，避免低容量适配器无法吸收双向信号的不稳定问题。适用于任何分阶段训练中引入新信息流的场景。来自本文训练流程消融。

5. **单模型多工作点**：通过帧率条件化实现单一模型覆盖 4.0~12.5 Hz（RTF 1.17~0.57），无需部署多个模型。适用于异构设备部署（云端高质量 / 端侧低延迟）。来自本文 FlexiSLM 的 quality-speed tradeoff。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 📋 核验结论（技术元数据）

> 从 frontmatter relocate 来的"已 verify + [§X]"prose 版结论。
> 来源标注：`[已 verify §X / Eq.X / Tab.X / Fig.X]`，三层 verify 见 `references/no-hallucination-rules.md §11`。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start from Qwen2.5-7B-Instruct（与 Qwen2.5-Omni / Kimi-Audio 使用相同 backbone） | [已 verify §3.1] |
| 训练 loss | 三路加权 CE：$\lambda_{\text{text}}=2$ 文本 + $\lambda_{\text{speech}}=1$ FSQ code + $\lambda_{\text{speech\_len}}=1$ 帧长度；非语音序列 speech/len loss 置零 | [已 verify §3.5] |
| Tokenizer 架构 | 输入侧 Qwen2.5-Omni audio encoder（25 Hz 连续特征）+ Frame Merging；输出侧 FlexiCodec FSQ 语义 token（仅用语义分支，省略 RVQ 声学分支） | [已 verify §3.1 + Appendix H] |
| 多任务 | true——Stage 2/3 混合 dialog-s2s / TTS / ASR / audio understanding 四类任务训练 | [已 verify §3.4 + Tab.2] |
| 训练数据 | Stage 1: ~100K h 英文 TTS（Emilia + MLS）；Stage 2/3: FlexiSLM-Data 9.9K h dialog + Emilia 50K h TTS + MLS 50K h TTS + LibriSpeech 1K h ASR + MLS 50K h ASR + LLaSO-instruct 24K h audio understanding（按采样比，非全量使用） | [已 verify §3.4 + Tab.2 + Appendix A] |
| 后训练 | 无——论文 Limitations 明确说"have not explored RLHF or DPO" | [已 verify §Limitations] |
| Codec 细节 | FlexiCodec：SenseVoice ASR feature → Frame Merging → FSQ 量化（单码本）；12.5 Hz base rate 可动态降至 ~3 Hz；语义分支 450M 参数（含 SenseVoice encoder）；解码器为 NAR Flow-Matching Transformer (363M, VoiceBox style) + Vocos vocoder，均冻结 | [已 verify §3.1 + Appendix G + Appendix H] |

## 完整公式

### 公式1: [[Cosine Similarity|帧间余弦相似度]]

$$
s_t = \frac{\mathbf{x}_t \cdot \mathbf{x}_{t+1}}{\|\mathbf{x}_t\| \, \|\mathbf{x}_{t+1}\|}, \quad t = 1, \ldots, T{-}1
$$

**含义**：衡量相邻语音帧的冗余度，决定是否合并

**符号说明**：
- $\mathbf{x}_t$: 第 $t$ 帧的特征向量
- $s_t$: 相邻帧相似度，$s_t > \tau$ 时触发合并
- $\tau$: 合并阈值（训练时 $\sim \mathcal{U}(0.85, 1.0)$）

### 公式2: [[Frame Rate|平均帧率定义]]

$$
\text{Average FR} = \frac{\text{Total number of frames after merging}}{\text{Audio duration in seconds}}
$$

**含义**：定义动态帧率序列的等效帧率

### 公式3: [[Positional Encoding|帧率正弦编码]]

$$
\text{PE}(r) = [\sin(r\omega_1), \cos(r\omega_1), \ldots, \sin(r\omega_d), \cos(r\omega_d)]
$$

**含义**：将标量帧率 $r$ 编码为高维向量，注入 Talker 的每个位置

**符号说明**：
- $r$: 目标平均帧率（Hz）
- $\omega_i = 10{,}000^{-2i/d}$: 频率基
- $d$: 编码维度

### 公式4: [[Cross-Entropy Loss|加权多流 CE Loss]]

$$
\mathcal{L} = \lambda_{\text{text}} \mathcal{L}_{\text{text}} + \lambda_{\text{speech}} \mathcal{L}_{\text{speech}} + \lambda_{\text{speech\_len}} \mathcal{L}_{\text{speech\_len}}
$$

**含义**：三路交叉熵总损失

**符号说明**：
- $\mathcal{L}_{\text{text}}$: 文本 token 交叉熵损失
- $\mathcal{L}_{\text{speech}}$: FlexiCodec FSQ code 交叉熵损失
- $\mathcal{L}_{\text{speech\_len}}$: 每 token 帧长度交叉熵损失
- $\lambda_{\text{text}} = 2$, $\lambda_{\text{speech}} = \lambda_{\text{speech\_len}} = 1$

## 完整图表

### Figure 1: 动态帧率策略高层示意

![[_resources/2606.31247/figures/fig-008.png]]

**说明**：展示 Frame Merging Module 的核心思想——固定帧率（12.5/25 Hz）的语音帧经 Frame Merging Module 后根据信息密度合并为动态帧率序列（<=12.5 Hz）。合并阈值可控。

### Figure 4: 不同帧率下的语音输出可视化

![[_resources/2606.31247/figures/fig-028.png]]

**说明**：对同一句话 "The Hubble Space Telescope was launched in 1990" 展示固定 12.5 Hz（44 token）和 FlexiSLM 在 8.0 Hz（29 token）、6.25 Hz（22 token）、4.0 Hz（14 token）下的输出对比。低帧率时 token 数减少但帧长度增大，保持了词级对齐。

### Table 1: 与代表性 SLM 的能力对比

| Model | FR (Hz) | FR Ctrl. | Dynamic FR |
|---|---|---|---|
| Qwen3-Omni-30B | 12.5 | ✗ | ✗ |
| Fun-Audio-Chat-8B | 25 (5.0 effective) | ✗ | ✗ |
| GLM 4-Voice-9B | 12.5 | ✗ | ✗ |
| Mimo-Audio-7B | 25 (6.25 effective) | ✗ | ✗ |
| Kimi-Audio-7B | 12.5 | ✗ | ✗ |
| Qwen2.5-Omni-7B | 25 in / 50 out | ✗ | ✗ |
| BPE Text Tokens | 4.5 | - | - |
| **FlexiSLM-7B** | **4.0~12.5** | **✓** | **✓** |

**说明**：FlexiSLM 是唯一同时支持帧率控制和动态帧率的 SLM。Fun-Audio-Chat 和 Mimo-Audio 使用 patching 降低 LLM 侧有效帧率，但这与动态帧率压缩正交。

### Table 2: Stage 2 训练数据

| Dataset | Task | Ratio | Utts. | Hours |
|---|---|---|---|---|
| FlexiSLM-Data | Dialog-s2s | 3.0 | 1.4M | 9.9K |
| TriviaQA+Web Q. | Dialog-s2s | 3.0 | 140K | 0.4K |
| TriviaQA+Web Q. | Dialog-t2t | 1.0 | 140K | -- |
| Emilia-EN | TTS | 0.15 | 14M | 50K |
| MLS | TTS | 0.15 | 12M | 50K |
| LibriSpeech | ASR | 1.0 | 280K | 1K |
| MLS | ASR | 0.1 | 12M | 50K |
| LLaSO-instruct | Audio Und. | 1.0 | 7M | 24K |

**说明**：Ratio 为单 epoch 采样比。Dialog-s2s 采样比 3.0（过采样），TTS 采样比 0.15（欠采样）。

### Table 4: 帧率控制精度

| Method | Target | Llama Q | Web Q | TriviaQA | Alpaca |
|---|---|---|---|---|---|
| Threshold ($\tau$=0.90) | ~8 Hz | 8.34 ($\sigma$=0.70) | 7.91 ($\sigma$=0.66) | 8.18 ($\sigma$=0.78) | 8.08 ($\sigma$=0.40) |
| Threshold ($\tau$=0.86) | ~6 Hz | 6.44 ($\sigma$=0.59) | 6.03 ($\sigma$=0.59) | 6.32 ($\sigma$=0.65) | 6.06 ($\sigma$=0.45) |
| **Direct FR** (6.25 Hz) | 6.25 | 6.25 ($\sigma$=0.05) | 6.25 ($\sigma$=0.04) | 6.24 ($\sigma$=0.06) | 6.24 ($\sigma$=0.03) |
| **Direct FR** (4.0 Hz) | 4.0 | 3.99 ($\sigma$=0.05) | 4.00 ($\sigma$=0.04) | 4.00 ($\sigma$=0.05) | 4.00 ($\sigma$=0.03) |

**说明**：直接帧率控制的方差比阈值控制低一个数量级。

### Table 6: 消融实验

| Method | OAB+VB AVG (s2t/s2s) | ASR WER (clean/other) | TTS WER |
|---|---|---|---|
| FlexiSLM-Stage2 (baseline) | 68.7 / 63.0 | 2.92 / 7.20 | 3.11 |
| w/o dynamic output FR | 67.7 / 61.0 | 3.14 / 7.67 | 4.95 |
| w/o dynamic input FR | 67.5 / 62.9 | 2.97 / 7.97 | 3.12 |
| w/ threshold-controlled output | 68.2 / 61.7 | 2.96 / 7.24 | 3.53 |

**说明**：动态输出帧率提升 s2s 和 TTS 质量（去掉后 TTS WER 3.11→4.95）；动态输入帧率改善细粒度理解任务（去掉后 ASR other WER 7.20→7.97）；直接帧率控制优于阈值控制（s2s 63.0 vs 61.7）。

### Table 8: 语音生成 WER

| Model | FR | TTS WER | Dialog WER |
|---|---|---|---|
| CosyVoice | 25 | 3.20 | -- |
| Qwen3-Omni | 12.5 | 3.34 | 4.32 |
| Qwen2.5-Omni | 25 | 3.18 | 6.33 |
| **FlexiSLM** | **12.5** | **2.14** | **4.52** |
| FlexiSLM | 8.0 | 2.47 | 4.41 |
| FlexiSLM | 6.25 | 2.87 | 5.83 |
| FlexiSLM | 5.0 | 4.16 | 9.03 |

**说明**：FlexiSLM 在 12.5 Hz 达到最低 TTS WER 2.14%，优于所有对比系统。8.0 Hz 仍优于所有基线。6.25 Hz 开始轻微退化但仍可用。

### Table 10: 附加消融（Encoder / Backbone / 架构选择）

| Method | OAB+VB AVG (s2t/s2s) | ASR WER (clean/other) | TTS WER |
|---|---|---|---|
| Baseline (Stage 2) | 68.7 / 63.0 | 2.92 / 7.20 | 3.11 |
| Switch to SenseVoice encoder | 68.4 / 62.2 | 2.73 / 6.54 | 3.49 |
| Switch to Qwen-ASR encoder | 65.9 / 59.8 | **2.08** / **4.42** | **2.94** |
| Switch to Qwen2.5-Omni Thinker | 67.0 / 61.6 | 2.27 / 5.33 | 3.11 |
| w/o input merging Transformer | 65.7 / 60.1 | 6.45 / 12.33 | 3.66 |
| Activate T2T link in Stage 2 | 60.5 / 55.9 | 7.75 / 12.67 | 3.15 |

**说明**：
- Qwen-ASR encoder ASR 最强但 spoken QA 下降——ASR 预训练的 encoder 偏向转录特征，损失高层语义
- Qwen2.5-Omni Thinker 替换 Qwen2.5-7B-Instruct 后 QA 下降但 ASR 改善——文本 LLM backbone 知识更丰富
- 去掉 input merging Transformer 导致 ASR 崩溃（WER 2.92→6.45）——证明合并后需要再对齐
- Stage 2 过早激活 Talker-to-Thinker 严重退化——LoRA 容量不足

### Table 11: 音频理解（LLaSO-Eval）

| Model | FR | Emotion | Accent | Vocal | Music Source | Instrument | Gender | Avg ACC |
|---|---|---|---|---|---|---|---|---|
| Gemini 2.5-Pro | -- | 8/17 | 38/47 | 79 | 41 | 36 | 75/94 | 48.3 |
| LLaSO-3B | 50 | 27/24 | 75/54 | 74 | 57 | 46 | 76.5/99 | 58.3 |
| Kimi-Audio-7B | 12.5 | 32/55 | 19/38 | 83.5 | 38 | 26 | 66/98 | 46.8 |
| **FlexiSLM (12.5)** | **12.5** | **49/50** | **81/64** | **91** | **84** | **52** | **74.5/100** | **65.8** |
| FlexiSLM (8.0) | 8.0 | 49/80 | 61/61 | 90 | 82 | 53 | 74.5/100 | 64.7 |
| FlexiSLM (6.25) | 6.25 | 47/49 | 81/60 | 91 | 81 | 52 | 73.5/99 | 64.0 |
| FlexiSLM (4.0) | 4.0 | 48/49 | 79/64 | 88.5 | 79 | 55 | 75.5/99 | 64.1 |

**说明**：FlexiSLM 在 12.5 Hz 达到 65.8% 最高均准确率。音频理解任务对帧率压缩不敏感（4.0 Hz 仍有 64.1%），因为这类序列级分类任务依赖全局声学统计。

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | Table 3/5/8 中与 Qwen2.5-Omni / Kimi-Audio 的对比（OpenAudioBench + VoiceBench + LibriSpeech ASR） | 使用 Kimi-Audio-Evalkit 公开评测套件 + GPT-5.5 作为 LLM judge；benchmark 和评测协议完全公开；多个 7B 基线公平对比 |
| **中** | Table 6/10 消融实验 | 消融用更小训练预算（8 GPU / 160K steps / Stage 2 only），不是完整 Stage 3 配置；可能低估各组件在完整训练下的贡献 |
| **中** | Table 11 音频理解 | LLaSO-Eval 基线不完全公平（GLM-4-Voice 等未训练 LLaSO 任务，FlexiSLM 训练了） |
| **低** | RTF 对比中 FlexiSLM vs Qwen2.5-Omni 的绝对值 | 两者推理架构不同（FlexiSLM Thinker 处理含 speech-length 序列维护双向信息流 vs Qwen2.5-Omni Thinker 文本结束即停），直接 RTF 对比受架构差异影响 |

## 核心 Claim 审查

1. **Paper Claim**："FlexiSLM outperforms fixed-frame-rate 7B models including Qwen2.5-Omni and Kimi-Audio at its high-quality operating points"
   **My Assessment**：在 Kimi-Audio-Evalkit 的 OpenAudioBench + VoiceBench 综合分数上确实领先（+5.7/+3.9 over Qwen2.5-Omni, +2.7/+10.0 over Kimi-Audio）。但需注意 FlexiSLM 在 12.5/12.5 Hz 的 RTF 为 1.17（>1，非实时），而 Qwen2.5-Omni 在 25/50 Hz 的 RTF 为 1.57，两者都非实时。"outperforms" 成立于该评测框架内，但 FlexiSLM 的优势部分可能来自 FlexiSLM-Data 蒸馏数据的质量（从 Qwen3-Omni 30B 蒸馏）。

2. **Paper Claim**："at 6.25 Hz it roughly halves inference time relative to 12.5 Hz while retaining strong speech-to-speech quality"
   **My Assessment**：RTF 从 1.17 降至 0.59 确认"roughly halves"。s2s Overall 从 67.2 降至 66.2（-1.0 点），在 100 分制上降幅不大，"retaining strong quality" 基本成立。但 TTS WER 从 2.14% 升至 2.87%，dialog WER 从 4.52% 升至 5.83%，语音可懂度有可量化退化。

3. **Paper Claim**："the first SLM that supports dynamic and controllable frame rates"
   **My Assessment**：在 SLM（端到端 spoken language model）范围内，据现有文献确实未见先例。动态帧率 codec（FlexiCodec 等）已有，但集成到 SLM 框架内是首次。

## 批判性思考

### 优点
1. **概念清晰且验证充分**：动态帧率 SLM 的想法简洁优雅——用余弦相似度做帧合并、用正弦编码注入帧率条件——且通过完整消融逐一验证了各组件贡献
2. **实用的 quality-speed tradeoff**：单一模型覆盖 RTF 1.17~0.57，无需重训，对异构部署有实际价值
3. **直接帧率控制的精度**：控制误差 < 0.1 Hz（$\sigma$ < 0.06）远优于阈值控制，使推理加速可预测
4. **音频理解对帧率压缩鲁棒**：4.0 Hz 仍有 64.1% 准确率（vs 12.5 Hz 65.8%），说明 Frame Merging 保留了全局声学统计

### 局限性
1. **非流式**：论文明确承认当前模型不支持流式处理。Frame Merging Module 需要完整序列计算相邻帧相似度，与 causal/chunk-based 流式处理不兼容。实时对话场景需要额外适配
2. **仅英文**：训练数据以英文为主（Emilia-EN + MLS + LibriSpeech），多语种能力未验证
3. **蒸馏数据依赖**：核心 FlexiSLM-Data（9.9K h）从 Qwen3-Omni 30B 蒸馏，意味着对话能力上限受 teacher 模型约束。且用 Qwen3-TTS 合成语音、Fish-Audio 合成 prompt，合成数据质量上限有待验证
4. **消融训练预算不对等**：消融用 8 GPU / 160K steps（约完整训练的 1/3），低估了完整训练可能带来的额外收益
5. **4.0 Hz 工作点实用性存疑**：s2s 从 67.2 降至 56.5（-10.7 点），TTS WER 从 2.14% 升至 4.16%，在质量敏感场景可能不够用
6. **单轮对话限制**：训练数据 FlexiSLM-Data 限于单轮 QA/指令跟随，多轮对话、推理密集任务未覆盖

### 潜在改进方向
1. 将 Frame Merging 适配为 chunk-based 流式版本（如局部窗口内做帧合并），支持实时交互
2. 探索 RLHF/DPO 后训练改善对话质量和韵律自然度
3. 结合 patching（如 Mimo-Audio 的 4-frame grouping）与动态帧率实现更激进的压缩
4. 多语种扩展——FlexiCodec 的 SenseVoice encoder 本身支持多语种，理论上可迁移

### 可复现性评估
- [x] 代码计划开源（https://github.com/AmphionTeam/FlexiSLM，截至 2026-07-07 未发布）
- [ ] 预训练模型（计划发布）
- [x] 训练细节完整（Appendix C 给出完整超参数）
- [x] 数据集可获取（FlexiSLM-Data 计划发布；Emilia/MLS/LibriSpeech 公开）

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[SpeechLM-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §路线 2 Codec LM — Thinker-Talker 子范式（与 [[Qwen2.5-Omni]] 同框架，但引入动态帧率）
- **核心问题**：[[TTS-核心挑战]] §挑战 3 延迟/效率（通过降帧率换加速）；§挑战 5 Codec 设计（FlexiCodec 动态帧率 tokenizer）
- **表示层位置**：[[TTS-表示层地图]] §semantic-token（FlexiCodec FSQ 语义 token，12.5 Hz base 可动态降至 4.0 Hz）
- **相邻工作**：[[Qwen2.5-Omni]]（同 Thinker-Talker 架构+同 backbone，但固定帧率）/ [[Kimi-Audio]]（固定 12.5 Hz 并行 speech-text）/ [[FlexiCodec]]（本文的 codec 基础，此前仅验证于 0.3B TTS）/ Mimo-Audio（patching 降有效帧率，与动态压缩正交）

## 🔄 后续重估

- **2026-07-07**：初读。FlexiSLM 首次验证了动态帧率在 SLM 中的可行性和实用性。在 12.5 Hz 高质量点表现优异（overall 72.4/67.2），6.25 Hz 可实现近乎无损的推理减半。但当前不支持流式、仅英文、未做后训练，距实际部署还有距离。帧率可控这一特性本身对异构设备部署有明确的工程价值。需持续关注其代码发布和社区复现情况。

---

## 关联笔记

### 基于
- [[FlexiCodec]]: FlexiSLM 的 codec 基础——动态帧率 FSQ 语义 token + NAR Flow-Matching 解码器
- [[Qwen2.5-Omni]]: Thinker-Talker 架构来源 + Audio Encoder + LLM backbone 初始化

### 对比
- [[Kimi-Audio]]: 同为 7B SLM，固定 12.5 Hz，FlexiSLM 在 s2s 上领先 +10 点
- [[Qwen2.5-Omni]]: 同 Thinker-Talker 架构但固定 25/50 Hz，FlexiSLM 在 Overall 上领先 +5.7/+3.9 点

### 方法相关
- [[FSQ]]: FlexiCodec 使用的量化方式（替代 RVQ）
- [[Flow Matching]]: Audio Decoder 使用的 NAR 解码范式
- [[Vocos]]: 将 mel 频谱图转为 24 kHz 波形的声码器
- [[LoRA]]: Stage 2 对 Thinker LLM 的轻量级适配方法
- [[SenseVoice]]: FlexiCodec 语义编码分支的 ASR 特征提取器

### 硬件/数据相关
- [[Emilia]]: Stage 1/2 TTS 训练数据来源
- [[MLS]]: Stage 1/2 TTS + ASR 训练数据来源
- [[LibriSpeech]]: ASR 训练和 TTS WER 评测数据

---

## 速查卡片

> [!summary] FlexiSLM: A Dynamic and Controllable Frame Rate Spoken Language Model
> - **核心**: 首个动态+可控帧率 SLM，通过 Frame Merging 实现 4.0~12.5 Hz 连续可控
> - **方法**: Thinker-Talker 架构 + Frame Merging Module（余弦相似度合并）+ 直接帧率条件化（正弦编码）+ FlexiCodec FSQ token
> - **结果**: 12.5 Hz 时 Overall s2t/s2s 72.4/67.2（7B SLM 最优）；6.25 Hz 推理减半（RTF 1.17→0.59）仅损 1 点 s2s；非流式、仅英文、未做后训练
> - **代码**: https://github.com/AmphionTeam/FlexiSLM（计划发布，截至 2026-07-07 未上线）

---

*笔记创建时间: 2026-07-07*
