---
title: "Wan-Streamer v0.1: End-to-end Real-time Interactive Foundation Models"
method_name: "Wan-Streamer"
authors: [Lianghua Huang, Zhi-Fan Wu, Wei Wang, Yupeng Shi, Mengyang Feng, Junjie He, Chen-Wei Xie, Yu Liu, Jingren Zhou, Ang Wang, Bang Zhang]
year: 2026
venue: arXiv
arxiv_id: "2606.25041"
tags: [omni-model, full-duplex, audio-visual-interaction, streaming, flow-matching, end-to-end, real-time, multimodal-generation]
note_tier: standard

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: true
post_training_type: custom
streaming: true

# === 知识地图联动 ===
domain: Omni
subdomain: audio-visual-interaction
routes: [speech-llm-tts, streaming-tts, dialogue-tts]
problems: [latency, streaming, interrupt-handling, dialogue-integration]
representations: [continuous-latent]
related_maps:
  - "[[TTS-SpeechLM-Dialogue关系]]"
  - "[[TTS-核心挑战]]"
related_surveys: []
evidence_level: low
maturity: exploratory
last_repositioned: 2026-06-26

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.25041/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.25041/paper.html"
figures_dir: "_resources/2606.25041/figures/"
github_local:
cached_at: 2026-06-26

# === 通用元数据 ===
image_source: online
arxiv_html: "https://arxiv.org/html/2606.25041"
created: 2026-06-26
---

# 论文笔记：Wan-Streamer v0.1: End-to-end Real-time Interactive Foundation Models

> **笔记分级**：standard（方法架构清晰、值得精读的端到端多模态交互系统，但 v0.1 报告细节有限）。分级标准见 `references/quality-standards.md §模板分级`。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Alibaba Group (Wan Team) |
| 日期 | June 2026 |
| 项目主页 | [wan-streamer.com](https://wan-streamer.com/) |
| 对比基线 | [[Moshi]] / [[GPT-4o]] / [[Qwen2.5-Omni]] |
| 链接 | [arXiv](https://arxiv.org/abs/2606.25041) / 代码未开源 |

## 一句话总结

> 单一 Transformer 统一文本/语音/视频的输入输出，以 block-causal attention 实现原生流式全双工音视频交互，模型端延迟约 200 ms。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 必要图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源 [§X] 不写在这里，统一收到「附录·核验结论」表。
> **图表**：关键图表就近嵌入，按「论点 → 证据（图/表）→ 结论」组织。

## 核心贡献

1. **原生流式端到端多模态交互模型**：在单一 Transformer 中统一文本、语音、视频三模态的输入与输出，无需级联外部模块（VAD / ASR / TTS / 数字人渲染等），感知、推理、生成、响应时机、轮次管理、跨模态同步全部联合学习
2. **全因果多模态架构**：包含因果音频/视频 VAE、因果编解码器、[[Block-Causal Attention]] 和全历史自回归流式生成，保证严格因果性
3. **Thinker-Performer 低延迟推理系统**：通过 KV-cache 交换实现流水线并行，模型端响应延迟约 200 ms，总交互延迟约 550 ms（含 350 ms 网络延迟）

## 问题背景

### 要解决的问题

人类交互天然是流式、全双工的——人在持续感知的同时也在持续回应。现有多模态交互系统多为级联架构（cascaded pipeline），拼接独立的 VAD、ASR、LLM、TTS、音频驱动动画或视频生成模块。

### 现有方法的局限

级联系统存在三个结构性问题：
- **模块边界引入延迟**：每个模块间的序列化处理累积延迟
- **误差级联放大**：上游模块的错误不可逆地传递给下游
- **轮次管理难以联合学习**：响应时机、打断处理、跨模态同步需要跨模块协调，但各模块独立训练

### 本文的动机

如果将感知、推理、生成、响应时机全部建模在单一因果流式模型中，可以消除级联开销，让模型端到端地学习何时回应、如何同步语音和视觉输出。

## 方法详解

### 领域定位

Wan-Streamer 属于 **端到端多模态交互** 路线，与 [[Moshi]]（纯语音全双工）和 [[Body of Her]]（端到端拟人 agent）同类，但在模态覆盖度上更进一步——同时支持文本/语音/视频三模态的输入和输出。核心差异在于：Moshi 只处理 speech-to-speech，Wan-Streamer 增加了视频通道并在单一模型内生成同步的语音 + 视频输出。

### 端到端数据流（先地图后街景）

Wan-Streamer 的完整流水线：**用户音视频流** → **因果编码器**（将原始音频/视频编码为连续 latent）→ **统一 Transformer**（block-causal attention 处理交错的文本/音频/视频 token，预测文本 token + 音视频 velocity field）→ **flow-matching solver**（从噪声 latent 生成干净的音视频 latent）→ **因果解码器**（latent→音频波形 + 视频帧）→ **输出流**。

![Figure 1: Overview of Wan-Streamer](https://arxiv.org/html/2606.25041v2/x1.png)

> **Figure 1**：Wan-Streamer 系统总览。单一 Transformer 接收用户的文本/音频/视频 token 流，通过 block-causal attention 实现增量式流式生成，同时输出文本 token、音频 latent 和视频 latent。语言通道用离散 token + cross-entropy loss，音视频通道用连续 latent + [[Conditional Flow Matching]] loss。每个 streaming unit 为 160 ms。

下面逐个放大每个关键模块。

### 流式建模的数学框架

交互被建模为连续因果流。在第 $k$ 个 streaming unit（160 ms）：
- 用户观测：$u_k = (u_k^t, u_k^a, u_k^v)$（文本、音频、视频）
- Agent 响应：$y_k = (y_k^t, y_k^a, y_k^v)$（文本、音频、视频）

联合分布按流式因果分解：

$$
p_\theta(y_{1:K} \mid u_{1:K}) = \prod_{k=1}^{K} p_\theta\bigl(y_k^t, y_k^a, y_k^v \mid u_{\leq k}^t, u_{\leq k}^a, u_{\leq k}^v, y_{<k}^t, y_{<k}^a, y_{<k}^v\bigr)
$$

**为什么这样设计**：因果分解保证了每个 streaming unit 只依赖过去的观测和历史响应，天然适配流式推理。生成后的 $y_k$ 以干净 latent 形式提交回历史，成为后续 unit 的条件上下文——这消除了自回归生成中常见的 exposure bias 风险（因为 context 是 clean latent 而非 noisy 中间状态）。

### 双通道生成：离散语言 + 连续音视频

**语言通道**：离散 token，标准 cross-entropy next-token prediction。

**音视频通道**：连续 latent 空间，使用 [[Conditional Flow Matching]]。Flow matching 通过学习从噪声到目标 latent 的速度场实现生成。

插值路径与速度场定义：

$$
z_\tau^m = (1 - \tau) z_0^m + \tau \epsilon^m, \quad \frac{\partial z_\tau^m}{\partial \tau} = \epsilon^m - z_0^m
$$

其中 $z_0^m$ 是目标干净 latent，$\epsilon^m \sim \mathcal{N}(0, I)$ 是高斯噪声，$\tau$ 是 flow time。

Flow matching loss：

$$
\mathcal{L}_{\text{FM}}^m = \mathbb{E}_{\epsilon^m} \left\| f_\theta(z_\tau^a, z_\tau^v, c_k, \tau) - \frac{\partial z_\tau^m}{\partial \tau} \right\|_2^2
$$

其中 $c_k = \{u_{\leq k}^t, u_{\leq k}^a, u_{\leq k}^v, y_{<k}^t, y_{<k}^a, y_{<k}^v\}$ 是干净的流式上下文。

**为什么 work**：关键设计是**同一个 clean context $c_k$ 同时条件化音频和视频的速度场预测**。这意味着语音韵律和面部动态、嘴唇运动共享同一条件信息，实现了天然的跨模态同步——说什么话时做什么表情/动作由模型联合优化，而非后处理对齐。

### Block-Causal Attention

Transformer 使用 block-causal attention 处理交错的视觉、音频和文本 token。每个 streaming unit 内的 token 可以互相 attend，但不能 attend 到未来 unit 的 token。这在标准 causal attention 的基础上放松了 unit 内的约束，同时严格保持 unit 间的因果性。

**具体例子**：假设 streaming unit 大小为 160 ms，对应 4 个视频帧（25 FPS）。unit $k$ 内的 4 帧视频 token + 对应音频 token + 文本 token 可以互相 attend（形成一个 block），但整个 block 不能看到 unit $k+1$ 及之后的任何 token。

### 因果音视频 VAE

音频和视频都通过**严格因果**的 VAE 编码到连续 latent 空间。因果性的含义是：编码和解码都只依赖当前和过去的输入，不需要未来帧——这是流式推理的前提。论文强调这些 VAE 是"从一开始就为因果性而设计的"。

### 训练流程

训练分三个阶段：

**Stage 1 — 独立任务预训练**：
- Transformer 从语言模型 warm start 初始化（引用了 Qwen2.5 和 Qwen3）
- 在 LM 周围训练多模态接口
- 因果音视频编码器与 Transformer 联合训练
- 理解任务（图像/音频/视频理解、文本对话、ASR、TTS、音频对话）与生成任务（图像/音频/视频生成、音视频联合生成）混合训练

**Stage 2 — 端到端交互训练**：
- 在全双工交互数据上训练，数据包含交错的用户/agent 文本/音频/视频
- 模型学习从当前观测更新状态、生成同步响应、并将干净 latent 提交回历史
- 响应时机、主动倾听、打断处理、长上下文一致性都在此阶段学习

**Stage 3 — 蒸馏加速**：
- 用更强的 teacher（带 CFG + 更多 flow-matching solver 步数）蒸馏到高效 student
- CFG 效果被吸收进 student；solver 步数减少
- **Rolling distillation**：student 在连续多个 streaming unit 上自回归 rollout，使用自身生成的历史进行训练
- 采用 **self-forcing** 策略 + **distribution matching** 对齐 student 在真实 rollout 条件下的轨迹与 teacher，减少 train-test mismatch

### 推理流程

虽然训练是单一模型，但推理部署拆分为 **Thinker** 和 **Performer** 两个角色，分别放在两块 GPU 上：

![Figure 2: Thinker-Performer streaming inference pipeline](https://arxiv.org/html/2606.25041v2/x2.png)

> **Figure 2**：Thinker-Performer 流水线。在 unit $k$，Thinker 编码当前用户观测 $u_k$、更新 KV cache、解码上一步的响应 latent $y_{k-1}$ 用于即时输出。Performer 接收当前 KV slice，运行 flow-matching solver 生成下一步的干净音视频 latent $y_k$，在下一个 unit 返回给 Thinker。

**Thinker（GPU 1）**：因果音视频编码器 → token-causal Transformer path（语言预测 + 状态更新 + KV-cache 构建）→ 因果音视频解码器。

**Performer（GPU 2）**：仅负责 flow-matching solver（latent 去噪生成）。

**第 $k$ 步推理协议**：
1. Thinker 消费当前用户音视频观测，运行因果编码器 + token-causal 解码 → 产生当前 KV-cache slice
2. Thinker 接收 Performer 上一步生成的干净音视频 latent $y_{k-1}$
3. Thinker 将新 KV slice 发送给 Performer
4. Thinker 解码 $y_{k-1}$ 为音视频输出，即时输出
5. Performer 将收到的 KV slice 拼接到全历史 cache，运行 flow-matching solver 生成 $y_k$
6. 干净 latent 留在 Performer，下一步发给 Thinker

**为什么 work**：这种流水线设计的精妙之处在于**时间错位**——Thinker 处理当前帧感知/状态更新的同时，Performer 并行生成下一帧的音视频 latent。只要 Performer 的计算时间 + KV/latent 传输开销 < 160 ms（一个 streaming unit），就能保持实时。

**实时约束**：Performer 计算时间 + KV/latent 通信开销 $\leq$ 160 ms。

**额外优化**：CUDA graph capture、编译优化、优化 kernel、KV-cache 高效交换。

### 自然行为：倾听态、打断、主动发言

全双工行为从交错交互数据中端到端学习，而非手工设计 turn-taking 规则：
- **空闲态**：保持身份、注视方向、姿态、呼吸、微妙面部运动
- **倾听态**：产生注视转移、点头、微表情、姿态变化，与用户语音/视觉线索时间耦合
- **打断处理**：模型在生成自身响应的同时持续消费用户音视频，可以在自然打断时停止、缩短或重定向语音
- **主动发言**：当输入流中出现显著视觉事件时，可以主动发起评论

## 关键结果

> 本文无定量 benchmark 评测（无 MOS / WER / SIM-O 等指标），结果主要是延迟对比和定性描述。

**核心证据**：Table 1 延迟对比是全文最核心的定量数据。

### Table 1: 与语音/全模态系统的响应延迟对比

| 系统 | 交互模式 | 用户可见响应延迟 | 模型端指标 |
|------|---------|----------------|-----------|
| Doubao Realtime Voice | speech-to-speech | ~1 s | ~700 ms bare-model |
| Seeduplex | speech-to-speech | N/R | -250 ms endpoint, -300 ms interrupt vs. Doubao |
| GPT-4o / Realtime API | speech-to-speech, audio+vision in | protocol-dependent | 232/320 ms audio; ~500 ms API TTFB; ~800 ms target |
| Hume EVI 3 | speech-to-speech | 0.9-1.4 s web-app | <300 ms model |
| Gemini Live API | speech-to-speech | 1.2-3.6 s | N/R model-side |
| Sesame web app | speech-to-speech | 0.8-1.2 s | N/R model-side |
| [[Moshi]] | speech-to-speech | N/R product path | 160 ms 理论; 200 ms 实际 |
| Qwen3/3.5-Omni | audio-video-text in, speech/text out | N/R interaction loop | first-packet: 234/547 ms |
| MiniCPM-o 4.5 | audio-video in, speech/text out | N/R interaction loop | 0.58 s first-token; RTF 0.20-0.27 |
| **Wan-Streamer** | **text/audio/video in+out** | **~550 ms total (含 350 ms 网络)** | **~200 ms model-side; 25 FPS video** |

**结论**：Wan-Streamer 的模型端延迟 (~200 ms) 与 Moshi (~200 ms 实际) 相当，但模态覆盖范围显著更广（三模态 I/O vs. 纯语音）。在所有同时支持语音+视频输出的系统中，它是延迟最低的。但需注意：不同系统的延迟测量协议不统一，直接数值对比的可信度有限。

### Table 2: 与数字人/Avatar 系统的运行时对比

| 系统 | 视觉范围 | 报告运行时 |
|------|---------|----------|
| Body of Her | 端到端拟人 agent | 下一帧 42 ms @24 FPS（初步，无部署延迟） |
| VASA-1 | 音频驱动 talking face | 40 FPS + 170 ms 前置延迟（仅渲染器） |
| StreamAvatar | 流式 talking/listening avatar | FFD 0.33-0.39 s; 视频延迟 ~1.20 s |
| AvatarForcing (Cui) | 单步流式 talking avatar | 34 ms/帧; 0.51 s audio-to-visual |
| LiveTalk | 多模态交互 avatar | 24.82 FPS; 0.33 s 首帧延迟 |
| OmniForcing | text-to-audio-video streaming | TTFC ~0.7 s; ~25 FPS |
| **Wan-Streamer** | **三模态感知对话 + 同步语音视频** | **25 FPS; ~550 ms total; ~200 ms model-side** |

**结论**：现有数字人/Avatar 系统多为渲染器或依赖外部语音模型，不具备端到端对话推理能力。Wan-Streamer 是唯一在单一因果 Transformer 中同时完成对话推理 + 语音生成 + 视频生成的系统。

## 可复用的设计模式

1. **Thinker-Performer 流水线拆分**：将感知/推理和生成拆分到两块 GPU 上流水线执行，通过 KV-cache 交换实现并行。适用于任何需要实时生成的多步 pipeline。来自本文推理架构。

2. **Clean latent 提交回历史**：生成的音视频 latent 以 clean（去噪后）形式提交回自回归历史，而非带噪中间状态。避免了误差在流式自回归中累积。适用于任何连续 latent 自回归生成场景。来自本文流式建模框架。

3. **Rolling distillation + Self-forcing**：蒸馏时让 student 在自身生成的历史上 rollout，而非用 teacher 的历史，减少 train-test mismatch。适用于流式生成模型的加速蒸馏。来自本文 Stage 3 训练。

4. **Block-causal attention 实现流式多模态**：unit 内多模态 token 互相 attend，unit 间严格因果。相比 token-level causal 更高效（unit 内并行），相比 full attention 可以流式。适用于多模态流式生成。来自本文 attention 设计。

5. **统一 flow-matching 共享 context 实现跨模态同步**：音频和视频速度场共享同一条件 context，自然同步嘴唇运动和语音韵律。适用于需要多模态时间对齐的生成任务。来自本文 Eq. (3)。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 📋 核验结论（技术元数据）

> 来源标注：`[已 verify §X / Eq.X / Tab.X / Fig.X]` 或 `[GitHub: <path>:<line>]`。
> **注意**：本文为 v0.1 技术报告，许多关键细节未披露。未披露项标注为"论文未披露"。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start，从语言模型初始化，引用 Qwen2.5 和 Qwen3，但未明确说用哪个具体 checkpoint/size | [已 verify §2.3 Stage 1] |
| 训练 loss | 语言通道: cross-entropy next-token prediction; 音视频通道: conditional flow matching loss (Eq. 3); Stage 3 额外加 distribution matching distillation loss | [已 verify §2.1 Eq.1-3, §2.3] |
| Tokenizer 架构 | 文本: 离散 token; 音视频: 连续 latent（通过因果 VAE 编解码），不使用离散 codec | [已 verify §2.1] |
| 多任务 | true，Stage 1 混合理解+生成任务，Stage 2 端到端交互 | [已 verify §2.2-2.3] |
| 训练数据 | 分三类（理解/生成/交互），具体规模（小时数/样本量）论文未披露 | [已 verify §2.2，但规模未给] |
| 后训练 | Stage 3 蒸馏: teacher (CFG + 多步 solver) → student (少步 solver + 无 CFG); rolling distillation + self-forcing + distribution matching | [已 verify §2.3 Stage 3] |
| 模型参数量 | 论文未披露 | N/A |
| 音视频 VAE 架构 | 严格因果 VAE，连续 latent，具体层数/维度/下采样率论文未披露 | [已 verify §2.1 定性描述，细节未给] |
| Block-causal attention 细节 | 描述为"block-causal attention for incremental streaming"，具体 mask 结构/block 大小未披露 | [已 verify §2.1 定性描述，细节未给] |
| 输出分辨率 | 192p（v0.1 初步结果） | [已 verify §Conclusion] |
| 推理硬件 | 两块 GPU（thinker + performer） | [已 verify §2.4] |

## 完整公式

### 公式 1: [[Autoregressive|流式因果分解]]

$$
p_\theta(y_{1:K} \mid u_{1:K}) = \prod_{k=1}^{K} p_\theta\bigl(y_k^t, y_k^a, y_k^v \mid u_{\leq k}^t, u_{\leq k}^a, u_{\leq k}^v, y_{<k}^t, y_{<k}^a, y_{<k}^v\bigr)
$$

**含义**：将多模态交互序列按 streaming unit 进行因果分解，每个 unit 的响应条件化于所有过去的观测和历史响应。

**符号说明**：
- $u_k = (u_k^t, u_k^a, u_k^v)$: 第 $k$ 个 unit 的用户观测（文本、音频、视频）
- $y_k = (y_k^t, y_k^a, y_k^v)$: 第 $k$ 个 unit 的 agent 响应（文本、音频、视频）
- $K$: streaming unit 总数

### 公式 2: [[Flow Matching|Flow Matching 插值路径]]

$$
z_\tau^m = (1 - \tau) z_0^m + \tau \epsilon^m, \quad \frac{\partial z_\tau^m}{\partial \tau} = \epsilon^m - z_0^m
$$

**含义**：定义从目标干净 latent $z_0^m$ 到噪声 $\epsilon^m$ 的线性插值路径，速度场为常数。

**符号说明**：
- $z_0^m$: 模态 $m$ 的目标干净 latent
- $\epsilon^m \sim \mathcal{N}(0, I)$: 高斯噪声
- $\tau \in [0,1]$: flow time（$\tau=0$ 时为干净 latent，$\tau=1$ 时为纯噪声）
- $m \in \{a, v\}$: 音频或视频模态

### 公式 3: [[Conditional Flow Matching|CFM Loss]]

$$
\mathcal{L}_{\text{FM}}^m = \mathbb{E}_{\epsilon^m} \left\| f_\theta(z_\tau^a, z_\tau^v, c_k, \tau) - \frac{\partial z_\tau^m}{\partial \tau} \right\|_2^2
$$

**含义**：条件 flow matching 损失，训练模型预测从噪声到干净 latent 的速度场。

**符号说明**：
- $f_\theta$: 统一 Transformer 的速度场预测
- $c_k$: 干净流式上下文，包含所有过去的观测和响应
- $\tau$: flow time / 噪声级别

## 完整图表

### Figure 1: System Overview

![Figure 1](https://arxiv.org/html/2606.25041v2/x1.png)

**说明**：Wan-Streamer 系统架构总览。单一 Transformer 处理交错的文本/音频/视频 token，使用 block-causal attention 保证流式因果性。语言通道产生离散 token，音视频通道通过 flow matching 在连续 latent 空间生成。

### Figure 2: Thinker-Performer Inference Pipeline

![Figure 2](https://arxiv.org/html/2606.25041v2/x2.png)

**说明**：推理时的 Thinker-Performer 流水线。Thinker 负责感知编码 + 语言推理 + 音视频解码输出；Performer 仅负责 flow-matching solver 生成下一步 latent。两者通过 KV-cache slice 和 clean latent 进行通信，在相邻 streaming unit 之间流水线并行。

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **低** | 延迟数字 (~200 ms model-side, ~550 ms total) | 自报指标，无第三方验证；不同系统延迟测量协议不统一，Table 1 中大量 N/R 条目；无开源代码可复现 |
| **低** | 自然行为（倾听态、打断、主动发言） | 仅定性描述 + 项目主页 demo，无定量评测（无 MOS、无 naturalness 评分、无 turn-taking 准确率） |
| **低** | 视频生成质量 | 仅 192p 分辨率，无 FID / FVD / SSIM 等视频质量指标 |
| **低** | 语音生成质量 | 无 MOS / WER / SIM-O 等语音质量指标 |

## 核心 Claim 审查

1. **Paper Claim**："real-time, low-latency, full-duplex audio-visual interaction"，"approximately 200 ms model-side response latency"
   **My Assessment**：200 ms 模型端延迟的数字在 speech-to-speech 系统中属于领先水平（与 Moshi 相当），但这是自报数字，无第三方验证。更重要的是，总交互延迟 550 ms 中有 350 ms 是网络延迟假设——实际网络条件下可能更高。此外，这个延迟数字是在 192p 分辨率下取得的，扩展到更高分辨率时延迟会增加。

2. **Paper Claim**："eliminates reliance on external modules" — 感知、推理、生成、响应时机全部联合学习
   **My Assessment**：架构设计确实是端到端的——单一 Transformer 处理所有模态。但论文未给出消融实验证明端到端比级联好多少，也未给出理解/生成能力的定量对比。"消除外部模块"是架构声明而非实验验证。

3. **Paper Claim**："substantially reduces train-test mismatch" (关于 rolling distillation)
   **My Assessment**：rolling distillation + self-forcing 在流式生成蒸馏中是合理的设计（解决 exposure bias），但论文未给消融证明具体 improvement 幅度。

## 批判性思考

### 优点
1. **架构设计的完整性**：从因果 VAE 到 block-causal attention 到 Thinker-Performer 推理，整个系统设计为原生流式，不是在已有模型上加流式 wrapper
2. **模态覆盖范围领先**：在已知系统中，唯一同时支持文本/语音/视频三模态 I/O 的端到端模型
3. **Thinker-Performer 流水线是优雅的工程方案**：通过 KV-cache 交换实现感知/推理与生成的并行，且保持单模型训练的一致性

### 局限性
1. **v0.1 信息极度稀疏**：无模型参数量、无训练数据规模、无 VAE 架构细节、无 attention mask 具体设计、无消融实验。作为技术报告，缺少几乎所有可复现的关键信息
2. **无定量评测**：无 MOS / WER / SIM-O / FID / FVD 等任何标准指标，无法判断生成质量
3. **192p 分辨率**：远低于实用标准（通常至少 720p），论文称"扩展到更高分辨率是直接的未来工作"但未给证据
4. **延迟数字缺乏严格基准**：Table 1 中不同系统的延迟测量方式、硬件条件、网络假设各不相同，直接数值对比意义有限
5. **全双工行为仅定性展示**：打断处理、主动发言等行为只有描述和 demo，无 turn-taking 准确率、打断响应时间等定量指标

### 潜在改进方向
1. 提升到 720p+ 分辨率并测量延迟 trade-off
2. 加入标准 benchmark 评测（语音质量: MOS/WER/SIM-O；视频质量: FVD/SSIM；对话: turn-taking accuracy）
3. 消融 end-to-end vs. cascaded 的实际差异
4. 公开 VAE、attention mask、训练数据等细节

### 可复现性评估
- [ ] 代码开源（未开源）
- [ ] 预训练模型（未发布）
- [ ] 训练细节完整（严重不完整：无参数量、无数据规模、无 VAE 细节）
- [ ] 数据集可获取（不可获取，未披露）

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[Omni-领域总览]]（音视频文本三模态统一）
- **技术路线**：[[TTS-SpeechLM-Dialogue关系]] 位置 ④ — 端到端全模态交互（超越纯语音对话）
- **核心问题**：[[TTS-核心挑战]] 挑战 3（延迟/流式）+ 挑战 8（对话系统集成）
- **表示层位置**：连续 latent（非离散 token），音视频通过因果 VAE 映射到连续空间
- **相邻工作**：[[Moshi]]（纯语音全双工，160 ms streaming unit）/ [[Body of Her]]（端到端拟人 agent）/ [[Qwen2.5-Omni]]（多模态理解+语音输出，但非原生流式视频生成）

## 🔄 后续重估

- **2026-06-26**：初读。架构设计理念清晰且前瞻——原生流式多模态交互是正确方向。但 v0.1 报告信息极度稀疏（无参数量/数据规模/质量评测/消融），实际生成质量和系统能力无法判断。192p 分辨率说明离实用仍有距离。需等 v0.2 或后续论文补充细节后重估。当前定位为 exploratory，evidence_level=low。

---

## 关联笔记

### 基于
- [[Qwen2.5-Omni]]: Transformer 初始化来源（引用 Qwen2.5/Qwen3）
- [[Conditional Flow Matching]]: 音视频生成的核心方法
- [[Flow Matching]]: CFM 的理论基础

### 对比
- [[Moshi]]: 最接近的先驱——纯语音全双工、160 ms streaming unit、200 ms 实际延迟
- [[MiniCPM-o]]: 多模态理解+语音输出，但非端到端视频生成
- [[VASA-1]]: 音频驱动 talking face，但仅渲染器无对话推理

### 方法相关
- [[Block-Causal Attention]]: 核心 attention 机制
- [[Conditional Flow Matching]]: 音视频 latent 生成方法
- [[Full-Duplex]]: 全双工交互范式
- [[KV Cache]]: Thinker-Performer 通信基础
- [[Classifier-Free Guidance]]: Stage 3 蒸馏中被吸收的技术

### 硬件/数据相关
- 推理需两块 GPU（Thinker + Performer）

---

## 速查卡片

> [!summary] Wan-Streamer v0.1
> - **核心**: 单一 Transformer 统一文本/语音/视频 I/O 的原生流式全双工交互模型
> - **方法**: 因果 VAE + block-causal attention + CFM 生成 + Thinker-Performer 流水线推理
> - **结果**: ~200 ms 模型端延迟, ~550 ms 总延迟 (含 350 ms 网络), 25 FPS @ 192p; 无质量评测
> - **代码**: 未见开源

---

*笔记创建时间: 2026-06-26*
