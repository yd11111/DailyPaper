---
title: "OmniFlatten: An End-to-end GPT Model for Seamless Voice Conversation"
method_name: "OmniFlatten"
authors: [Qinglin Zhang, Luyao Cheng, Chong Deng, Qian Chen, Wen Wang, Siqi Zheng, Jiaqing Liu, Hai Yu, Chaohong Tan, Zhihao Du, Shiliang Zhang]
year: 2024
venue: arXiv
tags: [full-duplex, speech-llm, dialogue, flatten, post-training, gpt, omni, low-latency]
zotero_collection: Speech-LLM/全双工
image_source: online
arxiv_html: https://arxiv.org/html/2410.17799v2
created: 2026-05-19
---

# 论文笔记：OmniFlatten: An End-to-end GPT Model for Seamless Voice Conversation

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | 阿里巴巴 Tongyi Lab |
| 日期 | Oct 2024 (v2: Jan 2025) |
| 项目主页 | https://omniflatten.github.io/ |
| 对比基线 | [[Moshi]], [[LLaMA-Omni]], [[GLM-4-Voice]], [[Mini-Omni]], [[SpeechGPT]], [[VITA]], [[SyncLLM]] |
| 链接 | [arXiv](https://arxiv.org/abs/2410.17799) / [HTML](https://arxiv.org/html/2410.17799v2) / [Demo](https://omniflatten.github.io/) |

---

## 一句话总结

> 把用户/助手 × 语音/文本四路信号"切块再展平"成单条序列，用 SFT 三阶段把 [[Qwen2-0.5B]] 后训练成低延迟全双工语音对话 LLM，**完全不改 GPT 架构**。

---

## 核心贡献

1. **Flatten 范式**: 提出 chunk-based [[Flatten 操作]]，把 user-speech / user-text / assistant-text / assistant-speech 四个并行流以固定 chunk 大小（speech=10 token / text=2 token）交错串成一条序列，让原生 [[GPT]] 自回归就能处理全双工对话——避免 [[Moshi]] 那种多流并行需要 acoustic delay + inner monologue 的复杂设计。
2. **三阶段渐进式后训练**: [[Modality Alignment]]（ASR+TTS SFT 做语音-文本对齐）→ [[Half-duplex Dialogue Training]]（轮替对话）→ [[Full-duplex Dialogue Training]]（先 3-stream 再 2-stream），符合 [[Curriculum Learning]] 思想。
3. **零架构改动**: 完全复用 [[Qwen2-0.5B]] 的 GPT backbone，仅靠数据组织和 SFT 把文本 LLM 改造成实时语音对话模型，不依赖耗算的预训练。
4. **数据合成 pipeline**: 从开源文本对话→[[CosyVoice]] 合成→真实交互模式时序仿真（包含打断/沉默/响应延迟）→[[MUSAN]] 加噪，产出 2000 小时多通道全双工对话数据。
5. **Turn-taking 显著超越 [[Moshi]]**: Assistant turn-taking 平均响应时延 193 ms（Moshi 553 ms），User turn-taking 287 ms（Moshi 753 ms）；Acc@5 也大幅领先。

---

## 问题背景

### 要解决的问题

构建端到端**低延迟、自然交互**的[[全双工]] 语音对话系统，要支持：[[Turn-taking]]、[[Backchannel]]、[[Barge-in]]、overlapping speech 等真实人类对话现象。

### 现有方法的局限

- **协作式系统**（[[Qwen-Audio]] / [[SALMONN]] + 外接 [[TTS]]）：模块拼接，必然引入级联延迟，无法做真正的 full-duplex。
- **端到端但半双工**（[[SpeechGPT]] / [[LauraGPT]] / [[Mini-Omni]] / [[LLaMA-Omni]] / [[GLM-4-Voice]]）：是 turn-based 模型，不支持同时听说。
- **端到端全双工 [[Moshi]]**：多流并行（user-speech / assistant-text / assistant-speech）建模功能强，但**不被原生 GPT 自回归支持**，需要复杂的 acoustic delay 和 [[Inner Monologue]] 设计。
- **端到端全双工 [[SyncLLM]]**：用 chunk 化 + deduplication 简化建模，但去重导致音频重建误差，且只产 speech 不产 text，丢掉了语义引导。
- **[[LSLM]]**：靠外接 TTS 实现"边听边说"+随时打断。

### 本文的动机

如果能把多流多任务数据**直接 flatten 成单条序列**，那就可以无脑套 GPT 自回归——既保留 GPT 简单性，又避开 Moshi 的并行流定制设计；同时**chunk 粒度**是显式可调旋钮，控制语义完整性 vs 延迟的折中。

---

## 方法详解

### 模型架构

OmniFlatten 采用 **decoder-only [[GPT]] 单序列自回归**架构：

- **Backbone**: [[Qwen2-0.5B]]（小模型省算力，但仍有竞争力）
- **音频离散化**: [[CosyVoice]] 的 [[Speech Tokenizer]]——单码本 [[Vector Quantization|VQ]]，4096 个 code，由多语种 ASR 监督训练，输出**语义 token**（不是声学 token），benefit 语音理解和生成内容一致性
- **音频还原**: [[Optimal-transport Conditional Flow Matching|OT-CFM]]（CosyVoice 同款）将 token → Mel，然后 [[HiFi-GAN]] vocoder → waveform；OT-CFM 比扩散更快、梯度更简单
- **核心创新**: [[Flatten 操作]] + 三阶段后训练，**无任何模型结构改动**
- **总参数**: 0.5B（远小于 [[Moshi]] 7B、[[LLaMA-Omni]] 8B、[[GLM-4-Voice]] 9B）

### 核心模块

#### 模块 1: Audio Tokenization & Detokenization

**设计动机**: 用 [[Semantic Token]] 而非 [[Acoustic Token]]，让对话的"内容流"和文本流天然对齐——这是 flatten 操作能 work 的前提。

**具体实现**:
- 编码器 + 单码本 VQ → 4096 codes，每帧一个 token（CosyVoice 帧率约 25 Hz / 50 Hz）
- 解码用 [[OT-CFM]] 从 token 生成 Mel，[[HiFi-GAN]] 从 Mel 生成 waveform

#### 模块 2: Modality Alignment（阶段 1）

**设计动机**: 让原本只懂文本的 [[Qwen2-0.5B]] 学会"说话"和"听话"，是后续对话学习的前置能力。

**具体实现**: 多任务 SFT，复用 GPT 的 next-token prediction，仅靠特殊 task ID 切换任务：

ASR 样本格式：
```
[ASR][SOS] S_seq [EOS][SOT] T_seq [EOT]
```

TTS 样本格式：
```
[TTS][SOT] T_seq [EOT][SOS] S_seq [EOS]
```

其中 `[ASR]`/`[TTS]` 是 task ID，`[SOS]/[EOS]` 是语音段起止，`[SOT]/[EOT]` 是文本段起止。`S_seq` 是离散 speech token 序列，`T_seq` 是文本 token 序列。

#### 模块 3: Half-duplex Dialogue Training（阶段 2）

**设计动机**: 全双工太难，[[Curriculum Learning]] 先学半双工——半双工里没 overlapping speech，跟 ASR/TTS 数据分布最一致，作为过渡。

**具体实现**: 把多轮 turn-based 对话 flatten 成 4-stream 单序列，按 speaker turn 排：
- Turn N-1：User Speech Tokens（红方块）→ User Text Tokens（红圆）
- Turn N：Assistant Text Tokens（蓝圆）→ Assistant Speech Tokens（蓝方块）

模型相当于在序列里依次做：**ASR**（user 语音→text）→ **textual response**（user text→assistant text）→ **TTS**（assistant text→assistant speech）。

#### 模块 4: Full-duplex Dialogue Training（阶段 3.1：3-stream）

**设计动机**: 真正全双工要求**实时**——不能等用户说完才转写，必须流式产 assistant 输出。把 user text stream 砍掉。

**具体实现**:
- **三流**: user speech / assistant text / assistant speech
- **Chunk 化** + **Relaxed alignment**: 不强制 token 级语音文本对齐
- **Chunk 大小**: speech chunk = 10 token, text chunk = 2 token（文本信息密度高，chunk 小一些，避免 text 跑得太超前于 speech，同时尽量保留 TTS 能力）
- **Flatten 顺序**: input speech → output text → output speech（每个 chunk 内部依次拼接）
- **Padding**: 文本结束后用 `silent_text_token` 填空，语音静音段用 `silent_speech_token`
- **Chunk 内自回归**: dashed arrow 表示模型把预测出的 assistant text+speech token 作为 input 继续 AR 解码

#### 模块 5: Full-duplex Dialogue Training（阶段 3.2：2-stream）

**设计动机**: 进一步减少延迟，去掉对中间文本的依赖，专注 speech-to-speech。

**具体实现**: 删去 assistant text stream，只保留 user speech 和 assistant speech。Chunk N-1 输入 5 个 user speech token，输出 5 个 assistant speech token。代价是丧失文本语义引导，对话质量明显下降（见 Table 3）。

### 训练策略关键点

- **Loss masking**: 对话学习阶段在 user channel 上做 loss masking——观察发现这能稳定训练（user 通道有合成噪声）
- **Loss**: 全程 standard cross-entropy
- **Optimizer**: AdamW (β1=0.9, β2=0.95, weight decay 0.1), max LR=2e-5, warm-up + cosine decay
- **Batch**: 100M tokens / batch
- **Epochs**: 5
- **Max seq len**: alignment 阶段 1024 → dialogue 阶段 8192

---

## 关键公式

### 公式 1: [[Modality Alignment]] ASR 训练样本格式

$$
\text{ASR sample} = [ASR]\,[SOS]\,S_{seq}\,[EOS]\,[SOT]\,T_{seq}\,[EOT]
$$

**含义**: 把 ASR 任务转成"先听语音 token、再吐文本 token"的下一 token 预测；标签 `[ASR]` 让模型识别任务方向。

**符号说明**:
- $S_{seq}$: 离散语音 token 序列（CosyVoice 单码本 4096 codes）
- $T_{seq}$: 对应的文本 token 序列（Qwen tokenizer）
- $[SOS]/[EOS]$: speech 段起止
- $[SOT]/[EOT]$: text 段起止

### 公式 2: [[Modality Alignment]] TTS 训练样本格式

$$
\text{TTS sample} = [TTS]\,[SOT]\,T_{seq}\,[EOT]\,[SOS]\,S_{seq}\,[EOS]
$$

**含义**: 反向——先给文本 token 再产语音 token，与 ASR 共享同一 GPT，唯一区别就是 task ID 和顺序。

**符号说明**: 同公式 1。

### 公式 3: [[Flatten 操作]] 全双工 3-stream chunk 单序列

$$
\underbrace{[U^S_{n-1}]}_{\text{10 user speech}}\;
\underbrace{[A^T_{n-1}]}_{\text{2 asst text}}\;
\underbrace{[A^S_{n-1}]}_{\text{10 asst speech}}\;
\underbrace{[U^S_{n}]}_{\text{10 user speech}}\;
\underbrace{[A^T_{n}]}_{\text{2 asst text}}\;
\underbrace{[A^S_{n}]}_{\text{10 asst speech}}\;\cdots
$$

**含义**: 每个 chunk 顺序拼接"用户语音 → 助手文本 → 助手语音"三段；多个 chunk 沿时间方向继续拼。GPT 自回归只看左侧上下文，因此天然满足"流式输入边听边说"。

**符号说明**:
- $U^S_n$: 第 $n$ 个 chunk 的用户语音 token 段（长度 10）
- $A^T_n$: 第 $n$ 个 chunk 的助手文本 token 段（长度 2）
- $A^S_n$: 第 $n$ 个 chunk 的助手语音 token 段（长度 10）
- 静音段填 $\text{silent\_speech\_token}$ / $\text{silent\_text\_token}$

### 公式 4: [[Flatten 操作]] 全双工 2-stream

$$
\underbrace{[U^S_{n-1}]}_{\text{5 user speech}}\;\underbrace{[A^S_{n-1}]}_{\text{5 asst speech}}\;\underbrace{[U^S_{n}]}_{\text{5 user speech}}\;\underbrace{[A^S_{n}]}_{\text{5 asst speech}}\;\cdots
$$

**含义**: 移除 assistant text stream，纯 speech-to-speech，进一步降延迟但牺牲语义质量。

### 公式 5: 训练目标（标准交叉熵 + user channel loss mask）

$$
\mathcal{L} = -\sum_{t \in \mathcal{T}_{\text{assist}}} \log P_\theta(x_t \mid x_{<t})
$$

**含义**: 只在 assistant 通道（assistant text+speech）的 token 位置算 cross-entropy；user 通道因为是带噪输入，做 loss mask 跳过，提升训练稳定性。

**符号说明**:
- $\mathcal{T}_{\text{assist}}$: 序列里属于 assistant 通道的 token 位置集合
- $x_t$: 第 $t$ 位 token
- $\theta$: GPT 参数

---

## 关键图表

### Figure 1(a): Overall Architecture / 整体架构

![Figure 1a](https://arxiv.org/html/2410.17799v2/x1.png)

**说明**: OmniFlatten 端到端全双工对话模型总览。输入两个并行流（user speech、assistant speech），经过 [[Speech Tokenizer]] 离散化 → flatten 成单序列 → [[Qwen2-0.5B]] GPT backbone 自回归 → 输出 speech token + 可选 text token → [[OT-CFM]] + [[HiFi-GAN]] 合成 waveform。三阶段训练流水线：modality alignment（ASR+TTS）→ half-duplex dialogue → full-duplex dialogue（先 3-stream 再 2-stream）。

### Figure 1(b): Half-duplex Dialogue Training / 半双工对话训练

![Figure 1b](https://arxiv.org/html/2410.17799v2/x2.png)

**说明**: 4-stream flatten。按 speaker turn 排序：Turn N-1 是 User Speech Tokens（红方块）+ User Text Tokens（红圆），Turn N 是 Assistant Text Tokens（蓝圆）+ Assistant Speech Tokens（蓝方块）。本质是序列内串联 ASR → text dialogue → TTS 三段任务，作为从 modality alignment 到 full-duplex 的过渡。

### Figure 1(c): Full-duplex 3-stream Training / 全双工三流训练

![Figure 1c](https://arxiv.org/html/2410.17799v2/x3.png)

**说明**: 删除 user text stream，剩 user speech / assistant text / assistant speech 三流；按 chunk 切分（speech chunk = 10 token, text chunk = 2 token）后**flatten** 成单序列，顺序为 input speech → output text → output speech。Chunk N-1 输入 5 个 user speech token，模型输出 2 个 assistant text token + 5 个 assistant speech token。虚线箭头表示 chunk 内自回归——模型把刚预测的 assistant text/speech token 续到上下文继续解码。这是真正"边听边说"的训练形态。

### Figure 1(d): Full-duplex 2-stream Training / 全双工双流训练

![Figure 1d](https://arxiv.org/html/2410.17799v2/x4.png)

**说明**: 进一步去掉 assistant text stream，仅 user speech 和 assistant speech 两流；目标是降延迟、纯 speech-to-speech。代价是失去文本语义引导，Table 3 显示 chat 质量大幅下降。

### Figure 2: Data Simulation Pipeline / 对话数据合成管线

![Figure 2](https://arxiv.org/html/2410.17799v2/x5.png)

**说明**: 全双工对话训练数据的合成流水线。
1. 收集开源**纯文本对话**（[[Alpaca]] / [[Moss]] / [[BelleCN]] / [[UltraChat]]）→ 启发式过滤掉含代码/数学/罕见符号/长文本的样本，保留 **390K 多轮 session**
2. 用 [[CosyVoice]] 合成 speech；user 通道音色从 [[LibriSpeech]] / [[3D-Speaker]] 采样，assistant 通道用固定音色
3. **时序仿真**模拟三种交互模式：(a) 用户问完→助手立即响应；(b) 用户尝试打断→助手立即停止；(c) 助手讲完→保持沉默等待用户
4. 给 user 通道叠加 [[MUSAN]] 噪声（SNR 15-30 dB），模拟真实环境
5. 最终得到 **2000 小时多通道全双工对话数据**，按 1%/1%/98% 切 val/test/train

### Table 1: ASR 结果（modality alignment 后）

| Model | Librispeech test-clean (WER) | test-other (WER) | Wenetspeech test-meeting (CER) | test-net (CER) |
|---|---|---|---|---|
| [[Whisper]]-S | 3.13 | 7.37 | 25.62 | 16.66 |
| [[Whisper]]-L | **1.82** | **3.5** | 18.87 | **10.48** |
| [[VITA]] | 8.14 | 18.4 | **12.15** | 16.53 |
| **OmniFlatten** | 7.91 | 19.21 | 26.1 | 19.0 |

**说明**: OmniFlatten 在 0.5B 参数下 ASR 跟 [[VITA]] 在英文 test-clean 上接近，但中文 test-meeting 显著落后（26.1 vs 12.15），与 [[Whisper]]-Large 仍有差距。说明小模型 + 100K 小时混合数据做 modality alignment 已能初步具备 ASR 能力，但还不能跟专门 ASR 模型比。

### Table 2: TTS 结果（modality alignment 后）

| Model | LibriTTS (WER) | AIShell-3 (CER) |
|---|---|---|
| Original (人声) | 2.66 | 2.52 |
| [[ChatTTS]] | 8.32 | 3.87 |
| [[CosyVoice]] | **2.89** | **3.82** |
| **OmniFlatten** | 4.51 | 4.46 |

**说明**: OmniFlatten 的 TTS（用 Whisper-Large-v3 / Paraformer-zh 反转后算 WER/CER）显著优于 [[ChatTTS]]，与专门的 [[CosyVoice]] 接近——modality alignment 阶段成功把 GPT 改造成可用的 TTS。注：未评 [[MOS]]。

### Table 3: 全双工对话能力（LLM-as-Judge 由 Qwen-Max 评 1-10 分）

| Model | Params | En Score(Text) | En Score(Speech+ASR) | Zh Score(Text) | Zh Score(Speech+ASR) |
|---|---|---|---|---|---|
| Qwen2-0.5B-Instruct | 0.5B | 6.75 | – | 6.98 | – |
| Qwen2-7B-Instruct | 7B | **8.37** | – | **8.09** | – |
| [[LLaMA-Omni]] | 8B | 6.01 | **5.50** | 4.17 | 3.89 |
| [[Moshi]] | 7B | 3.92 | 3.46 | – | – |
| [[GLM-4-Voice]] | 9B | 6.97 | 6.40 | 7.02 | **6.69** |
| **OmniFlatten directly 3-stream** | 0.5B | 2.99 | 2.59 | 4.94 | 3.95 |
| **OmniFlatten 3-stream w/o half-duplex** | 0.5B | 3.89 | 3.54 | 5.25 | 4.76 |
| **OmniFlatten 3-stream full process** | 0.5B | 4.88 | 3.92 | 5.6 | 5.15 |
| **OmniFlatten 2-stream full process** | 0.5B | – | 2.19 | – | 3.06 |
| Ground Truth Response | – | 7.65 | – | 6.83 | – |

**说明 / 关键发现**:
1. **三阶段训练有效**：直接 3-stream（2.99/4.94）→ 加 modality alignment（3.89/5.25）→ 再加 half-duplex（4.88/5.6），单调递增，验证 [[Curriculum Learning]] 思路
2. **2-stream 大幅掉点**：去掉 assistant text 后从 3.92→2.19（En Speech）和 5.15→3.06（Zh Speech），说明文本流对 chat 语义至关重要
3. **加 speech 模态会损失 chat 能力**：Qwen2-0.5B-Instruct（纯文本）6.75，加上语音改造后变 4.88
4. **小模型 vs 大模型的劣势**：0.5B 全程落后 8B [[LLaMA-Omni]]（En）和 9B [[GLM-4-Voice]]
5. **[[Moshi]] 中文为零**：不支持中文，且英文常反问而非直接回答，比 OmniFlatten 还差
6. 中文表现 OmniFlatten 反超 [[LLaMA-Omni]]（5.6 vs 4.17），因为 LLaMA-Omni dialogue learning 没用中文数据
7. 作者怀疑 [[GLM-4-Voice]] 训练数据可能泄露了本测试集

### Table 4: Turn-taking Accuracy & Response Time

| Model | Assistant Turn-taking Acc@1/5/10/25 (%) | Avg Assistant Response Time (ms) | User Turn-taking Acc@1/5/10/25 (%) | Avg User Response Time (ms) |
|---|---|---|---|---|
| [[Moshi]] | 2.9 / 18.8 / 38.5 / 55.1 | 553 | 0.0 / 6.2 / 14.8 / 45.7 | 753 |
| **OmniFlatten** | **20.6 / 53.6 / 66.3 / 71.7** | **193** | **10.9 / 30.9 / 41.8 / 51.8** | **287** |

**说明**: OmniFlatten 在两类 turn-taking 上**全面碾压** [[Moshi]]：
- Assistant turn-taking @1：20.6% vs 2.9%（用户问完后**第 1 个 token** 就答得对）
- 平均 assistant 响应延迟：193 ms（Moshi 553 ms，差 ~3 倍）
- 平均 user 打断响应延迟：287 ms（Moshi 753 ms）

但作者承认两个模型在 25 token 内的 user turn-taking 成功率都不太高，说明"准确识别用户打断意图"仍是公开问题。

### 评价指标定义

- **Assistant Turn-taking Acc@k**: 用户讲完后，模型是否在第 $k$ 个 token 时**正确预测非静音 token**（即开始说话）
- **User Turn-taking Acc@k**: 模型正在说话时，用户开始说话后，模型是否在第 $k$ 个 token **正确预测静音 token**（即停下来听）
- **阈值**: 1.5 秒未响应（开始/停止）算失败

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| Modality Alignment 混合集 | ~100K hours | 70% 私有 + 30% 开源（[[AISHELL]]-3 / [[LibriTTS]] / [[TED-LIUM]] / [[VoxPopuli]] / [[LibriSpeech]] / [[MLS]] / [[WenetSpeech]]\* 15%）；中英双语 | 阶段 1 SFT |
| Simulated Voice Chat | 2000 hours, 390K 多轮 session | [[CosyVoice]] 合成 + [[MUSAN]] 噪声叠加 + 时序模拟 | 阶段 2/3 SFT |
| 评测集 | – | LibriSpeech (ASR), Wenetspeech (ASR), LibriTTS (TTS), AIShell-3 (TTS), 1% 自建 voice chat test set | 评测 |

\* 仅用 Wenetspeech 训练集的 15% 高质量转录子集

### 实现细节

- **Backbone**: [[Qwen2-0.5B]]
- **优化器**: AdamW, β1=0.9, β2=0.95, weight decay 0.1
- **学习率**: max 2e-5, warm-up + cosine decay
- **Batch size**: 100M tokens / batch
- **训练轮数**: 5 epochs
- **Max sequence length**: alignment 1024 → dialogue 8192
- **Loss**: standard cross-entropy + user channel loss mask
- **硬件**: 论文未明确报告 GPU 数量

### 推理设置

测试时用 ground truth 的 user 通道 speech 作为固定输入，模型按 chunk 大小交替填充预测的 assistant speech 和 text，模拟真实流式推理。

---

## 批判性思考

### 优点

1. **优雅简洁**：纯 SFT + 数据组织技巧把文本 LLM 改成全双工语音模型，完全不动 backbone，降低工程门槛——和 [[Moshi]] 的多流定制设计形成鲜明对比
2. **Turn-taking 实测强**：193 ms 响应时延和 20.6% Acc@1 在公开评测里很有竞争力；这是用户真正会感知到的对话流畅度
3. **Chunk 设计有道理**：text=2 / speech=10 token 比例反映"文本信息密度高"的事实，避免文本超前于语音
4. **三阶段消融充分**：Table 3 把"直接 3-stream / 无 half-duplex / 全流程"三种配置都比较了，证明 curriculum 真的有用
5. **Speech tokenizer 选 [[CosyVoice]] 的语义 token 而非声学 token**：让 flatten 后的 speech-text 对齐更容易学

### 局限性

1. **小模型限制**：0.5B 参数下 chat 质量明显落后 7B-9B 的 [[LLaMA-Omni]] / [[GLM-4-Voice]]，论文也承认是 limitation
2. **Speech vs Text 的"模态税"**：Qwen2-0.5B-Instruct 6.75 → OmniFlatten 4.88，加 speech 模态后 chat 能力掉了 28%
3. **2-stream 性能崩塌**：理论上更接近真正全双工，但 chat 质量大幅下降；说明纯 speech-to-speech 还远没到位
4. **Backchannel 不支持**：作者明确承认无法处理 user/assistant 的"嗯""哦"等 backchannel，对话自然度还差一截
5. **Chunk size 是固定的**：不能根据语速/语义边界动态调，可能在某些场景出现"半个词"切分问题
6. **没报告 [[MOS]] / [[UTMOS]]**：声学质量没主观/客观评分，只是用 WER/CER 间接反映；可能 flatten 训练对声音自然度有损害
7. **2000 小时合成数据规模偏小**：[[Moshi]] 用了 7M+ 小时；数据规模差距是 chat 质量差距的重要原因
8. **数据合成里只有"打断/沉默/响应"三种交互**：缺真实人类对话里的犹豫、自我修正、笑声等
9. **公平对比有疑问**：与 [[Moshi]] 比 turn-taking 没说清楚 Moshi 是否在中文上微调过；GLM-4-Voice 数据可能泄露

### 潜在改进方向

1. **scale up backbone**：Qwen2-1.5B / 3B / 7B 同方法重复，看是否能追上 Moshi/GLM-4-Voice
2. **动态 chunk**：基于 VAD / 语义边界自适应切 chunk
3. **加 backchannel 数据**：在合成 pipeline 里注入"嗯/哦/对"等短反馈，扩充交互模式
4. **加 RLHF / Speech DPO**：现在只是 SFT，加偏好学习应能进一步提升 chat 质量
5. **扩展到 Vision**：作者在 conclusion 提到要做视觉全双工——本质上 flatten 范式可以无缝扩展到任意模态
6. **更大规模数据**：把合成数据从 2K 小时扩到 10K+ 小时

### 可复现性评估

- [ ] 代码开源 (only demo site, no public code at v2 release)
- [ ] 预训练模型
- [x] 训练细节完整 (optimizer / lr / batch / epoch / seq len 都给了)
- [x] 数据集可获取 (Modality Alignment 30% 是开源数据；Simulated 数据没开源但 pipeline 描述清楚可复现)

---

## 关联笔记

### 基于

- [[Qwen2-0.5B]]: 文本 LLM backbone
- [[CosyVoice]]: 提供 speech tokenizer 和 OT-CFM detokenizer
- [[OT-CFM]]: token → mel 的生成器
- [[HiFi-GAN]]: mel → waveform vocoder

### 对比 / 同期工作

- [[Moshi]]: 全双工标杆，多流并行；OmniFlatten 用 flatten 单流避免其复杂设计
- [[SyncLLM]]: 同样 chunk-based 全双工，但 dedup 损害音质且无 text 输出
- [[LLaMA-Omni]]: 半双工大参数（8B），中文弱
- [[GLM-4-Voice]]: 半双工大参数（9B），强但可能数据泄露
- [[Mini-Omni]] / [[Mini-Omni 2]]: 半双工，命令式打断
- [[SpeechGPT]] / [[LauraGPT]]: 早期端到端但半双工
- [[VITA]]: 双模块全双工方案
- [[dGSLM]]: 早期端到端全双工
- [[LSLM]]: TTS+listener 实现 listen-while-speaking
- [[SALMONN]] / [[Qwen-Audio]]: 只支持 speech-in / text-out

### 方法相关

- [[Flatten 操作]]: 核心创新
- [[Modality Alignment]]: 阶段 1
- [[Half-duplex Dialogue Training]]: 阶段 2
- [[Full-duplex Dialogue Training]]: 阶段 3
- [[Curriculum Learning]]: 整体训练范式哲学
- [[Turn-taking]]: 全双工核心评价维度
- [[Barge-in]]: 用户打断
- [[Speech Tokenizer]]: 语义 token 化

### 数据相关

- [[LibriSpeech]] / [[LibriTTS]] / [[AISHELL]] / [[WenetSpeech]] / [[MLS]] / [[TED-LIUM]] / [[VoxPopuli]]: ASR/TTS 训练
- [[Alpaca]] / [[Moss]] / [[BelleCN]] / [[UltraChat]]: 文本对话源
- [[MUSAN]]: 噪声数据
- [[3D-Speaker]]: 音色采样

---

## 速查卡片

> [!summary] OmniFlatten (Alibaba Tongyi, Oct 2024)
> - **核心**: 把 user/assistant × speech/text 四流按 chunk **flatten** 成单序列，复用 [[Qwen2-0.5B]] GPT 自回归即可全双工
> - **训练**: 三阶段 SFT — modality alignment（ASR+TTS）→ half-duplex → full-duplex (3-stream → 2-stream)
> - **关键参数**: speech chunk=10 token, text chunk=2 token, speech tokenizer 用 [[CosyVoice]] 单码本 4096 codes
> - **结果**: Turn-taking 全面优于 [[Moshi]]（193 ms vs 553 ms 响应时延，Acc@1 20.6% vs 2.9%），但 0.5B 小模型 chat 质量落后 [[LLaMA-Omni]] / [[GLM-4-Voice]]
> - **创新点**: 不改 GPT 架构、不依赖大规模预训练；flatten 范式可扩展到任意模态
> - **代码**: 项目主页 https://omniflatten.github.io/（v2 论文未释出代码）

---

*笔记创建时间: 2026-05-19*
