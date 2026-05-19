---
title: "MiniCPM-o 4.5: Towards Real-Time Full-Duplex Omni-Modal Interaction"
method_name: "MiniCPM-o 4.5"
authors: [Junbo Cui, Bokai Xu, Chongyi Wang, Tianyu Yu, Yuan Yao, Zhiyuan Liu, Maosong Sun, Xu Han, Yankai Lin, Chaojun Xiao, MiniCPM-o Team]
year: 2026
venue: arXiv
tags: [full-duplex, omni-modal, speech-llm, streaming, proactive-interaction, edge-deployment]
zotero_collection: Speech-LLM/全双工
image_source: online
arxiv_html: https://arxiv.org/html/2604.27393v1
created: 2026-05-19
---

# 论文笔记：MiniCPM-o 4.5: Towards Real-Time Full-Duplex Omni-Modal Interaction

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | OpenBMB / 清华大学 NLP 组 (Tsinghua / ModelBest) |
| 日期 | April 2026 (arXiv:2604.27393v1) |
| 项目主页 | [MiniCPM-o 4.5 Demo / Model / Code](https://github.com/OpenBMB/MiniCPM-o) |
| 对比基线 | [[Gemini 2.5 Flash]] / [[Qwen3-Omni]] / [[Qwen3-VL]] / [[InternVL3.5]] / [[Kimi-Audio]] / [[GPT-5]] / [[CosyVoice|CosyVoice2]] / [[LiveCC]] / [[StreamingVLM]] |
| 链接 | [arXiv](https://arxiv.org/abs/2604.27393) / [Code](https://github.com/OpenBMB/MiniCPM-o) |

---

## 一句话总结

> 提出 [[Omni-Flow]] 时分多路复用框架，用共享时间轴对齐视/听/说三路 token，把 turn-based 多模态交互改造成 9B 端侧可跑的实时全双工 Omni-LLM。

---

## 核心贡献

1. **首个 9B 全双工 Omni-LLM**: 在 <12 GB RAM 的边缘设备上同时 see / listen / speak，并能基于持续场景理解 **proactively** 主动提醒/评论。
2. **[[Omni-Flow]] 统一流式建模框架**: 用 [[时分多路复用|TDM]] 思想把 env-visual / env-audio / out-stream 三路按毫秒级时间窗 $t$ 对齐成单一因果序列，让 perception 与 response 在 token 级并行；turn-taking 退化为输出 `[listen]` 的特殊情况。
3. **[[TAIL|Time-Aligned Interleaving]] 时延感知文-语交错**: 按累积播放进度自适应决定每个 chunk 的文本量，避免文本跑得比语音快导致 stale response，配 bounded look-ahead 兼顾发音连读上下文。
4. **强对比能力**: 与 [[Gemini 2.5 Flash]] 视觉持平、超越 [[Qwen3-Omni]] 30B-A3B 在 omni 理解 + 语音生成质量；同时 RTX 4090 上 INT4 推理吞吐 212 tok/s、显存 11 GB，效率显著优于 30B-A3B。

---

## 问题背景

### 要解决的问题

现有 [[MLLM]] 的多模态交互瓶颈不再是 modality 覆盖或 [[首包延迟]]，而是 **交互范式本身**：

1. **Perception / response 仍是交替阶段**：模型生成时无法吸收新到的输入做即时调整。
2. **Reactive that's all**：必须等到显式 user request 才响应，无法在 evolving 多模态环境中主动行动（例如 long-horizon assistance、ambient interaction）。

### 现有方法的局限

- **Turn-based 范式**（VAD → 语音识别 → LLM → TTS）：信息流被阻塞，I/O 串行化。
- **[[Mini-Omni|Mini-Omni 2]] / [[Step-Audio]] 等让 backbone 直接吐 speech token**：~25 tok/s 解码效率被拖低、且核心语言能力会退化（[[Catastrophic Forgetting]]）。
- **现有流式 TTS**：要么先生成大段文本再吐音（text lead 太大），要么固定 text-speech 比例（text 进度跑赢播放）→ 两者都让说话内容滞后于 evolving environment。

### 本文的动机

- 把 user request 不当作 privileged role，而当作 always-on 多模态环境状态的一部分；
- 让模型在每个时间窗自主决定 **whether / when / what** to output；
- 用文本作为 [[LLM Backbone]] 的内部"language of thought"，把 speech token 生成下放给轻量 [[Speech Token Decoder]]。

---

## 方法详解

### 模型架构

[[MiniCPM-o 4.5]] 采用 **end-to-end 三件套 + token-level 连续连接** 架构：

- **输入流**:
  - env-visual: [[LLaVA-UHD]] 切片 + [[SigLIP ViT]] (0.4B) → [[Resampler]] 16× 压缩，全双工模式下 max 448×448、否则 2240×2240，每 slice → 64 tokens
  - env-audio: [[Whisper]] Medium encoder (0.3B) chunk 流式编码 50 frames/s → 两层 MLP 5× 时间压缩 → **10 audio tokens/s**
- **Backbone**: [[Qwen3-8B]] (8.2B) 仅在文本域生成，**3-4 decoding steps/s** 即可（人类语速）
- **Speech Token Decoder**: 轻量 Llama 风格 (~0.3B)，输入 = `LLM hidden state (MLP reshape) + 自回归 speech token` → 输出 [[CosyVoice|S3 Token]]，25 frames/s
- **Waveform Synthesis**: 基于 [[Flow Matching]] 的 streaming decoder，将 S3 token → waveform，参考音色由 multimodal system prompt 中的 reference audio 决定（支持 [[Voice Cloning]]）
- **总参数**: 9.34B（learnable，bfloat16）

> 关键设计：**文本是思考语言**，speech token 生成下沉到轻量 decoder，避免 backbone 做高频 25 tok/s 解码导致的效率塌陷与语言能力退化。

### 核心模块

#### 模块 1: [[Omni-Flow]]（全双工统一序列化）

**设计动机**: 借 [[时分多路复用]] 思想把视/听/说三路对齐到一个共享毫秒级时间轴，使 perception 与 response 自然并行。

**Time-Aligned Streams**: 三条 stream
- `env-visual`: 实时视觉观测
- `env-audio`: 声学场景（包含 user speech；user 不再是 privileged role）
- `out-stream`: 助手的文本+语音输出

**Unified Serialization**: 第 $k$ 个 chunk 内
- 视觉 token 序列 $\mathbf{v}^{k}$、音频 token 序列 $\mathbf{a}^{k}$、输出 token 序列 $\mathbf{o}^{k}$
- 不该输出时 $\mathbf{o}^{k}$ 仅含特殊 `[listen]` token
- 三段拼成 group $\mathbf{g}_{k}=[\mathbf{v}^{k};\mathbf{a}^{k};\mathbf{o}^{k}]$，多个 group 串行成单一因果序列
- chunk 内先消化感知 token、再生成输出 token → 每次输出都条件于最新观测
- 减小 $t$ 提高刷新率，模型自然支持 [[Proactive Behavior|主动行为]]，无需外置 [[VAD]]

**Design Tradeoffs**（详见 Table 1 消融）：
- **Temporal granularity**: chunk size 1.0 s / 0.2 s / 0.1 s — chunk 越短响应越及时、但每 chunk 内可用建模预算越少；**1.0 s 最稳**。
- **Boundary explicitness**: 显式 special token 分隔 group 一致优于隐式（区分新输入 vs 新输出非平凡）。
- **Control formulation**:
  - **Listen-Speak (LS)**: 先预测 binary `listen / speak` 控制 token，再生成内容
  - **Listen-Text (LT)**: 直接在共享输出空间预测 `[listen]` 或文本
  - LS 优于 LT（"是否说话" 与 "说什么" 解耦更稳）

#### 模块 2: [[TAIL|Time-Aligned Interleaving]]（时延感知交错语音生成）

**设计动机**: text 生成时间 ≠ speech 播放时间。token 平均 vocalization 时长是 context-dependent 的；如果 chunk 内文本播放耗时 ≫ chunk 时长，则语音逐渐落后于模型最新状态，user 听到的是过期回答。

**具体实现**:
- 在第 $k$ 个 chunk，模型根据 **累积播放进度** 决定本 chunk 生成多少文本，使刚刚生成内容播完后语音流接近时间边界 $kt$
- 若前几个 chunk 累积出现轻微 playback delay → 本 chunk 生成更少文本让语音追平
- 训练监督来自 full-duplex 数据中的 token 起止时间标注：start time ∈ $[(k-1)t, kt)$ 的文本 token 及其对应 speech token 划入第 $k$ chunk
- **Look-Ahead 机制**: chunk $k$ 末尾若干文本 token 的 speech token 推迟到 chunk $k+1$ 发音（覆盖如 `the apple` vs `the car` 的连读上下文），避免 text 显著领先 audio

#### 模块 3: 端到端训练 pipeline（4 阶段）

1. **Speech Pretraining**: 冻结 [[MiniCPM-V]] 4.5 + Whisper，仅训 audio projector / LLM-to-speech projector / speech decoder，对齐 Whisper feature 与 LLM 隐空间、把隐状态映射到语义+韵律 grounded 的 speech token。
2. **Joint Pretraining**: 全参数解冻，跨 vision-language / speech / omni-modal 数据联合预训；不同 modality 组合分到不同 [[Data Parallelism|DP rank]]，每 step 内固定 modality 比例；统一 next-token loss。
3. **Joint SFT**: 两阶段（大规模指令 → 高质量人工标注），随机化 0.2-0.4 MP 分辨率 + 1-5 FPS 实现质量-效率 tradeoff。
4. **RL**: [[GRPO]] 提升推理与指令跟随；引入下方 smooth length reward；用 [[RLAIF-V]] 缓解视觉幻觉（且发现幻觉缓解能从图文迁移到 omni-modal full-duplex）。

---

## 关键公式

### 公式 1: [[Omni-Flow|时间对齐 group 序列化]]

$$
\mathbf{g}_{k} = [\mathbf{v}^{k}; \mathbf{a}^{k}; \mathbf{o}^{k}]
$$

**含义**: 第 $k$ 个时间 chunk（时长 $t$）的 group 把视觉、音频、输出 token 按"先感知后输出"顺序拼接，多个 group 串成单一因果序列供标准 causal LM 消费。

**符号说明**:
- $t$: 时间窗时长，论文最优为 1.0 s
- $\mathbf{v}^{k}$: chunk 内来自 env-visual 的视觉 token
- $\mathbf{a}^{k}$: chunk 内来自 env-audio 的音频 token
- $\mathbf{o}^{k}$: chunk 内 out-stream 输出，无输出时仅含 `[listen]` token

### 公式 2: [[TAIL|TAIL chunk 文本分配]]（描述性）

$$
\text{Tokens assigned to chunk } k = \{\, \text{text token } w_i : t_i^{\text{start}} \in [(k-1)t,\, kt) \,\}
$$

**含义**: 用 token 起止时间标注构造监督，把每个文本 token 及其对应 speech token 划入起始时间所在的 Omni-Flow chunk，使模型学到 history-dependent 的交错策略。

**符号说明**:
- $w_i$: 第 $i$ 个文本 token
- $t_i^{\text{start}}$: 该 token 在播放时间轴上的开始时刻
- 末尾若干 token 的 speech 部分通过 look-ahead 延后到 chunk $k+1$

### 公式 3: [[Smooth Length Reward]]（5.4 节正则）

$$
r_{\mathrm{len}}(i) = \begin{cases} s_{i}, & r_{i}=1 \\ \min(0,\, s_{i}), & r_{i}=0 \end{cases},\quad
s_{i} = \left(0.5 - \frac{\ell_{i}-\ell_{\min}}{\ell_{\max}-\ell_{\min}}\right) \times \min\!\left(1,\, \frac{\ell_{\max}-\ell_{\min}}{\tau}\right)
$$

**含义**: GRPO 的辅助 length reward。短的正确回答得正分、短的错误回答不被奖励；当同 prompt 下 length 差距很小时（除 $\tau$ 项）整体 reward 缩水避免噪声干扰；平滑 shaping，避免 [[Kimi K1.5]] style 过度激进 length penalty 与 accuracy reward 冲突。

**符号说明**:
- $r_{i} \in \{0,1\}$: 第 $i$ 条 response 的正确性
- $\ell_{i}, \ell_{\min}, \ell_{\max}$: 同 prompt 内 response 长度统计
- $\tau$: 长度差距下界，下行差距小则缩小 reward 幅度
- $\min(0, s_i)$ 分支：错误响应只能扣分不能加分

### 公式 4: 视觉编码压缩比

$$
\text{Token compression ratio} = \frac{1024}{64} = 16\times
$$

**含义**: 每个 [[LLaVA-UHD]] slice 经 [[SigLIP ViT]] 编码为 1024 token，再被 [[Resampler]] 压到 64 token，相比常见 4× 压缩更激进，显著降低视觉 token 预算。

**符号说明**:
- 1024: SigLIP 输出 patch token 数（14×14 patch + 448×448 输入）
- 64: Resampler query 数

### 公式 5: 音频编码降采样

$$
50\,\text{Hz} \xrightarrow{\;\text{2-layer MLP, 5}\times\;} 10\,\text{Hz}
$$

**含义**: Whisper Medium encoder 输出 50 frame/s 特征经 2 层 MLP 做 5× 时间压缩到 10 audio tokens/s 给 LLM，换取更小的 backbone token 预算。

---

## 关键图表

### Figure 1: 综合能力雷达图

![Figure 1: Evaluation results on diverse capabilities](https://arxiv.org/html/2604.27393v1/media/radar_minicpmo4.5.png)

**说明**: 跨视觉语言、OCR、多图、视频、音频理解、语音生成、Omni 理解、Streaming 八个维度的对比。MiniCPM-o 4.5 在同尺度开源模型中达 SOTA，逼近 [[Gemini 2.5 Flash]]，整体超越 [[Qwen3-Omni]] 30B-A3B 与提供更高质量语音生成。

### Figure 2: AI 交互范式演进

![Figure 2: Evolution of AI interaction paradigms](https://arxiv.org/html/2604.27393v1/x4.png)

**说明**: text-only → multimodal understanding → omni live streaming → human-like full-duplex 的演化曲线。MiniCPM-o 4.5 把进度推到"perception 与 response 同时进行"的 full-duplex 节点。

### Figure 3: Turn-based vs Full-Duplex Streaming

![Figure 3: From turn-based interaction to full-duplex streaming](https://arxiv.org/html/2604.27393v1/x5.png)

**说明**: 现有范式将 perception 与 response 切成交替阶段，导致信息流阻塞、被动响应；MiniCPM-o 4.5 在说话同时持续接收多模态流，可实时更新输出并主动行动。

### Figure 4: 端到端 Omni-Modal 架构

![Figure 4: End-to-end omni-modal architecture of MiniCPM-o 4.5](https://arxiv.org/html/2604.27393v1/x6.png)

**说明**: 三件套 modality encoders → LLM backbone → speech decoders 通过 token-level hidden state 端到端可训，多模态输入输出在毫秒级共享时间轴上对齐用于 [[Omni-Flow]]。

### Figure 5: 流式语音生成策略对比

![Figure 5: Comparison of streaming speech generation strategies](https://arxiv.org/html/2604.27393v1/x7.png)

**说明**:
- (a) **Large text lead**：先生成长文本再合成，文本远跑赢播放；
- (b) **Fixed text-speech ratio**：固定比例交错，仍假设 token vocalization 时长固定；
- (c) **TAIL (Ours)**：自适应让每 chunk 生成的文本对应约 $t$ 秒播放时长，spoken 与 evolving environment 紧贴。

### Figure 6: Length Reward 训练曲线

![Figure 6: Training set accuracy using different length penalty methods](https://arxiv.org/html/2604.27393v1/x8.png)

**说明**: [[Kimi K1.5]] style 在后期出现训练 accuracy 减速甚至下降（length penalty 与 accuracy reward 冲突）；本文 smooth length reward 曲线接近 baseline、且仍能拿到可观长度压缩 → 平滑 shaping 优于激进 shaping。

---

### Table 1: Full-Duplex 设计选择消融

| Chunk Size | Boundary | Control | AdvBench | AlpacaEval | IFEval | SDQA | MMLU |
|---|---|---|---|---|---|---|---|
| 1.0 s | Explicit | LS | **0.98** | 3.56 | **0.29** | **0.36** | **0.65** |
| 1.0 s | Explicit | LT | 0.92 | **3.60** | 0.24 | 0.35 | 0.56 |
| 1.0 s | Implicit | LT | 0.96 | 3.31 | 0.22 | 0.28 | 0.45 |
| 0.2 s | Explicit | LS | 0.81 | 1.22 | 0.10 | 0.09 | 0.45 |
| 0.1 s | Explicit | LS | 0.67 | 2.40 | 0.10 | 0.13 | 0.32 |

**关键发现**:
- 1.0 s + Explicit + LS 综合最优；
- chunk 越小退化越剧烈 → latency-capacity tradeoff；
- 显式边界、控制与内容解耦（LS）一致更稳。

---

### Table 2: 视觉语言（Instruct 模式）

| Benchmark | [[Gemini 2.5 Flash]] | [[InternVL3.5]] 8B | [[Qwen3-VL]] 8B | [[Qwen3-Omni]] 30B-A3B | **MiniCPM-o 4.5 9B** |
|---|---|---|---|---|---|
| OpenCompass | 78.5 | 75.8 | 76.5 | 75.7 | **77.6** |
| MMBench EN v1.1 | 86.6 | 79.5 | 84.5 | 84.9 | **87.6** |
| MMBench CN v1.1 | 86.0 | 80.0 | 84.7 | 84.1 | **87.2** |
| MathVista | 75.3 | 78.4 | 77.2 | 75.9 | **80.1** |
| MMVet | 81.4 | 83.1 | 73.7 | 74.8 | 74.4 |
| MMMU | 76.3 | 73.4 | 69.6 | 69.1 | 67.6 |
| MMStar | 75.8 | 69.3 | 70.9 | 68.5 | 73.1 |
| AI2D | 87.7 | 84.0 | 85.7 | 85.2 | **87.6** |
| MMT-Bench (val) | 70.0 | 66.7 | 60.9 | **70.4** | 69.7 |
| MM-IFEval | 75.8 | 56.3 | 59.4 | 65.7 | 66.3 |
| OCRBench | 864 | 840 | **896** | 880 | 876 |
| TextVQA (val) | 74.3 | 78.2 | 82.9 | **84.1** | 83.8 |
| DocVQA (val) | 93.0 | 92.3 | **96.1** | 95.4 | 94.7 |
| OmniDocBench (EN) ↓ | 0.214 | 0.322 | 0.255 | 0.216 | **0.109** |
| OmniDocBench (CN) ↓ | 0.290 | 0.416 | 0.319 | 0.363 | **0.162** |
| HallusionBench | 59.1 | 54.5 | 61.1 | 59.7 | **63.2** |
| MMHal-Score | 4.6 | 3.8 | **4.7** | 4.6 | **4.7** |
| MMHal-Hallrate ↓ | **23.9** | 34.7 | 29.9 | 31.6 | 24.3 |
| Mantis-Eval | 72.8 | 70.5 | 74.2 | 78.3 | **79.7** |
| MUIRBench | 74.5 | 55.8 | 64.4 | 61.9 | 72.0 |
| MMSI-Bench | 12.1 | – | 11.3 | 14.2 | **16.6** |
| Video-MME (w/o subs) | **75.6** | 66.0 | 71.4 | 70.5 | 70.4 |
| LVBench | **62.2** | – | 58.0 | 50.2 | 50.9 |
| MLVU (M-Avg) | **77.8** | 70.2 | **78.1** | 75.2 | 76.5 |
| LongVideoBench (val) | – | 62.1 | 66.4 | **66.9** | 66.0 |
| MotionBench | – | **62.3** | 59.5 | 61.7 | 61.4 |

**说明**: 9B 参数量下 OpenCompass 77.6 领先同 scale 开源、逼近闭源 Gemini 2.5 Flash；OmniDocBench EN/CN 大幅领先（0.109 / 0.162）；多图 (Mantis / MMSI) 与幻觉 (HallusionBench) 显著最佳。

---

### Table 3: 视觉语言（Thinking 模式）

| Benchmark | [[Gemini 2.5 Flash]] | [[GPT-5]] | [[Qwen3-VL]] 8B | [[Qwen3-Omni]] 30B-A3B | **MiniCPM-o 4.5 9B** |
|---|---|---|---|---|---|
| OpenCompass | **79.9** | 79.7 | 77.3 | 78.5 | 78.2 |
| MMBench EN v1.1 | 87.1 | 85.5 | 85.3 | 88.2 | **89.0** |
| MMBench CN v1.1 | 87.3 | 85.6 | 85.5 | **87.7** | 87.6 |
| MathVista | 79.4 | **81.9** | 81.4 | 80.0 | 81.0 |
| MMVet | **81.2** | 77.6 | 69.8 | 74.8 | 73.6 |
| MMMU | 77.7 | **81.8** | 74.1 | 75.6 | 70.2 |
| MMStar | **76.5** | 75.7 | 75.3 | 74.9 | 73.6 |
| HallusionBench | 63.5 | **65.2** | 65.4 | 62.8 | 62.6 |
| AI2D | 88.7 | **89.5** | 84.9 | 86.1 | 88.5 |
| MMT-Bench (val) | 70.7 | **72.7** | 68.1 | 70.9 | 69.7 |
| MM-IFEval | 75.7 | **83.1** | 73.5 | 69.9 | 68.2 |
| OCRBench | 853 | 807 | 819 | 859 | **879** |
| TextVQA (val) | 73.8 | 77.8 | 77.8 | **80.8** | 79.8 |
| DocVQA (val) | 92.8 | 91.3 | **95.3** | 94.2 | 92.3 |

**说明**: Thinking 模式下 MMBench EN 89.0 居首；OCRBench 879 首位；STEM 类（MMMU/MMVet）距 GPT-5 / Gemini 仍有差距，反映 thinking-mode 训练数据规模与小模型 scale 双重限制。

---

### Table 4: 音频理解

| Benchmark | [[Kimi-Audio]] 9B | [[Qwen3-Omni]] 30B-A3B | **MiniCPM-o 4.5 9B** |
|---|---|---|---|
| AISHELL-1 ↓ | **0.6** | **0.6** | 0.9 |
| AISHELL-2 ↓ | 2.6 | **2.3** | 2.5 |
| WenetSpeech test-net ↓ | 6.3 | **4.7** | 5.9 |
| WenetSpeech test-meeting ↓ | **5.4** | 5.9 | 5.7 |
| LibriSpeech test-clean ↓ | **1.3** | **1.2** | 1.4 |
| LibriSpeech test-other ↓ | **2.4** | 2.5 | 2.8 |
| GigaSpeech test ↓ | 9.4 | 8.7 | **8.5** |
| VoxPopuli V1-En ↓ | 8.0 | 6.4 | **6.2** |
| CoVoST 2 en→zh | 36.6 | 46.6 | **49.9** |
| CoVoST 2 zh→en | 18.3 | **29.4** | 26.4 |
| MMAU | 68.4 | **77.5** | 76.9 |
| MELD | 59.1 | 56.8 | **60.2** |
| VoiceBench AlpacaEval ∗ | 4.46 | 4.74 | **4.81** |
| Speech TriviaQA | 41.9 | 62.9 | **75.5** |
| Speech Web Questions | 46.4 | **74.9** | 70.2 |
| Speech CMMU | **67.0** | 47.8 | 59.2 |

**说明**: ASR 与 [[Kimi-Audio]] / [[Qwen3-Omni]] 处于同一水平，差距在 0.1-1 WER 内；语义类（CoVoST en→zh、MELD、VoiceBench、Speech TriviaQA）大幅领先 → 体现 backbone 强语言能力的迁移。Speech Web Questions / CMMU 仍有差距（中文知识检索类）。

---

### Table 5: 语音生成

| Model | SeedTTS-ZH CER↓ | SIM-o | SeedTTS-EN WER↓ | SIM-o | LongTTS EN WER↓ | LongTTS ZH CER↓ | Expresso ∗ | ESD ∗ |
|---|---|---|---|---|---|---|---|---|
| [[CosyVoice|CosyVoice2]] | 1.45 | **74.8** | 2.57 | **65.2** | 14.80 | **5.27** | 17.9 | 53.4 |
| [[Qwen3-Omni]] | 1.41 | N/A | 3.39 | N/A | 17.33 | 18.99 | N/A | N/A |
| **MiniCPM-o 4.5** | **0.86** | 74.5 | **2.38** | 64.9 | **3.37** | 6.58 | **29.8** | **82.1** |

**说明**: SeedTTS 双语 CER/WER 均最低；LongTTS EN WER 3.37 远低于 baseline 14-17（长形稳定性大幅领先）；ESD 情感控制 82.1 显著最佳。SIM-o 略低于 CosyVoice2 暗示音色克隆精度仍有空间。

---

### Table 6: 文本能力

| Model | IFEval-PLS | BBH | CMMLU | MMLU | HumanEval | MBPP | Math500 | GSM8K | Avg |
|---|---|---|---|---|---|---|---|---|---|
| Qwen3-8B-Instruct | 83.0 | 69.4 | 78.7 | **81.7** | 86.6 | 75.9 | **84.0** | 93.4 | 81.6 |
| MiniCPM-o 4.5 | **84.7** | **81.1** | **79.6** | 77.0 | 86.6 | **76.7** | 77.0 | **94.5** | **82.1** |

**说明**: 在多模态训练后 7/8 个文本榜单优于纯文 backbone Qwen3-8B-Instruct，平均 +0.5 → omni-modal 训练并未显著伤害文本能力（部分原因可能是数据 mixing + RL）。

---

### Table 7: Omni-Modal 理解（simplex 模式）

| Benchmark | [[Gemini 2.5 Flash]] | [[Qwen3-Omni]] 30B-A3B | **MiniCPM-o 4.5 9B** |
|---|---|---|---|
| Daily-Omni | 79.3 | 70.7 | **80.2** |
| WorldSense | 52.6 | 54.0 | **55.7** |
| Video-Holmes | 51.3 | 50.4 | **64.3** |
| JointAVBench | 55.6 | 53.1 | **60.0** |
| AVUT-Human | 65.4 | 74.2 | **78.6** |
| FutureOmni | 55.6 | **62.1** | 56.1 |
| Video-MME-Short (w/ audio) | **85.5** | 81.3 | 84.7 |

**说明**: 7 项中 5 项最佳；Video-Holmes 复杂视频推理 +13 个点超越 Gemini，AVUT-Human 比 Qwen3-Omni 30B 高 4 点。

---

### Table 8: Vision-only Full-Duplex Benchmark

| Benchmark | [[LiveCC]] 8B | [[StreamingVLM]] 8B | **MiniCPM-o 4.5 9B** |
|---|---|---|---|
| LiveSports-3K-CC | 41.5 | 45.6 | **54.4** |

**说明**: 持续视觉流场景下 win rate 54.4，比 LiveCC / StreamingVLM 高 12.9 / 8.8 点 → [[Omni-Flow]] 把 perception 与 response 编排到共享时间轴让回答更贴合 evolving scene。

---

### Table 9: Length Reward 策略对比

| Length Reward | Avg (Thinking) | Avg (Instruct) | Length ↓ Thinking | Length ↓ Instruct |
|---|---|---|---|---|
| No Length Reward | 73.5 | 70.9 | – | – |
| [[Kimi K1.5]] Style | 73.0 | 70.1 | **50.7%** | 20.2% |
| **Ours (smooth)** | **74.3** | **70.9** | 35.3% | **20.5%** |

**关键发现**: Kimi style 激进压缩导致 accuracy 下滑；本文 smooth shaping 在 -35.3% 长度时 accuracy 反而 +0.8。

---

### Table 10: 不同 Speech 生成模式对比（Seed TTS test）

| Interleaving Mode | ZH CER ↓ | ZH SIM-o ↑ | EN WER ↓ | EN SIM-o ↑ |
|---|---|---|---|---|
| No interleave | 1.44 | 74.1 | 2.70 | 64.9 |
| Fixed text | **0.86** | **74.5** | **2.38** | 64.9 |
| Dynamic text ([[TAIL]]) | 1.04 | 74.1 | 3.93 | **65.1** |

**关键发现**: 非交错最差；fixed-text 交错 CER/WER 最佳（适合 turn-based 高质量场景）；TAIL 在 full-duplex 必要的时延对齐上有取舍 → EN WER 略升 (3.93)，但仍可用，属于 streaming-quality tradeoff。

---

### Table 11: vLLM 推理效率（NVIDIA RTX 4090）

| Model | Dtype | Throughput (tok/s) ↑ | First-token Latency (s) ↓ | Memory (GB) ↓ |
|---|---|---|---|---|
| [[Qwen3-Omni]] 30B-A3B | BF16 | OOM | OOM | OOM |
| **MiniCPM-o 4.5** | BF16 | 154.3 | **0.59** | 19 |
| [[Qwen3-Omni]] 30B-A3B | INT4 | 147.8 | 0.98 | 20 |
| **MiniCPM-o 4.5** | INT4 | **212.3** | **0.58** | **11** |

**说明**: BF16 下 30B-A3B 直接 OOM；INT4 下 MiniCPM-o 4.5 吞吐高 ~44%、首 token 延迟近半、显存仅 ~55%。64-frame 视觉输入下首 token 0.58 s 已达消费级实时门槛。

---

### Table 12: 不同推理框架对比（RTF / 显存）

| Framework | Dtype | RTX 4090 RTF↓ | RTX 4090 Mem(GB)↓ | DGX Spark RTF↓ | DGX Spark Mem(GB)↓ |
|---|---|---|---|---|---|
| PyTorch | BF16 | OOM | OOM | 2.43 | 26 |
| PyTorch | INT4 | 1.26 | 14 | 1.27 | 14 |
| **llama.cpp-omni** | FP16 | 0.27 | 19 | 0.46 | 19 |
| **llama.cpp-omni** | INT4 | **0.21** | **11** | **0.20** | **11** |

**说明**: 自研 [[llama.cpp-omni]] 推理框架在 INT4 下 RTF 0.20-0.21，显著优于 PyTorch INT4 的 1.26-1.27 → 真实端侧实时全双工可行性的关键工程支撑。

---

### Table 13: 模型架构超参（9.34B total，bfloat16）

| Component | 关键超参 |
|---|---|
| **[[SigLIP ViT]] 视觉编码器 (417.8M)** | hidden 1152, 27 layers, 16 heads, FFN 4304, GELU-tanh, patch 14×14 |
| **[[Resampler]] (88.9M)** | 64 query tokens, embed 4096, 32 heads |
| **[[Whisper]] Medium 音频编码器 (307.2M)** | hidden 1024, 24 layers, 16 heads, FFN 4096, GELU, 80 mel bins |
| **音频投影 (21.0M)** | 2-layer MLP + ReLU, $1024\to 4096\to 4096$ |
| **[[Qwen3-8B]] LLM Backbone (8189.2M)** | hidden 4096, 36 layers, 32 heads, [[GQA]] KV=8, head dim 128, FFN 12288, SiLU, [[RMSNorm]] $\epsilon=10^{-6}$, vocab 151748, ctx 40960, [[RoPE]] $\theta=10^6$, no weight tying |
| **Backbone-to-Decoder Projector (10.5M)** | 2-layer MLP+ReLU, $4096\to 768\to 768$ |
| **Speech Token Decoder** | text embed 116.8M, transformer 188.8M, hidden 768, 20 layers, 12 heads, FFN 3072, SiLU, ctx 4096, codebook 6562, 单码本 25 frames/s |

---

## 实验

### 数据集

| 类型 | 来源 / 处理 | 用途 |
|---|---|---|
| **大规模自然语音** | 多源未标注 → [[Silero VAD]] + Whisper + [[Paraformer]] + diarization 多 pipeline 标注 | zero-shot TTS / ASR / 多轮多说话人对话 |
| **专业录音对话** | LLM 生成口语化指令对话 → 人工演员录制（保留口语化、变化情绪/语速/重音、固定声纹） | TTS instruction-following / QA / 多轮自然对话 |
| **Vision-Language 数据** | 基于 [[MiniCPM-V]] 4.5 数据扩展 + 改进 [[CapsFusion]] caption 合成 + relevance-aware OCR masking + 密集视频描述 + reward-model filtering | 视觉理解 |
| **Omni-Modal Full-Duplex Web** | Web audio-video 段，过滤单人主播 + 弱视听相关；OCR 字幕去除、talking-head 检测、ASR 转录过滤 | Omni 训练 |
| **Full-Duplex Task 数据** | 人工标注的连续场景描述、proactive reminding 等任务 | Full-duplex 能力 |
| **评测集** | OpenCompass, MMBench, OmniDocBench, AISHELL-1/2, WenetSpeech, LibriSpeech, GigaSpeech, VoxPopuli, CoVoST 2, MMAU, MELD, VoiceBench, [[SeedTTS-eval]], LongTTS, Expresso, [[ESD]], Daily-Omni, WorldSense, Video-Holmes, LiveSports-3K-CC | 全维度评测 |

### 实现细节

- **Backbone**: [[Qwen3-8B]]
- **Visual Encoder**: [[SigLIP ViT]] + [[LLaVA-UHD]] + [[Resampler]]（16× 压缩）
- **Audio Encoder**: [[Whisper]] Medium，chunk-based streaming，5× 时间压缩
- **Speech Decoder**: 0.3B Llama-style，输出 [[CosyVoice|S3 Token]]，配 streaming Flow-Matching 声码
- **训练精度**: bfloat16
- **总参数**: 9.34B
- **RL**: [[GRPO]] + 自研 smooth length reward + [[RLAIF-V]]，前 480 step 不开 length reward
- **推理后端**: vLLM / 自研 [[llama.cpp-omni]]（macOS / Windows / Linux）

---

## 批判性思考

### 优点

1. **范式创新明确**: [[Omni-Flow]] 把 turn-taking 转写为时分多路复用的 token 拼接，用 standard causal LM 即可吃下，工程友好。
2. **三件套合理分工**: backbone 只在文本域 3-4 step/s 解码、speech token 下放给 0.3B 小 decoder，避开了 [[Mini-Omni]] / [[Step-Audio]] 让 backbone 直出 speech token 的效率/语言能力 tradeoff。
3. **TAIL 解决"音落后于思"问题**: 累积播放进度感知 + bounded look-ahead，比固定比例与"先文后音"更适合 evolving environment。
4. **[[Proactive Behavior|主动行为]] 自然生发**: out-stream 每 chunk 自主决定 listen/speak，无需外置 [[VAD]] 与 turn-taking 模块。
5. **9B / 12 GB / RTX 4090 全双工**: 配合自研 llama.cpp-omni INT4 RTF 0.20，端侧落地有真实工程性。
6. **消融充分**: chunk size、boundary、control formulation 三个维度交叉做实验，给出明确 1.0 s + Explicit + LS 推荐。
7. **视觉/文本能力没塌**: 多模态训练后文本榜单平均反而 +0.5（vs Qwen3-8B-Instruct backbone）。

### 局限性

1. **Speech 部分 SIM-o 略低于 [[CosyVoice|CosyVoice2]]**（74.5 vs 74.8 ZH，64.9 vs 65.2 EN），TAIL 模式 EN WER 升至 3.93 → streaming 模式下音色克隆与发音稳定性仍有损失。
2. **MMMU / MMVet 落后** Gemini 2.5 Flash 与 Qwen3-VL 较多（thinking 模式下 MMMU 70.2 vs GPT-5 81.8）→ STEM 推理和小模型 scale 都有限制。
3. **Speech Web Questions / Speech CMMU 仍有差距** → 中文 speech 知识检索类不及对手，提示语音端到端检索覆盖弱。
4. **TAIL 训练监督依赖精确 token 起止时间标注**，构造成本高，对工业级数据 pipeline 要求严格。
5. **Proactive 能力作者亲承"还相对简单"**，未做长 horizon planning / 主动协助等高阶 agentic 评测。
6. **闭源数据**：百万小时自然语音 + 录音棚演员录制 + web audio-video pipeline，复现门槛高。
7. **未与 [[Moshi]] / [[GLM-4-Voice]] / [[VITA]] 等真正全双工对话基线直接对比** —— LiveSports-3K-CC 是 vision-only audio-free 基准，omni 全双工没有公开同尺度对比。

### 潜在改进方向

1. 引入 [[Speech DPO]] / 偏好建模进一步抬高 SIM-o 与表达力；
2. TAIL 监督半自动化（用 forced alignment 自动生成 token 起止时间）；
3. 更激进的 chunk size 自适应（按上下文动态选 0.5/1.0/2.0 s）；
4. Proactive 行为接入 [[Long-Horizon Memory]] / 计划型 agent；
5. 针对中文 speech 知识检索类做 RAG 风格增强；
6. 与 [[Moshi]]/[[GLM-4-Voice]] 在统一 full-duplex protocol 下做对比 benchmark。

### 可复现性评估

- [x] 代码开源（[OpenBMB/MiniCPM-o](https://github.com/OpenBMB/MiniCPM-o)）
- [x] 预训练模型（HuggingFace 上可下载）
- [x] 端侧推理框架（llama.cpp-omni 提供）
- [ ] 训练数据：百万小时自然语音 + 演员录制 + web audio-video 大规模管线均未开放
- [x] 训练细节：4 阶段 pipeline 与 RL 设计写得相对完整
- [x] 评测集均为公开 benchmark

---

## 关联笔记

### 基于
- [[MiniCPM-V]] 4.5: 视觉语言基底；本文从其 pretrained checkpoint 出发
- [[Qwen3-8B]]: 文本 backbone
- [[Whisper]] Medium: 流式音频编码器
- [[CosyVoice|CosyVoice2]]: streaming flow-matching decoder + S3 token 沿用
- [[LLaVA-UHD]]: 任意宽高比高分辨率切片
- [[SigLIP ViT]]: 视觉骨干
- [[Kimi K1.5]]: length reward 思路对比基础

### 对比
- [[Gemini 2.5 Flash]]: 闭源同尺度旗舰，本文逼近其视觉能力
- [[Qwen3-Omni]] 30B-A3B: 同期开源 Omni 直接对手
- [[Mini-Omni|Mini-Omni 2]] / [[Step-Audio]]: 让 backbone 直出 speech token 的代表，本文反之
- [[Moshi]]: 全双工双流建模另一思路（双流 codec）
- [[GLM-4-Voice]]: 端到端语音对话
- [[LiveCC]] / [[StreamingVLM]]: vision-only full-duplex 基线

### 方法相关
- [[Omni-Flow]]: 本文核心机制
- [[TAIL|Time-Aligned Interleaving]]: 时延感知文-语交错
- [[Proactive Behavior]]: out-stream `[listen]` 触发的主动行为
- [[GRPO]]: RL 算法
- [[RLAIF-V]]: 视觉幻觉缓解
- [[Smooth Length Reward]]: 自研 length reward
- [[llama.cpp-omni]]: 端侧推理框架
- [[Resampler]]: 视觉 token 16× 压缩
- [[Flow Matching]]: 声码器范式

### 硬件/数据相关
- [[NVIDIA RTX 4090]]: 端侧推理标杆
- [[DGX Spark]]: 专业级硬件对照
- [[SeedTTS-eval]]: 语音生成评测
- [[VoiceBench]]: speech instruction-following 评测
- [[Daily-Omni]] / [[WorldSense]] / [[Video-Holmes]] / [[JointAVBench]]: omni 评测套件
- [[LiveSports-3K-CC]]: vision-only full-duplex 评测

---

## 速查卡片

> [!summary] MiniCPM-o 4.5
> - **核心**: 用 [[Omni-Flow]] 时分多路复用把 turn-based 多模态交互改造成 9B 端侧实时全双工 Omni-LLM
> - **方法**: 三件套（视/听 encoder + Qwen3-8B + 0.3B speech decoder + Flow-Matching 声码）+ chunk=1.0s 共享时间轴序列化 + [[TAIL]] 时延感知文-语交错 + smooth length reward + [[RLAIF-V]] 反幻觉
> - **结果**: OpenCompass 77.6（同 scale 开源 SOTA、逼近 Gemini 2.5 Flash），SeedTTS CER/WER 双语最低，Omni 7 项 5 项最佳，RTX 4090 INT4 RTF 0.20、12 GB RAM
> - **代码**: [github.com/OpenBMB/MiniCPM-o](https://github.com/OpenBMB/MiniCPM-o)

---

*笔记创建时间: 2026-05-19*
