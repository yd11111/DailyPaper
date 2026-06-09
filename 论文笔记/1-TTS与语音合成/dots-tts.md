---
title: "dots.tts Technical Report"
method_name: "dots.tts"
authors: [Shi Lian, Changtao Li, Bohan Li, Hankun Wang, Da Zheng, Junfeng Tian, Yufeng Ma, Colin Zhang, Kai Yu]
year: 2026
venue: arXiv
arxiv_id: "2606.07080"
tags: [continuous-ar-tts, flow-matching, self-corrective-alignment, meanflow-distillation, zero-shot-cloning, streaming-tts]
note_tier: standard

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: false
post_training_type: custom
streaming: true

# === 知识地图联动 ===
domain: TTS
subdomain: continuous-ar-tts
routes: [tokenizer-free-tts, streaming-tts, diffusion-flow-tts]
problems: [zero-shot-cloning, speaker-similarity, latency, streaming, multilinguality, long-form-stability]
representations: [continuous-latent]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-表示层地图]]"
  - "[[TTS-核心挑战]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-06-09

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.07080/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.07080/paper.html"
figures_dir: "_resources/2606.07080/figures/"
github_local: "~/DailyPaper/.cache/papers/2606.07080/github/rednote-hilab_dots.tts"
cached_at: 2026-06-09

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2606.07080v1
created: 2026-06-09
---

# 论文笔记：dots.tts Technical Report

> **笔记分级**：standard（完整精读）。分级标准见 `references/quality-standards.md §模板分级`。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Xiaohongshu (dots) + X-LANCE Lab, Shanghai Jiao Tong University |
| 日期 | June 2026 |
| 项目主页 | [Demo](https://rednote-hilab.github.io/dots.tts-demo) |
| 对比基线 | [[CosyVoice3]], [[Qwen3-TTS]], [[IndexTTS2]], [[FireRedTTS2]], [[SeedTTS]], [[DiTAR]], [[VibeVoice]], [[VoxCPM]], [[F5-TTS]], [[MegaTTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2606.07080) / [Code](https://github.com/rednote-hilab/dots.tts) / [Model](https://huggingface.co/collections/rednote-hilab/dotstts) |

## 一句话总结

> 小红书联合上交 X-LANCE 提出的 2B 连续自回归 TTS 基础模型，通过 AudioVAE + LLM + AR Flow-Matching Head 三模块解耦设计，加 SOAR 自纠正后训练和 MeanFlow 蒸馏，在 Seed-TTS-Eval 上取得最优平均 WER/SIM，流式模式首包延迟 54 ms。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 必要图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源统一收到「附录·核验结论」表。

## 核心贡献

1. **全连续 AR TTS 架构**：去除离散 codec token，在 128 维连续 VAE latent 空间做自回归生成，避免 tokenizer 信息瓶颈
2. **三模块解耦设计**：Semantic Encoder（内容摘要） + LLM（长程文本-内容建模） + AR-FM Head（局部声学渲染），让语义推理和声学生成不竞争同一模块
3. **Reward-free 自纠正后训练（SOAR）**：将 SOAR 方法应用于 AR flow-matching head，暴露模型于自身 off-trajectory 推理误差进行自纠正，无需 reward model
4. **CFG-aware MeanFlow 蒸馏**：将 CFG 融入蒸馏目标，推理时只需单次前向，实现 NFE=4 下接近全精度质量

## 问题背景

### 要解决的问题

连续表示 AR TTS（Route 3）在去除离散 token 量化瓶颈的同时，面临**长程误差累积**问题：离散 token 有天然的"量化缓冲"把不完美的预测 snap 回合法配置，但连续 latent 没有这种保护。

### 现有方法的局限

- **NAR 路线**（[[Voicebox]], [[F5-TTS]]）：需要已知时长，不适合实时流式对话
- **Discrete-token AR 路线**（[[CosyVoice]], [[Qwen3-TTS]], [[SeedTTS]]）：tokenizer 的码本设计限定了 LM 能表达的内容上限，难以用单一分布覆盖 speech + paralinguistics + singing
- **已有连续 AR 路线**（KALL-E, [[DiTAR]], [[VibeVoice]], VoxCPM）：连续 latent 缺乏量化缓冲导致长程漂移

### 本文的动机

通过三个互补设计缓解连续 AR 的不稳定性：(1) 让 VAE latent 空间同时具备语义结构性和预测友好性；(2) 将语义推理与声学渲染分离到独立模块；(3) 用自纠正训练让 DiT 学会从自身推理偏差中恢复。

## 方法详解

### 领域定位

dots.tts 属于 **[[Continuous Autoregressive TTS]]** 路线（Route 3），与 [[DiTAR]] / [[VibeVoice]] / VoxCPM 同类。核心差异在于三点：(a) 采用 HoliTok 式多目标 AudioVAE 使 latent 空间兼具语义和声学信息；(b) 三模块解耦而非端到端单体 LM；(c) 引入 SOAR 自纠正后训练，这是同类工作中首次针对 flow-matching head 做 reward-free alignment。

### 端到端数据流（先地图后街景）

dots.tts 的完整流水线：

**48 kHz 语音** → **AudioVAE Encoder**（冻结，输出 128 维 @25 Hz latent 流）→ **Semantic Encoder**（4 倍下采样到 6.25 Hz，提取内容摘要）→ **LLM**（消费 BPE text + 6.25 Hz audio-semantic embedding，输出 hidden state）→ **AR Flow-Matching Head**（条件于 LLM hidden + 全历史 clean patch，生成下一个 4 帧 VAE latent patch）→ **AudioVAE Decoder**（冻结，BigVGAN 解码到 48 kHz 波形）

关键设计：LLM 只看 semantic 压缩后的摘要（6.25 Hz），不看原始 VAE latent（25 Hz）。AR-FM Head 则能看到全部历史 clean patch，保持声学连贯性。

![Figure 1: Overview of the dots.tts backbone](https://arxiv.org/html/2606.07080v1/x1.png)

> **Figure 1**：dots.tts 主干架构概览。BPE 文本 token 与 6.25 Hz audio-semantic embedding 共享一个 LLM 序列；每个 LLM hidden state 条件化 AR-FM head 生成下一个 4 帧 VAE latent patch，该 patch 经 semantic encoder 反馈到 LLM 的下一步输入。AudioVAE 独立训练后冻结。

### AudioVAE：语义结构化的连续语音空间

**为什么这样设计**：连续 AR TTS 的生成目标质量直接取决于 VAE latent 空间的性质。如果 latent 只优化重建而缺乏语义结构，LLM 难以在该空间做有效的长程预测。dots.tts 借鉴 HoliTok 的多目标训练策略，让 latent 同时具备高保真重建和语义可学习性。

**架构**：全因果卷积编码器，下采样步幅 [2,2,2,4,6,10]（总 1920 倍），输出 128 维 latent @25 Hz。解码器采用因果 [[BigVGAN]]-v2 架构，输出 48 kHz 波形。所有卷积层严格因果，支持流式推理。

**两阶段训练**：

- **Stage 1（重建质量）**：多周期 + 多尺度 sub-band CQT 对抗 loss、多尺度 mel 重建 loss、feature matching loss、KL + flow 正则化。500K steps，9.6 秒裁剪片段。
- **Stage 2（可学习性）**：保持 Stage 1 全部 loss，增加两个目标——(i) 帧级对齐 loss 对标冻结的 [[WavLM]] 第 23 层 hidden；(ii) 多任务下游模块（小 encoder + 小 LLM head），联合训练 ASR + 情感 + 说话人分类。200K steps。训练完后**丢弃下游 LLM head**，只保留 encoder（即后续 backbone 中的 semantic encoder）。

**为什么 Stage 2 有效**：WavLM 对齐让 latent 具备语义结构；多任务 supervision 确保 latent 保留足够的内容、情感、说话人信息。这使后续 LLM 能在 6.25 Hz 语义摘要空间做高效推理。

### LLM Backbone：文本 LLM 到语音的迁移

**初始化**：从 [[Qwen2.5]]-1.5B Base 预训练权重初始化。输入为 raw BPE text token（不用 phoneme），音频侧以 6.25 Hz semantic embedding 输入。

**两种序列布局**：
1. **Plain 模式**：完整文本作为前缀，后接音频 span（标准 TTS）
2. **1T1A 交错模式**：`T A T A ... T A <eot> A A ... A`，文本和音频 1:1 交错，实现双流流式。上游对话 LLM 每生成一个 text token，dots.tts 就能开始合成对应的语音片段。

**为什么从文本 LLM 初始化**：继承文本理解能力带来更好的韵律、文本归一化能力、以及自然语言风格指令跟随的可能性。代价是需要更多 speech-text paired data（比 phoneme-based 方案成本高）。

### Semantic Encoder：25 Hz → 6.25 Hz 的内容压缩

AudioVAE Stage 2 训练出的下游 encoder 被直接迁移为 backbone 的 semantic encoder。架构：strided 因果卷积投影器 + 因果 Transformer（hidden 1024, FFN 4096），4 倍下采样将 25 Hz latent patch 压到 6.25 Hz LLM token。严格因果，支持流式。

**设计动机**：让 LLM 只消费内容级摘要，不直接处理声学细节。AR-FM Head 携带完整 LLM hidden history，本质上是一个完整的 text-conditioned 声学生成器（类似 [[ARDiT]]），这推动 LLM 编码语义而非声学信息。

### AR Flow-Matching Head：全历史条件的声学渲染

**为什么这样设计**：连续 AR 的核心挑战是长程漂移。dots.tts 让 AR-FM Head 在每一步都能看到**全部**历史 clean patch，而非仅最近几步，从而保持长程声学一致性。

**DiT 架构**：18 层 [[DiT]]，hidden 1024, FFN 4096, [[RoPE]], [[RMSNorm]]+QK-norm, adaLN-zero 调制（由 diffusion timestep + 说话人 [[X-Vector|x-vector]] 驱动）。

**每步生成过程**：
1. 输入序列由三流拼接：LLM hidden state $H_n$（1 token）+ 所有历史 clean patch $P_{<n}$（各 4 token）+ 当前噪声 patch $Z_n$（4 token）
2. DiT 预测 rectified-flow velocity field，ODE 求解器积分得到 clean patch $P_n$
3. $P_n$ 经 semantic encoder 反馈到 LLM，更新 audio-semantic 输入

**Block-Causal 训练**：单次前向并行处理所有 N 个 patch，通过精心设计的注意力掩码和 [[RoPE]] position ID 复现推理时的逐步上下文。序列分两半：**Cause 部分 C**（clean history）和 **Generation 部分 Z**（noisy targets），四个子块的注意力规则确保训练和推理行为完全一致。

![Figure 2: Attention masks and RoPE position IDs](https://arxiv.org/html/2606.07080v1/x2.png)

> **Figure 2**：AR flow-matching head 的注意力掩码和 RoPE position ID。(a) block-causal 训练掩码；(b) 逐步推理掩码；(c) position ID 对齐方式——Z 段的位置在红色边界处重置，确保训练和推理的 RoPE 相位一致。

**说话人条件**：冻结 [[CAM++]] 提取 [[X-Vector|x-vector]]，作为全局 adaLN-zero condition。[[Classifier-Free Guidance|CFG]]：LLM hidden 和 speaker 流在训练时各自独立 drop（论文写 $p_{\text{drop}}=0.5$），推理时联合做 CFG，scale $\gamma=1.2$。

**具体例子**：假设推理到第 3 步（$n=2$），AR-FM Head 的输入序列为 $[H_0, P_0, H_1, P_1, H_2, Z_2]$（1+4+1+4+1+4=15 token）。DiT 对 $Z_2$ 做 10 步 Euler ODE 积分（加 CFG 则 effective NFE=20），得到 $P_2$。$P_2$ 替换 $Z_2$，经 semantic encoder 得到 1 个 6.25 Hz embedding 送入 LLM，LLM 输出 $H_3$，循环继续。

### 训练流程

**AudioVAE 训练**（独立，完成后冻结）：AdamW, LR 从 $10^{-4}$ 指数衰减到 $10^{-6}$，Stage 1: 500K steps，Stage 2: 200K steps。

**Backbone 预训练**（三阶段，AdamW + WSD schedule）：

| 阶段 | 更新范围 | 数据 | 批大小 | 步数 | 峰值 LR |
|------|---------|------|--------|------|---------|
| Stage 1 (对齐) | Semantic Encoder + AR-FM（LLM 冻结） | 仅 Emilia | ~0.5h/batch | 100K | $2 \times 10^{-4}$ |
| Stage 2 (通用) | 全部模块 | 完整 1.5M 小时 | ~8h/batch | 700K (~4 epochs) | $2 \times 10^{-4}$ |
| Stage 3 (退火) | 全部模块 | 高质量子集 | — | 100K (~1 epoch) | $2 \times 10^{-4} \to 3 \times 10^{-5}$ |

Stage 1 限制为 Emilia 数据是因为全混合数据**严重不稳定**。尽管 LLM 冻结，Stage 1 仍能产出可懂语音（~42% WER），说明 Semantic Encoder + AR-FM Head 自身具备基本生成能力。

### 推理流程

**标准推理**（Pretrain/SOAR 模型）：
1. 编码 prompt 音频 → AudioVAE → latent → semantic encoder → LLM prefill
2. 逐步 AR 循环：LLM 输出 $H_n$ → AR-FM Head 做 10 步 Euler ODE（+ CFG, effective NFE=20）→ 输出 $P_n$ → semantic encoder → 反馈 LLM
3. Stop head（detached MLP on LLM hidden）判断停止

**MeanFlow 蒸馏推理**：用蒸馏后的 student DiT，NFE=2~4，无需 CFG（已融入蒸馏目标），每步只需单次条件前向。

**流式模式**：Plain 模式首包 85 ms（RTF 0.231），1T1A 交错模式首包 54 ms（RTF 0.245），单张 H800 GPU。

### 后训练：SOAR 自纠正对齐 + MeanFlow 蒸馏

**SOAR（Self-corrective Alignment）**：
- **动机**：弥合训练（clean teacher-forced 输入）和推理（自生成、可能偏移的输入）之间的分布差异
- **做法**：只更新 DiT，其他模块冻结。从当前时间步做 detached Euler rollout 一步，得到 off-trajectory 状态，re-noise 后生成 N=6 个辅助训练点，训练 DiT 在这些偏移状态上也能正确回归 clean target
- **Reward-free**：不需要 reward model、人类偏好数据或外部 teacher，纯粹基于 flow-matching 自身一致性
- 50K steps，4h/batch，$\lambda_{\text{aux}}=1.0$，$\gamma_{\text{soar}}=1.2$

**CFG-aware MeanFlow 蒸馏**：
- 冻结 SOAR 后的 DiT 作 teacher，student 继承 teacher 参数 + 新增 interval-duration embedder
- Teacher 用 16 步 Euler+CFG 生成轨迹，student 学习 interval-conditioned mean velocity
- CFG 融入蒸馏目标 → 推理时无需 CFG（省一半前向计算）
- 50K steps，8h/batch，峰值 LR $1 \times 10^{-4}$

## 关键结果

> 只列支撑主结论的核心表/图。完整表格见附录。

**核心证据 1：Seed-TTS-Eval**

| Model | Params | Avg WER(%)↓ | Avg SIM↑ |
|-------|--------|------------|---------|
| CosyVoice 3 | 1.5B | 3.06 | 75.3 |
| Qwen3-TTS | 1.7B | 3.07 | 74.5 |
| Seed-TTS | — | 3.65 | 77.8 |
| VoxCPM 2 | 2B | 3.65 | 76.7 |
| **dots.tts (Pretrain)** | **2B** | **2.92** | **78.8** |
| **dots.tts (SOAR)** | **2B** | **2.95** | **79.2** |
| dots.tts (MF NFE=4) | 2B | 2.94 | 78.2 |

**结论**：dots.tts 在 Seed-TTS-Eval 上取得最优平均 WER（2.92）和 SIM（79.2），SOAR 在 SIM 上领先 Seed-TTS 1.4 个点。MF NFE=4 以约 1 SIM 点代价保持同等 WER。

**核心证据 2：EmergentTTS-Eval 表达力**

dots.tts Pretrain 以 49.2% 总分领跑开源模型（接近 gpt-4o-mini-tts 的 50% baseline），SOAR 在 Syntactic Complexity 维度达 65.7%——全场最高（包括闭源系统），超过 Gemini-2.5-Pro（61.8%）。

**核心证据 3：效率**

| 模式 | 首包延迟 (ms) | RTF |
|------|-------------|-----|
| Plain | 85.4 | 0.231 |
| 1T1A Interleaved | 54.4 | 0.245 |

MF NFE=4 蒸馏 + vLLM 连续批处理 + torch.compile，单张 H800。

## 可复用的设计模式

1. **多目标 VAE 训练策略**：Stage 1 优化重建 + Stage 2 加 SSL 教师对齐 + 多任务 supervision。适用于任何需要 latent 空间同时具备高保真度和语义结构性的生成任务（不限于语音）。来自 AudioVAE 设计。
2. **语义-声学解耦的三模块架构**：将 LLM 的输入限制为低帧率语义摘要，让流生成 head 独立处理声学细节。适用于 LLM-based 生成系统中语义推理与像素/声学渲染的解耦。来自 Semantic Encoder + LLM + AR-FM Head 分离。
3. **Block-Causal 并行训练**：通过精心设计的注意力掩码和 position ID 重置，让 AR 生成的全历史条件模型能在单次前向中并行训练所有步。适用于任何 AR 生成 + per-step diffusion/flow 的架构。来自 §2.5.2。
4. **Reward-free 自纠正后训练**：利用模型自身的 off-trajectory rollout 产生自纠正训练数据，无需外部 reward。适用于 flow-matching / diffusion 模型的推理鲁棒性提升。来自 SOAR 应用。
5. **CFG-aware 蒸馏**：把 CFG 融入蒸馏目标，推理时省去无条件分支的前向计算。适用于任何使用 CFG 的 diffusion/flow 模型加速。来自 MeanFlow 蒸馏。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 核验结论（技术元数据）

> 从 frontmatter relocate 来的 prose 版结论。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | Warm-start from Qwen2.5-1.5B Base。代码中使用 `Qwen2ForCausalLM._from_config(llm_config)` 初始化 LLM，并从预训练 checkpoint 加载权重 | [已 verify §2.3] [已 verify GitHub: core.py:69-72] |
| 训练 loss | Backbone pretraining: $\mathcal{L}_{\text{pre}} = \mathcal{L}_{\text{fm}} + \mathcal{L}_{\text{eos}}$，权重均为 1.0。代码确认 `ce_weight=1.0, fm_weight=1.0, eos_weight=1.0`（ce_loss 用于 LLM text token，fm_loss 用于 flow-matching，eos_loss 用于停止预测）。AudioVAE 有独立 loss（Eq. 2） | [已 verify §2.6, Eq.3-5] [已 verify GitHub: config.py:41-44, model.py:852-874] |
| Tokenizer 架构 | Text + speech 分离：BPE text token 与 6.25 Hz audio-semantic embedding 共享 LLM 序列，但 audio 位置用特殊 span token 标记，输入 embedding 被 semantic encoder 输出替换 | [已 verify §2.3] [已 verify GitHub: core.py:162-177] |
| 多任务 | False。Backbone 训练单任务（TTS）。AudioVAE Stage 2 有 ASR+emotion+speaker 多任务，但下游 head 训练完即丢弃 | [已 verify §2.2, §2.6.2] |
| 训练数据 | 1.5M 小时：~1.2M in-house（中英为主） + ~300K 开源（Emilia, LibriTTS-R, HiFi-TTS, MLS 等） + ~7K caption 数据 | [已 verify §3.1] |
| 后训练 | SOAR（reward-free self-corrective alignment），只更新 DiT，50K steps，无 reward model | [已 verify §2.6.3, §3.2.3] |
| AudioVAE 细节 | 128 维连续 latent @25 Hz，全因果卷积 encoder（strides [2,2,2,4,6,10]），因果 BigVGAN-v2 decoder，48 kHz 输出。hop_size 通过 vocoder 模块暴露 | [已 verify §2.2] [已 verify GitHub: model.py:157, modules/vocoder/bigvgan.py] |
| AR-FM Head DiT | 18 层, hidden 1024, FFN 4096, modulation=True, rotary_bias=True, RMSNorm, QK-norm | [已 verify §2.5.1] [已 verify GitHub: config.py:26-38] |
| Speaker Encoder | 冻结 CAM++ 提取 x-vector，512 维 embedding | [已 verify §2.5.1] [已 verify GitHub: config.py:63, modules/speaker/campplus.py] |
| Semantic Encoder | 因果 Transformer，in_dim=latent_dim, out_dim=llm_hidden_size，4x 下采样（25→6.25 Hz）。代码默认 6 层（_EncoderConfig），论文称 24 层——实际模型从预训练 config 加载，可能覆盖默认值 | [已 verify §2.4] [已 verify GitHub: config.py:8-22, modules/backbone/semantic_encoder.py] |
| CFG | 论文：$p_{\text{drop}}=0.5$，代码默认 `cfg_droprate=0.2, xvec_drop_rate=0.2`。实际模型加载预训练 config 可能覆盖 | [已 verify §2.5.1] [已 verify GitHub: config.py:56,61] |

**代码 vs 论文差异注记**：
- Semantic Encoder 层数：代码 `_EncoderConfig` 默认 6 层，论文 §2.4 称 24 层。预训练 config.json 可能覆盖 Python 默认值。
- CFG drop rate：代码默认 0.2，论文写 0.5。同理，预训练 config 可能覆盖。

## 完整公式

### 公式 1: [[Flow Matching|Flow-Matching Loss]]

$$
\mathcal{L}_{\text{fm}} = \mathbb{E}_{n, t, \epsilon} \left[ \left\| v_\theta(Z_n^t, t, H_{\leq n}, P_{<n}, s) - (P_n - \epsilon) \right\|^2 \right]
$$

**含义**：AR flow-matching head 的核心训练目标，回归直线速度场。

**符号说明**：
- $\epsilon \sim \mathcal{N}(0, I)$：标准高斯噪声
- $P_n$：ground-truth 4 帧 VAE latent patch
- $Z_n^t = (1-t)\epsilon + tP_n$：线性插值的噪声 patch
- $v_\theta$：DiT 预测的速度场
- $H_n$：LLM hidden state
- $s$：全局说话人 x-vector

### 公式 2: [[Cross Entropy|Stop Loss]]

$$
\mathcal{L}_{\text{eos}} = -\frac{1}{2} \log p_{N-1} - \frac{1}{2(N-1)} \sum_{n=0}^{N-2} \log(1 - p_n)
$$

**含义**：平衡二分类交叉熵，让唯一的正样本位置（最后一个 patch）获得与所有负样本相同的总权重。

**符号说明**：
- $p_n = \sigma(\text{MLP}(\text{sg}(H_n)))$：停止概率，MLP 作用于 detached LLM hidden
- $N$：总 patch 数

### 公式 3: AudioVAE 总 Loss

$$
\mathcal{L}_{\text{vae}} = \underbrace{[\mathcal{L}_{\text{mel}} + \mathcal{L}_{\text{adv}} + \mathcal{L}_{\text{fm}} + \beta_{\text{kl}} \mathcal{L}_{\text{kl}}]}_{\text{Stage 1}} + \underbrace{[\lambda_{\text{wavlm}} \mathcal{L}_{\text{wavlm}} + \lambda_{\text{sup}} \mathcal{L}_{\text{sup}}]}_{\text{Stage 2}}
$$

**含义**：AudioVAE 的完整训练目标，Stage 1 追求重建质量，Stage 2 追求语义可学习性。

### 公式 4: SOAR On-trajectory Loss

$$
\ell_{\text{on}}^{(b)} = \omega(\tau^{(b)}) \left\| v_\theta(x_{\tau^{(b)}}^{(b)}, \tau^{(b)}, c^{(b)}) - (x_1^{(b)} - x_0^{(b)}) \right\|^2
$$

### 公式 5: SOAR Off-trajectory 生成

$$
\hat{x}_{\tau_+}^{(b)} = \text{sg}\left[ x_{\tau^{(b)}}^{(b)} + (\tau_+^{(b)} - \tau^{(b)}) \cdot v_\theta^{\text{cfg}}(x_{\tau^{(b)}}^{(b)}, \tau^{(b)}, c^{(b)}; \gamma_{\text{soar}}) \right]
$$

**含义**：detached Euler rollout 一步，产生模型自身推理轨迹上的偏移状态。

### 公式 6: MeanFlow Student Loss

$$
\mathcal{L}_{\text{mv}} = \mathbb{E}\left[ w_{\text{mv}} \cdot \left\| v_\phi(x_{t_a}, t_a, \Delta t, c) - \bar{v}_{t_a \to t_b}^{T, \text{cfg}} \right\|^2 \right]
$$

**含义**：Student 学习 teacher（SOAR 后 DiT）在 $[t_a, t_b]$ 区间的 CFG-aware mean velocity。

**符号说明**：
- $\bar{v}_{t_a \to t_b}^{T,\text{cfg}} \approx (x_{t_b}^{T,\text{cfg}} - x_{t_a}^{T,\text{cfg}}) / (t_b - t_a)$：teacher 轨迹的平均速度
- $w_{\text{mv}} = (\text{sg}(\ell_{\text{mv}}) + \epsilon_{\text{mv}})^{-1/2}$：自适应 per-sample 权重

## 完整图表

### Table 1: AudioVAE 重建质量（LibriSpeech test-other）

| Model | Sample Rate | FPS | PESQ↑ (NB/WB) | STOI↑ | UTMOS↑ | SIM↑ | WER(%)↓ |
|---|---|---|---|---|---|---|---|
| Ground Truth | — | — | 4.55/4.64 | 1.000 | 3.50 | 1.000 | 4.59 |
| XY-Tokenizer | 16kHz | 100 | 2.80/2.23 | 0.89 | 3.46 | 0.82 | 6.19 |
| WavTokenizer | 16kHz | 75 | 2.40/1.96 | 0.87 | 3.22 | 0.68 | 13.35 |
| X-codec2 | 16kHz | 50 | 2.83/2.26 | 0.90 | 3.64 | 0.81 | 6.85 |
| SAC | 16kHz | 62.5 | 2.92/2.39 | 0.90 | 3.84 | 0.85 | 5.77 |
| SemanticVAE | 16kHz | 40 | 3.99/3.80 | 0.969 | 3.76 | 0.963 | 4.15 |
| MingTok-Audio | 16kHz | 50 | 4.23/4.12 | 0.981 | 3.75 | 0.950 | 4.27 |
| **dots.tts VAE** | **48kHz** | **25** | **4.09/3.95** | **0.973** | **3.75** | **0.969** | **4.14** |

**说明**：dots.tts VAE 在 48 kHz/25 Hz 条件下实现接近 ground truth 的重建质量（WER 4.14 vs GT 4.59），重建不构成下游瓶颈。连续表示在所有指标上大幅优于离散 token。

### Table 2: Seed-TTS-Eval 完整结果

| Model | Params | en WER/SIM | zh WER/SIM | zh-hard WER/SIM | Avg WER/SIM |
|---|---|---|---|---|---|
| CosyVoice 3 | 1.5B | 2.22/72.0 | 1.12/78.1 | 5.83/75.8 | 3.06/75.3 |
| F5-TTS | 0.3B | 2.00/67.0 | 1.53/76.0 | 8.67/71.3 | 4.10/71.4 |
| FireRedTTS 2 | 1.5B | 1.95/66.5 | 1.14/73.6 | 8.98/70.3 | 4.02/70.1 |
| IndexTTS 2 | 1.5B | 2.23/70.6 | 1.03/76.5 | 7.12/75.5 | 3.46/74.2 |
| Qwen3-TTS | 1.7B | 1.23/71.7 | 1.22/77.0 | 6.76/74.8 | 3.07/74.5 |
| Seed-TTS | — | 2.25/76.2 | 1.12/79.6 | 7.59/77.6 | 3.65/77.8 |
| DiTAR | 0.6B | 1.69/73.5 | 1.02/75.3 | —/— | —/— |
| VoxCPM 2 | 2B | 1.84/75.3 | 0.97/79.5 | 8.13/75.3 | 3.65/76.7 |
| **dots.tts (Pretrain)** | **2B** | 1.34/76.8 | 0.96/80.5 | 6.46/79.2 | **2.92/78.8** |
| **dots.tts (SOAR)** | **2B** | 1.30/77.1 | 0.94/81.0 | 6.60/79.5 | **2.95/79.2** |
| dots.tts (MF4) | 2B | 1.29/76.2 | 0.94/80.0 | 6.60/78.5 | 2.94/78.2 |
| dots.tts (MF3) | 2B | 1.41/75.9 | 1.02/79.9 | 7.19/78.6 | 3.21/78.1 |
| dots.tts (MF2) | 2B | 1.51/75.2 | 1.04/79.1 | 7.74/76.7 | 3.43/77.0 |

### Table 3: MiniMax 多语言测试集（24 语言平均）

| Model | Avg WER(%)↓ | Avg SIM↑ |
|---|---|---|
| MiniMax-Speech | 2.8 | 76.6 |
| Fish-Audio S2 | 3.7 | 78.0 |
| VoxCPM 2 | 5.7 | 82.3 |
| **dots.tts (SOAR)** | 6.8 | **83.9** |

**说明**：SOAR 在 24 语言中 19 个取得最高 SIM，平均 SIM 83.9 领先 VoxCPM 2 (+1.6)。但 WER 在低资源语言（Arabic 36%, Hindi 14%）较高，BPE 方案在脚本差异大的语言上有劣势。

### Table 4: CV3-Eval

| Model | zh WER | en WER | hard-zh WER | hard-en WER | en→zh SIM | zh→en SIM |
|---|---|---|---|---|---|---|
| CosyVoice 3 | 3.91 | 4.99 | 9.77 | 10.55 | 66.9 | 66.4 |
| dots.tts (SOAR) | 3.71 | 4.50 | 9.22 | 4.49 | 75.0 | 72.8 |
| dots.tts (MF4) | 3.95 | 4.05 | 9.10 | 4.37 | 73.8 | 70.9 |

**说明**：跨语言 voice cloning SIM 大幅领先 CosyVoice 3（+6~8 SIM 点），hard-en WER 降至 4.37%（MF4）。

### Table 5: EmergentTTS-Eval（按 Overall 排序，摘选）

| Model | Voice | WER(%)↓ | Overall↑ | Syntax↑ |
|---|---|---|---|---|
| Gemini-2.5-Flash-TTS | Zephyr | 10.39 | 70.7% | 57.9% |
| gpt-4o-mini-tts (baseline) | Alloy | 10.61 | 50.0% | — |
| **dots.tts (Pretrain)** | basic_ref_en | 10.86 | 49.2% | 58.4% |
| **dots.tts (SOAR)** | basic_ref_en | 10.45 | 47.6% | **65.7%** |
| Qwen3-TTS | basic_ref_en | 17.32 | 42.8% | 60.4% |
| VoxCPM 2 | basic_ref_en | 11.84 | 41.1% | 52.3% |

**说明**：SOAR 的 Syntactic Complexity 65.7% 全场最高，表明 LLM backbone 的文本理解优势在复杂句法场景下显现。但 Complex Pronunciation (16%) 和 Foreign Words (39%) 是明显短板，反映 BPE 对生僻发音的弱点。

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | AudioVAE 重建质量（Table 1）| LibriSpeech 公开 benchmark，标准指标，可复现 |
| **高** | Seed-TTS-Eval（Table 2）| 公开 benchmark，标准协议，基线数字来自原始论文 |
| **中** | MiniMax 多语言测试集（Table 3）| 第三方测试集但非标准 community benchmark，基线来自各自论文/API |
| **中** | EmergentTTS-Eval（Table 5）| 用 Gemini 做评委的 AI 评测，可能有评委偏好；开源模型用统一 prompt 但闭源用各自最优 voice |
| **中** | 效率数据（85/54 ms）| 单 GPU 测量，但未说明负载条件；使用 vLLM + torch.compile 的特定优化配置 |
| **低** | CV3-Eval（Table 4）| CosyVoice 3 发布的评测集，非独立第三方，部分基线缺失 |

## 核心 Claim 审查

1. **Paper Claim**：dots.tts 在 Seed-TTS-Eval 上取得最优平均 WER 和 SIM。
   **My Assessment**：在作者报告的设置下（Whisper-Large-v3 / Paraformer ASR, WavLM-SV 相似度）成立。但"最优"限于已公开的模型，Seed-TTS 本身是闭源系统。

2. **Paper Claim**：SOAR 是 reward-free 的自纠正方法，改善了鲁棒性和声学质量。
   **My Assessment**：SOAR 在 SIM 上稳定提升（78.8→79.2），hard-en WER 从 5.99→4.49 有显著改善。但 EmergentTTS 的 Overall 从 49.2 下降到 47.6，Emotions 从 72.7→63.9，表明 SOAR 可能以表达力为代价换取稳定性。论文未讨论这一 trade-off。

3. **Paper Claim**：连续 AR 去除了离散 token 的信息瓶颈。
   **My Assessment**：Table 1 确认连续 VAE 在重建指标上大幅优于离散 codec。但"去除瓶颈"是否转化为下游 TTS 质量优势，仍需与使用同等规模训练的离散 AR 系统直接对比。当前基线中 Qwen3-TTS（离散 token, 1.7B）的 WER 3.07 与 dots.tts 的 2.95 差距不大。

4. **Paper Claim**：1T1A 交错模式实现 54 ms 首包延迟。
   **My Assessment**：数字可信（MF NFE=4, 单 H800），但实际部署中还需考虑上游对话 LLM 的 text token 生成速度——1T1A 模式的延迟取决于文本生成速度，54 ms 只是 TTS 侧的首包延迟。

## 批判性思考

### 优点
1. **系统性设计**：三模块解耦 + 多阶段训练 + 后训练 + 蒸馏，每个环节有明确的 engineering 动机和消融支持
2. **首个在连续 AR 路线上 beat 主流离散 AR 方案的工作**：之前 DiTAR/VibeVoice 虽然展示了可行性，但未在全面 benchmark 上领先
3. **Block-causal 训练的并行化设计**非常巧妙，是正确利用 attention mask 实现训练-推理一致性的工程范例
4. **完全开源**：code + checkpoints + training pipeline，Apache 2.0，可复现

### 局限性
1. **BPE vs phoneme 的 trade-off 未充分评估**：低资源语言（Arabic WER 36%）表明 BPE 方案在脚本差异大的语言上有严重缺陷，但论文仅在 Limitations 段简要提及
2. **SOAR 的表达力-稳定性 trade-off 未分析**：EmergentTTS 数据显示 SOAR 降低了表达力（Emotions -8.8%），这与 TTS 追求自然表达力的目标矛盾
3. **缺乏消融实验**：没有 AudioVAE Stage 2 vs 不加 Stage 2、三模块解耦 vs 端到端单体 LM、SOAR vs 标准 fine-tuning 的 ablation
4. **模型规模优势未控制**：2B 参数 vs CosyVoice 3 的 1.5B、Qwen3-TTS 的 1.7B，部分性能优势可能来自规模而非架构
5. **训练数据量差异大**：1.5M 小时（其中 1.2M in-house），其他基线的数据规模未必可比

### 潜在改进方向
1. 加入 phoneme auxiliary input 缓解低资源语言问题
2. 在 SOAR 中加入表达力保持约束
3. 探索 Semantic Encoder 与 LLM 共享参数以减少参数量
4. 在 AudioVAE 上支持 singing/music 扩展

### 可复现性评估
- [x] 代码开源（Apache 2.0）
- [x] 预训练模型（pretrain / SOAR / MF 三个 checkpoint）
- [ ] 训练细节部分完整（1.2M in-house 数据不可获取）
- [ ] 数据集部分可获取（300K 开源部分可获取，in-house 不可获取）

---

# 三、知识系统层

## 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §Route 3 连续表示 AR 生成 + §Route 4 Diffusion/Flow TTS（AR-FM Head）
- **核心问题**：[[TTS-核心挑战]] §挑战 1 零样本克隆 / §挑战 3 延迟与流式
- **表示层位置**：[[TTS-表示层地图]] §连续 latent（128 维 VAE @25 Hz）
- **相邻工作**：[[DiTAR]]（同属连续 AR + per-patch flow-matching）/ [[VibeVoice]]（同属连续 AR）/ [[VoxCPM]]（同属连续 AR）/ [[ARDiT]]（灵感来源，decoder-only DiT）/ [[CosyVoice3]]（离散 AR 强基线）/ [[Qwen3-TTS]]（LLM-native TTS 对照）

## 后续重估

- **2026-06-09**：初读。连续 AR TTS 路线的重要里程碑，首次在全面 benchmark 上整体优于主流离散 AR 方案。三模块解耦 + SOAR 自纠正 + MeanFlow 蒸馏的工程完成度高。但 2B 参数 + 1.5M 小时数据的规模优势需要控制，低资源语言弱点反映 BPE 方案的固有局限。SOAR 的表达力-稳定性 trade-off 值得关注——后续如果有 SOAR + 表达力保持的改进版，需重新评估。

---

## 关联笔记

### 基于
- [[Qwen2.5]]: LLM backbone 的预训练基座
- [[BigVGAN]]: AudioVAE decoder 架构
- [[WavLM]]: AudioVAE Stage 2 的 SSL 教师

### 对比
- [[DiTAR]]: 同属连续 AR + per-patch flow，但 dots.tts 规模更大 + 加了 SOAR
- [[VibeVoice]]: 同属连续 AR，dots.tts 在 Seed-TTS-Eval 上显著领先
- [[CosyVoice3]]: 离散 AR 强基线，dots.tts 平均 WER 低 0.11 个点、SIM 高 3.9 个点
- [[Qwen3-TTS]]: LLM-native TTS 对照，dots.tts 连续 AR 路线 vs Qwen3-TTS 离散 token 路线

### 方法相关
- [[Continuous Autoregressive TTS]]: 核心范式
- [[Flow Matching]]: AR-FM Head 的生成范式
- [[Classifier-Free Guidance]]: 推理时的引导策略
- [[DiT]]: AR-FM Head 的 backbone 架构

### 硬件/数据相关
- [[Emilia-YODAS|Emilia]]: 开源训练数据的主要来源
- [[Seed-TTS-eval]]: 主评测 benchmark

---

## 速查卡片

> [!summary] dots.tts Technical Report
> - **核心**: 2B 参数连续 AR TTS，三模块解耦（AudioVAE + LLM + AR-FM Head）
> - **方法**: HoliTok 式多目标 AudioVAE + Qwen2.5-1.5B warm-start LLM + 18 层 DiT flow-matching head + SOAR 自纠正后训练 + CFG-aware MeanFlow 蒸馏
> - **结果**: Seed-TTS-Eval 最优平均 WER 2.92/SIM 79.2；首包延迟 54 ms（1T1A 模式）；Syntactic Complexity 65.7% 全场最高
> - **代码**: [GitHub](https://github.com/rednote-hilab/dots.tts)（Apache 2.0，含 pretrain/SOAR/MF 三个 checkpoint）

---

*笔记创建时间: 2026-06-09*
