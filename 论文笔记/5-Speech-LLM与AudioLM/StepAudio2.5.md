---
title: "StepAudio 2.5 Technical Report"
method_name: "StepAudio 2.5"
authors: [StepFun-Audio Team]
year: 2026
venue: arXiv (Tech Report)
arxiv_id: "2605.23463"
tags: [speech-llm, unified-omni, asr, tts, realtime-dialogue, moe, multi-token-prediction, rlhf, industrial-tech-report]
zotero_collection: 5-Speech-LLM与AudioLM

# === 论文核心技术元数据（三层 verify）===
lm_init: "warm-start from a textual MoE LLM, then 2.2T-token continual multimodal pre-training [§3.2 paper.html L329-345]"
training_loss: "Stage-1 adaptor-only CE on 3B ASR tokens; Stage-2 unified next-token CE on 1.6T text+speech tokens with progressively annealed MoE auxiliary loss; Stage-3 cooldown CE on 600B high-quality tokens. ASR head adds weighted MTP CE (公式 1-2). TTS/Realtime add RLHF (GRM-shaped reward, PPO + KL) [§3.2, §4.1, §5.1, §6.1.3]"
tokenizer_arch: "audio-encoder → adaptor → MoE LLM decoder; unified sequence space mixing text tokens and (vocabulary-expanded) audio tokens; TTS branch completely removes encoder+adaptor, uses LLM-only NTP on audio tokens [§2.1, §5 paper.html L699-700]"
multitask: true "[§3.2] 主预训练 800B speech 含 ASR / TTS / S2T translation / 文本-语音 interleaved continuation / S2S conversation 五种任务格式"
training_data: "Pretrain 2.2T tokens = 800B text + 800B speech + 600B cooldown; ASR SFT 100K h short-form + 50K h long-form pseudo-label (ROVER 3-system, drop disagreement-rate ê>0.05) [§3.2 + §4.2]; TTS SFT data 部分来自 Step-Audio-EditX 合成（无具体小时数） [§5.2]; Realtime SFT data 来自 10K+ native personas + algorithmic fission → 百万级 persona matrix [§6.2]"
post_training: "TTS: GRM-shaped reward $r_{hf} = s(r_\\phi(x, y, y^*))$，pairwise scalar preference vs 高质量 reference [§5.1 Eq.1]. Realtime: PPO + KL 正则，GRM + interaction rubrics，multi-turn + single-turn 混合 [§6.1.3]"
codec_detail: "未明示。tokenizer 设计未披露 RVQ/FSQ 选型 / 码本大小 / 帧率 / 码率。仅说明 audio encoder 输出经 adaptor 映射到 LLM hidden space，并向 LLM 词表新增 audio tokens [§2.1 + §3.2 p3]。L2 verify 不可用（未开源）"

# === 知识地图联动 ===
domain: SpeechLM
subdomain: unified-omni
routes: [speech-llm-tts, llm-asr, full-duplex-dialogue, controllable-tts]
problems: [asr-accuracy, asr-throughput, instruction-following, prosody-control, dialogue-integration, persona-consistency, paralinguistic, latency]
representations: [audio-token, unified-token-space]
related_maps:
  - "[[SpeechLM-领域总览]]"
  - "[[TTS-SpeechLM-Dialogue关系]]"
  - "[[TTS-技术路线图]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-评测体系]]"
related_surveys:
  - "[[ControllableTTS-Survey]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-26

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2605.23463/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2605.23463/paper.html"
figures_dir: "_resources/2605.23463/figures/"
github_local: ""
cached_at: 2026-05-26

# === 通用元数据 ===
image_source: online
arxiv_html: "https://arxiv.org/html/2605.23463v1"
created: 2026-05-26
---

# 论文笔记：StepAudio 2.5 Technical Report

> **本笔记按工业 tech report 标准撰写**：重点抽取数据规模、训练 recipe、超参、serving 指标、可复现细节。工程价值 > 学术新颖性。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | StepFun（阶跃星辰） |
| 发布日期 | 2026-05-22 |
| 系列上下文 | Step-Audio → Step-Audio 2 → **StepAudio 2.5** → Step-Audio-EditX（合成数据生成器） |
| 项目主页 | 未提供 |
| 是否开源 | **未开源**（无 GitHub repo / 无 checkpoint） |
| 链接 | [arXiv](https://arxiv.org/abs/2605.23463) / [HTML](https://arxiv.org/html/2605.23463v1) |
| 对比基线 | ASR: [[VibeVoice-ASR]] / [[FunASR-Nano]] / Doubao-ASR-2603 / Qwen3-ASR-1.7B；TTS: MiniMax-2.8-HD / [[ElevenLabs-v3]] / Gemini-3.1-Flash-TTS；Realtime: GPT-realtime / Gemini Live / Doubao Realtime（隐式） |

---

## 一句话总结

> 用一个 MoE audio-language 基座 + RLHF 主导的后训练，把 ASR / TTS / 实时对话三条产品线压在同一 backbone 上，并在三个方向都达到或超过专用系统的水平。

---

## 核心工程交付（按工业价值排序）

| # | 交付 | 关键指标 | 出处 |
|---|---|---|---|
| 1 | **生产级 LLM-based ASR + MTP-5 加速** | RTF=0.0053 / 单 H800 / 单并发，比 Qwen3-ASR-1.7B 快 ~1.8× | Tab.2 |
| 2 | **三方向 SOTA**（同一 backbone） | ASR avg CER 2.97% (zh) / WER 3.68% (en)；TTS arena win-rate 67.6%；Realtime +10.0 主观 margin | Tab.1 / Fig.4 / Fig.5 |
| 3 | **Long-form ASR 数据管线**（3-system ROVER） | 50K h 伪标 long-form 训练集，drop ê>0.05 段 | §4.2 + Fig.3 |
| 4 | **Persona × Fission Realtime 数据基建** | 10K+ native personas → 百万级 persona matrix × 百万级真实场景对话 | §6.2 |
| 5 | **统一 RLHF 范式** | TTS / Realtime 都用 GRM-shaped reward，单一规范覆盖两条线 | §5.1 + §6.1.3 |

---

## 1. 总体设计（核心论点）

[已 verify §1] 论点："**Once text and audio share a well-shaped representational space, the differences among downstream tasks migrate away from architecture toward operational regimes: data, objectives, and decoding constraints.**"

工程含义：不为 ASR / TTS / Realtime 各造一套架构，而是 (1) 共享 backbone (2) 用数据组合和后训练目标做"方向化推理"。

三方向的部署需求差异（必须在同一 backbone 下满足）：

| 分支 | 核心目标 | 主要挑战 |
|---|---|---|
| ASR | 准确 + 高效 long-form 转录 | RTF + long-form context |
| TTS | 可控 + 富表达力 | global / inline 双层级控制 + 评测 |
| Realtime | 低延迟 + persona 一致性 + 副语言响应 | reward sparsity + 多轮一致性 |

---

## 2. 统一基座（Shared Backbone）

### 2.1 架构组成

[已 verify §2.1] 经典 audio-encoder → adaptor → LLM-decoder 三件套：

- **Audio Encoder**：冻结的声学编码器，输出紧凑声学嵌入
- **Adaptor**：轻量映射层，把声学嵌入投影到 LLM hidden space
- **MoE LLM Decoder**：从一个文本 MoE LLM 初始化；词表扩展加入新的 audio token；在统一序列空间中 text 与 audio token 共存
- **TTS 分支特例**：[已 verify §5 p1] 完全去掉 encoder + adaptor，只用 LLM backbone 做 next-token prediction 生成 audio token

### 2.2 三方向推理 (Directional Inference)

| 方向 | 输入 | 输出 | 输出空间特性 |
|---|---|---|---|
| ASR | audio embeddings | transcript tokens | 窄、离散、被声学信号强锚定 |
| TTS | text + control instructions | audio tokens / 中间表示 | 丰富，挑战在自然度与可控性 |
| Realtime | audio | latent reasoning trace → response | 在 turn 级延迟约束下耦合理解+生成 |

> 关键：foundation 不区分"理解"与"生成"，只需一个高质量多模态先验 + 路由不同输出空间的机制。三方向都是对同一多模态记忆的不同查询。

### 2.3 架构图

![Figure 1](https://arxiv.org/html/2605.23463v1/2605.23463v1/figures/stepaudio25-unified-foundation-architecture.png)

**说明**: 共享 audio-language stack 服务三个 deployment goal 不同的分支（ASR/TTS/Realtime）。

---

## 3. 数据基础与多阶段预训练

### 3.1 通用数据生产管线

[已 verify §3.1] 自动化 pipeline，**同时服务**理解 + TTS + 对话三类任务：

1. **SED + VAD** 过滤低质量非语音段
2. 相邻 VAD 段合并，重新切分为"语义相对完整 + 时长适中"的 base sample
3. **Audio-level annotation**：质量评分 / 合成语音检测 / 说话人数标注
4. **Dual-ASR 转录** + LID；用 WER / edit-distance / 语速做交叉验证
5. 基于转录做语义完整性评分 + 内容分类
6. 按 metadata（语种 / 时长 / 语义质量 / 音质）分级，预训练不同阶段抽不同质量

### 3.2 Progressive Foundation Training

[已 verify §3.2] **总训练量 2.2T tokens**。分四阶段执行：

| Stage | 数据规模 | 序列长 | 可训模块 | 关键设置 |
|---|---|---|---|---|
| **(A) Adaptor 对齐** | 3B tokens（纯 ASR） | – | **仅 adaptor**（encoder + LLM 冻结） | 继承 Step-Audio 2 |
| **(B) 多模态 Warmup** | 128B tokens | 16K | 全部 | adaptor / embedding / output 高 LR；MoE router 低 LR（保护文本模态） |
| **(C) 主预训练** | ~1.47T tokens（800B text + 800B speech 减去 warmup） | 16K | 全部 | LR 归一；**MoE auxiliary loss 系数和 router LR 渐退**，平衡 expert 利用率与 top-$k$ 路由概率 |
| **(D) Cooldown** | 600B 高质量 tokens | **扩 32K** | 全部 | 新增 **Audio Caption + Instruct TTS**；强调高质量多模态 + 长上下文 |

Speech 数据格式（Stage B/C）：ASR / TTS / S2T 翻译 / 文本-语音 interleaved continuation / S2S 对话。**语音不只是转录输入，而是作为通用 sequence modality 出现在多种 in/out 配置中**。

> 工程意义：32K 长上下文窗口是后续 ASR long-form / Realtime 多轮的能力基础——不是评测时临时扩，而是从预训练就准备好的。

---

## 4. ASR 专用化

### 4.1 架构：encoder-adaptor-decoder + MTP-5

[已 verify §4 paper.html L354] 在共享 decoder 之上 append **5 个并行 future-token branches**，单步前向生成 6-token proposal（主分支 + 5 个 MTP）。

![Figure 2](https://arxiv.org/html/2605.23463v1/2605.23463v1/figures/stepaudio25-asr-mtp-architecture.png)

**MTP block 结构**（每个分支）：
- 接收前一分支 hidden state + shifted token embedding
- 两输入分别 normalize → concat → 投影到 decoder hidden size → 一层 Transformer block
- 所有分支**共享** embedding layer 与 vocabulary output head

**推理验证机制**：proposal 只接受"经验证的前缀"——一旦未来 token 与正常自回归路径不一致，丢弃后续；MTP 严格只做加速、不引入错误。

### 4.2 训练 Recipe（关键超参）

[已 verify §4.1] 两阶段训练：

**(1) ASR SFT**（先把基座调成可靠 AR recognizer）

| 超参 | 值 |
|---|---|
| 数据 | 100K h short-form + 50K h long-form 伪标 |
| 序列长度 | 32K（packed） |
| 数据增强 | SpecAugment 时间+频率 masking |
| 冻结模块 | audio encoder |
| 可训模块 | adapter + LLM decoder |
| Steps | 10K |
| Peak LR | $2 \times 10^{-5}$ |
| Global batch | 32 |
| Warmup | 100 steps |
| LR schedule | cosine decay → $1 \times 10^{-6}$ |

**(2) MTP 训练**（两子阶段）

| Sub-stage | 可训模块 | Peak LR | Steps | 备注 |
|---|---|---|---|---|
| Frozen-branch alignment | **仅 5 个 MTP blocks** | $2 \times 10^{-4}$ | 10K | Transformer 层从**最后一层 decoder 初始化**（继承 linguistic prior）；branch-specific 投影新初始化；共享 embedding + LM head 全部冻结 |
| Joint calibration | adapter + LLM + MTP | $2 \times 10^{-5}$ | 10K | 让 backbone 状态与 MTP 分支对齐，把 MTP 训成 calibrated proposal |

两子阶段都继承 32K 序列长度 / batch=32 / 10K steps。

### 4.3 关键公式

**MTP 分支权重**（指数递减，反映串行依赖）：

$$
w_h = \frac{\alpha^{h-1}}{\sum_{j=1}^{H}\alpha^{j-1}}, \quad H = 5, \quad \alpha = 0.9
$$

**ASR + MTP 联合损失**（每个位置 $t$）：

$$
\mathcal{L}_t = \mathrm{CE}(p_t, x_{t+1}) + \sum_{h=1}^{H} w_h \, \mathrm{CE}(p_{t,h}, x_{t+1+h})
$$

- $p_t$ / $p_{t,h}$：主分支 / 第 $h$ MTP 分支的输出分布
- $x_{t+1+h}$：第 $h$ 步未来的真实 token

**Long-form 数据可靠性度量**（段级分歧率，决定是否丢弃）：

$$
\hat{e} = \frac{\#\,\mathrm{disagreed\ positions}}{\#\,\mathrm{text\ units}}
$$

策略：$\hat{e} > 0.05$ 的段直接丢弃。

### 4.4 Long-form ASR 数据管线（50K h）

[已 verify §4.2] 五步管线：

![Figure 3](https://arxiv.org/html/2605.23463v1/2605.23463v1/figures/stepaudio25-asr-data-pipeline.png)

1. **VAD 分段** ≤30s
2. **三 ASR 系统转录** 得到多个候选 hypothesis
3. **Surface-form normalization**（统一大小写 / 标点 / 格式）以聚焦真实识别误差
4. **ROVER 对齐 + token 级投票**：至少 2/3 系统支持的 token 才被接受；非共识位置标 disagreement
5. **段级分歧率过滤**：$\hat{e} > 0.05$ 丢弃；通过的相邻段拼接成 long-form 训练样本
6. **LLM-based 精修阶段**：恢复标点 / inverse text normalization / 跨段术语和实体一致性

> 工程意义：用便宜的"3 系统投票 + 阈值过滤 + LLM 精修"造出 50K h **session-level**伪标数据，绕开人工标注瓶颈。这是 long-form 32K 上下文能力的核心数据基础。

### 4.5 ASR 评测结果

**Serving 设置**：单 NVIDIA H800 / 单并发；Doubao-ASR-2603 通过官方 API；不原生支持 long-form 的 baseline（如 FunASR-Nano）用 VAD 切 ≤30s。

#### Table 1: 三 benchmark 分组 Error Rate (%)

| 类别 | Test set | VibeVoice-ASR | FunASR-Nano | Doubao-ASR-2603 | Qwen3-ASR-1.7B | **StepAudio 2.5** | w/o MTP |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **中文** | AISHELL-1 | 5.19 | 1.88 | 2.07 | 1.49 | **0.71** | 0.79 |
| | AISHELL-2 ios | 5.10 | 2.61 | 2.70 | 2.50 | **2.29** | 2.30 |
| | WenetSpeech testnet | 14.79 | 5.30 | **4.03** | 4.44 | 4.54 | 4.57 |
| | WenetSpeech testmeeting | 17.09 | 5.31 | 5.09 | **4.66** | 4.70 | 4.73 |
| | FLEURS zh | 8.77 | 3.19 | 2.83 | 2.74 | **2.63** | 2.63 |
| | **Avg (zh)** | 10.19 | 3.66 | 3.34 | 3.17 | **2.97** | 3.00 |
| **英文** | LibriSpeech clean | 2.30 | 1.80 | 2.94 | 1.69 | **1.38** | 1.40 |
| | LibriSpeech other | 5.79 | 4.43 | 5.98 | 3.57 | **3.16** | 3.14 |
| | Common Voice v11 | 20.03 | 11.05 | 14.06 | **7.50** | 7.57 | 7.62 |
| | FLEURS en | 5.20 | 4.96 | 6.74 | **3.23** | 3.55 | 3.74 |
| | VoxPopuli cleaned AA | **2.38** | 3.97 | 3.61 | 3.28 | 2.76 | 3.23 |
| | **Avg (en)** | 7.14 | 5.24 | 6.67 | 3.85 | **3.68** | 3.83 |
| **Long-form** | LibriSpeech clean long | 1.66 | 2.34 | 2.81 | 1.95 | **1.27** | 1.27 |
| | LibriSpeech other long | 3.48 | 4.89 | 5.59 | 3.81 | **2.90** | 2.81 |
| | WenetSpeech testnet long | 8.73 | 4.74 | **3.72** | 4.15 | 4.09 | 4.09 |
| | Earnings22 cleaned AA | **5.62** | 10.38 | 12.33 | 6.90 | 6.52 | 6.34 |
| | **Avg (long-form)** | 4.87 | 5.59 | 6.11 | 4.20 | **3.70** | 3.63 |

**关键观察**：
- 在 3 类 benchmark 的 average 上都建立新 SOTA（zh / en / long-form 平均分别 2.97 / 3.68 / 3.70）
- **MTP 对准确率的影响 <0.06 abs 点**（with vs without 列），验证了 strict-prefix verification 的设计正确性
- LibriSpeech long 上明显碾压其它系统，归因于原生 32K context（无 boundary errors）

#### Table 2: RTF（100 个 30s clip 平均）

| Model | VibeVoice-ASR | FunASR-Nano | Doubao-ASR-2603 | Qwen3-ASR-1.7B | **StepAudio 2.5 ASR** |
|---|---|---|---|---|---|
| RTF ↓ | 0.1039 | 0.0591 | 0.0640 | 0.0094 | **0.0053** |

**关键观察**：MTP-5 把"decoder 越大 token-by-token 延迟越线性增长"的规律打破——大多 step 一次性 emit 多个 verified token。**比 Qwen3-ASR-1.7B 快 ~1.77×**，更比 VibeVoice-ASR 快约 20× 。

#### Table 3: MTP-{3/5/7} 严格逐位置接受率（WenetSpeech meeting）

| Config | 1st | 2nd | 3rd | 4th | 5th | 6th | 7th | Avg Length |
|---|---|---|---|---|---|---|---|---|
| MTP-3 | 0.96 | 0.88 | 0.80 | – | – | – | – | 3.6 / 4 |
| **MTP-5** | 0.95 | 0.88 | 0.80 | 0.71 | 0.64 | – | – | **5.0 / 6** |
| MTP-7 | 0.96 | 0.88 | 0.80 | 0.72 | 0.65 | 0.59 | 0.53 | 6.1 / 8 |

**关键观察**：
- 早期位置接受率与总分支数**几乎无关** → 每个 MTP head 学到独立的稳定预测任务
- 从第 2 位起接受率以 ~0.9 衰减
- 3→5：+39% avg accepted length；5→7：仅 +22%，且 6/7 位失败会触发 KV-cache rollback，反而拖累 streaming
- → **MTP-5 是效率-复杂度的最优 trade-off**

> 工程级洞见（§4.3 末）："**Grounded generation tasks can be accelerated more aggressively than free-form text generation, precisely because the external modality reduces semantic branching.** Grounding is not only a source of information; it is also a source of algorithmic structure."

---

## 5. TTS 专用化

### 5.1 架构差异：去掉 encoder + adaptor

[已 verify §5 paper.html L699-700] TTS 分支**完全不要 encoder + adaptor**，audio token 当作 LLM 的一种"新语言"，TTS = pure next-token prediction（NTP）。核心挑战：text ↔ audio 表示空间的对齐。

### 5.2 两阶段 SFT

[已 verify §5.1]

| Stage | 数据 | 目标 |
|---|---|---|
| **Stage 1** 大规模 zero-shot TTS + global instruction supervision | Step-Audio-EditX 合成 | 学粗粒度音色 / speaking style / 整体韵律控制 |
| **Stage 2** 高质量录音 + global & inline 双标注 | 内部录音 | 学 utterance / segment 级细粒度表达控制 |

→ 既支持 global instruction following 又支持 inline expressive control。

### 5.3 TTS RLHF（核心）

[已 verify §5.1 Eq.1]

1. 先训练 **Generative Reward Model** $r_\phi$
2. 对每个 prompt $x$，policy $\pi_\theta$ 生成候选 $y$；高质量 reference $y^*$
3. GRM 对 $(x, y, y^*)$ 输出 pairwise scalar preference
4. 最终 reward 经 reward shaping：

$$
r_{hf}(x, y, y^*) = s\!\left( r_\phi(x, y, y^*) \right)
$$

- $r_\phi$：GRM 原始打分
- $s(\cdot)$：reward shaping 变换

### 5.4 SFT 数据生产管线（recorded data）

[已 verify §5.2] 沿用 **Emotional-Context-Speech** 标注框架（Huggingface 仓库），核心改造在标注 target：

1. **Whisper-Large-v3** 转录 → **MFA** 词级时间戳 → 切 utterance
2. 严重对齐错 / 转录不完整 / 极短样本 drop
3. 每条 utterance 收集对话上下文 / 剧本 metadata
4. **离散化韵律特征**：F0、语速、停顿统计、谱重心、RMS energy、MFCC variance、HNR
5. 把 transcript + context + quantized acoustic features 喂给**标注 LLM**，产出双标注：
   - **Global control**：整句 speaking style / 韵律 / 情感概述
   - **Inline expression**：在文本中插入 local directives 标 span 级表达行为
6. 训练前再用 ASR 模型交叉验证转录，drop mismatch 段

### 5.5 评测：Arena Win-Rate

[已 verify §5.3] 作者明确否定客观指标（CER / SS / LLM-as-judge / MOS）对 LLM-based TTS 的可靠性，转用 arena 评测。

**评测协议**：
- 听感敏感度筛选评测员，评测期内固定不变
- 音对随机化 + 评测位置随机化
- 要求评测员说明偏好理由
- 周期性 spot-check 干预偏差，事后复审大分歧 case
- 774 prompts，对每个 baseline 选官方推荐 voice preset

![Figure 4](https://arxiv.org/html/2605.23463v1/2605.23463v1/figures/stepaudio25-tts-arena.png)

**结果**：StepAudio-2.5-TTS 对 MiniMax-2.8-HD / ElevenLabs-v3 / Gemini-3.1-Flash-TTS **总胜率 67.6%**，对每个 baseline 都一致领先。

> ⚠️ 评估薄弱点：**完全没有报告 WER / SIM / SECS / MOS 等标准客观指标**，arena 评测集不公开，胜率无置信区间。

---

## 6. Realtime 专用化

### 6.1 架构：原封不动复用基座

[已 verify §6 p1] 不改架构：audio encoder + adaptor + decoder。decoder 输出**显式 latent reasoning trace** 再生成 response。

### 6.2 四大挑战

1. Conversational coherence：跨多轮维持话题/风格/对话状态
2. Persona consistency：在多样和 adversarial 输入下守住人格
3. Paralinguistic sensitivity：理解犹豫 / 笑声 / 叹气 / 节奏变化
4. **Reward sparsity**：对话属性（自然度 / 情感契合）无单一 ground-truth → 难以单靠 verifiable reward 优化

### 6.3 三阶段训练管线

[已 verify §6.1]

**(A) Audio-Centric Mid-Training** —— 完全继承基座，保留 audio-grounded perception + long-form reasoning。

**(B) Progressive SFT** —— 三维度渐进 curriculum：

| 维度 | 数据 | 目标 |
|---|---|---|
| Conversational Alignment | 多轮对话 + spoken-language artifacts | turn 级连贯；处理 disfluency / mid-utterance interruption；偏好口语化 / 韵律友好 response |
| Persona & Stylistic Control | persona-conditioned data | 从 curated seed 出发**算法 fission** 产出百万级 persona × verbal habit 组合矩阵；学 compositional generalization |
| Paralinguistic Sensitivity | real spoken interactions | 在 latent reasoning trace 中注册副语言线索，动态调整 response 语气 + pacing；融合"who is speaking"与"how the user is speaking" |

**Dynamic rehearsal schedule**：interaction-specific 数据持续与通用 instruction + reasoning 任务交错，按 validation metric 动态调整比例，防 catastrophic forgetting。

**(C) RLHF with Generated Rewards** —— PPO + KL 正则；GRM 对参考 response 打分 + 显式 interaction rubrics（多轮一致性 / 对早期用户内容的忠实度）；混合 **multi-turn**（一致性主导）+ **single-turn**（更长推理 + 更丰富偏好表达）。

### 6.4 数据基建（三条流）

[已 verify §6.2]

| 数据流 | 规模/构造 |
|---|---|
| Conversational backbone | 自然口语多轮对话，倾向 turn-to-turn 连贯 / 省略 / disfluent / mid-utterance revision；书面体 response 降权 |
| Persona-conditioned | **10,000+ native personas（人写 + 人审）** → 算法 fission 重组 personality / 用语习惯 / 情感边界 / 互动 archetype → **百万级 persona matrix**；每个合成 persona 配对**百万级真实场景对话** |
| Paralinguistic cue | 带 atmosphere descriptor（语速 / 重音 / subtext）+ cue label（hesitation / 轻笑 / 叹气 / 呼吸 / 节奏变化 / 降调） |

整体管线：in-character 一致性检查 + annotation 交叉验证 + 去 fission 引入的近重复。

### 6.5 评测（5 个套件）

[已 verify §6.3]

| 套件 | 类型 | 描述 |
|---|---|---|
| Step-Dialogue-Human-Eval | 主观 mobile-app | 通用对话 |
| step_Dialogue_general | 客观 API | 通用对话 |
| step-Dialogue-car | 客观 API | 车载对话 |
| Step-Dialogue-Understanding | 87 个音频样本 | 从音频信号推断 age / gender / 语速 等声学属性 |
| Step-SPQA | 11 类 audio-Q/audio-A | 沿用 Step-Audio 2 提出的 benchmark |

![Figure 5](https://arxiv.org/html/2605.23463v1/2605.23463v1/figures/stepaudio25-realtime-evaluation.png)

**关键结果**：5 个套件**全部领先**；主观 +10.0 margin；Step-SPQA +16.6 margin；Step-Dialogue-Understanding 强表现 → 副语言条件化没有损害通用 reasoning。

> ⚠️ 评估薄弱点：5 个套件**只有 Step-SPQA 有外部参照**（来自 Step-Audio 2），其余 4 个均为自建，外部可比性有限。

---

## 7. 实验对比矩阵（按方向）

### 7.1 ASR：每条 baseline 的对比维度

| 维度 | StepAudio 2.5 | Qwen3-ASR-1.7B | Doubao-2603 | FunASR-Nano | VibeVoice-ASR |
|---|---|---|---|---|---|
| Avg CER (zh) | **2.97** | 3.17 | 3.34 | 3.66 | 10.19 |
| Avg WER (en) | **3.68** | 3.85 | 6.67 | 5.24 | 7.14 |
| Long-form avg | **3.70** | 4.20 | 6.11 | 5.59 | 4.87 |
| RTF | **0.0053** | 0.0094 | 0.0640 | 0.0591 | 0.1039 |
| 原生 long-form | ✓ (32K ctx) | ✓ | ✓ | ✗ (需 VAD) | ✓ |

### 7.2 TTS：arena 模式（无客观指标）

| 维度 | StepAudio 2.5 | MiniMax-2.8-HD | ElevenLabs-v3 | Gemini-3.1-Flash-TTS |
|---|---|---|---|---|
| Total arena win-rate | **67.6%** | — | — | — |
| WER / SIM / MOS | **未报告** | — | — | — |

### 7.3 Realtime：自建 benchmark 主导

| 套件 | StepAudio 2.5 vs next-best |
|---|---|
| Step-Dialogue-Human-Eval（主观）| **+10.0** |
| Step-SPQA | **+16.6** |
| 其它 3 个 | "consistently outperforms"（无具体数字） |

---

## 8. 结果可信度分层

| 可信度 | 结果 | 理由 |
|---|---|---|
| **高** | ASR zh/en/long-form 三类公开 benchmark CER/WER；RTF | 公开标准集 + 强基线 + 单卡单并发统一 serving；MTP 数字可独立测量 |
| **高** | MTP 加速 vs 准确率 trade-off（≤0.06 abs 点） | 直接表中给 with/without 对照 |
| **中** | TTS 67.6% arena 胜率 | 774 prompt + 流程标准化，但评测集不公开 + 无置信区间 + 无客观指标交叉验证 |
| **中** | Realtime +10.0 / +16.6 margin | 主观 mobile-app + API 客观混合，benchmark 4/5 自建 |
| **低** | TTS 内容一致性 / 说话人相似度的绝对水平 | 论文完全没报 WER / SIM 等 |

---

## 9. 批判性思考（按工程视角）

### 9.1 核心 Claim 审查

1. **Paper Claim**: "achieves state-of-the-art results across ASR, TTS, and Realtime"
   **My Assessment**: ASR 在 3 类公开 benchmark + 强基线下成立（CER 2.97% / WER 3.68% / long-form 3.70% / RTF 0.0053 都 best）。TTS 仅 arena 胜率成立，**绝对 quality 无法判断**。Realtime 受限于自建 benchmark，外部可比性弱。

2. **Paper Claim**: "MTP acts strictly as an acceleration primitive"（不损害准确率）
   **My Assessment**: 完全成立。Table 1 with/without 列差异 ≤0.06 abs 点，**这是 strict-prefix verification 设计带来的**——只接受经验证的前缀，KV-cache rollback 保证最终一致性。

3. **Paper Claim**: "A singular audio-language foundation can successfully internalize the distinct deployment objectives"
   **My Assessment**: 技术上确实共享 backbone + 不同后训练实现三模式。**但 TTS 分支完全去掉 encoder + adaptor**——这种情况下"同一模型"的定义边界存在模糊。论文未量化三分支实际共享的参数比例。

### 9.2 工程优点

1. **Recipe 完整度高**：从 4 阶段 pretraining 到 MTP 两子阶段训练，给出了具体 token 数、序列长、LR、batch、warmup、衰减。对工业团队来说是可借鉴的 ground truth。
2. **MTP-5 设计精巧且可推广**：grounded generation 的加速思路（外部 modality 减少语义分支 → 验证式 multi-token）对任何 "speech/vision-conditioned LM" 都适用。
3. **数据基建是真正护城河**：长音频 3-system ROVER 伪标管线 + Realtime 10K personas → 算法 fission 百万矩阵，这些都是几个月工作量。
4. **统一 RLHF 范式**：TTS / Realtime 都用 GRM-shaped reward，规范统一便于团队扩展。

### 9.3 工程局限

1. **TTS 评估严重薄弱**：不报 WER / SIM / MOS / UTMOS，arena 集不公开。**外部团队无法独立 verify TTS 的绝对水平**。
2. **MoE 参数量完全黑盒**：论文一字不提 active / total params / expert 数。无法估算服务成本。
3. **Codec 内部架构未披露**：speech tokenizer 用了什么量化方式（RVQ/FSQ/其它）/ 帧率 / 码率全部缺失。这是 TTS 工程能力的核心组件之一，缺失严重。
4. **未开源代码 + 未发模型 + 未发评测集**：L2 verify 全部不可用。
5. **Realtime benchmark 4/5 自建**：外部对比能力弱，公允性受质疑。

### 9.4 复现性评估

- [ ] 代码开源（**未开源**）
- [ ] 预训练 / SFT / RLHF checkpoint（**未公开**）
- [x] 训练流程描述（**ASR 部分详尽**，TTS / Realtime 偏粗）
- [ ] 数据集（ASR SFT 部分含公开集 + 自建；TTS / Realtime 数据均未公开）
- [ ] 评测集（Realtime 4/5 套件自建未发布；TTS 774 prompt 未发布）
- [x] long-form 评测附属工具（[wenetspeech-testnet-long](https://github.com/lawlict/wenetspeech-testnet-long) 已开源用于复现 long-form 评测）

### 9.5 工业落地视角的可借鉴点

1. **MTP-5 + autoregressive verification**：直接可移植到任何 grounded-generation 任务（ASR / VQA）。设计精巧，工程成本低。
2. **3-system ROVER + LLM 精修**：替代昂贵人工标注 long-form ASR 数据的标准做法。
3. **算法 fission persona matrix**：解决"persona 数据稀缺"的工程方案。
4. **GRM + reward shaping**：取代 scalar reward model 的趋势型做法，对 TTS / 对话两类没有"唯一正确答案"的任务更合适。
5. **统一基座 + 数据/目标差异化分支**：是 unified speech foundation 路线的工业级实证（vs Qwen3-Omni 同期）。

---

## 🗺️ 在知识地图中的定位

- **所属领域**：[[SpeechLM-领域总览]]（unified audio-language foundation）
- **技术路线**：
  - [[TTS-技术路线图]] §speech-llm-tts —— 在 SpeechLM 内嵌 TTS 路线的工业代表（与 [[Qwen3-Omni]] 同类）
  - [[TTS-技术路线图]] §controllable-tts —— global + inline 双层级 instruction control
- **核心问题**：
  - [[TTS-核心挑战]] §instruction-following / §dialogue-integration（覆盖三方向）
  - 跨方向：在同一 backbone 内做 ASR (transduction) + TTS (generation) + Realtime (interactive) 的统一
- **表示层位置**：[[TTS-表示层地图]] §unified-token-space —— text + audio token 共享 LLM decoder 词表
- **在 SpeechLM/对话框架内的位置**：[[TTS-SpeechLM-Dialogue关系]] **位置 ①②③ 全覆盖**（理解 + 生成 + 对话），但 TTS 分支独立去 encoder 形态特殊
- **相邻工作**：[[Step-Audio 2]]（前代）/ [[StepAudio-EditX]]（数据生成器）/ [[Qwen3-Omni]]（同期统一基座竞品）/ [[GPT-4o]] / [[Gemini]] / Doubao / [[Moshi]]（实时对话标杆）

---

## 🔄 后续重估

- **2026-05-26（首次精读 + 从头重写为工业 tech report 格式）**：核心定位是 unified speech LLM 工业化的代表作之一。ASR 部分**工程价值极高**（具体 recipe + RTF 0.0053 + MTP-5 设计），可作为 LLM-based ASR 服务部署的 reference。TTS / Realtime 部分**透明度不足**——TTS 缺客观指标，Realtime 缺独立 benchmark。MoE 参数量 + codec 内部架构 + 完整 SFT/RL 数据规模均未披露，与 CosyVoice 系列开源 + 详尽 ablation 的风格形成鲜明对比。整体定位：值得跟踪的 unified speech foundation 路线工业代表，但**不可作为 L2 verify 的可靠 ground truth**。

---

## 关联笔记

### 基于
- [[Step-Audio 2]]：直接前代，Stage A adaptor 对齐策略沿用
- [[StepAudio-EditX]]：TTS SFT Stage 1 的合成数据生成器

### 对比
- [[Qwen3-Omni]]：同期同路线（unified audio-language foundation）
- [[Qwen3-ASR-1.7B]]：ASR 主对照 baseline（RTF + 准确率双维度）
- [[MiniMax-2.8-HD]] / [[ElevenLabs-v3]] / Gemini-3.1-Flash-TTS：TTS arena 基线
- [[Moshi]]：实时对话延迟标杆（StepAudio 2.5 未报具体延迟，无法直接比对）
- [[CosyVoice3]]：同期工业 tech report（独立 TTS 路线）对照

### 方法相关
- [[Multi-Token Prediction]]：ASR MTP-5 的方法基础
- [[RLHF]]：TTS / Realtime 共享的后训练范式
- [[MoE]]：基座 LLM 架构
- [[ROVER]]：long-form 数据 voting 算法
- [[SpecAugment]]：ASR SFT 数据增强

### 数据/硬件相关
- [[WenetSpeech]] / [[LibriSpeech]] / [[AISHELL-1]] / [[AISHELL-2]] / [[FLEURS]] / [[Common Voice]] / [[VoxPopuli]] / [[Earnings22]]：ASR benchmark
- [[Step-SPQA]]：Realtime audio-Q/A benchmark（继承自 Step-Audio 2）
- [[H800]]：评测硬件

---

## 速查卡片

> [!summary] StepAudio 2.5 Technical Report (StepFun, 2026-05)
> - **核心**：统一 MoE audio-language 基座 + RLHF 主导后训练 → ASR / TTS / Realtime 三方向同 backbone SOTA
> - **训练规模**：2.2T tokens（800B text + 800B speech + 600B cooldown）；32K 上下文；ASR SFT 100K h + long-form 50K h ROVER 伪标
> - **ASR 核心**：MTP-5 验证式加速；avg CER 2.97% (zh) / WER 3.68% (en) / RTF 0.0053（单 H800 单并发；比 Qwen3-ASR-1.7B 快 ~1.77×）
> - **TTS 核心**：去 encoder/adaptor，pure NTP；GRM-shaped reward RLHF；774-prompt arena 67.6% win-rate（无客观指标）
> - **Realtime 核心**：10K+ native personas → 百万级 persona matrix；PPO + KL + GRM；主观 +10.0 / Step-SPQA +16.6 margin
> - **未开源**：无 code / model / benchmark；MoE 参数量、codec 架构、TTS 数据规模均缺
> - **工程亮点**：4 阶段 pretraining recipe / MTP-5 设计与训练 recipe / 3-system ROVER long-form 数据管线 / persona algorithmic fission 数据基建

---

*笔记创建时间: 2026-05-26（从头重写为工业 tech report 格式）*
