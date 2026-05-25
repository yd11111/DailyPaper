---
title: "AudioLM: a Language Modeling Approach to Audio Generation"
method_name: "AudioLM"
authors: [Zalán Borsos, Raphaël Marinier, Damien Vincent, Eugene Kharitonov, Olivier Pietquin, Matt Sharifi, Dominik Roblek, Olivier Teboul, David Grangier, Marco Tagliasacchi, Neil Zeghidour]
year: 2023
venue: TMLR
tags: [speech-llm, audio-generation, semantic-token, acoustic-token, hierarchical-lm, language-model, self-supervised, codec]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2209.03143
created: 2026-05-25
---

# 论文笔记：AudioLM: a Language Modeling Approach to Audio Generation

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Google Research |
| 日期 | Sep 2022 (TMLR 2023) |
| 项目主页 | [Demo Samples](https://google-research.github.io/seanet/audiolm/examples) |
| 对比基线 | [[GSLM]], [[Perceiver AR]], [[Jukebox]] |
| 链接 | [arXiv](https://arxiv.org/abs/2209.03143) |

---

## 一句话总结

> 首次将音频生成定义为语言建模问题，通过 [[Semantic Token]] + [[Acoustic Token]] 分层建模实现语义连贯且高保真的语音/音乐续写，无需任何文本标注。

---

## 核心贡献

1. **语义-声学分层 token 框架**: 提出将 [[w2v-BERT]] 的 [[Semantic Token]] 与 [[SoundStream]] 的 [[Acoustic Token]] 结合的混合 tokenization 方案，前者保证长程语义连贯，后者保证高保真音质
2. **三阶段分层语言模型**: 设计 semantic → coarse acoustic → fine acoustic 的级联 [[Autoregressive]] 建模范式，兼顾长程结构与声学细节
3. **无文本标注的语音生成**: 仅用纯音频训练，在 3 秒 prompt 下生成语法/语义连贯的语音续写，人类评估者无法与真实语音区分（51.2% 正确率，与随机猜测无显著差异）
4. **跨领域泛化**: 同一框架在钢琴音乐续写上也有效，主观评测中 83.3% 偏好率优于纯声学 token 建模
5. **生成检测**: 简单分类器即可以 98.6% 准确率检测 AudioLM 生成内容，为安全评估提供基线

---

## 问题背景

### 要解决的问题
如何在不依赖任何文本/符号标注的情况下，从短音频片段自回归生成既语义连贯又音质高保真的音频续写。

### 现有方法的局限
- **WaveNet** 等强自回归波形模型在无文本条件时生成不连贯的"babbling"
- **Textless NLP / GSLM**: 用 [[HuBERT]] 的离散 token 做自回归，语义连贯但音质差（单说话人、干净语音限制），因为低码率 semantic token 丢失了声学细节
- **Jukebox**: 分层 token 生成音乐，时域连贯但有明显伪影
- **Perceiver AR**: 在高码率 [[SoundStream]] token 上建模钢琴音乐，音质好但时域结构弱

核心矛盾：**语义信息（长程结构）与声学信息（音质细节）天然分布在不同尺度的表示中**，单一 token 类型无法兼顾。

### 本文的动机
将 semantic token（捕捉语言/音乐内容、天然低码率、高语音判别力但不可逆重建）与 acoustic token（捕捉声学细节、可高质量重建但语音判别力差）在分层模型中互补组合。

---

## 方法详解

### 模型架构

AudioLM 采用 **三阶段分层自回归** 架构：

- **输入**: 单通道音频 $x \in \mathbb{R}^T$（16 kHz 采样率）
- **Tokenizer 1 — Semantic**: [[w2v-BERT]] XL (0.6B) → 第 7 层 MLM embeddings → k-means ($K=1024$) → 25 Hz [[Semantic Token]] 序列 $z$
- **Tokenizer 2 — Acoustic**: [[SoundStream]] encoder → 50 Hz embeddings → 12 层 [[RVQ]] ($N=1024$) → [[Acoustic Token]] 矩阵 $Y \in \{1,...,N\}^{T_A \times Q}$
- **Language Model**: 三个独立的 decoder-only [[Transformer]]，各 0.3B 参数
- **Detokenizer**: [[SoundStream]] decoder，从 acoustic token 重建波形

### 核心模块

#### 模块 1: Semantic Tokenizer (w2v-BERT)

**设计动机**: 利用 [[SSL Speech Representation]] 的高层语义抽象能力，提取内容/语言层面的离散表示

**具体实现**:
- 使用 [[w2v-BERT]] XL 模型（0.6B 参数，[[Conformer]]-based），同时以 contrastive loss 和 MLM loss 训练
- 取 MLM 模块第 7 层的 1024 维 embeddings（通过 ABX / sWUGGY / sBLIMP 选层）
- Embeddings 做零均值单位方差归一化后 k-means 聚类（$K=1024$），centroid 索引即为 semantic token
- 帧率 25 Hz（每 40 ms 一个 token），码率 250 bps
- 连续重复的 semantic token 做去重处理

**关键性质**: 语音判别力强（ABX within-speaker 6.7%），但重建质量极低（ViSQOL 1.1）——只承载"说什么"而非"怎么说"。

#### 模块 2: Acoustic Tokenizer (SoundStream)

**设计动机**: 利用 [[Audio Codec]] 的高保真重建能力，捕捉说话人身份、韵律、录音环境等声学细节

**具体实现**:
- [[SoundStream]] 编码器：4 个卷积块，strides (2, 4, 5, 8) → 320 倍下采样 → 50 Hz embeddings
- [[RVQ]] 量化：12 层残差向量量化器，每层码本大小 $N=1024$
- 前 $Q'=4$ 层为 coarse tokens（码率 2000 bps），后 8 层为 fine tokens（额外 4000 bps），总码率 6000 bps
- 端到端训练含重建损失 + 对抗损失

**关键性质**: 重建质量高（ViSQOL 3.9 @6000 bps），但语音判别力差（ABX within-speaker 22.4%）——完整保留"怎么说"但不直接编码"说什么"。

#### 模块 3: 三阶段分层语言模型

**Stage 1 — Semantic Modeling**:
- 建模 $p(z_t \mid z_{<t})$：纯 semantic token 的自回归生成
- 负责长程语义/语法结构
- 训练时 random crop 30s 音频

**Stage 2 — Coarse Acoustic Modeling**:
- 以全部 semantic token 为条件，生成 coarse acoustic token（前 $Q'=4$ 层 [[RVQ]]）
- Acoustic token 矩阵做 row-major flatten + offset 编码
- 训练时 random crop 10s 音频

**Stage 3 — Fine Acoustic Modeling**:
- 以 coarse acoustic token 为条件，生成 fine acoustic token（后 8 层 RVQ）
- 不再需要 semantic token 条件（条件独立假设）
- 在不重叠的 3s chunk 上独立运行，可批处理

---

## 关键公式

### 公式 1: [[Autoregressive|语言模型目标]]

$$
\max \prod_{t=1}^{T'} p(h_t \mid h_{<t})
$$

**含义**: 对离散 token 序列做标准自回归极大似然训练

**符号说明**:
- $h = (h_1, \dots, h_{T'})$: 离散 token 序列
- $T'$: token 序列长度（比原始波形 $T$ 短 2-3 个数量级）
- $p(h_t \mid h_{<t})$: 由 decoder-only Transformer 参数化的条件分布

### 公式 2: [[Semantic Token|语义条件独立假设]]

$$
p(z_t \mid z_{<t}, y_{<t}) \approx p(z_t \mid z_{<t})
$$

**含义**: Semantic token 的生成不依赖 acoustic token，这一假设使得 Stage 1 可以独立建模 semantic 序列

**符号说明**:
- $z_t$: 第 $t$ 步的 semantic token
- $y_{<t}$: 之前所有 acoustic token

### 公式 3: [[RVQ|Acoustic Token Flattening]]

$$
y = (y_1^1, y_1^2, \dots, y_1^Q, y_2^1, \dots, y_{T_A}^Q)
$$

$$
o_i = \bigl((i-1) \bmod Q\bigr) \cdot N
$$

**含义**: 将 $T_A \times Q$ 的 acoustic token 矩阵按 row-major 顺序展平为一维序列，并通过 offset 向量为不同量化层创建不重叠的 token 索引空间

**符号说明**:
- $y_t^q$: 时间步 $t$、第 $q$ 层 RVQ 的 token
- $Q$: RVQ 总层数（12）
- $N$: 每层码本大小（1024）
- $o_i$: 第 $i$ 个位置的 offset

### 公式 4: [[Autoregressive|Stage 2 条件分布]]

$$
p(y_t^q \mid z, \, y_{<t}^{\leq Q'}, \, y_t^{<q}) \quad \text{for } q \leq Q'
$$

**含义**: Coarse acoustic token 的生成以全部 semantic token $z$ 和已生成的 acoustic token 为条件

**符号说明**:
- $Q' = 4$: coarse 层数
- $z$: 完整 semantic token 序列
- $y_{<t}^{\leq Q'}$: 之前时间步的 coarse acoustic tokens
- $y_t^{<q}$: 当前时间步已生成的更粗层 tokens

### 公式 5: [[Autoregressive|Stage 3 条件分布]]

$$
p(y_t^q \mid y^{\leq Q'}, \, y_{<t}^{>Q'}, \, y_t^{<q}) \quad \text{for } q > Q'
$$

**含义**: Fine acoustic token 的生成仅以 coarse acoustic token 为条件，不再需要 semantic token（条件独立假设）

### 公式 6: [[RVQ|码率计算]]

$$
\text{bitrate} = f_s \times Q \times \log_2 N
$$

**含义**: 计算 acoustic token 的总码率

**符号说明**:
- $f_s = 50$ Hz: SoundStream 帧率
- $Q$: RVQ 层数
- $N = 1024$: 码本大小
- 示例: $50 \times 12 \times 10 = 6000$ bps

---

## 关键图表

### Figure 1: Tokenizers Overview / 两种 Tokenizer 对比

![Figure 1](https://ar5iv.labs.arxiv.org/html/2209.03143/assets/x1.png)

**说明**: AudioLM 使用两种互补的 tokenizer。上方：[[SoundStream]] 通过 [[RVQ]] 将波形压缩为 acoustic token（高重建质量、低语音判别力）。下方：[[w2v-BERT]] 通过 k-means 将 SSL 表示离散化为 semantic token（高语音判别力、低重建质量）。两者互补是 AudioLM 的核心洞察。

### Figure 2: Three-Stage Hierarchical Modeling / 三阶段分层建模

![Figure 2](https://ar5iv.labs.arxiv.org/html/2209.03143/assets/x2.png)

**说明**: AudioLM 的三阶段推理流程。Stage 1 自回归生成 semantic token 序列（保证语义连贯）；Stage 2 以 semantic token 为条件生成 coarse acoustic token（恢复说话人身份和韵律）；Stage 3 以 coarse acoustic token 为条件生成 fine acoustic token（补充声学细节）。最终由 [[SoundStream]] decoder 重建波形。

### Figure 3(a): w2v-BERT Layer Selection / ABX 分数随层变化

![Figure 3(a)](https://ar5iv.labs.arxiv.org/html/2209.03143/assets/x3.png)

**说明**: [[w2v-BERT]] MLM 模块不同层的 ABX 语音判别力分数。第 7 层在 within-speaker 和 across-speaker 两个设置下都取得最优 ABX 分数，因此被选为 semantic token 的提取层。

### Figure 3(b): k-means Cluster Number / 聚类数对语言学评测的影响

![Figure 3(b)](https://ar5iv.labs.arxiv.org/html/2209.03143/assets/x4.png)

**说明**: 固定 w2v-BERT 第 7 层，改变 k-means 聚类数 $K$ 对 sWUGGY（词汇判别）和 sBLIMP（语法判别）的影响。$K=1024$ 时两项指标均达到较优水平。

### Table I: Semantic vs Acoustic Token 对比

| Tokenization | Bitrate (bps) | ABX within/across (%, ↓) | ViSQOL (↑) |
|---|---|---|---|
| Semantic (w2v-BERT) | 250 | 6.7 / 7.6 | 1.1 |
| Semantic (w2v-BERT) | 6000 | 5.6 / 6.2 | 1.4 |
| Acoustic (SoundStream) | 2000 | 22.4 / 28.7 | 3.3 |
| Acoustic (SoundStream) | 6000 | 17.8 / 26.6 | 3.9 |

**表格说明**: 核心对比。即使在匹配码率（6000 bps）下，semantic token 的 ABX 远优于 acoustic token（5.6 vs 17.8），而 acoustic token 的 ViSQOL 远优于 semantic token（3.9 vs 1.4）。两者特性互补，验证了分层建模的动机。

### Table II: ASR Error Rates / 语义内容保留验证

| | Original | Recon. w/ SoundStream | AudioLM (acoustic gen.) | GSLM unit-to-speech |
|---|---|---|---|---|
| [[CER]] (%) | 0.8 | 0.9 | 3.4 | 2.9 |
| [[WER]] (%) | 2.5 | 2.6 | 6.0 | 6.6 |

**表格说明**: 从 ground-truth semantic token 出发做 acoustic generation，再用 [[Conformer]] Transducer-L 做 ASR。AudioLM WER 6.0% 与 [[GSLM]] 的 6.6% 相当，但 AudioLM 可处理多说话人+噪声环境，GSLM 仅限单说话人干净语音。误差主要来自专有名词和句末 token 对齐。

### Table III: Speaker Classification Accuracy / 说话人信息分布

| Setting | Accuracy (%) |
|---|---|
| Recon. w/ SoundStream | 100.0 |
| Acoustic generation w/ AudioLM | 3.2 |
| Continuation w/ AudioLM | 92.6 |

**表格说明**: 关键实验。Acoustic generation（仅从 semantic token 出发）说话人分类仅 3.2%（接近随机 0.3%），证明 semantic token 几乎不携带说话人信息。而 continuation 模式（prompt 包含 acoustic token）分类准确率高达 92.6%，证明说话人身份主要由 acoustic token 决定。

### Table IV: Zero-Shot Linguistic Evaluation / 零资源语言学评测

| Model | sWUGGY all | sWUGGY in-vocab | sBLIMP |
|---|---|---|---|
| **Text-based toplines** | | | |
| Forced alignment topline | 92.2 | — | 63.7 |
| Phone topline | 97.9 | — | 66.8 |
| **Non-causal** | | | |
| BERT baseline | 67.7 | 75.6 | 56.1 |
| HuBERT-only | 70.9 | 79.8 | 59.5 |
| CPC-BERT | — | 80.0 | 59.9 |
| **Causal** | | | |
| GSLM | — | 68.7 | 57.1 |
| **AudioLM** | **71.5** | **83.7** | **64.7** |

**表格说明**: AudioLM 在 ZeroResource Challenge 2021 的两项零资源语言学评测上全面领先。sWUGGY in-vocab 83.7% 超越 CPC-BERT（80.0%）3.7 个点。sBLIMP 64.7% 甚至超越了有监督的 forced alignment topline（63.7%）。这表明 AudioLM 的分层建模确实学到了深层的词汇和语法知识。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Libri-Light]] unlab-60k | 60k 小时 | 英文有声书，包含噪声环境 | 语音全流程训练 |
| [[LibriSpeech]] test-clean | 2.2 小时 | 干净英文评测集 | 语音评测 |
| 内部钢琴数据集 | 40k 小时 | 初学者到专家级，各种声学条件 | 钢琴训练 |
| [[Maestro]] | — | 专业钢琴演奏 | 钢琴评测 prompt |

### 实现细节

- **Semantic Tokenizer**: [[w2v-BERT]] XL, 0.6B 参数, [[Conformer]]-based
- **Acoustic Tokenizer**: [[SoundStream]], 4 conv blocks, strides (2,4,5,8), 12 层 [[RVQ]], $N=1024$
- **LM (3 个 stage 共用架构)**: Decoder-only [[Transformer]], 12 layers, 16 heads, $d=1024$, FFN $d_{ff}=4096$, dropout 0.1, T5-style relative positional embeddings, **0.3B 参数/stage**
- **训练长度**: Stage 1 = 30s, Stage 2 = 10s, Stage 3 = 3s (random crop)
- **硬件**: 16 TPUv4, batch size 256, 1M steps/stage
- **推理温度**: Stage 1 = 0.6, Stage 2 = 0.8, Stage 3 = 0.6
- **Prompt 时长**: 语音 3s, 钢琴 4s
- **Semantic token 去重**: Stage 1-2 训练/推理时移除连续重复 token

### 主观评测结果

- **语音续写 human evaluation**: 100 个 10s 样本（50 真实 + 50 AudioLM 续写），10 名评估者。正确分类率 **51.2%**，二项检验 $p=0.23$，与 50% 随机猜测无统计显著差异。人类无法分辨 AudioLM 语音与真实语音
- **钢琴续写偏好**: 15 对 20s 样本比较（acoustic-only vs full AudioLM），10 名评估者，**83.3%** 偏好完整 AudioLM
- **生成检测**: 简单 CNN 分类器达到 **98.6%** 检测准确率（对比的是 SoundStream 压缩后的真实语音，排除压缩伪影混淆）

---

## 批判性思考

### 优点
1. **范式开创性**: 首次将"音频生成 = 语言建模"形式化，semantic + acoustic 分层思路被 [[VALL-E]] / [[SoundStorm]] / [[MusicLM]] 等后续工作直接继承，成为 [[Speech LLM]] 的标准范式
2. **无文本依赖**: 完全无需转录、phoneme 标注或任何文本对齐，纯音频训练即可生成语义连贯的语音
3. **实验设计扎实**: 通过 acoustic generation（隔离 semantic token 贡献）、speaker classifier（量化信息分布）、linguistic probing（sWUGGY/sBLIMP）等精巧实验，严谨验证了设计假设
4. **跨领域泛化**: 同一框架在语音和钢琴音乐上都有效，证明 semantic-acoustic 解耦是通用原则

### 局限性
1. **三阶段串行推理延迟高**: 三个 0.3B Transformer 顺序执行，不适合实时/流式场景；后续 [[SoundStorm]] 用 MaskGIT 并行化解决了这个问题
2. **无条件控制能力**: 只做 audio continuation，不支持 text-conditioned generation（text-to-speech），限制了实用性
3. **仅 16 kHz**: 采样率偏低，不适合高保真音频应用
4. **未开源**: 代码和模型未公开，可复现性受限
5. **评测有限**: 语音主观评测仅 50 个续写样本，且仅英文；钢琴评测使用内部数据集，无法外部复现

### 潜在改进方向
1. 用 [[Flow Matching]] / non-autoregressive 方法替代 Stage 2-3，减少推理延迟
2. 引入文本条件实现 TTS（后续 [[VALL-E]] / [[SpearTTS]] 走了这条路）
3. 统一 semantic 和 acoustic tokenizer（后续 [[SpeechTokenizer]] / [[X-Codec]] 尝试了这个方向）

### 可复现性评估
- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（架构、超参、数据量均详细报告）
- [x] 数据集可获取（Libri-Light 公开，钢琴为内部数据）

---

## 关联笔记

### 基于
- [[w2v-BERT]]: 提供 semantic tokenizer
- [[SoundStream]]: 提供 acoustic tokenizer 和 decoder
- [[GSLM]]: textless NLP 先驱，AudioLM 在此基础上增加声学建模

### 后续
- [[VALL-E]]: 继承 semantic + acoustic 分层，加入文本条件做 zero-shot TTS
- [[SoundStorm]]: 用 MaskGIT 并行化 AudioLM 的 acoustic stage
- [[MusicLM]]: 将 AudioLM 扩展到文本条件音乐生成
- [[SpearTTS]]: Google 基于 AudioLM 的 TTS 系统

### 方法相关
- [[Semantic Token]]: 核心组件——承载语义/语言内容
- [[Acoustic Token]]: 核心组件——承载声学细节
- [[RVQ]]: SoundStream 的量化策略，coarse/fine 分层是 AudioLM 三阶段的基础
- [[Transformer]]: 三个 stage 共用的 LM 架构
- [[Autoregressive]]: 所有三个 stage 的生成范式

### 硬件/数据相关
- [[Libri-Light]]: 60k 小时语音训练集
- [[LibriSpeech]]: 评测集

---

## 速查卡片

> [!summary] AudioLM: a Language Modeling Approach to Audio Generation
> - **核心**: 将音频生成视为离散 token 的语言建模，semantic + acoustic 分层解耦长程语义与声学细节
> - **方法**: w2v-BERT semantic token (25 Hz, 250 bps) + SoundStream acoustic token (50 Hz, 6000 bps) + 三阶段 decoder-only Transformer (0.3B x 3)
> - **结果**: 人类无法区分 3s prompt 的续写与真实语音（51.2% 正确率 ~ 随机）；sBLIMP 64.7% 超越有监督 topline
> - **代码**: 未开源

---

*笔记创建时间: 2026-05-25*
