---
title: "Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers"
method_name: "VALL-E"
authors: [Chengyi Wang, Sanyuan Chen, Yu Wu, Ziqiang Zhang, Long Zhou, Shujie Liu, Zhuo Chen, Yanqing Liu, Huaming Wang, Jinyu Li, Lei He, Sheng Zhao, Furu Wei]
year: 2023
venue: arXiv
tags: [zero-shot-tts, codec-language-model, in-context-learning, autoregressive, rvq, large-scale-training]
image_source: online
arxiv_id: "2301.02111"
created: 2026-05-25
---

# 论文笔记：Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft |
| 日期 | January 2023 |
| 项目主页 | https://aka.ms/valle |
| 对比基线 | [[YourTTS]], [[AudioLM]], [[GSLM]] |
| 链接 | [arXiv](https://arxiv.org/abs/2301.02111) / [Code](https://github.com/microsoft/unilm) |

---

## 一句话总结

> 首次将 TTS 重定义为条件 codec 语言建模任务，用 AR+NAR transformer 在 [[EnCodec]] 离散码上建模，仅需 3 秒 prompt 实现 zero-shot 语音合成，开创了 codec LM TTS 范式。

---

## 核心贡献

1. **范式转换 -- TTS as Conditional Language Modeling**: 将中间表示从 [[Mel-Spectrogram]] 换为 [[Discrete Audio Token]]（[[EnCodec]] [[RVQ]] 码），将回归任务转为语言建模任务
2. **大规模半监督训练**: 使用 60K 小时 LibriLight 数据（比以往系统大数百倍），首次证明 TTS 也能从 scaling data 中获益
3. **In-Context Learning 能力**: 类似 GPT-3 的 prompt 机制，3 秒 enrolled recording 即可 zero-shot 复刻音色、情感、声学环境，无需 fine-tune

---

## 问题背景

### 要解决的问题

传统 TTS 系统（[[Tacotron 2]]、[[FastSpeech 2]]）依赖高质量录音棚数据，面对未见说话人时音色相似度和自然度急剧下降，难以泛化到 open-domain 零样本场景。

### 现有方法的局限

- 中间表示用 [[Mel-Spectrogram]]，需要连续信号回归，一对一映射导致多样性不足
- 训练数据规模小（通常 <= 600 小时），说话人覆盖有限
- Speaker adaptation 方法（fine-tune / speaker embedding）对未见说话人泛化差
- 基于 speaker encoder 的方法虽能做 zero-shot，但合成质量在域外显著退化

### 本文的动机

NLP 领域 GPT-3 等大规模语言模型展示了强大的 [[In-Context Learning]] 能力。如果将语音也离散化为 token，就可以借鉴语言建模范式——用大规模数据训练 codec language model，让 TTS 也具备 in-context learning 能力，仅靠 prompt 即可泛化到未见说话人。

---

## 方法详解

### 模型架构

VALL-E 采用 **hierarchical conditional codec language model** 架构：

- **输入**: [[Phoneme]] 序列 $\mathbf{x} = \{x_0, x_1, \dots, x_L\}$ + 3 秒 acoustic prompt $\tilde{\mathbf{C}}$
- **Tokenizer**: [[EnCodec]] 将 24 kHz 波形编码为 75 Hz 的 8 层 [[RVQ]] 离散码矩阵 $\mathbf{C}^{T \times 8}$
- **AR Model**: Decoder-only [[Transformer]]，自回归预测第 1 层 codec token（决定韵律和内容）
- **NAR Model**: [[Transformer]]，非自回归并行预测第 2-8 层 codec token（补充音质细节）
- **Decoder**: [[EnCodec]] 解码器从 8 层 token 重建波形
- **总参数**: AR 和 NAR 各 12 层 transformer，embedding dim 1024，FFN dim 4096

### 核心模块

#### 模块 1: Autoregressive Codec Language Model（AR 模型）

**设计动机**: 第 1 层 [[RVQ]] 码对重建质量贡献最大，承载了语义和韵律的主要信息，适合用 [[Autoregressive Model|自回归]] 方式逐帧生成以保证连贯性。

**具体实现**:
- 使用 decoder-only [[Transformer]]，12 层，16 头注意力
- 输入为 phoneme embedding $W_x$ 和 acoustic embedding $W_a$ 的拼接
- Phoneme 序列和第 1 层 codec 序列各自追加 `<EOS>` token
- 位置编码：phoneme 和 acoustic prompt 分别独立计算 sinusoidal position embedding
- 输出层与 acoustic embedding $W_a$ 共享参数（weight tying）
- 训练时，任意前缀 $\mathbf{c}_{<t,1}$ 天然作为后续部分的 prompt（prefix LM 训练）
- 推理时，enrolled recording 的 phoneme 转录拼在目标 phoneme 前，其 acoustic token 作为 AR 解码的前缀

#### 模块 2: Non-Autoregressive Codec Language Model（NAR 模型）

**设计动机**: 第 2-8 层 [[RVQ]] 码补充声学细节（音质、高频），各层之间相对独立且信息量递减，适合 NAR 并行生成以加速推理。

**具体实现**:
- 同样 12 层 [[Transformer]]，但为非自回归结构
- 包含 8 套独立的 acoustic embedding 层 $W_a^1, \dots, W_a^8$
- 训练时随机采样 stage $i \in [2, 8]$，以前 $i-1$ 层为条件预测第 $i$ 层
- 当前帧的条件 embedding 为前 $i-1$ 层 embedding 之和
- Acoustic prompt 的 embedding 固定为全 8 层之和（提供完整说话人信息）
- 使用 [[Adaptive Layer Normalization]]（AdaLN）注入 stage 信息：$\mathrm{AdaLN}(h, i) = a_i \cdot \mathrm{LayerNorm}(h) + b_i$
- 推理时调用 7 次（stage 2 到 8），每次 greedy decoding

---

## 关键公式

### 公式 1: [[Autoregressive Model|AR Codec Language Model]]

$$
p(\mathbf{c}_{:,1} \mid \mathbf{x}, \tilde{\mathbf{C}}_{:,1}; \theta_{\mathrm{AR}}) = \prod_{t=0}^{T} p(c_{t,1} \mid \mathbf{c}_{<t,1}, \tilde{\mathbf{c}}_{:,1}, \mathbf{x}; \theta_{\mathrm{AR}})
$$

**含义**: 自回归生成第 1 层 codec token 序列，每一帧条件于所有前帧、acoustic prompt 的第 1 层码、和 phoneme 序列。

**符号说明**:
- $\mathbf{c}_{:,1}$: 第 1 层 codebook 的所有帧 token 序列
- $\mathbf{c}_{<t,1}$: 前 $t$ 帧的第 1 层 token（自回归上文）
- $\tilde{\mathbf{c}}_{:,1}$: 3 秒 acoustic prompt 的第 1 层 token
- $\mathbf{x}$: phoneme 序列
- $\theta_{\mathrm{AR}}$: AR 模型参数

### 公式 2: [[RVQ|NAR Codec Language Model]]

$$
p(\mathbf{C}_{:,2:8} \mid \mathbf{x}, \tilde{\mathbf{C}}; \theta_{\mathrm{NAR}}) = \prod_{j=2}^{8} p(\mathbf{c}_{:,j} \mid \mathbf{C}_{:,<j}, \mathbf{x}, \tilde{\mathbf{C}}; \theta_{\mathrm{NAR}})
$$

**含义**: 非自回归生成第 2-8 层 codec token，每层条件于所有前层、phoneme 和完整 acoustic prompt。

**符号说明**:
- $\mathbf{C}_{:,2:8}$: 第 2 到 8 层 codebook 的 token 矩阵
- $\mathbf{C}_{:,<j}$: 前 $j-1$ 层所有 token
- $\tilde{\mathbf{C}}$: acoustic prompt 的完整 8 层 token 矩阵

### 公式 3: 联合预测

$$
p(\mathbf{C} \mid \mathbf{x}, \tilde{\mathbf{C}}; \theta) = p(\mathbf{c}_{:,1} \mid \tilde{\mathbf{C}}_{:,1}, \mathbf{x}; \theta_{\mathrm{AR}}) \cdot \prod_{j=2}^{8} p(\mathbf{c}_{:,j} \mid \mathbf{c}_{:,<j}, \mathbf{x}, \tilde{\mathbf{C}}; \theta_{\mathrm{NAR}})
$$

**含义**: VALL-E 的整体建模分解——AR 生成第 1 层，NAR 依次生成第 2-8 层，两阶段级联。

### 公式 4-5: [[RVQ|Acoustic Embedding 计算]]

$$
e_{c_{t,j}} = W_a^j \odot c_{t,j}
$$

$$
\mathbf{e}_{c_t} = \sum_{j=1}^{i-1} e_{c_{t,j}}
$$

**含义**: 在 NAR 模型的 stage $i$ 训练/推理时，将前 $i-1$ 层的 acoustic embedding 逐层求和作为当前帧的条件输入。

**符号说明**:
- $W_a^j$: 第 $j$ 层 codebook 的 embedding 矩阵
- $c_{t,j}$: 第 $t$ 帧第 $j$ 层的 token
- $\odot$: embedding lookup 操作

### Acoustic Prompt Embedding

$$
\mathbf{e}_{\tilde{c}_t} = \sum_{j=1}^{8} e_{\tilde{c}_{t,j}}
$$

**含义**: Acoustic prompt 的 embedding 固定使用全 8 层之和，保证提供完整的说话人身份信息。

---

## 关键图表

### Figure 1: Overview / 系统概览

![Figure 1: Overview of VALL-E](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/prompt.jpg)

**说明**: VALL-E 的整体流程。与传统 TTS 的 phoneme -> [[Mel-Spectrogram]] -> waveform 路径不同，VALL-E 采用 phoneme -> [[Discrete Audio Token|discrete codec code]] -> waveform。给定目标文本的 phoneme 和 3 秒 acoustic prompt，VALL-E 条件生成 [[EnCodec]] 离散码，再用 codec decoder 重建波形。该框架还可与 GPT-3 等文本模型结合，实现语音内容创作和语音编辑。

### Figure 2: Neural Audio Codec Model / 神经音频编解码器

![Figure 2: Neural audio codec model revisit](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/codec.jpg)

**说明**: [[EnCodec]] 的 [[RVQ]] 结构示意。由于采用残差量化，第 1 个 quantizer 对重建质量贡献最大，后续 quantizer 的贡献递减。这一特性是 VALL-E 分层建模（AR 处理第 1 层、NAR 处理后续层）的设计基础。

### Figure 3: Conditional Codec Language Modeling / 条件编解码语言建模结构

![Figure 3: Structure of conditional codec language modeling](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/x1.png)

**说明**: 分层建模架构的详细结构。左侧为 AR 模型，输入 phoneme 和 acoustic prompt 的第 1 层 token，自回归预测目标第 1 层 token；右侧为 NAR 模型，以前层 token 和完整 prompt 为条件，非自回归预测后续层 token。NAR 实际调用 7 次（stage 2 到 8）。

### Figure 4: Diversity Analysis / 多样性分析

**(a) LibriSpeech 样本**

![Figure 4a: Diversity - LibriSpeech](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/x2.png)

**说明**: 同一文本 "After early nightfall, the yellow lamp would light up here and there the squalid quarter of the brothels" 使用不同随机种子合成两次，得到不同的语速和短语时长分布。Sample 1 语速更快。

**(b) VCTK 样本**

![Figure 4b: Diversity - VCTK](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/x3.png)

**说明**: 同一文本 "I must do something about it" 合成两次，产生不同口音风格。Sample 2 在 "must" 上有更大振幅强调。这展示了 sampling-based 解码带来的多样性优势——传统 TTS 的确定性解码无法实现。

### Table 1: VALL-E vs 传统 TTS 系统对比

| 方面 | 传统 TTS 系统 | VALL-E |
|------|-------------|--------|
| 中间表示 | [[Mel-Spectrogram]] | [[Discrete Audio Token\|audio codec code]] |
| 目标函数 | 连续信号回归 | 语言模型 |
| 训练数据 | <= 600 小时 | 60K 小时 |
| In-context learning | 不支持 | 支持 |

**说明**: 范式层面的根本差异。VALL-E 在中间表示、目标函数、数据规模、泛化能力四个维度都实现了突破。

### Table 2: LibriSpeech 客观评测

| Model | [[WER]] (%) ↓ | SPK Similarity ↑ |
|-------|------|-----|
| GroundTruth | 2.2 | 0.754 |
| [[GSLM]] | 12.4 | 0.126 |
| [[AudioLM]]* | 6.0 | - |
| [[YourTTS]] | 7.7 | 0.337 |
| **VALL-E** | **5.9** | **0.580** |
| VALL-E-continual | 3.8 | 0.508 |

**说明**: VALL-E 在 [[WER]] 上显著优于 [[YourTTS]]（5.9% vs 7.7%），说话人相似度从 0.337 提升到 0.580。VALL-E-continual 因前 3 秒来自真实音频，[[WER]] 进一步降至 3.8%。Speaker similarity 用 [[WavLM]]-TDNN 模型计算。

### Table 3: LibriSpeech 主观评测

| Model | [[SMOS]] ↑ | [[CMOS]] (vs. VALL-E) |
|-------|------|------|
| [[YourTTS]] | 3.45 +- 0.09 | -0.12 |
| **VALL-E** | **4.38 +- 0.10** | 0.00 |
| GroundTruth | 4.50 +- 0.10 | +0.17 |

**说明**: VALL-E 在说话人相似度 [[SMOS]] 上大幅领先 [[YourTTS]] +0.93 分（4.38 vs 3.45），接近真实录音的 4.50。自然度 [[CMOS]] 与 ground truth 仅差 0.17，差距极小。12 名英语母语评测员参与。

### Table 4: NAR 模型消融实验

| 配置 | [[WER]] (%) ↓ | SPK ↑ | 说明 |
|------|------|-----|------|
| NAR-no prompt | 19.6 | 0.518 | 无任何 prompt |
| NAR-phn prompt | 3.0 | 0.541 | 仅 phoneme prompt |
| NAR-2 prompts | 2.8 | 0.732 | phoneme + acoustic prompt |

**关键发现**: Phoneme prompt 让 [[WER]] 从 19.6% 暴降至 3.0%（减少 84.7%），说明 phoneme 信息对保证内容准确性至关重要。Acoustic prompt 进一步将 speaker similarity 从 0.541 提升到 0.732（+35.3%），证明 acoustic prompt 是说话人身份信息的关键载体。

### Table 5: AR 模型消融实验

| 配置 | [[WER]] (%) ↓ | SPK ↑ | 说明 |
|------|------|-----|------|
| VALL-E | 5.9 | 0.585 | 完整模型 |
| w/o acoustic prompt | 5.9 | 0.236 | 移除 acoustic prompt |

**关键发现**: 移除 acoustic prompt 后 [[WER]] 不变（5.9%），但 speaker similarity 从 0.585 暴跌到 0.236（-59.7%）。说明 acoustic prompt 对 AR 模型的语义生成影响不大，但对说话人身份保持极为关键。

### Table 6: VCTK 说话人相似度（自动评测）

| 配置 | 3s prompt | 5s prompt | 10s prompt |
|------|-----------|-----------|------------|
| **108 全部说话人** | | | |
| [[YourTTS]]* | 0.357 | 0.377 | 0.394 |
| **VALL-E** | **0.382** | **0.423** | **0.484** |
| GroundTruth | 0.546 | 0.591 | 0.620 |
| **11 未见说话人** | | | |
| [[YourTTS]] | 0.331 | 0.337 | 0.344 |
| **VALL-E** | **0.389** | **0.380** | **0.414** |
| GroundTruth | 0.528 | 0.556 | 0.586 |

**说明**: 在 VCTK 上，108 个说话人中 VALL-E 训练时未见过任何一个，而 [[YourTTS]] 训练集包含 97 个。即便如此，VALL-E 在所有 prompt 长度上都优于 [[YourTTS]]。更长的 prompt 带来更高的相似度（3s: 0.382 -> 10s: 0.484）。

### Table 7: VCTK 主观评测

| Model | [[SMOS]] ↑ | [[CMOS]] (vs. VALL-E) |
|-------|------|------|
| [[YourTTS]]* | 3.70 +- 0.09 | -0.23 |
| **VALL-E** | **3.81 +- 0.09** | 0.00 |
| GroundTruth | 4.29 +- 0.09 | -0.04 |

**说明**: VALL-E 的 [[CMOS]] 比 ground truth 仅低 0.04，"与真实录音无统计显著差异"。[[YourTTS]] 落后 VALL-E 0.23 [[CMOS]]。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| LibriLight | 60K 小时, ~7000 说话人 | 英文有声书，噪声大但说话人多样 | 训练 |
| [[LibriSpeech]] test-clean | 2.2 小时（4-10 秒片段） | 干净英文 | 测试 |
| VCTK | 108 说话人 | 英文多口音 | 跨域测试 |
| EmoV-DB | 5 种情感 | 情感语音 | 定性分析 |

### 实现细节

- **Tokenizer**: [[EnCodec]] 6K bitrate, 24 kHz, 75 Hz frame rate, 8 层 [[RVQ]], 每层 1024 entries
- **ASR 标注**: Hybrid DNN-HMM (Kaldi)，30ms frameshift phoneme alignment
- **AR & NAR**: 各 12 层 [[Transformer]]，16 heads, dim 1024, FFN 4096, dropout 0.1
- **训练音频裁剪**: 随机 10-20 秒片段
- **NAR prompt**: 随机 3 秒片段
- **优化器**: AdamW, warmup 32K steps, peak LR $5 \times 10^{-4}$, linear decay
- **Batch Size**: 6K acoustic tokens / GPU
- **训练步数**: 800K steps
- **硬件**: 16x NVIDIA V100 32GB GPU
- **Baseline**: [[YourTTS]]（在 VCTK + [[LibriTTS]] + TTS-Portuguese 上训练）
- **Speaker Similarity 评测**: [[WavLM]]-TDNN（VoxSRC 2021/2022 top-ranked）
- **WER 评测**: [[HuBERT]]-Large fine-tuned on [[LibriSpeech]] 960h, CTC-based (no LM fusion)
- **主观评测**: [[CMOS]] (12 英语母语者, [-3, +3]) + [[SMOS]] (6 人, [1-5], 0.5 步长)

### 定性发现

1. **多样性**: Sampling-based 解码产生同一文本的多种合成结果（不同语速、重音、口音），有助于 ASR 数据增强
2. **声学环境保持**: 当 prompt 有混响时，VALL-E 合成的语音也带混响；传统 TTS 只输出干净语音
3. **情感保持**: 使用 EmoV-DB 的情感 prompt 时，VALL-E 自动保持 prompt 的情感（amused / angry / sleepy / disgusted），无需情感 TTS 微调

---

## 批判性思考

### 优点

1. **范式开创性**: 首次成功将 TTS 转化为 codec language modeling，证明了语音领域也可以受益于 LLM 的 scaling law 和 in-context learning，直接催生了 [[VALL-E 2]]、[[VALL-E X]]、[[SeedTTS]]、[[MaskGCT]] 等后续工作
2. **数据 scaling 验证**: 60K 小时训练首次证明了 TTS 中 data scaling 的有效性，打破了"TTS 只需要少量高质量数据"的传统认知
3. **无需 speaker embedding**: 完全依靠 prompt 机制实现 zero-shot，无需额外的 speaker encoder 或 fine-tune，架构更简洁

### 局限性

1. **鲁棒性不足**: 自回归 attention 对齐不稳定，存在词语不清晰、遗漏、重复等问题（autoregressive TTS 的经典痛点，后续 VALL-E 2 用 Repetition Aware Sampling 解决）
2. **数据覆盖偏差**: 60K 小时以英文有声书为主（LibriLight），朗读风格单一，口音覆盖不足，对非标准口音的泛化仍有限
3. **推理效率低**: AR + 7 次 NAR 调用，推理速度慢于非自回归方案；论文未报告 RTF
4. **两阶段分离**: AR 和 NAR 是两个独立模型，未能实现端到端统一建模
5. **仅限英文**: 未验证多语种能力

### 潜在改进方向

1. 统一 AR/NAR 为单一模型（后续 [[VALL-E 2]] 部分实现）
2. 全 NAR 加速推理（如 [[MaskGCT]] 的 masked prediction 方式）
3. 扩展到多语种和跨语言 TTS
4. 解决 AR 解码的鲁棒性问题（repetition / skipping）

### 可复现性评估

- [x] 代码开源（github.com/microsoft/unilm，社区复现版众多）
- [ ] 预训练模型（官方未公开 checkpoint）
- [x] 训练细节完整（架构、超参、数据处理流程详尽）
- [x] 数据集可获取（LibriLight 公开可用）

---

## 关联笔记

### 基于

- [[EnCodec]]: 提供离散音频 token 化，是 VALL-E 的底层表示
- [[AudioLM]]: 三段式 semantic -> coarse -> fine 的分层离散音频生成，VALL-E 借鉴了层级建模思路
- [[HuBERT]]: 论文中 [[WER]] 评测使用的 ASR 模型

### 对比

- [[YourTTS]]: 主要 baseline，speaker encoding 方式的 zero-shot TTS
- [[GSLM]]: speech-to-speech 离散语言模型，speaker similarity 极低（0.126）
- [[AudioLM]]: speech-to-speech 模型，VALL-E 加入了 text conditioning

### 方法相关

- [[RVQ]]: 残差向量量化，VALL-E 分层建模的设计基础
- [[Zero-shot TTS]]: VALL-E 开创的 in-context learning 范式
- [[Discrete Audio Token]]: 将连续语音离散化的核心理念
- [[Adaptive Layer Normalization]]: NAR 模型注入 stage 信息的技术
- [[In-Context Learning]]: VALL-E 通过 acoustic prompt 实现的泛化机制

### 后续工作

- [[VALL-E 2]]: 改进鲁棒性（Repetition Aware Sampling + grouped code modeling）
- [[SeedTTS]]: 字节跳动，基于 codec LM 范式的大规模 TTS
- [[MaskGCT]]: 全 NAR codec LM TTS
- [[CosyVoice]]: 阿里，supervised semantic token + flow matching 的方案

### 数据相关

- [[LibriSpeech]]: 测试集
- [[LibriTTS]]: YourTTS baseline 的训练数据之一

---

## 速查卡片

> [!summary] VALL-E: Neural Codec Language Models are Zero-Shot TTS
> - **核心**: 首次将 TTS 定义为条件 codec 语言建模，用 AR+NAR transformer 在 EnCodec 离散码上建模
> - **方法**: Phoneme + 3s acoustic prompt -> AR (layer 1) + NAR (layer 2-8) -> EnCodec decode -> waveform
> - **结果**: LibriSpeech SMOS 4.38 (+0.93 vs YourTTS), WER 5.9%, SPK 0.580; 60K h 训练数据
> - **代码**: https://github.com/microsoft/unilm

---

*笔记创建时间: 2026-05-25*
