---
title: "CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training"
method_name: "CosyVoice3"
authors: [Zhihao Du, Changfeng Gao, Yuxuan Wang, Fan Yu, Tianyu Zhao, Hao Wang, Xiang Lv, Hui Wang, Chongjia Ni, Xian Shi, Keyu An, Guanrou Yang, Yabin Li, Yanni Chen, Zhifu Gao, Qian Chen, Yue Gu, Mengzhe Chen, Yafeng Chen, Shiliang Zhang, Wen Wang, Jieping Ye]
year: 2025
venue: arXiv (Tech Report)
arxiv_id: "2505.17589"
tags: [tts, zero-shot-tts, codec-lm-tts, flow-matching, multilingual, post-training, industrial-tech-report]
zotero_collection: 1-TTS与语音合成
note_tier: heavy

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: false
post_training_type: custom
streaming: partial

# === 知识地图联动 ===
domain: TTS
subdomain: codec-lm-tts
routes: [codec-lm-tts, controllable-tts, instruction-tts, streaming-tts, voice-cloning]
problems: [zero-shot-cloning, prosody-control, multilinguality, data-scale, codec-design, instruction-following, latency, evaluation]
representations: [semantic-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-表示层地图]]"
  - "[[TTS-代表模型谱系]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-评测体系]]"
  - "[[TTS-趋势判断]]"
related_surveys:
  - "[[ControllableTTS-Survey]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-06-01

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2505.17589/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2505.17589/paper.html"
figures_dir: "_resources/2505.17589/figures/"
github_local: "~/DailyPaper/.cache/papers/2505.17589/github/FunAudioLLM_CosyVoice/"
cached_at: 2026-05-26

# === 通用元数据 ===
image_source: local
arxiv_html: https://arxiv.org/html/2505.17589v2
created: 2026-05-25
---

# 论文笔记：CosyVoice 3

> **笔记分级**：heavy（工业系统 tech report，百万小时数据 + 多子系统）。
>
> **结构**：一、阅读层（核验后口径）/ 二、研究审计层（核验来源 + 完整表格公式 + 超参 + 可信度 + 批判）/ 三、知识系统层。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Speech Team, Tongyi Lab, Alibaba Group |
| 日期 | 2025-05 |
| 系列 | [[CosyVoice]] (2024-07) → [[CosyVoice 2]] (2024-12) → **CosyVoice 3** (2025-05) |
| 项目主页 | [funaudiollm.github.io/cosyvoice3](https://funaudiollm.github.io/cosyvoice3) |
| 对比基线 | [[MaskGCT]] / [[F5-TTS]] / [[Seed-TTS]] / [[CosyVoice 2]] / [[Spark-TTS]] / [[Qwen2.5-Omni]] / [[FireRedTTS]] |
| 开源 | ✅ 代码 + 部分 ckpt + CV3-Eval；❌ DiffRO 训练代码 / 1M h 数据 / tokenizer 训练 |
| 链接 | [arXiv](https://arxiv.org/abs/2505.17589) / [Code](https://github.com/FunAudioLLM/CosyVoice) |

## 一句话总结

> CosyVoice 系列第三代：监督多任务 tokenizer(MinMo 基座) + DiffRO 后训练 + 数据从 17 万 h 扩至 100 万 h + LM 从 0.5B 扩至 1.5B，在 SEED-TTS-Eval 上 test-zh CER 0.71%(超 Human 的 1.26%)、test-en WER 1.45%，覆盖 9 语种 + 19 方言。

---

# 一、阅读层（主文）

## 核心贡献

1. **监督多任务 Speech Tokenizer**：用 [[MinMo]]（1.4M h 预训练的多模态 LLM）替代 v2 的 SenseVoice-Large 作 FSQ 基座，5 任务监督训练使 token 同时携带语义+副语言信息。
2. **DiffRO 后训练**：在 token 层直接计算 reward（绕过 CFM/vocoder），用 [[Gumbel-Softmax]] 使离散采样可微，CER/WER 降 12-35%。
3. **百万小时 scaling**：训练数据从 170K h 扩至 1M h（9 语+19 方言），LM 0.5B→1.5B，CFM 100M→DiT 300M。
4. **CV3-Eval benchmark**：发布覆盖 9 语种×500 样本 + 跨语言 + 情感 + 方言 + 困难子集的公开测评集。

## 问题背景

### 要解决的问题
[[CosyVoice 2]] 在中英文广播场景下表现良好，但在 in-the-wild 场景仍有四个明显局限：语言仅覆盖中英文、领域/风格单一、数据和模型规模未充分探索、缺少有效后训练策略。

### 本文的动机
从四个维度同时推进：更强的 tokenizer 基座（MinMo）、token 层可微的后训练（DiffRO）、数据规模 6× 扩展、模型规模 3× 扩展。

## 方法详解

### 领域定位

CosyVoice 3 属于 **[[codec-lm-tts|codec-LM]] + [[Conditional Flow Matching|CFM]]** 两阶段零样本 TTS 路线，与 [[VALL-E]]（AR codec-LM）、[[Seed-TTS]]（AR 闭源工业标杆）同类。核心差异不在架构范式（v2 已定型），而在三个工程层面的推进：tokenizer 基座升级、后训练策略、数据+模型 scaling。

### 端到端数据流

CosyVoice 3 推理时的完整流水线：

**输入文本** → Qwen2 BPE Tokenizer → text tokens
→ **Text-to-Speech LM**（[[Qwen2]] warm-start, 0.5B/1.5B）自回归生成 25 Hz speech tokens（vocab 6561）
→ **Chunk-aware Causal CFM**（[[DiT]] 300M backbone）将 speech tokens 转为 50 Hz Mel spectrogram（80 dim, 24 kHz）
→ **Causal HiFi-GAN Vocoder**（含 NSF F0 predictor）合成 24 kHz 波形

同时，**参考语音**经 [[S3Tokenizer|MinMo-based FSQ Tokenizer]] 提取 speaker/style embedding 注入 CFM。

![[_resources/2505.17589/figures/fig2.png]]

> **Figure 2**：CosyVoice 3 系统总览与训练四阶段。上半部分是推理流水线：文本经 LM 生成 speech token，经 CFM 转 Mel，经 vocoder 合成波形；参考语音经 tokenizer 提取 speaker 信息注入 CFM。下半部分是训练四阶段（详见"训练流程"一节）。注意与 v2 的关键差异：移除了独立 text encoder 和 length regularization module，CFM 升级为 DiT backbone。

### 监督多任务 Speech Tokenizer：为什么换基座

**v2 的问题**：v2 用 SenseVoice-Large（纯 ASR 模型）作 FSQ 基座，提取的 token 主要携带语义信息，副语言信息（情感、语种、说话人特征）较弱。

**v3 的思路**：换成 [[MinMo]]——一个在 1.4M h 语音上预训练的多模态 LLM，本身已在对话、多语种 ASR、情感识别等任务上达 SOTA。其 intermediate representations 天然携带丰富副语言信息。

**怎么做**：在 MinMo 的 Voice Encoder₁（12 层 Transformer + RoPE）中间插入 [[Finite Scalar Quantization|FSQ]] 模块——投影到 D=8 维低秩空间，每维量化到 [-1, 1]，得到码本大小 $Q=(2K+1)^D=3^8=6561$ 的单 codebook。量化后的表征继续通过 Voice Encoder₂ + MinMo LLM 做 5 任务监督训练（ASR 365K h / LID 85K h / SER 48K h / AED 21K h / SA 11K h）。

$$
\bar{H} = \mathrm{ROUND}(\mathrm{Proj}_{down}(H)), \quad \hat{H} = \mathrm{Proj}_{up}(\bar{H})
$$

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j}\,(2K+1)^j
$$

**为什么 work**：多任务监督迫使 FSQ 保留语义+情感信息（因为下游要做 SER），同时丢弃不需要的声学细节（因为码本只有 6561，信息瓶颈强制取舍）。Table 12 验证：在 3000 h 数据上，监督式 tokenizer 在 CER 和 SS 两个维度同时优于自监督的 W2v-BERT 2.0 和声学的 SoundStream。一个反直觉的发现：FSQ 的信息瓶颈对特定任务有正则化效果——FSQ-MinMo 在 Fleurs CN 上的 WER (3.35) 甚至低于未量化 MinMo (6.71)。

### DiffRO：在 token 层直接做 RL

**现有 RL 的问题**：传统 TTS RL（如 F5R-TTS 用 GRPO）需要把 speech token 经 CFM+vocoder 还原为音频后才能算 reward——计算开销大，且生成的音频高度相似导致正负样本难区分。

**DiffRO 的核心 insight**：绕过 CFM/vocoder，直接在 token 空间算 reward。

**怎么做**（四步）：
1. 训练一个 **Token2Text 模型**（类似反向 ASR）：输入 speech token 序列，输出文本后验概率
2. **Reward** = Token2Text 给出的 log 概率——衡量"这组 token 能不能被清楚识别回文本"
3. **Gumbel-Softmax** 对 LM 输出的 token logits 采样——使离散选择可微，梯度直接从 reward 回传到 LM
4. **Token-level KL** 约束——在每个时间步的 logits 上算 KL（而非 sequence-level posterior），粒度更细更稳定

$$
\pi_\theta^* = \max_{\pi_\theta} \mathbb{E}[R(Y)] - \beta\, D_{\mathrm{KL}}[\pi_\theta \,\|\, \pi_{\mathrm{ref}}]
$$

**具体例子**：假设 LM 在时间步 $t$ 输出 6561 维 logits，Gumbel-Softmax 采出一个近似 one-hot 向量 $\tilde{\mu}_t$（可微），送入 Token2Text 模型计算"这个 token 选择对最终文本识别的贡献"。如果选 token #3721 让下游 ASR 识别正确的概率从 0.7 升到 0.9，reward 梯度就会增强选 #3721 的倾向。同时 KL 项防止 LM 偏离太远。

**为什么 work**：因果链是——token 层 reward 直接反馈 → LM 知道每个 token 选择对可懂度的影响 → 内容一致性提升 → CER/WER 降低 12-35%。在低资源语言效果最大（韩语 WER 相对降 68.7%），因为这些语言的 pretraining 数据不足，后训练修正空间大。

**⚠️ 复现警示**：DiffRO 的公式完整公开（§2.2 Eq.3-7），但开源 repo 中 **Token2Text / Gumbel-Softmax / token-level KL 均未找到实现代码**。仅有 DPO 的实现。复现 DiffRO 收益需自行实现。

**已知副作用**：DiffRO 优化 WER 时会轻微降低 speaker similarity（0.780→0.774），因为 reward 只看内容不看音色——属于 reward hacking，作者承认未解决。

### 训练流程

四阶段顺序训练：

1. **Large-scale Pretraining**：用全部 1M h 数据，LM 从 Qwen2 checkpoint warm-start，CFM 从头训练。产出：基础零样本 TTS 能力。
2. **DiffRO Post-training**：在筛选数据上用 DiffRO 优化 LM 的 token 选择。产出：内容一致性提升。
3. **Continual Pretraining**：在情感/指令/多语言数据上继续预训练。产出：风格迁移能力。
4. **Speaker Fine-tune**：在多说话人数据上微调，随机 mask speaker/style prompt 防止灾难性遗忘。产出：说话人适应。

### 推理流程

1. **时长估计**：无显式 duration predictor——LM 自回归生成直到 EOS，由 token/mel ratio=2 决定 mel 帧数
2. **LM 解码**：`ras_sampling` top_p=0.8, top_k=25, win_size=10, tau_r=0.1
3. **CFM 解码**：euler solver + cosine scheduler，推理 CFG rate=0.7
4. **流式**：CausalCFM chunk=25 token，支持 chunk-aware 流式推理。**但论文未报告任何首包延迟 / RTF 数字**

## 关键结果

**核心证据**：Table 4（SEED-TTS-Eval），全文最强证据——10 个 baseline 在同一公开 benchmark 上公平对比。

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ |
|---|---|---|---|---|---|
| Human | 1.26 | 0.755 | 2.14 | 0.734 | – |
| [[F5-TTS]] | 1.56 | 0.741 | 1.83 | 0.647 | 8.67 |
| [[Seed-TTS]] | 1.12 | **0.796** | 2.25 | **0.762** | 7.59 |
| [[CosyVoice 2]] | 1.45 | 0.748 | 2.57 | 0.652 | 6.83 |
| **CV3-0.5B + DiffRO** | **0.75** | 0.774 | 1.76 | 0.695 | **5.09** |
| **CV3-1.5B + DiffRO** | **0.71** | 0.775 | **1.45** | 0.695 | 5.66 |

**结论**（同 benchmark 对比，可信度高）：
- v2→v3：test-zh CER **-51%**（1.45→0.71），test-en WER **-44%**（2.57→1.45）
- CV3 是**唯一超 Human**的系统（test-zh CER 0.71 < 1.26）
- DiffRO 贡献 12-35% 相对提升
- **SS 上 [[Seed-TTS]] 仍领先**（0.796 vs 0.775），作者归因于训练数据规模差距
- ⚠️ DiffRO 轻微降低 SS（reward hacking 未解决）
- ⚠️ 1.5B 在 test-hard 上反而不如 0.5B（5.66 vs 5.09），归因于困难样本训练数据不足

**多语言覆盖**：CV3 是唯一支持全部 9 语种（zh/en/ja/ko/de/es/fr/it/ru）的系统。DiffRO 在所有语种正向提升，韩语提升最大（-68.7%）。完整多语言表见附录。

## 可复用的设计模式

1. **Token-level RL（绕过 renderer）**：在离散 token 空间用 Gumbel-Softmax + token-level KL 做可微 reward 优化。适用于任何 "LM → discrete token → downstream renderer" 的 pipeline（不限 TTS）。来自本文 DiffRO。
2. **监督式 tokenizer 设计模式**：在大型预训练 speech model 中间层插入量化瓶颈，通过多任务监督选择性保留目标信息（语义+情感），丢弃不需要的信息（细粒度声学）。适用于需要可控信息分离的 codec 设计。来自本文 FSQ-MinMo。
3. **Pronunciation Inpainting**：用 mixed text+phoneme 输入解决多音字/罕见词——只替换单音字为音素（RepMono），保留多音字原文避免 G2P 误差。适用于任何 BPE-based TTS 的可控发音。来自本文 §2.3。
4. **多语种数据管线 6 步 cookbook**：VAD+说话人分离 → MossFormer2 降噪 → 3 路 ASR 交叉验证（保留 pairwise WER<15%）→ MFA 停顿标点 → 峰值归一 → 长度比过滤。适用于百万级 in-the-wild 语音数据构建。来自本文 §3。
5. **跨语言能力迁移**：通过 continual pretraining + 多语种辅助数据，将单语说话人变为多语说话人（polyglot training），不需要每个语种都有对应说话人数据。来自本文 §2.6。

---

# 二、研究/审计层（附录）

## 📋 核验结论（技术元数据）

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start from Qwen2 checkpoint via `Qwen2ForCausalLM.from_pretrained()`；hidden_size=896（与 Qwen2-0.5B 一致）| [已 verify GitHub: cosyvoice/llm/llm.py:226-240, conf:30-31] |
| 训练 loss | LM: speech-token-only label-smoothed CE，text/instruct 位置 mask 不计 loss，无文本 loss 无 KL；CFM: mask-only flow matching loss | [已 verify GitHub: llm.py:687-693, conf:45] |
| Tokenizer 架构 | 监督多任务 FSQ，基于 MinMo voice encoder；FSQ 插入 Voice Encoder₁ 之后；推理时为 ONNX 黑盒，内部超参不可 verify | [已 verify §2.1 + GitHub: onnx.py] |
| 多任务 | tokenizer 预训练阶段 true（5 任务 530K h）；LM 训练阶段 false（单任务 CE）| [已 verify §2.1 Tab.3 + GitHub: llm.py:351-405] |
| 训练数据 | TTS 主训练 1M h 9 语+19 方言；tokenizer 530K h 5 任务；SFT 5000 h 100+风格 | [已 verify §4.2 + §2.1 Tab.3 + §2.5 Tab.1] |
| 后训练 | 论文：DiffRO（Token2Text + Gumbel-Softmax + token-level KL）；开源 repo：仅 DPO 实现，DiffRO 代码未公开 | [已 verify §2.2 Eq.3-7 + GitHub: losses.py:24-57, llm.py:407-456] |
| Codec 细节 | FSQ 单 codebook 6561=3⁸（K=1,D=8）；25 Hz；24 kHz sample rate | [已 verify §2.1 Eq.1-2 + conf:26,13,8] |

## 完整公式

### 公式1: [[Finite Scalar Quantization|FSQ 量化]]（Eq.1）

$$
\bar{H} = \mathrm{ROUND}(\mathrm{Proj}_{down}(H)), \quad \hat{H} = \mathrm{Proj}_{up}(\bar{H})
$$

**含义**：将中间表征投影到 D 维低秩空间并四舍五入量化，再投影回原维度。训练用 STE 近似梯度。

### 公式2: Speech Token 索引（Eq.2）

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j}\,(2K+1)^j
$$

**含义**：D 维量化向量编码为 $(2K+1)$ 进制标量索引。$Q = 3^8 = 6561$。

### 公式3: [[Gumbel-Softmax]] 采样（Eq.3）

$$
\tilde{\mu}_t = \mathrm{GumbelSoftmax}\, P_{\pi_\theta}(\mu_t \mid \mu_{1:t-1}; Y)
$$

**含义**：使 LM 的离散 token 选择可微。

### 公式4: ASR Reward（Eq.4）

$$
R_{\mathrm{ASR}}(Y) = \log P_{\mathrm{ASR}}(\tilde{Y}_n = Y_n \mid Y_{1:n-1}; \tilde{\mu}_{1:T})
$$

**含义**：Token2Text 模型给出的文本后验概率作为 reward。

### 公式5: DiffRO 优化目标（Eq.5）

$$
\pi_\theta^* = \max_{\pi_\theta} \mathbb{E}[R(Y)] - \beta\, D_{\mathrm{KL}}[\pi_\theta(\mu \mid Y) \,\|\, \pi_{\mathrm{ref}}(\mu \mid Y)]
$$

**含义**：最大化 reward 同时约束与参考策略的 KL 散度。

### 公式6: Token-level KL（Eq.6）

$$
D_{\mathrm{KL}} = \sum_{t=1}^{T} \sum_{k=0}^{Q} P_{\pi_\theta}(\mu_t=k) \log \frac{P_{\pi_\theta}(\mu_t=k)}{P_{\pi_{\mathrm{ref}}}(\mu_t=k)}
$$

**含义**：在每个时间步 logits 上计算 KL，粒度比 sequence-level 更细。

### 公式7: Multi-Task Reward（Eq.7）

$$
R_{\mathrm{MTR}}(Y, \{A_i\}_{i=1}^{K}) = \sum_i \log P_{\mathrm{task}_i}(\tilde{A_i} = A_i \mid \tilde{\mu})
$$

**含义**：除 ASR 外可加入 SER/MOS/AED 等多任务 reward。DiffRO-EMO 是 SER reward 的实例。

### 公式8: 音量归一化（Eq.8）

$$
\mathrm{norm\_wav} = \frac{\mathrm{raw\_wav}}{\max(\mathrm{raw\_wav})} \times 0.6
$$

## 完整实验表格

### Table 4 完整版: SEED-TTS-Eval

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ | test-hard SS↑ |
|---|---|---|---|---|---|---|
| Human | 1.26 | 0.755 | 2.14 | 0.734 | – | – |
| Vocoder Resyn | 1.27 | 0.720 | 2.17 | 0.700 | – | – |
| MaskGCT | 2.27 | 0.774 | 2.62 | 0.714 | 10.27 | 0.748 |
| E2 TTS (32 NFE) | 1.97 | 0.730 | 2.19 | 0.710 | – | – |
| F5-TTS (32 NFE) | 1.56 | 0.741 | 1.83 | 0.647 | 8.67 | 0.713 |
| F5R-TTS | 1.37 | 0.754 | – | – | 8.79 | 0.718 |
| Seed-TTS | 1.12 | **0.796** | 2.25 | **0.762** | 7.59 | **0.776** |
| FireRedTTS | 1.51 | 0.635 | 3.82 | 0.460 | 17.45 | 0.621 |
| Qwen2.5-Omni-7B | 1.70 | 0.752 | 2.72 | 0.632 | 7.97 | 0.747 |
| Qwen2.5-Omni-7B_RL | 1.42 | 0.754 | 2.33 | 0.641 | 6.54 | 0.752 |
| CosyVoice | 3.63 | 0.723 | 4.29 | 0.609 | 11.75 | 0.709 |
| CosyVoice 2 | 1.45 | 0.748 | 2.57 | 0.652 | 6.83 | 0.724 |
| Spark-TTS | 1.20 | 0.672 | 1.98 | 0.584 | – | – |
| CV3-0.5B | 1.16 | 0.780 | 2.02 | 0.718 | 6.08 | 0.758 |
| CV3-0.5B_RL | **0.75** | 0.774 | 1.76 | 0.695 | **5.09** | 0.750 |
| CV3-1.5B | 1.12 | 0.781 | 2.21 | 0.720 | 5.83 | 0.758 |
| CV3-1.5B_RL | **0.71** | 0.775 | **1.45** | 0.695 | 5.66 | 0.750 |

### Table 5: CV3-Eval 多语言（WER↓）

| Model | zh | en | ja | ko | de | es | fr | it | ru |
|---|---|---|---|---|---|---|---|---|---|
| CV2 | 4.08 | 6.32 | 9.13 | 19.7 | – | – | – | – | – |
| CV3-0.5B + DiffRO | **2.89** | **3.68** | **5.15** | **4.02** | **4.51** | **2.99** | **8.56** | **2.94** | **3.79** |
| CV3-1.5B + DiffRO | 3.01 | 3.71 | 5.27 | 4.01 | 3.93 | 3.26 | 8.09 | 2.72 | 4.11 |

### Table 9: 情感克隆（准确率）

| Model | TR-happy | TR-sad | TR-angry | TF-happy | TF-sad | TF-angry |
|---|---|---|---|---|---|---|
| F5-TTS | 0.92 | 0.52 | 0.72 | 0.80 | 0.28 | 0.64 |
| CosyVoice 2 | 0.84 | 0.72 | 0.58 | 0.56 | 0.44 | 0.38 |
| CV3-0.5B | 0.92 | 0.70 | 0.72 | 0.64 | 0.42 | 0.58 |
| **DiffRO-EMO** | **0.98** | 0.68 | **0.84** | **0.98** | 0.50 | **0.68** |

### Table 10-12: Tokenizer Ablation（完整版见原笔记 v1）

**核心工程结论**：
- 声学 token (SoundStream) 缺乏语义 → 内容一致性差
- HuBERT 语义 token 有 SS 但中文 CER 极高（语言特异性）
- 监督多任务 tokenizer 在 CER + SS 两维同时领先
- FSQ 信息瓶颈对特定任务有正则化效果（FSQ-MinMo Fleurs CN WER 3.35 < 未量化 MinMo 6.71）
- FSQ 量化后 LID 无损、情感识别反而提升（62.4→68.4），但性别/年龄/vocal sound 退化

### Table 13: Pronunciation Inpainting

RepMono + MixPhn：中英文均 100% 纠正率。

### Table 14: 指令式语音生成

Style SIM 相对 v2 提升约 11%（72.99→81.06）。Expresso WER 偏高，归因于 ASR 偏好标准发音。

## 训练超参（可复现参考）

### LM

| 参数 | 值 |
|---|---|
| Base class | `CosyVoice3LM(Qwen2LM)` → `Qwen2ForCausalLM.from_pretrained()` |
| dim | 896 |
| vocab | 6561 + 200 = 6761 |
| Loss | LabelSmoothingLoss（lsm=0, length-normalized）|
| Sampling | ras_sampling top_p=0.8 top_k=25 win_size=10 tau_r=0.1 |
| LR | 1e-5, constantlr, warmup 2500 steps |
| Max epoch | 200 |
| Grad clip / accum | 5 / 2 |

### CFM（DiT）

| 参数 | 值 |
|---|---|
| dim/depth/heads/dim_head | 1024/22/16/64 |
| ff_mult | 2 |
| Solver | euler, cosine schedule |
| CFG | train 0.2 / inference 0.7 |
| sigma_min | 1e-6 |
| only_mask_loss | True |

### Vocoder（Causal HiFi-GAN + NSF）

| 参数 | 值 |
|---|---|
| Base channels | 512, nb_harmonics=8 |
| Upsample rates | [8, 5, 3] |
| Discriminator | MPD + MRD |
| F0 predictor | CausalConvRNNF0Predictor |

## 多语种数据管线（6 步，含具体工具）

| Step | 操作 | 工具 |
|---|---|---|
| 1 | 说话人分离 + VAD + 音频事件检测 → ≤30s 片段 | 内部模块 |
| 2 | 降噪 + 首尾能量截断 | MossFormer2 |
| 3 | LID → 3 路 ASR 转录 → 交叉验证（avg pairwise WER<15%）| Faster-Whisper V3 / NeMo Canary-1B / seamlessM4T-V2-large |
| 4 | 基于词间停顿调整标点（≥300ms 加逗号 / ≤50ms 删标点）| MFA |
| 5 | 峰值归一化 ×0.6 | Eq.8 |
| 6 | speech_token_len / text_token_len 比过滤（丢弃 min 1% + max 5%）| – |

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | SEED-TTS-Eval 全部指标 (Tab.4) | 公开 benchmark + 10 baseline + 公开评测器 |
| **高** | Tokenizer ablation (Tab.10-12) | 替换 token 控制变量，公开 benchmark |
| **中** | CV3-Eval 多语/跨语/困难 | benchmark 已发布，但多数 baseline 不支持小语种，缺公平对比 |
| **中** | 情感克隆 (Tab.9) | emo2vec-large-plus 作分类器，开源但结果对分类器敏感 |
| **中-低** | DiffRO 提升数字 | 公式完整但实现代码未在 repo 公开，第三方需自行复刻 |
| **低** | 1M h 数据质量 | 数据闭源，管线只给原理无具体阈值 |
| **低** | 流式延迟 | 论文未报告任何 RTF / 首包延迟数字 |

## 核心 Claim 审查

1. **Paper Claim**：提出 novel speech tokenizer derived from large audio understanding LLM
   **My Assessment**：SenseVoice→MinMo 是基座升级而非范式创新。但 FSQ 信息瓶颈使情感识别反而提升（Tab.11 62.4→68.4）是有价值的发现。

2. **Paper Claim**：DiffRO applicable to other discrete-token-based speech synthesis models
   **My Assessment**：公式上通用，但开源 repo 未提供代码——"applicable"在工程上有 gap。

3. **Paper Claim**：Achieves SOTA on multiple benchmarks
   **My Assessment**：SEED-TTS-Eval CER/WER 确实 SOTA 且超 Human。但 SS 落后 Seed-TTS。CV3-Eval 多语种因 baseline 不支持小语种属 first-mover 优势而非横向碾压。

### 优点
1. 数据+模型双 scaling 工业实证（1M h + 0.5B→1.5B）
2. Tokenizer ablation 极扎实（5 种 tokenizer × 2 规模完整对比）
3. 数据管线 6 步 cookbook 每步点名工具，可复刻
4. DiffRO 思路精巧（绕过 renderer 做 RL）
5. 诚实暴露问题（reward hacking / 1.5B 不如 0.5B / 歌唱未支持）

### 局限性
1. DiffRO 代码不开源——最大复现障碍
2. Tokenizer 是 ONNX 黑盒（无训练代码 / 内部超参不可 verify）
3. 1M h 训练数据闭源
4. 音色不可通过文本指令控制
5. 不支持歌唱
6. 流式延迟未报告（1.5B 流式是否可用是问号）
7. RL reward hacking (SS↓) 未解决

### 可复现性评估
- [x] 代码开源（主模块 + conf）
- [x] 部分 checkpoint 开源
- [-] Tokenizer 仅 ONNX 推理
- [-] DiffRO 实现不开源
- [ ] 1M h 训练数据不开源
- [x] CV3-Eval 测评集公开
- [x] 架构描述+超参完整

### 与同期工业 tech report 对照

| 维度 | CosyVoice 3 | StepAudio 2.5 | Qwen3-TTS |
|---|---|---|---|
| 定位 | 独立 TTS 产品 | Unified ASR/TTS/Realtime | 独立 TTS 产品 |
| LM init | Qwen2 warm-start [已 verify GitHub] | 文本 MoE LLM warm-start | Qwen3 warm-start (paper claim) |
| Codec | FSQ 6561 / 25 Hz | 未披露 | 双 codebook / 多 sub_talker |
| 训练数据 | 1M h | 2.2T tokens | 未明示 |
| 后训练 | DiffRO (公式公开,代码未开源) | GRM-shaped RLHF | 各自方案 |
| 开源 | 代码+部分ckpt+benchmark | 全闭源 | 部分开源 |
| 工程透明度 | ★★★ | ★ | ★★ |

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]（独立 TTS 系统）
- **技术路线**：[[TTS-技术路线图]] §路线 2（Codec LM + CFM 两阶段）；LM 为 Qwen2 warm-start，v1→v2/v3 是 init 范式的关键跃迁
- **核心问题**：[[TTS-核心挑战]] §挑战1 零样本克隆 / §挑战4 数据规模 / §挑战5 Codec 设计 / §挑战3 流式延迟
- **表示层位置**：[[TTS-表示层地图]] §FSQ——监督多任务单 codebook 6561 / 25 Hz
- **相邻工作**：[[CosyVoice 2]]（直接前作）/ [[Seed-TTS]]（SS 仍领先的闭源标杆）/ [[StepAudio2.5]]（同期对照）/ [[Qwen3-TTS]]（阿里另一条线）

## 🔄 后续重估

- **2026-05-25**：初读，建立核心框架。
- **2026-05-26**：GitHub 三层 verify——确认 Qwen2 warm-start / DiffRO 代码未开源 / tokenizer ONNX 黑盒。从头重写为工业 tech report 格式。
- **2026-06-01**：按新标准重写——阅读层/审计层分离；frontmatter relocate 为枚举；方法段改叙事式（端到端数据流+why 因果链+DiffRO 具体例子）；新增"可复用的设计模式"段。内容基于原笔记 v1 所有已 verify 事实，无新增来源。

---

## 关联笔记

### 基于
- [[CosyVoice 2]]: 直接前作，沿用 LLM + chunk-aware CFM + HiFi-GAN 三段式
- [[CosyVoice]]: 系列初代，提出 supervised semantic token
- [[MinMo]]: speech tokenizer 基座（1.4M h 多任务 speech understanding LLM）
- [[Qwen2]]: LM warm-start 基座

### 对比
- [[StepAudio2.5]]: 同期工业 tech report（unified 路线）
- [[Qwen3-TTS]]: 阿里另一条产品线
- [[Seed-TTS]]: AR 闭源标杆（SS 仍领先）
- [[F5-TTS]]: NAR flow matching 代表
- [[MaskGCT]]: NAR masked generative codec
- [[Spark-TTS]]: 单 LLM + BiCodec 路线

### 方法相关
- [[FSQ]]: Finite Scalar Quantization
- [[DiffRO]]: 核心后训练方法
- [[Conditional Flow Matching]] / [[DiT]]: CFM decoder
- [[Gumbel-Softmax]]: 使离散采样可微
- [[ERes2Net]] / [[WavLM]]: SS 评测器

### 数据/工具相关
- [[Seed-TTS-eval]] / [[CV3-Eval]]: 评测集
- [[MossFormer2]]: 降噪
- [[Faster-Whisper]] / [[NeMo Canary]] / [[seamlessM4T]]: ASR 交叉验证
- [[MFA]]: 标点调整

---

## 速查卡片

> [!summary] CosyVoice 3 (Alibaba Tongyi, 2025-05)
> - **核心**: 监督多任务 FSQ tokenizer（MinMo 基座）+ DiffRO token-level RL + 1M h 9 语种 scaling
> - **架构**: Qwen2 warm-start LM → 25 Hz speech token (FSQ 6561) → DiT CausalCFM → CausalHiFiGAN
> - **结果**: SEED-TTS-Eval test-zh CER 0.71%（超 Human，v2 对比 -51%）/ test-en WER 1.45%（-44%）/ 9 语种覆盖 / DiffRO 12-35% 相对提升
> - **可复用**: token-level RL / 监督式 tokenizer 模式 / pronunciation inpainting / 6 步多语种数据管线
> - **⚠️ 复现**: DiffRO 训练代码未开源 / tokenizer ONNX 黑盒 / 1M h 数据闭源

---

*笔记创建: 2026-05-25 · 工业 tech report 重写: 2026-05-26 · 新标准重写(阅读层/审计层分离+可复用 idea): 2026-06-01*
