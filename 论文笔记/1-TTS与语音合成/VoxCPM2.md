---
title: "VoxCPM2 Technical Report"
method_name: "VoxCPM2"
authors: [Yixuan Zhou, Guoyang Zeng, Xin Liu, Xiang Li, Renjie Yu, Jiancheng Gui, Jiaheng Wu, Ziyang Wang, Xudong Shen, Runchuan Ye, Zhisheng Zhang, Jiuyang Zhou, Bingsong Bai, Weiyue Sun, Mengyuan Deng, Qundong Shi, Zhiyong Wu, Zhiyuan Liu]
year: 2026
venue: arXiv
arxiv_id: "2606.06928"
tags: [zero-shot-tts, tokenizer-free, hierarchical-modeling, flow-matching, multilingual-tts, controllable-tts, instruction-tts, voice-cloning, continuous-latent]
note_tier: heavy

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: false
post_training_type: none
streaming: true

# === 知识地图联动 ===
domain: TTS
subdomain: multilingual-controllable
routes: [diffusion-flow-tts, controllable-tts, instruction-tts, streaming-tts]
problems: [zero-shot-cloning, multilinguality, instruction-following, prosody-control, emotion-style-control]
representations: [continuous-latent]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-表示层地图]]"
related_surveys: []
evidence_level: medium
maturity: emerging
last_repositioned: 2026-06-09

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.06928/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.06928/paper.html"
figures_dir: "_resources/2606.06928/figures"
github_local: "~/DailyPaper/.cache/papers/2606.06928/github/OpenBMB_VoxCPM"
cached_at: 2026-06-09

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2606.06928v1
created: 2026-06-09
---

# 论文笔记：VoxCPM2 Technical Report

> **笔记分级**：heavy（2B 参数、2M+ 小时训练数据、30 语种、多模式统一的工业级 TTS 基础模型系统报告）。分级标准见 `references/quality-standards.md §模板分级`。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Tsinghua SIGS (THUHCSI), THUNLP, ModelBest (面壁智能), OpenBMB |
| 日期 | June 2026 |
| 项目主页 | https://github.com/OpenBMB/VoxCPM |
| 对比基线 | [[CosyVoice]]、[[CosyVoice2]]、[[CosyVoice3]]、[[F5-TTS]]、[[MaskGCT]]、[[Spark-TTS]]、[[FireRedTTS]]、[[FireRedTTS2]]、[[IndexTTS2]]、[[Qwen3-TTS]]、[[Fish-Speech]] S2、[[LongCat-Audio-DiT]] |
| 链接 | [arXiv](https://arxiv.org/abs/2606.06928) / [Code](https://github.com/OpenBMB/VoxCPM) / [Nano-vLLM](https://github.com/a710128/nanovllm-voxcpm) |

## 一句话总结

> VoxCPM2 将无 tokenizer 的分层连续 latent TTS 框架扩展至 2B 参数/2M+ 小时/30 语种/48 kHz，通过统一序列组织实现基础合成、语音设计、可控克隆、续写克隆四种模式在一套参数下运行，在多个公开 benchmark 上取得有竞争力的结果。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 必要图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源统一收到「附录·核验结论」表。

## 核心贡献

1. **分层连续 latent 框架工业化**：将 [[VoxCPM]] 的 TSLM-FSQ-RALM-LocDiT 四组件架构从 0.6B 扩展到 2B 参数，TSLM 升级至 [[MiniCPM]]-4-1B，FSQ bottleneck 从 256 维扩至 512 维，输出从 16 kHz 提升至 48 kHz
2. **非对称 AudioVAE V2**：编码器接收 16 kHz 输入，解码器输出 48 kHz，实现隐式超分辨率，同时避免了高采样率带来的序列长度爆炸和训练数据不统一问题
3. **统一序列组织**：通过 text/reference-audio/target-audio 三种 building block 的不同排列组合，在一套模型参数下实现基础多语言 TTS、自然语言声音设计、风格可控克隆、续写克隆四种生成模式
4. **30 语种覆盖**：内部评测平均 WER 1.68%，28 个语种低于 3%；Apache 2.0 全量开源

## 问题背景

### 要解决的问题

当前 TTS 领域存在两条主流路线的困境：离散 token + LM 路线量化不可避免地丢失细粒度声学细节；连续 latent 路线需要同时优化语义-韵律结构和局部声学纹理，导致优化困难。VoxCPM 提出了分层架构来解决这个问题，但 v1 受限于规模（0.6B/16 kHz）和功能（仅基础 TTS + 续写）。

### 现有方法的局限

- 离散 codec LM 路线（[[VALL-E]]、[[CosyVoice]] 系列等）依赖外部 tokenizer，量化残差导致细节损失
- 多阶段 pipeline（AR LM + diffusion/flow decoder）割裂了语义规划和声学渲染
- VoxCPM v1 仅 0.6B 参数、16 kHz 输出、不支持自然语言可控生成

### 本文的动机

VoxCPM 的分层连续 latent 范式（TSLM 做语义规划 → FSQ 压缩为稳定骨架 → RALM 恢复声学细节 → LocDiT 生成连续 latent patch）已被验证可行。VoxCPM2 的目标是在三个维度上推进：**能力**（多语言 + 可控）、**质量**（48 kHz）、**规模**（2B/2M+ 小时）。

## 方法详解

### 领域定位

VoxCPM2 属于 **连续 latent + diffusion-autoregressive 混合**路线，与 [[DiTAR]]（引入 LocDiT 并被 VoxCPM 系列复用）、CLEAR、VibeVoice 等同属分层 diffusion-autoregressive 家族。核心差异在于：VoxCPM2 在**单一连续 latent 骨架内**实现语义-声学分层（通过可微 [[FSQ]] bottleneck），不依赖任何外部离散 codec tokenizer。这使得它是目前 tokenizer-free 连续 latent TTS 中规模最大（2B）、语种覆盖最广（30 语种 + 9 方言）的开源系统。

### 端到端数据流（先地图后街景）

VoxCPM2 的完整流水线：

**输入**（文本 + 可选参考音频 + 可选描述） → **AudioVAE V2 Encoder**（16 kHz 波形 → 64 维 latent @25 Hz） → **LocEnc**（latent patch → 上下文化的 acoustic history，P=4 合并得到 6.25 Hz token rate） → **TSLM**（MiniCPM-4-1B，因果 LM，做文本-音频联合建模，输出经 FSQ 量化的语义骨架） → **Fusion + RALM**（FSQ 骨架 + acoustic history 拼接投影后送入 RALM，恢复声学残差细节） → **LocDiT**（接收 TSLM 语义隐状态 + RALM 残差隐状态 + 上一步 patch 作条件，用 [[Conditional Flow Matching]] 生成当前步的连续 latent patch） → **AudioVAE V2 Decoder**（连续 latent → 48 kHz 波形输出）

![Figure 1: Overall architecture of VoxCPM2](https://arxiv.org/html/2606.06928v1/x1.png)

> **Figure 1**：VoxCPM2 整体架构。从左到右：输入文本经 TSLM 的 embedding 层处理，音频经 AudioVAE V2 编码后通过 LocEnc 得到 patch-level embedding；两路在 TSLM 内因果建模，TSLM 输出经 FSQ bottleneck 量化后产生语义骨架；语义骨架与 acoustic embedding 拼接投影后送入 RALM 恢复残差；TSLM 和 RALM 的隐状态分别投影为 LocDiT 的独立 prefix token（而非 v1 的求和），连同前一步 patch 和噪声 patch 一起输入 LocDiT，由 CFM solver 去噪后得到当前步 latent patch；最终所有 patch 送入 AudioVAE V2 Decoder 输出 48 kHz 波形。

下面逐个放大每个关键子系统。

### 子系统 A：AudioVAE V2（非对称编解码器）

**为什么这样设计**：高采样率（48 kHz）输出能显著提升保真度，但如果编码器也用 48 kHz 输入，会带来两个问题：(1) 现有大量训练语料是 16 kHz 的，无法复用；(2) latent 序列变长，增加 AR 循环成本。解决方案是非对称设计——编码端固定 16 kHz，解码端输出 48 kHz，让解码器学习隐式超分辨率。

**怎么做**：

- **Encoder**：因果 CNN，下采样率 [2, 5, 8, 8]（总 640 倍时域压缩），将 16 kHz 波形映射为 64 维 latent frames @25 Hz，输出 mu 和 logvar（VAE 形式）
- **Decoder**：更深更宽的因果 CNN，上采样率 [8, 6, 5, 2, 2, 2]（总 960 倍），接受可选的目标采样率条件（通过 `SampleRateConditionLayer` 以 scale-bias 注入），输出 48 kHz 波形
- Patch size P=4 → LM 侧 token rate = 25 Hz / 4 = **6.25 Hz**（每步 160 ms）

**具体例子**：一段 10 秒的 16 kHz 音频（160,000 samples）经 encoder 下采样 640 倍得到 250 个 latent frames；以 P=4 合并后 TSLM 只需处理 63 个 token，每步 LocDiT 生成 4 帧 latent；decoder 将 250 帧 latent 上采样 960 倍输出 240,000 samples @48 kHz（即 5 秒——因为 48 kHz 下 5 秒 = 240k samples，但实际输出时长仍为 10 秒，因为 decoder 内部帧率匹配不同）。

### 子系统 B：TSLM（文本-语义语言模型）

**为什么这样设计**：语义规划需要对文本和音频上下文做全局因果建模。采用 MiniCPM-4-1B 作为骨架（28 层，H=2048），利用其预训练语言知识加速收敛。

**怎么做**：
- 文本和音频通过 modality indicator 合并为统一序列：`combined = text_mask * text_embed + audio_mask * feat_embed`
- 因果 attention 处理后，音频位置的输出经 FSQ 量化（512 维，9 级/维），文本位置保持原始输出
- FSQ 用 straight-through estimator（训练时 round 后 detach 梯度），产生"语义骨架"

### 子系统 C：Fusion + RALM（残差声学语言模型）

**为什么这样设计**：FSQ 压缩了信息，RALM 的任务是恢复被量化丢掉的声学细节。v1 用 element-wise 求和融合 FSQ 输出和 acoustic history，会丢失信息；v2 改为拼接-投影融合。

**怎么做**：

$$
h_i^{\text{res\_in}} = W_{\text{fuse}} [h_i^{\text{FSQ}} \| E_i]
$$

**含义**：将 FSQ 量化后的语义骨架和 LocEnc 编码的 acoustic history 沿特征维拼接，经线性投影后送入 RALM。

**符号**：$h_i^{\text{FSQ}}$ = TSLM 第 $i$ 步经 FSQ 量化的输出，$E_i$ = LocEnc 对位置 $i$ 音频特征的编码，$W_{\text{fuse}} \in \mathbb{R}^{d \times 2d}$ = 可学习投影矩阵。

RALM 配置：8 层，H=2048，**无位置编码**（NoPE 设计）。去除 RoPE 是因为 RALM 做的是局部声学修正，不需要绝对位置信息，移除 PE 能减少对训练长度的过拟合，提升长音频稳定性。

### 子系统 D：LocDiT（局部扩散 Transformer）

**为什么这样设计**：LocDiT 在每个 AR 步内做局部的 [[Conditional Flow Matching]] 生成。v1 将 TSLM 语义隐状态、RALM 残差隐状态、timestep embedding 求和成单个 token 作为 LocDiT 条件；v2 改为分开投影成独立 prefix token，让 LocDiT 能通过 attention 自行学习如何利用各信息源。

**怎么做**：LocDiT 的输入序列为：

$$
[\mu_{\text{sem}}, \mu_{\text{res}}, \mu_t, z_{i-1}^{(1)}, \ldots, z_{i-1}^{(P)}, \tilde{z}_i^{(1)}, \ldots, \tilde{z}_i^{(P)}]
$$

**含义**：$\mu_{\text{sem}}$ = TSLM 语义隐状态投影，$\mu_{\text{res}}$ = RALM 残差隐状态投影，$\mu_t$ = timestep embedding，$z_{i-1}^{(1..P)}$ = 上一步 latent patch（作条件），$\tilde{z}_i^{(1..P)}$ = 当前步噪声 patch（待去噪目标）。LocDiT 12 层全 attention 处理后，在噪声 patch 位置预测速度场。

配置：12 层，H=1024，CFM solver（Euler 方法，默认 10 步），sway sampling + CFG-Zero* 默认启用。

### 子系统 E：统一序列组织

VoxCPM2 通过三种 building block（text、reference audio、target audio）的不同排列实现五种生成模式：

| 模式 | 序列布局 | 用途 |
|------|---------|------|
| 基础 TTS | text -> target | 标准多语言合成 |
| 语音设计 | (voice description + text) -> target | 用自然语言描述声音特征 |
| 参考克隆 | [ref_audio] \| text -> target | 用参考音频克隆音色 |
| 可控克隆 | [ref_audio] \| (style desc + text) -> target | 克隆音色 + 指令控制风格 |
| 续写克隆 | (prompt_text + target_text) \| prompt_audio -> target | 续写式克隆（需配对文本） |

关键设计：(1) voice description / style description 直接拼接在 text 前面，作为 TSLM 的普通文本前缀处理，不需要额外的 style encoder/adapter；(2) reference audio 通过 REF_START/REF_END 特殊 token 界定，插在序列开头，不参与 loss 计算，仅作纯条件信号；(3) 续写模式可与参考模式叠加（Reference + Continuation），实验显示这种组合在 SIM 指标上最优。

### 训练流程

**目标函数**：两项 loss —— (1) patch-level CFM flow matching loss（MSE on velocity field），(2) binary stop-prediction loss（CE on TSLM-FSQ hidden states）。两项 loss 等权（lambda=1.0:1.0）。Loss 仅在 target audio 段计算。CFG dropout rate = 10%。

**三阶段渐进课程**：

| 阶段 | 数据 | 序列长度 | 重点 |
|------|------|---------|------|
| Stage 1: 多语言预训练 | 大规模多语言 (text, audio) 对 | <=4096，音频<=60s | 建立 30 语种发音和韵律基础 |
| Stage 2: 联合预训练 | 保留大量纯 TTS + 逐步引入可控数据 | <=8192，音频<=3min | 引入 NL 描述、参考音频三元组 |
| Stage 3: 高质量退火 SFT | 精选高质量子集，更多表达性语音 | 8192，2s-5min | 提高可控准确率，语种均衡采样 |

### 推理流程

**CFG 推理**：

$$
\hat{v} = v_{\text{uncond}} + \alpha(v_{\text{cond}} - v_{\text{uncond}})
$$

默认 $\alpha = 2.0$，推荐范围 [1.5, 3.0]。

**Sway sampling**：将更多 solver 步分配到高噪声区间。**CFG-Zero***：在最初约 4% 的步数中将预测置零，减少早期步伪影。两者默认启用，无额外可学习参数。

**流式推理**：因果 TSLM/RALM + patch-local LocEnc/LocDiT 天然支持 chunk-based 流式。每生成一个 patch 立即通过有状态 AudioVAE V2 Decoder 解码输出。AudioVAE V2 Decoder 通过 `StreamingVAEDecoder` 维护因果卷积的 padding buffer 实现无重叠流式解码。

## 关键结果

> 只列支撑主结论的核心表/图。完整表格见附录。

### 零样本克隆（Seed-TTS-Eval）

VoxCPM2 在开源模型中取得强竞争力，EN WER 1.84 / SIM 75.3，ZH CER 0.97 / SIM 79.5：

| 模型 | 参数 | 开源 | EN WER | EN SIM | ZH CER | ZH SIM |
|------|------|------|--------|--------|--------|--------|
| Qwen3-TTS | 1.7B | Y | 1.23 | 71.7 | 1.22 | 77.0 |
| Fish Audio S2 | 4B | Y | 0.99 | -- | 0.54 | -- |
| LongCat-Audio-DiT | 3.5B | Y | 1.50 | 78.6 | 1.09 | 81.8 |
| MOSS-TTS | 8B | Y | 1.85 | 73.4 | 1.20 | 78.8 |
| **VoxCPM2** | **2B** | **Y** | **1.84** | **75.3** | **0.97** | **79.5** |

**结论**：VoxCPM2 的 speaker similarity（SIM）在开源模型中仅次于 LongCat-Audio-DiT（3.5B），且在 ZH CER 上表现优秀（0.97）。但在 EN WER 上弱于 Qwen3-TTS 和 Fish Audio S2。Reference + Continuation 组合推理是最优 recipe（EN WER 0.99, SIM 79.5）。

### 多语言能力（MiniMax-MLS-Test）

VoxCPM2 在 24 语种 SIM 评测中 **22/24 语种排名第一**，平均 SIM 显著领先所有对比系统。在 WER 方面，强势语种（中/英/德/荷/芬/土耳其）具竞争力，但在阿拉伯语、捷克语、罗马尼亚语等低资源语种上仍有显著差距。

### 可控生成（InstructTTSEval）

| 模型 | ZH-APS | ZH-DSD | ZH-RP | EN-APS | EN-DSD | EN-RP |
|------|--------|--------|-------|--------|--------|-------|
| Gemini-TTS-Pro | 89.0 | 90.1 | 75.5 | 87.6 | 86.0 | 67.2 |
| Qwen3-TTS-VD | 85.2 | 81.1 | 65.1 | 82.9 | 82.4 | 68.4 |
| **VoxCPM2** | **85.2** | 71.5 | 60.8 | **84.2** | **83.2** | **71.4** |

**结论**：英文方面整体最优（APS/DSD/RP 三项均最高或次高）。中文 APS 并列第一，但 DSD/RP 弱于 Gemini-TTS-Pro 和 Qwen3-TTS-VD。

### 推理效率

| 推理路径 | 参数 | RTF | VRAM |
|---------|------|-----|------|
| VoxCPM2 (PyTorch) | 2B | 0.30 | ~8 GB |
| VoxCPM2 (Nano-vLLM) | 2B | 0.13 | ~8 GB |

使用 Nano-vLLM 在单张 RTX 4090 上实现 7 倍实时速度，8 GB 显存。

## 可复用的设计模式

1. **非对称编解码器超分辨率**：编码端用低采样率（降低 AR 成本 + 复用现有语料），解码端用高采样率（提升输出质量）。适用于任何编码-生成-解码架构中需要平衡效率与质量的场景。来自 AudioVAE V2 的 16kHz->48kHz 设计。

2. **可微 FSQ 做语义-声学分层**：在统一连续 latent 框架内用 Finite Scalar Quantization 创建信息瓶颈，将"高层语义规划"与"低层声学修正"自然分层，无需外部 tokenizer。适用于希望在单一骨架内实现分层建模的场景。来自 VoxCPM 系列的 TSLM-FSQ-RALM 设计。

3. **多 prefix token 条件注入**：LocDiT 将语义、残差、timestep 各自投影为独立 prefix token（而非求和成一个），让模型通过 attention 自行学习如何利用各信息源。适用于任何需要多条件融合的 DiT/diffusion 生成器。来自 VoxCPM2 对 LocDiT 的改进。

4. **统一序列组织的多模式支持**：用 building block（text/ref-audio/target-audio）的排列组合表达不同生成模式，共享一套参数。适用于需要多功能统一的 TTS/语音系统。voice description 直接作为文本前缀处理，无需专用模块。

5. **RALM NoPE 设计**：对负责局部声学修正的模块去除位置编码，减少对训练长度的过拟合，提升长音频推理稳定性。适用于任何做局部/patch-level 修正的 Transformer 模块。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 核验结论（技术元数据）

> 来源标注：`[已 verify §X / Eq.X / Tab.X / Fig.X]` 或 `[GitHub: <path>:<line>]`。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start from MiniCPM-4-1B（28 层 H=2048）；代码中 TSLM = `MiniCPMModel(config.lm_config)` 使用 `MiniCPM4Config`，加载预训练权重 | [已 verify §1.2, §3.3.3] [GitHub: model/voxcpm2.py:L179, modules/minicpm4/config.py] |
| 训练 loss | 两项：(1) patch-level CFM flow matching loss（MSE on velocity field）(2) binary stop-prediction loss（CE）；等权 1.0:1.0；仅在 target-audio 段计算 | [已 verify §3.5] [GitHub: model/voxcpm2.py:L360-371, conf/voxcpm_v2/voxcpm_finetune_all.yaml:L21-22] |
| Tokenizer 架构 | text+speech 分离（text 用 LlamaTokenizerFast，audio 通过 AudioVAE V2 编码为连续 latent，两路通过 modality mask 合并）；无离散 audio tokenizer | [已 verify §3.1] [GitHub: model/voxcpm2.py:L182, L327-328] |
| 多任务 | false——单任务 TTS，但通过统一序列组织支持多种生成模式（基础TTS/声音设计/克隆/续写），仍然是同一个 TTS loss | [已 verify §3.4, §3.5] |
| 训练数据 | 2M+ 小时多语言语音；中英占大部分；其余 28 语种约 1K-50K 小时/语种；可控数据含数万小时开源表达性语音 + 数千小时内部标注 | [已 verify §3.6] |
| 后训练 | 无 RLHF/DPO；Stage 3 是高质量退火 SFT，不属于后训练对齐 | [已 verify §3.5] |
| AudioVAE V2 细节 | Encoder: causal CNN, rates=[2,5,8,8], latent_dim=64, @25Hz; Decoder: causal CNN, rates=[8,6,5,2,2,2], 16kHz->48kHz, 带 SampleRateConditionLayer | [已 verify §3.2] [GitHub: modules/audiovae/audio_vae_v2.py:L359-370] |
| FSQ bottleneck | 512 维, 9 级/维, tanh + round + straight-through estimator | [已 verify §3.3.1] [GitHub: modules/layers/scalar_quantization_layer.py:L6-27, model/voxcpm2.py:L119-120] |
| RALM NoPE | `residual_lm_no_rope` 配置项控制；论文称 VoxCPM2 移除 RALM 的 RoPE | [已 verify §3.3.1] [GitHub: model/voxcpm2.py:L122, modules/minicpm4/config.py:L30] |
| 融合方式 | concat-projection: `fusion_concat_proj = nn.Linear(hidden_size*2, hidden_size)` | [已 verify §3.3.1] [GitHub: model/voxcpm2.py:L231, L334-335] |
| LocDiT multi-token | 独立 prefix：`x = torch.cat([mu, t.unsqueeze(1), cond, x], dim=1)`，mu/t/cond 作为独立 token | [已 verify §3.3.1] [GitHub: modules/locdit/local_dit_v2.py:L110] |
| CFG dropout | 10% (`training_cfg_rate: float = 0.1`) | [已 verify §3.5] [GitHub: modules/locdit/unified_cfm.py:L18] |
| 流式解码 | `StreamingVAEDecoder` 维护因果卷积 padding buffer，支持 chunk-by-chunk 无重叠解码 | [已 verify §3.7] [GitHub: modules/audiovae/audio_vae_v2.py:L504-580] |

## 完整公式

### 公式 1: [[Conditional Flow Matching|核心生成公式]]

$$
z_i \sim \text{LocDiT}(h_i^{\text{FSQ}}, h_i^{\text{residual}}, z_{i-1}; t)
$$

$$
h_i^{\text{FSQ}} = \text{FSQ}(\text{TSLM}(\mathbf{T}, \mathbf{E}_{<i}))
$$

$$
h_i^{\text{residual}} = \text{RALM}(\mathbf{H}_{\text{text}}^{\text{TSLM}}, \mathbf{H}_{\leq i}^{\text{FSQ}} \oplus \mathbf{E}_{<i})
$$

**含义**：第 $i$ 步的 latent patch $z_i$ 由 LocDiT 生成，以 TSLM 的 FSQ 语义骨架和 RALM 的残差信息为条件，结合前一步 patch $z_{i-1}$。

**符号说明**：
- $\mathbf{T}$ = 输入文本 token 序列
- $\mathbf{E}_{<i}$ = LocEnc 编码的 patch-level 声学历史
- $t$ = diffusion timestep
- $\text{FSQ}$ = 有限标量量化（per-dimension scalar quantization）
- $\oplus$ = 融合算子（VoxCPM2 中为 concat-projection）

### 公式 2: [[Conditional Flow Matching|拼接-投影融合]]

$$
h_i^{\text{res\_in}} = W_{\text{fuse}} [h_i^{\text{FSQ}} \| E_i]
$$

**含义**：将 FSQ 输出和 acoustic embedding 沿特征维拼接后线性投影，送入 RALM。

**符号说明**：
- $W_{\text{fuse}} \in \mathbb{R}^{d \times 2d}$ = 可学习投影矩阵
- $\|$ = 特征维拼接

### 公式 3: [[Classifier-Free Guidance|CFG 推理]]

$$
\hat{v} = v_{\text{uncond}} + \alpha (v_{\text{cond}} - v_{\text{uncond}})
$$

**含义**：用 unconditioned 和 conditioned 预测的加权组合增强生成质量。

**符号说明**：
- $\alpha$ = CFG scale，默认 2.0
- $v_{\text{cond}}$ / $v_{\text{uncond}}$ = 有/无条件下的速度场预测

### 公式 4: [[FSQ|FSQ 量化]]

$$
q(x) = \frac{\text{round}(\tanh(W_{\text{in}} \cdot h) \cdot s)}{s}
$$

$$
\text{output} = W_{\text{out}} \cdot q(x)
$$

**含义**：先线性投影到 latent 维（512 维），tanh 归一化到 [-1,1]，乘以 scale=9 后 round 再除回，straight-through estimator 传梯度。

**符号说明**：
- $s = 9$ = 量化级数
- $W_{\text{in}} \in \mathbb{R}^{512 \times d}$, $W_{\text{out}} \in \mathbb{R}^{d \times 512}$ = 投影矩阵

## 完整图表

### Table 1: VoxCPM 系列配置对比

| 组件 | VoxCPM | VoxCPM1.5 | VoxCPM2 |
|------|--------|-----------|---------|
| Backbone 参数 | ~0.6B | ~0.8B | ~2B |
| LocEnc | 4L, H=1024 | 8L, H=1024 | 12L, H=1024 |
| TSLM | MiniCPM-4-0.5B (24L, H=1024) | MiniCPM-4-0.5B (24L, H=1024) | MiniCPM-4-1B (28L, H=2048) |
| FSQ latent dim | 256 | 256 | 512 |
| RALM | 6L, H=1024 | 8L, H=1024 | 8L, H=2048 |
| LocDiT | 4L, H=1024 | 8L, H=1024 | 12L, H=1024 |
| Patch size P | 2 | 4 | 4 |
| LM 侧 token rate | 12.5 Hz | 6.25 Hz | 6.25 Hz |
| 最大序列长度 | 4096 | 4096 | 8192 |
| 输入采样率 | 16 kHz | 44.1 kHz | 16 kHz |
| 输出采样率 | 16 kHz | 44.1 kHz | 48 kHz |

### Table 3: Seed-TTS-Eval 零样本克隆完整结果

| 模型 | 参数 | 开源 | EN WER | EN SIM | ZH CER | ZH SIM | ZH-Hard CER | ZH-Hard SIM |
|------|------|------|--------|--------|--------|--------|-------------|-------------|
| CosyVoice3.5 | -- | N | 1.57 | 73.8 | 0.87 | 79.7 | 5.71 | 78.6 |
| MiniMax-Speech | -- | N | 1.65 | 69.2 | 0.83 | 78.3 | -- | -- |
| Seed-TTS | -- | N | 2.25 | 76.2 | 1.12 | 79.6 | 7.59 | 77.6 |
| F5-TTS | 0.3B | Y | 2.00 | 67.0 | 1.53 | 76.0 | 8.67 | 71.3 |
| CosyVoice 3 (0.5B) | 0.5B | Y | 2.02 | 71.8 | 1.16 | 78.0 | 6.08 | 75.8 |
| Qwen3-TTS | 1.7B | Y | 1.23 | 71.7 | 1.22 | 77.0 | 6.76 | 74.8 |
| Fish Audio S2 | 4B | Y | 0.99 | -- | 0.54 | -- | 5.99 | -- |
| OmniVoice | 0.8B | Y | 1.60 | 74.1 | 0.84 | 77.7 | -- | -- |
| LongCat-Audio-DiT | 3.5B | Y | 1.50 | 78.6 | 1.09 | 81.8 | 6.04 | 79.7 |
| MOSS-TTS | 8B | Y | 1.85 | 73.4 | 1.20 | 78.8 | -- | -- |
| VoxCPM | 0.6B | Y | 1.85 | 72.9 | 0.93 | 77.2 | 8.87 | 73.0 |
| **VoxCPM2** | **2B** | **Y** | **1.84** | **75.3** | **0.97** | **79.5** | **8.13** | **75.3** |

### Table 4: 推理 recipe 消融

| Recipe | EN WER | EN SIM | ZH CER | ZH SIM | ZH-Hard CER | ZH-Hard SIM |
|--------|--------|--------|--------|--------|-------------|-------------|
| Continuation only | 1.01 | 77.7 | 1.97 | 72.6 | 8.16 | 72.4 |
| Reference only | 1.10 | 75.3 | 1.81 | 67.0 | 6.85 | 70.0 |
| Ref + Continuation | 0.99 | 79.5 | 1.94 | 75.2 | 7.44 | 74.9 |

### Table 8: 内部 30 语种评测（部分）

| 语种 | 指标 | VoxCPM2 | Fish Audio S2 |
|------|------|---------|---------------|
| en | WER | 0.42 | 1.03 |
| zh | CER | 0.92 | 1.02 |
| km (高棉) | CER | 2.05 | 75.15 |
| lo (老挝) | CER | 1.90 | 87.40 |
| my (缅甸) | CER | 1.42 | 85.27 |
| th (泰语) | CER | 0.94 | 1.92 |
| **平均** | | **1.68** | -- |

**说明**：VoxCPM2 在高棉/老挝/缅甸等 Fish Audio S2 完全失败的语种上保持低 CER，显示 30 语种训练策略的有效性。

### Table 10: AudioVAE V2 重建质量

| VAE 模型 | 输入 SR | 输出 SR | VCTK MelD-48k | VCTK MelD-16k | VCTK STOI-16k | VCTK PESQ-16k |
|---------|---------|---------|---------------|---------------|---------------|---------------|
| VoxCPM (v1) | 16kHz | 16kHz | 1.787 | 0.801 | 0.911 | 3.940 |
| VoxCPM1.5 | 44kHz | 44kHz | 1.139 | 0.926 | 0.836 | 3.148 |
| VoxCPM2 | 16kHz | 48kHz | 1.335 | 0.813 | 0.907 | 3.906 |

### Table 12-14: 主观评测

| 评测 | 系统 | N-MOS | S-MOS/I-MOS |
|------|------|-------|-------------|
| 零样本克隆 | IndexTTS2 | 4.78 | S-MOS 4.71 |
| | Qwen3-TTS | 4.80 | S-MOS 4.69 |
| | **VoxCPM2** | **4.78** | **S-MOS 4.74** |
| 多语言 | OmniVoice | 4.76 | S-MOS 4.72 |
| | **VoxCPM2** | **4.78** | S-MOS 4.66 |
| 可控生成 | Qwen3-TTS-VD | 4.61 | I-MOS 4.41 |
| | **VoxCPM2** | 4.48 | **I-MOS 4.50** |

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | Seed-TTS-Eval 零样本克隆、CV3-Eval 多语言、MiniMax-MLS-Test | 公开 benchmark，标准化评测协议，与大量强基线公平对比，代码开源可复现 |
| **中** | InstructTTSEval 可控生成 | 公开 benchmark 但评测维度（APS/DSD/RP）定义可能与其他系统不完全一致；可控数据构造策略影响结果 |
| **中** | 主观评测（MOS）| ~100 样本、50 听众、双盲设计合理，但样本量中等；95% CI 约 +-0.02-0.04 |
| **中偏低** | 内部 30 语种评测 | 使用 Gemini 3.1 Flash Lite API 做 ASR（而非标准 Whisper），评测集未公开，跨系统 WER 不可比（Fish Audio S2 数据来源和 ASR 可能不同） |
| **低** | "2M+ 小时训练数据" | 仅声称，未给出数据清单、语种分布详细统计、数据质量筛选标准 |

## 核心 Claim 审查

1. **Paper Claim**："achieves SOTA or competitive performance on multiple public benchmarks"
   **My Assessment**：在 Seed-TTS-Eval 上 SIM 指标确实在开源模型中表现优异（仅次于 LongCat-Audio-DiT），但 WER 弱于 Qwen3-TTS 和 Fish Audio S2。应描述为"在 speaker similarity 维度上开源领先，intelligibility 维度有竞争力但非最优"。

2. **Paper Claim**："achieves an average WER of 1.68% on 30-language internal evaluation"
   **My Assessment**：该数字仅在内部评测集（未公开）上用 Gemini Flash Lite API 评测得到。评测集和 ASR 方法的选择可能显著影响 WER 值。在标准公开集上（如 MiniMax-MLS-Test），部分低资源语种（阿拉伯语 13.05%、罗马尼亚语 21.58%、印地语 19.70%）表现明显不佳。

3. **Paper Claim**："asymmetric AudioVAE V2 ...implicit super-resolution"
   **My Assessment**：AudioVAE V2 的 16kHz->48kHz 非对称设计从代码证实（encoder rates=[2,5,8,8] @16kHz, decoder rates=[8,6,5,2,2,2] @48kHz），Table 10 显示在 48kHz MelD 上优于 v1 但弱于 v1.5（后者用 44kHz 输入/输出）。"隐式超分辨率"的说法成立但有限定——在 16kHz 频带内重建质量（STOI/PESQ）略低于 v1。

4. **Paper Claim**："unified sequence organization enables all modes in single parameter set"
   **My Assessment**：代码证实所有模式共享参数，通过不同的序列构造（`_generate` 方法中四个分支）实现。设计合理且工程简洁。但可控数据在总训练数据中的占比较小（"数万小时"vs "2M+ 小时"），可控能力可能受限于数据规模而非架构。

## 批判性思考

### 优点

1. **架构原创性强**：在连续 latent 框架内通过可微 FSQ 实现语义-声学自然分层，不依赖外部 codec，是 tokenizer-free TTS 中最完整的系统
2. **多语种覆盖真实有效**：30 语种 + 9 方言，在高棉/老挝/缅甸等极低资源语种上仍可用（对比 Fish Audio S2 完全失败）
3. **统一序列组织设计简洁**：用 building block 排列组合替代多分支架构或多个 adapter，工程实现清晰
4. **完整开源**：Apache 2.0，代码/权重/fine-tuning 工具齐全，Nano-vLLM + vLLM-Omni 支持生产部署
5. **SIM 指标突出**：在 MiniMax-MLS-Test 24 语种中 22 个排名第一，说明 VoxCPM2 的说话人建模能力很强

### 局限性

1. **WER 非最优**：在 Seed-TTS-Eval 上 EN WER 1.84 弱于 Qwen3-TTS (1.23)、Fish Audio S2 (0.99)、ZipVoice (1.64) 等，intelligibility 不是此模型最强项
2. **低资源语种不均衡**：MiniMax-MLS-Test 上阿拉伯语 (13.05%)、罗马尼亚语 (21.58%)、印地语 (19.70%) 差距明显，"30 语种"宣称需加限定
3. **可控能力受限**：ZH-DSD (71.5) 和 ZH-RP (60.8) 弱于 Gemini-TTS-Pro 和 Qwen3-TTS-VD；可控数据占比小可能是瓶颈
4. **AudioVAE V2 的 16kHz 瓶颈**：encoder 固定 16kHz 意味着高于 8kHz 的声学细节完全依赖 decoder 的"超分辨率"能力，与 VoxCPM1.5（44kHz->44kHz）相比在 16kHz 频带内 STOI 和 PESQ 略低
5. **训练数据不透明**：2M+ 小时数据未给出详细清单、各语种分布、质量筛选标准。可控数据标注使用的 Step-Audio R1 和 Gemini 2.5 Pro 引入了模型间依赖

### 潜在改进方向

1. 引入 encoder 端 24kHz 或 32kHz 输入模式，减少超分辨率负担
2. 增加低资源语种可控数据比例，改善 DSD/RP 指标
3. 探索 RLHF/DPO 后训练对可控精度和韵律自然度的提升
4. LocDiT 推理步数优化（当前 10 步，可探索 4-6 步蒸馏）

### 可复现性评估

- [x] 代码开源（Apache 2.0）
- [x] 预训练模型（权重公开）
- [x] fine-tuning 工具（LoRA + 全参数 fine-tune 配置齐全）
- [ ] 训练细节完整（三阶段策略描述定性，缺具体 batch size/LR/epoch/计算资源）
- [ ] 数据集可获取（2M+ 小时数据未公开，仅描述来源类型）

---

# 三、知识系统层

## 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §连续 latent + diffusion-autoregressive 混合路线（路线 4/5 交汇）
- **核心问题**：[[TTS-核心挑战]] §挑战 1 (零样本克隆) + §挑战 2 (可控) + §挑战 1 (多语言克隆)
- **表示层位置**：[[TTS-表示层地图]] §continuous-latent（自训练 AudioVAE latent，非 codec token）
- **相邻工作**：[[VoxCPM]] / [[DiTAR]] / [[CLEAR]] / [[FELLE]] / VibeVoice

## 后续重估

- **2026-06-09**：初读。VoxCPM2 是目前 tokenizer-free 连续 latent TTS 路线中规模最大、语种最广、功能最全的开源系统。SIM 指标突出但 WER 并非最优。与离散 codec LM 路线的竞争仍在进行中——Qwen3-TTS/Fish Audio S2 在 intelligibility 上更强，但 VoxCPM2 在 speaker similarity 和极低资源语种上有独特优势。非对称 AudioVAE 的 16kHz 编码瓶颈是一个需要持续关注的限制。Apache 2.0 全量开源使其成为社区研究和工程复用的重要基础设施。

---

## 关联笔记

### 基于
- [[VoxCPM]]: VoxCPM2 的直接前身，建立了 TSLM-FSQ-RALM-LocDiT 四组件分层架构
- [[DiTAR]]: 引入 LocDiT（Local Diffusion Transformer），被 VoxCPM 系列复用

### 对比
- [[Qwen3-TTS]]: LLM-native 离散 codec 路线代表，WER 更优但 SIM 不如 VoxCPM2
- [[Fish-Speech]]: 4B 参数自回归 TTS，Seed-TTS-Eval WER 最低（0.99）
- [[LongCat-Audio-DiT]]: 3.5B 连续 latent DiT，Seed-TTS-Eval SIM 最高（78.6/81.8）
- [[CosyVoice3]]: 阿里最新 codec LM TTS，1.5B 闭源版在中文指标上很强
- [[IndexTTS2]]: 1.5B codec LM，在中文 CER 上竞争力强（1.03）

### 方法相关
- [[Conditional Flow Matching]]: LocDiT 内部使用的生成范式
- [[FSQ]]: 分层语义-声学分离的关键组件
- [[Classifier-Free Guidance]]: 推理时增强质量的方法
- [[MiniCPM]]: TSLM 的骨架 LLM

### 硬件/数据相关
- 训练数据：2M+ 小时多语言语音（未公开详细清单）
- 推理硬件：单 RTX 4090 (24GB) 即可运行，RTF 0.13 (Nano-vLLM)

---

## 速查卡片

> [!summary] VoxCPM2 Technical Report
> - **核心**: 2B 参数 tokenizer-free 分层连续 latent TTS 基础模型，30 语种 + 9 方言，48 kHz 输出
> - **方法**: TSLM(MiniCPM-4-1B) → FSQ(512d/9级) → RALM(NoPE) → LocDiT(CFM) → AudioVAE V2(16k→48k)，统一序列组织支持四种生成模式
> - **结果**: Seed-TTS-Eval SIM 75.3/79.5 (EN/ZH)，MiniMax-MLS-Test 22/24 语种 SIM 第一，InstructTTSEval EN 最优 (84.2/83.2/71.4)；30 语种平均 WER 1.68%（内部评测，限定条件较多）
> - **代码**: https://github.com/OpenBMB/VoxCPM (Apache 2.0)

---

*笔记创建时间: 2026-06-09*
