---
title: "StepAudio 2.5 Technical Report"
method_name: "StepAudio 2.5"
authors: [StepFun-Audio Team]
year: 2026
venue: arXiv
arxiv_id: "2605.23463"
tags: [speech-llm, unified-model, asr, tts, realtime-dialogue, rlhf, multi-token-prediction, moe]
zotero_collection: 5-Speech-LLM与AudioLM
image_source: online
arxiv_html: https://arxiv.org/html/2605.23463
created: 2026-05-25
---

# 论文笔记：StepAudio 2.5 Technical Report

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | StepFun (阶跃星辰) |
| 日期 | May 2026 |
| 前作 | [[StepAudio]] / [[Step-Audio-2]] / [[StepAudio-EditX]] |
| 对比基线 | ASR: [[Qwen3-ASR]] / [[VibeVoice]] / FunASR-Nano / Doubao-ASR; TTS: MiniMax-2.8-HD / ElevenLabs-v3 / Gemini-3.1-Flash-TTS; Realtime: GPT-Realtime / Gemini Live / Doubao Realtime |
| 链接 | [arXiv](https://arxiv.org/abs/2605.23463) / [HTML](https://arxiv.org/html/2605.23463) |

---

## 一句话总结

> 阶跃星辰把 ASR、TTS、Realtime 对话三个方向统一到**一个 MoE LLM backbone**，在 **2.2T token** 上渐进式预训练；下游三个分支分别靠 [[Multi-Token Prediction|MTP-5 可验证多 token 解码]]（ASR RTF **0.0053**）、[[RLHF]] + [[Generative Reward Model|GRM]]（TTS Arena **67.6% 胜率**）和渐进式 SFT + GRM-PPO（Realtime 全面领先 GPT-Realtime / Gemini Live）刷新各自 SOTA。

---

## 核心贡献

1. **统一基础架构**：冻结 audio encoder + 轻量 adaptor + MoE LLM decoder 的非对称设计，ASR / TTS / Realtime 三个方向共享同一 backbone，仅通过数据、优化目标和解码约束分化——验证了"一个高质量多模态先验 + 不同监督路由"的统一范式。
2. **ASR: 可验证多 token 解码 (MTP-5)**：每一步前向产生 6 个 token 提案（1 主 + 5 辅助），配合自回归验证确保准确性不损失；RTF 0.0053，比 [[Qwen3-ASR]]-1.7B 快 **1.8x**，比 [[VibeVoice]]-ASR 快 **20x**。
3. **TTS: 全 LLM 合成 + 偏好对齐**：完全去掉 encoder-adaptor，将语音合成建模为纯 next-token prediction；通过 [[Generative Reward Model|GRM]]-based [[RLHF]] 对齐人类偏好，Arena 评测 **67.6%** 胜率碾压三个商用基线。
4. **Realtime: 渐进式人格一致对话**：million-scale 人格矩阵 + 副语言敏感训练 + GRM-PPO，在主观评测中领先竞品 **+10.0 分**，Step-SPQA 领先 **+16.6 分**。
5. **大规模预训练数据引擎**：自动化 pipeline 覆盖 SED/VAD/多 ASR 交叉验证/ROVER 投票/LLM 后处理，支撑 800B text + 800B speech token 的统一多模态训练。

---

## 问题背景

### 要解决的问题

语音系统正经历**架构趋同**——ASR 从 CTC/RNN-T 走向 encoder-decoder + LLM，TTS 从手工 pipeline 走向生成式离散表示建模，Realtime 对话需要低延迟 + 人格一致 + 副语言理解。三条路径在底层都趋向于 "将语音当作另一种序列类型映射到 LLM 的共享潜在空间"。问题在于：**能否用一个统一基础模型同时做好这三件事？**

### 现有方法的局限

- **专用系统**：ASR / TTS / 对话系统独立训练和部署，参数冗余、知识无法共享。
- **早期 Speech LLM** ([[StepAudio]] / [[SpeechGPT]] / [[SALMONN]])：验证了统一可行性，但在各专项 benchmark 上仍弱于专用 SOTA。
- **推理效率**：LLM-based ASR 的自回归解码速度远慢于 CTC/Transducer，限制了大模型 ASR 的实用性。
- **TTS 对齐**：传统 MOS / CER / SIM 指标对 LLM-based TTS 有偏，且 SFT 后的表达力 / 自然度仍有优化空间。
- **Realtime 对话**：人格一致性、副语言敏感度和奖励稀疏性是核心未解问题。

### 本文的动机

"一旦 text 和 audio 共享一个多模态表示空间，任务特化就只是不同的数据、优化目标和解码约束的问题"——StepAudio 2.5 验证这个命题，三个方向的识别、合成和对话能力是"同一个多模态记忆的三种查询方式"。

---

## 方法详解

### 统一基础架构

StepAudio 2.5 采用 **audio-encoder-adaptor-LLM-decoder** 的非对称架构：

- **冻结 audio encoder**：将波形特征转为紧凑声学嵌入，提供稳定的声学抽象，在下游训练中始终冻结。
- **轻量 adaptor**：将 encoder 的声学嵌入映射到 LLM 的隐空间。
- **MoE LLM decoder**：从文本 [[Mixture of Experts|MoE]] LLM 初始化，在统一序列空间中同时处理 text token 和 speech token。承担语义理解、上下文管理、指令遵循和生成的全部负担。

**设计原则**：刻意的非对称——encoder 负责声学抽象，decoder 负责语义。"一旦语义主要在 decoder 里，下游任务即使输出不同也能共享大部分模型。"

### 任务特化 = 方向性推理

从同一个 backbone 出发，三个 "推理方向"：

| 方向 | 条件 | 输出空间 | 核心挑战 |
|------|------|----------|----------|
| **ASR** | 声学嵌入 → decoder | 窄、离散、强锚定于语音信号 | 准确性 + 解码效率 |
| **TTS** | 文本 + 控制指令 → decoder | 丰富、连续、需要自然表达 | 保真性 + 可控性 + 表达力 |
| **Realtime** | 实时音频 + 对话历史 → decoder | 低延迟 + 人格一致 + 副语言 | 延迟 + 一致性 + 副语言敏感 |

---

### 共享数据引擎与基础预训练 (Section 3)

#### 数据 Pipeline (Section 3.1)

原始音频经自动化处理：
1. **SED (Sound Event Detection) + VAD** → 过滤非语音
2. 相邻 VAD 段合并 → 按语义完整性重分段
3. 标注：质量评分、合成语音检测、说话人计数、**双 ASR 转录**、语言 ID
4. 通过 WER / 编辑距离 / 语速交叉验证
5. 按语言、时长、语义质量分、音频质量分分级

#### 渐进式基础训练 (Section 3.2)

**总量**：在 **2.2T token** (text + audio) 上持续预训练。

| 阶段 | 规模 | 序列长度 | 训练内容 | 训练模块 |
|------|------|----------|----------|----------|
| **Stage 1: Adaptor 对齐** | 3B ASR token | - | ASR 数据（沿用 Step-Audio 2） | 仅 adaptor（encoder + LLM 冻结） |
| **Stage 2: 统一多模态训练** | 800B text + 800B speech | 16K | ASR / TTS / 语音翻译 / text-speech 交替续写 / speech-to-speech 对话 | 全量 |
| **Stage 3: Cooldown** | 600B 高质量 token | 32K | 在 Stage 2 基础上引入 Audio Caption + Instruct TTS | 全量 |

**Stage 2 内部分两阶段**：
- **Warmup (128B token)**：稳定新引入的 speech vocabulary；adaptor / embedding / output 层用更大学习率，**MoE router 用更小学习率**以减少对文本模态的干扰。
- **主训练**：层级学习率归一化；**MoE auxiliary loss 系数和 router 学习率渐进退火**，平衡 expert 利用率与 top-k 路由概率。

---

### ASR 分支：可验证多 token 解码

#### MTP-5 架构

在共享 encoder-adaptor-decoder 之上加 **MTP-5 (Multi-Token Prediction) head**：

- 解码位置 $t$ 处：主分支预测 $x_{t+1}$，5 个辅助分支分别预测 $x_{t+2}, \ldots, x_{t+6}$
- **一次前向产生 6 个 token 提案**
- 推理时通过自回归验证——一旦某个 future token 与正常解码路径不一致，后续全部拒绝，从接受前缀恢复
- MTP "严格只作为加速原语"，不影响输出质量

每个 MTP block 的结构：
1. 取前一分支的隐状态 + 移位 token embedding
2. 归一化 + 拼接 → 投影回 decoder 隐藏维度
3. 通过 decoder-style Transformer block 处理
4. 所有分支共享 embedding 层和词汇输出头

#### ASR 训练流程

**Stage 1: ASR SFT**
- 短/长 form 数据混合训练，打包到 32K token 序列
- SpecAugment 时频 masking
- Audio encoder 冻结，优化 adaptor + decoder
- 10K steps, peak LR $2 \times 10^{-5}$, batch=32, cosine decay → $1 \times 10^{-6}$

**Stage 2: MTP 训练（两子阶段）**

| 子阶段 | 训练模块 | LR | 初始化 |
|--------|----------|-----|--------|
| 冻结分支对齐 | 仅 MTP blocks | $2 \times 10^{-4}$ | Transformer 层从 decoder 最后一层拷贝 |
| 联合校准 | adaptor + LLM + MTP | $2 \times 10^{-5}$ | 从上一阶段继续 |

两阶段均：32K 序列 / batch=32 / 10K steps。

#### ASR 数据

| 类别 | 规模 | 特点 |
|------|------|------|
| 短时有监督数据 | ~100K h | 中英 + code-switching + 垂直领域 + 远场/高噪 |
| 长时伪标签数据 | ~50K h | 三系统 ASR 转录 → ROVER 投票 → $\hat{e} > 0.05$ 丢弃 → LLM 精修 |

长时数据构建 pipeline：
1. VAD 分段 (≤30s) → 2. 三独立 ASR 转录 → 3. 表面形式归一化 → 4. **ROVER 对齐融合** → 5. 仅保留 ≥2 系统共识的 token → 6. 段级分歧率 $\hat{e} = \frac{\text{分歧位置数}}{\text{文本单元数}}$，$\hat{e} > 0.05$ 丢弃 → 7. 拼接为长 form → 8. LLM 补标点 / ITN / 跨段术语一致化

---

### TTS 分支：全 LLM 合成 + RLHF

#### 架构特点

TTS 分支**完全去掉 encoder-adaptor 模块**，仅依赖 LLM backbone。将 audio token 视为一种新"语言"，语音合成完全建模为 next-token prediction。

#### 训练流程

**SFT 两阶段**：

| 阶段 | 数据 | 目标 |
|------|------|------|
| Stage 1 | 大规模 zero-shot TTS + 全局指令 | 粗粒度控制（说话人特征、说话风格、整体韵律） |
| Stage 2 | 高质量内部语音 + 全局+内联指令 | 细粒度控制（句子级 + 片段级同步） |

核心训练目标：**zero-shot voice cloning TTS**，支持全局和内联双级别可控性。

**RLHF (基于 GRM)**：

对每个 prompt $x$，提供高质量参考响应 $y^*$，策略模型 $\pi_\theta$ 生成候选 $y$，[[Generative Reward Model|GRM]] $r_\phi$ 评估 $y$ 相对于 $y^*$ 的偏好得分：

$$r_{hf}(x, y, y^*) = s\!\left(r_\phi(x, y, y^*)\right)$$

其中 $s(\cdot)$ 是 reward shaping 变换。GRM 相比传统标量 reward model 能捕获更细粒度的人类偏好信号。

#### TTS 可控性

- **全局控制**：整个话语的说话风格、韵律模式、情感状态
- **内联控制**：插入文本中的局部指令，标记片段级表达行为
- **Zero-shot voice cloning**：无需说话人微调的任意说话人复制

#### SFT 数据构建

**1. 模型合成数据（全局指令控制）**：
- 用 [[StepAudio-EditX]] 生成，提供跨风格和情感属性的大规模合成能力

**2. 录制语音数据（全局 + 内联控制）**：
标注 pipeline 参照 Emotional-Context-Speech（[HuggingFace](https://huggingface.co/Insects/Emotional-Context-Speech)）：
1. [[Whisper]]-Large-v3 转录
2. Montreal Forced Aligner 词级时间戳对齐 → 话语级分段
3. 过滤对齐错误 / 不完整转录 / 过短样本
4. 收集对话历史 / 脚本上下文

**韵律特征提取与离散化**：F0 / 语速 / 停顿统计 / 谱质心 / RMS 能量 / MFCC 方差 / HNR → tokenize → 拼接转录和元数据 → LLM 标注输出：
- **全局控制描述**：整体风格/韵律/情感
- **内联表达描述**：片段级表达指令

---

### Realtime 分支：渐进式人格对话

#### 核心挑战

1. **对话连贯性**：跨多轮保持话题上下文、风格一致、对话状态
2. **人格一致性**：在多样/对抗性输入下保持特定人格特征和表达风格
3. **副语言敏感**：理解犹豫、笑声、叹气、节奏变化等非语言信号
4. **奖励稀疏**：自然度、情感匹配等属性缺乏单一 ground truth

#### 三阶段训练

**Stage 1: Audio-Centric Mid-Training**
- 继承基础模型，提供稳健的音频感知和长 form 推理能力

**Stage 2: 渐进式 SFT（三维度递进注入）**

| 维度 | 数据 | 目标 |
|------|------|------|
| 对话对齐 | 指令丰富的多轮对话 | 轮次连贯、口语化、处理不流畅/中断 |
| 人格风格控制 | million-scale 人格矩阵 + 百万级真实场景语料 | 组合泛化到未见人格 |
| 副语言敏感 | 带气氛描述的真实对话 | 在 latent reasoning trace 中注册非语言信号，动态调整语调和节奏 |

**防灾难性遗忘**：动态 rehearsal schedule 持续交织通用指令和推理任务。

**人格矩阵构建**：

> 从 **10,000+ 人工撰写并验证的原生人格描述**出发 → 算法性 fission 重组正交属性 → **million-scale 人格矩阵** → 每个合成人格与 million-scale 真实场景语料配对。

**Stage 3: RLHF + GRM-PPO**

PPO + KL 正则化，使用与 TTS 相同的 [[Generative Reward Model|GRM]] 架构：

$$r_{hf}(x, y, y^*) = s\!\left(r_\phi(x, y, y^*)\right)$$

**双重奖励信号**：
- **偏好对比**：整体响应质量
- **Rubric 评分**：指令敏感维度（跨轮一致性、忠实于用户内容）

**训练数据混合**：多轮对话（跨轮一致性）+ 单轮 prompt（更长推理、更丰富偏好）。

#### Realtime 数据构建

三个互补 SFT 数据流：

1. **对话骨干**：自然口语多轮对话，过滤保留轮次连贯、省略/不流畅表达、中途修正
2. **人格条件对话**：10K 原生人格 → million-scale 人格矩阵 → million-scale 真实场景配对
3. **副语言线索数据**：气氛描述（语速/重音/潜台词）+ 线索标签（犹豫/轻笑/叹气/呼吸/节奏变化/降调）

全语料统一 pipeline 检查：角色一致性、标注交叉验证、近重复去除。

---

## 关键公式

### 公式 1: MTP 分支权重衰减

$$w_h = \frac{\alpha^{h-1}}{\sum_{j=1}^{H} \alpha^{j-1}}, \quad H=5, \quad \alpha=0.9$$

**含义**: 越远的 future token 分支权重越小，指数衰减控制辅助分支对总 loss 的贡献。

### 公式 2: MTP 训练损失（位置 $t$）

$$\mathcal{L}_t = \text{CE}(p_t, x_{t+1}) + \sum_{h=1}^{H} w_h \, \text{CE}(p_{t,h}, x_{t+1+h})$$

**含义**: 标准 next-token CE + 加权的多 future token CE。$p_t$ 是主分支分布，$p_{t,h}$ 是第 $h$ 辅助分支分布。

### 公式 3: 段级分歧率（长时 ASR 数据过滤）

$$\hat{e} = \frac{\text{分歧位置数}}{\text{文本单元数}}$$

**含义**: 三系统 ROVER 对齐后，分歧超过 5% 的段丢弃，确保伪标签质量。

### 公式 4: GRM-based RLHF 奖励（TTS + Realtime 共用）

$$r_{hf}(x, y, y^*) = s\!\left(r_\phi(x, y, y^*)\right)$$

**含义**: Generative Reward Model 对候选 $y$ 和参考 $y^*$ 做成对比较，$s(\cdot)$ 做 reward shaping。比传统标量 reward model 捕获更细粒度的偏好信号。

---

## 关键图表

### Figure 1: 统一基础架构

![Figure 1: StepAudio 2.5 统一架构](https://arxiv.org/html/2605.23463v1/x1.png)

**说明**: 中央为共享的 audio encoder-adaptor-LLM decoder 栈，向三个方向分化：ASR（转录 token）、TTS（audio token）、Realtime（低延迟对话）。

### Figure 2: ASR MTP 架构

![Figure 2: MTP-5 多 token 解码](https://arxiv.org/html/2605.23463v1/x2.png)

**说明**: encoder-adaptor-decoder backbone + 5 个并行 future token 预测分支。每一步前向产生 6 个 token 提案，推理时自回归验证。

### Figure 3: 长时 ASR 数据 Pipeline

![Figure 3: Long-form ASR 数据构建](https://arxiv.org/html/2605.23463v1/x3.png)

**说明**: 原始录音 → VAD 分段 → 三独立 ASR 转录 → 归一化 → ROVER 投票融合 → 分歧过滤 → 段拼接 → LLM 后处理。

### Table 1: ASR 中文基准 (CER %)

| 测试集 | VibeVoice-ASR | FunASR-Nano | Doubao-ASR-2603 | Qwen3-ASR-1.7B | **StepAudio 2.5** | w/o MTP |
|---|---|---|---|---|---|---|
| AISHELL-1 | 5.19 | 1.88 | 2.07 | 1.49 | **0.71** | 0.79 |
| AISHELL-2 iOS | 5.10 | 2.61 | 2.70 | 2.50 | **2.29** | 2.30 |
| WenetSpeech testnet | 14.79 | 5.30 | **4.03** | 4.44 | 4.54 | 4.57 |
| WenetSpeech testmeeting | 17.09 | 5.31 | 5.09 | **4.66** | 4.70 | 4.73 |
| FLEURS zh | 8.77 | 3.19 | 2.83 | 2.74 | **2.63** | 2.63 |
| **Average** | 10.19 | 3.66 | 3.34 | 3.17 | **2.97** | 3.00 |

### Table 2: ASR 英文基准 (WER %)

| 测试集 | VibeVoice-ASR | FunASR-Nano | Doubao-ASR-2603 | Qwen3-ASR-1.7B | **StepAudio 2.5** | w/o MTP |
|---|---|---|---|---|---|---|
| LibriSpeech clean | 2.30 | 1.80 | 2.94 | 1.69 | **1.38** | 1.40 |
| LibriSpeech other | 5.79 | 4.43 | 5.98 | 3.57 | **3.16** | 3.14 |
| Common Voice v11 en | 20.03 | 11.05 | 14.06 | **7.50** | 7.57 | 7.62 |
| FLEURS en | 5.20 | 4.96 | 6.74 | **3.23** | 3.55 | 3.74 |
| VoxPopuli cleaned AA | **2.38** | 3.97 | 3.61 | 3.28 | 2.76 | 3.23 |
| **Average** | 7.14 | 5.24 | 6.67 | 3.85 | **3.68** | 3.83 |

### Table 3: ASR 长时基准 (Error Rate %)

| 测试集 | VibeVoice-ASR | FunASR-Nano | Doubao-ASR-2603 | Qwen3-ASR-1.7B | **StepAudio 2.5** | w/o MTP |
|---|---|---|---|---|---|---|
| LibriSpeech clean long | 1.66 | 2.34 | 2.81 | 1.95 | **1.27** | 1.27 |
| LibriSpeech other long | 3.48 | 4.89 | 5.59 | 3.81 | 2.90 | **2.81** |
| WenetSpeech testnet long | 8.73 | 4.74 | **3.72** | 4.15 | 4.09 | 4.09 |
| Earnings22 cleaned AA | 5.62 | 10.38 | 12.33 | 6.90 | 6.52 | **6.34** |
| **Average** | 4.87 | 5.59 | 6.11 | 4.20 | **3.70** | 3.63 |

**关键发现**: MTP-5 对识别精度的影响在 0.06 绝对点以内——验证了自回归验证确保转录质量。

### Table 4: ASR 解码效率 (RTF)

| 模型 | RTF |
|---|---|
| VibeVoice-ASR | 0.1039 |
| FunASR-Nano | 0.0591 |
| Doubao-ASR-2603 | 0.0640 |
| Qwen3-ASR-1.7B | 0.0094 |
| **StepAudio 2.5 ASR** | **0.0053** |

100 条 30 秒 clip 在单卡 H800 上测量。StepAudio 2.5 比 [[Qwen3-ASR]]-1.7B 快 **1.8x**，尽管使用更大的 decoder——证明 "decoder 规模不再线性转化为 token-by-token 延迟"。

### Table 5: MTP 接受率分析

| 配置 | 1st | 2nd | 3rd | 4th | 5th | 6th | 7th | 平均接受长度 |
|---|---|---|---|---|---|---|---|---|
| MTP-3 | 0.96 | 0.88 | 0.80 | - | - | - | - | 3.6 / 4 |
| MTP-5 | 0.95 | 0.88 | 0.80 | 0.71 | 0.64 | - | - | 5.0 / 6 |
| MTP-7 | 0.96 | 0.88 | 0.80 | 0.72 | 0.65 | 0.59 | 0.53 | 6.1 / 8 |

**关键发现**:
- 早期位置接受率不随总分支数变化——每个 MTP head 学到独立稳定的预测任务
- 从位置 2 起，接受率按 ~0.9/branch 衰减
- MTP-3→MTP-5：平均接受长度 **+39%**；MTP-5→MTP-7 仅 +22%，因高位置失败导致 KV cache 回滚
- **MTP-5 是效率-复杂度的最优平衡点**

### Figure 4: TTS Arena 胜率

**StepAudio-2.5-TTS 总胜率 67.6%**，在 774 条 prompt 的 arena 成对比较中一致击败 MiniMax-2.8-HD / ElevenLabs-v3 / Gemini-3.1-Flash-TTS。

### Figure 5: Realtime 评测

五个评测维度上 StepAudio 2.5 Realtime 全面领先：
- **主观人工评测**: 领先次优系统 **+10.0 分**
- **Step-SPQA**: 领先 **+16.6 分**
- **Step-Dialogue-Understanding**: 87 条多样音频样本测试声学特征推理

---

## 实验

### 评测设置

**ASR**：与 VibeVoice-ASR / FunASR-Nano / Doubao-ASR-2603 / [[Qwen3-ASR]]-1.7B 对比，单卡 H800 单并发（Doubao 通过 API）。

**TTS**：Arena 式成对评估，774 条 prompt。
- 评估者听力灵敏度筛选 → 固定评估者池
- 随机化模型音频对和评估位置
- 定期抽查 + 高差异案例后评估审查
- 基线：MiniMax-2.8-HD / ElevenLabs-v3 / Gemini-3.1-Flash-TTS（各用官方推荐最优声音预设）

论文明确讨论了传统指标的局限：CER / SIM 对 LLM-based 模型有内在偏差；ASR 指标在丰富副语言现象下不可靠；嵌入式说话人验证丢弃高频细节；LLM-as-judge 难以评估韵律质量。因此选用人类偏好 arena 评测。

**Realtime**：五个评测套件：
| 套件 | 类型 | 内容 |
|------|------|------|
| Step-Dialogue-Human-Eval | 主观/手机 App | 通用对话 |
| step_Dialogue_general | 客观/API | 通用对话 |
| step-Dialogue-car | 客观/API | 车载场景 |
| Step-Dialogue-Understanding | 客观 | 87 条音频的声学特征推理（年龄/性别/语速） |
| Step-SPQA | 客观 | 11 类音频问答（来自 Step-Audio 2） |

### 关键定量结果汇总

| 方向 | 核心指标 | 数值 | 意义 |
|------|----------|------|------|
| ASR (中文) | 平均 CER | **2.97%** | 优于 Qwen3-ASR (3.17) |
| ASR (英文) | 平均 WER | **3.68%** | 优于 Qwen3-ASR (3.85) |
| ASR (长时) | 平均 Error Rate | **3.70%** | 优于 Qwen3-ASR (4.20) |
| ASR | AISHELL-1 CER | **0.71%** | 绝对 SOTA，vs Qwen3-ASR 1.49 |
| ASR | LibriSpeech clean WER | **1.38%** | 绝对 SOTA，vs Qwen3-ASR 1.69 |
| ASR | RTF | **0.0053** | 比 Qwen3-ASR 快 1.8x |
| TTS | Arena 总胜率 | **67.6%** | 碾压三商用基线 |
| Realtime | 主观评测优势 | **+10.0** | vs 次优系统 |
| Realtime | Step-SPQA 优势 | **+16.6** | vs 次优系统 |

### MTP 的关键 insight

> "有锚定的生成任务（grounded generation）有时比自由文本生成更容易加速——因为外部模态（音频）约束了输出分布，降低了语义分支，锚定提供的不仅是信息，还有**算法结构**，使多 token 验证能以高接受率成功。"

---

## 批判性思考

### 优点

1. **统一范式真正 work 了**：三个方向都达到或超过专用系统 SOTA，证明 "一个 backbone 三种查询" 不是空话。ASR 中文 CER 0.71 (AISHELL-1) 和 RTF 0.0053 同时达到速度和精度 SOTA 尤其有说服力。
2. **MTP-5 的 grounding insight 深刻**：指出 grounded generation 比 free-form text 更适合 multi-token decoding，因为音频锚定降低了输出分布的不确定性——这个观察可推广到其他 grounded NLP 任务。
3. **GRM 是比标量 reward model 更好的 RLHF 方案**：在 TTS 和 Realtime 两个方向都展现了 generative reward model 对人类偏好的细粒度捕获能力。
4. **Realtime 的人格矩阵工程创新**：10K 原生人格 → million-scale 矩阵 → 组合泛化，是 scalable persona conditioning 的工程范本。
5. **长时 ASR 数据构建严谨**：三系统 ROVER + 分歧率过滤 + LLM 后处理，伪标签质量有保障。
6. **评测方法论严肃**：TTS 不用传统 MOS/CER 而用 Arena 评测，并在论文中明确论证了传统指标对 LLM-based TTS 的偏差——这种自省式方法论值得其他工作学习。

### 局限性

1. **模型细节严重缺失**：MoE LLM 的具体架构（参数量、expert 数、top-k routing）、audio encoder 架构、speech codec/tokenizer 设计（码本数/帧率/VQ 方式）均未公开。这是商业 tech report 的通病，但严重影响可复现性和学术价值。
2. **TTS 没有传统客观指标**：虽然 Arena 评测有道理，但完全不提 WER/SIM/MOS 让无法做横向对比。67.6% 胜率中 per-baseline 胜率细分也没给。
3. **Realtime 评测基线不透明**：竞品名称没有明确列出（只在 Figure 5 中），具体数值只给了 margin（+10.0 / +16.6）而非绝对分。
4. **WenetSpeech 上未赢**：testnet 和 testmeeting 两个中文子集分别输给 Doubao (4.03 vs 4.54) 和 Qwen3-ASR (4.66 vs 4.70)，说明在噪声/会议场景下还有提升空间。
5. **没有多语种 ASR 评测**：只有中英文，对于 "统一基础模型" 的定位来说语种覆盖不够。
6. **代码/模型未开源**：截至发布时无 GitHub / HuggingFace 链接，仅有论文。

### 潜在改进方向

1. **开放 codec/encoder 细节**：至少公开帧率、码本维度、token 数等基本参数，方便社区定位能力来源。
2. **TTS 补充客观评测**：在 Seed-TTS-eval 等标准 benchmark 上给出 WER/SIM，与 [[SemaVoice]] / [[IndexTTS 2]] / [[VoxCPM]] 做公平比较。
3. **多语种 ASR 扩展**：[[FLEURS]] 已覆盖 100+ 语种，评测范围应扩大。
4. **MTP 推广到 TTS/Realtime**：MTP 在 ASR 上效果显著，是否能加速 audio token 生成？论文未探讨。
5. **流式 Realtime 延迟**：论文未报告 first-turn latency / P50/P99 延迟等工程指标。

### 可复现性评估

- [ ] 代码开源（未发现）
- [ ] 预训练模型开源（未发现）
- [ ] 音频 encoder 架构公开（未公开）
- [ ] Speech codec 细节公开（未公开）
- [ ] MoE LLM 参数量公开（未公开）
- [x] 训练流程和超参描述完整（ASR 部分详尽；TTS/Realtime 部分较粗）
- [x] 评测基准可获取

---

## 关联笔记

### 前作 / 基础
- [[StepAudio]]: 初代统一语音交互系统（speech tokenizer + LLM + speech decoder）
- [[Step-Audio-2]]: 2.5 的直接前作，提供 adaptor 对齐方法和基础架构
- [[StepAudio-EditX]]: 语音编辑模型，在 TTS 分支中用于合成训练数据

### ASR 方向对比
- [[Qwen3-ASR]]: 1.7B 参数，最直接的 ASR 对手；StepAudio 2.5 在中英平均和 RTF 上全面超越
- [[Whisper]]: 经典 encoder-decoder ASR，StepAudio 2.5 代表了 LLM-based ASR 的下一代

### TTS 方向对比
- [[SemaVoice]]: 连续 AR TTS + SFM 对齐，Seed-TTS EN WER 1.71%
- [[CosyVoice 2]]: D-AR+C-NAR cascaded TTS
- [[IndexTTS 2]]: D-AR+C-NAR，SIM 强

### Realtime 方向对比
- [[Moshi]]: 开源全双工对话系统
- [[GLM-4-Voice]]: 清华/智谱的语音对话模型
- [[Qwen2.5-Omni]]: 阿里巴巴的 Omni 模型

### 方法相关概念
- [[Multi-Token Prediction]]: ASR 加速核心技术（MTP-5 可验证多 token 解码）
- [[Generative Reward Model]]: TTS + Realtime 共用的偏好对齐方案
- [[RLHF]]: TTS 和 Realtime 的后训练对齐
- [[Speech DPO]]: 相关的语音偏好优化方法
- [[Mixture of Experts]]: 基础 LLM backbone 架构
- [[Speech LLM]]: 本文所属范式
- [[Full-Duplex]]: Realtime 分支的相关概念

### 训练方法相关
- [[Modality Alignment]]: adaptor 对齐阶段
- [[Curriculum Learning]]: 渐进式预训练（3B→1.6T→600B）

---

## 速查卡片

> [!summary] StepAudio 2.5
> - **核心**: 统一 audio-language foundation model，一个 MoE LLM backbone 同时做 ASR / TTS / Realtime 对话，三个方向通过不同数据/优化目标/解码约束分化。
> - **预训练**: 2.2T token 渐进式训练（adaptor 对齐 3B → 统一多模态 1.6T → cooldown 600B），序列长度 16K→32K。
> - **ASR**: MTP-5 可验证多 token 解码，中文 CER 2.97% / 英文 WER 3.68% / AISHELL-1 **0.71%** / RTF **0.0053**（1.8x faster than Qwen3-ASR）。
> - **TTS**: 全 LLM 合成（去掉 encoder-adaptor），GRM-RLHF 偏好对齐，Arena 胜率 **67.6%**。
> - **Realtime**: 渐进式 SFT + million-scale 人格矩阵 + GRM-PPO，主观评测 **+10.0** / Step-SPQA **+16.6**。
> - **代码**: 未开源。

---

*笔记创建时间: 2026-05-25*
