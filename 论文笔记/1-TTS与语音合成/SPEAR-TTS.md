---
title: "Speak, Read and Prompt: High-Fidelity Text-to-Speech with Minimal Supervision"
method_name: "SPEAR-TTS"
authors: [Eugene Kharitonov, Damien Vincent, Zalán Borsos, Raphaël Marinier, Sertan Girgin, Olivier Pietquin, Matt Sharifi, Marco Tagliasacchi, Neil Zeghidour]
year: 2023
venue: arXiv
tags: [zero-shot-tts, low-resource-tts, backtranslation, semantic-token, discrete-speech, example-prompting, multi-speaker]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2302.03540
created: 2026-05-25
---

# 论文笔记：Speak, Read and Prompt: High-Fidelity Text-to-Speech with Minimal Supervision

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Google Research |
| 日期 | February 2023 |
| 项目主页 | [Demo](https://google-research.github.io/seanet/speartts/examples/) |
| 对比基线 | [[FastSpeech 2]], [[VALL-E]], [[YourTTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2302.03540) |

---

## 一句话总结

> Google 提出两阶段 TTS 系统 SPEAR-TTS：S1 "reading"（text -> [[Semantic Token]]）+ S2 "speaking"（semantic -> [[Acoustic Token]]），通过 [[Backtranslation]] 仅需 15 分钟标注数据即可实现高保真多说话人语音合成（MOS 4.96 vs GT 4.92）。

---

## 核心贡献

1. **两阶段解耦架构**: 将 TTS 分解为 S1（text -> semantic token）和 S2（semantic -> acoustic token），使 S2 可在纯音频数据上训练，语音质量不受标注数据量限制
2. **Backtranslation 极低资源训练**: 结合预训练和反向翻译技术，仅需 15 分钟单说话人标注数据即可训练 S1，CER 达 1.92%（LibriSpeech test-clean）
3. **Example Prompting 零样本声音克隆**: 通过 3 秒参考音频的 example prompting 机制控制说话人身份，无需显式说话人表示或 speaker-id 标签

---

## 问题背景

### 要解决的问题
传统 TTS 系统需要数百小时的高质量标注语音数据（text-speech 配对），这限制了其在低资源语言和方言中的应用。如何用极少量标注数据训练高质量多说话人 TTS？

### 现有方法的局限
- **[[FastSpeech 2]]** 等 NAR TTS：需大量单说话人标注数据，低资源时质量急剧下降（15min 数据 MOS 仅 1.72）
- **[[VALL-E]]**: 虽支持 [[Zero-shot TTS]]，但需要 60,000 小时 ASR 转录数据作为训练集，且推理时需要 prompt 的转录文本
- **Guided-TTS/Guided-TTS 2**: 依赖额外的音素分类器和说话人验证系统，监督信号并不"minimal"
- 直接端到端 text-to-waveform 建模时，文本与声学之间的巨大模态鸿沟难以用少量数据弥合

### 本文的动机
利用 [[AudioLM]] 的离散语音表示（[[Semantic Token]] + [[Acoustic Token]]），将 TTS 分解为两个较简单的 seq2seq 任务。S2 完全用无标注音频训练，S1 则可利用 NLP 领域成熟的预训练和 [[Backtranslation]] 技术大幅减少标注需求。

---

## 方法详解

### 模型架构

SPEAR-TTS 采用**两阶段级联**架构：

- **输入**: 文本（[[Phoneme]] 序列，47-token 词表）+ 可选 3 秒参考音频
- **S1 (Reading)**: [[T5]]-Large [[Encoder-Decoder Transformer]]，text -> [[Semantic Token]]
- **S2 (Speaking)**: 12 层 [[Decoder-only Transformer]]，semantic token -> [[Acoustic Token]]
- **解码器**: [[SoundStream]] decoder，acoustic token -> waveform
- **可选 S3**: bandwidth extension（16kHz -> 24kHz），T5-small

### 离散语音表示

#### Semantic Token

**设计动机**: 提供高层语言信息，剥离说话人身份和声学细节，作为 text 和 acoustic 之间的桥梁。

**具体实现**:
- 使用 [[w2v-BERT]]（结合 masked language modeling + contrastive learning 的自监督模型）
- 取第 7 层输出，经 mean-variance normalization 后用 [[k-means]]（K=512）聚类
- 聚类中心索引即为 semantic token
- 帧率 25 Hz，比特率 25 x log2(512) = 225 bit/s
- 去除连续重复 token（sequential repeats removed）

#### Acoustic Token

**设计动机**: 保留高保真声学细节，用于重建原始波形。

**具体实现**:
- 使用 [[SoundStream]] 神经音频编解码器
- [[RVQ]] 3 层量化，每层码本大小 1024
- 帧率 50 Hz，flat interleaving 后 150 tokens/s
- 比特率 1500 bit/s
- 词表大小 3 x 1024 = 3072 unique tokens

### 核心模块

#### 模块1: S1 — Reading（text -> semantic token）

**设计动机**: 将文本"朗读"为语义 token 序列，利用 [[Backtranslation]] 和预训练减少标注需求。

**预训练**: 受 [[BART]]/[[T5]] 启发，在纯语义 token 序列上做 denoising pretext task：
- 输入：随机删除 token 的 corrupted 序列（deletion probability = 0.6）
- 输出：原始无损序列
- 用 [[LibriLight]] 60,000h 语音提取的 semantic token 训练
- 架构：T5-Large（24 层 encoder-decoder）

**微调策略**:
- 冻结 encoder 上层 + 整个 decoder（除 cross-attention）
- 只更新 embedding + encoder 下层（4/6/8 层可选）
- 使用 [[Label Smoothing]] = 0.1

**Backtranslation 流程**:
1. 从预训练模型 P 出发
2. 冻结 encoder，微调 decoder 做反向任务（semantic -> text）
3. 用反向模型转录 [[LibriTTS]] 551h 纯音频数据，生成合成标注
4. 在合成数据上训练正向模型（也从 P 微调）
5. 最后在原始少量真实标注数据上继续微调

#### 模块2: S2 — Speaking（semantic -> acoustic token）

**设计动机**: 将语义 token "说"成声学 token，完全在无标注音频上训练。

**具体实现**:
- 12 层 decoder-only [[Transformer]]（12 heads x 64 dim，embed 768，FFN 2048）
- 在 [[LibriLight]] 60,000h 上训练 semantic-acoustic token 对
- 生成时语音自然带有随机变化的声音、语速和录音条件

**Example Prompting（声音克隆）**:
- 训练时：随机选取同一话语的两个不重叠语音窗口
- 计算两个窗口的 semantic + acoustic token
- 拼接顺序：(a) semantic prompt + (b) semantic target + (c) acoustic prompt + (d) acoustic target
- 训练目标：给 (a)-(c) 生成 (d)
- 段间用特殊分隔符分开防止伪影
- 推理时：3 秒参考音频提供 (a) 和 (c)，**无需 prompt 的文本转录**

**噪声控制**:
- 选择干净的 prompt 音频
- 生成 n_s=3 个样本，用 [[DNSMOS]] 类评估器选最优

---

## 关键公式

### 公式1: [[Semantic Token|语义 token 比特率]]

$$
R_{semantic} = f_{sr} \times \log_2(K) = 25 \times \log_2(512) = 225 \text{ bit/s}
$$

**含义**: 语义 token 的信息密度，每秒 225 bit 编码语言内容。

**符号说明**:
- $f_{sr} = 25$ Hz: semantic token 帧率
- $K = 512$: [[k-means]] 聚类簇数（码本大小）

### 公式2: [[Acoustic Token|声学 token 序列速率]]

$$
R_{acoustic} = f_{codec} \times N_{VQ} = 50 \times 3 = 150 \text{ tokens/s} = 1500 \text{ bit/s}
$$

**含义**: flat interleaving 后声学 token 的生成速率，acoustic 序列长度是 semantic 的 6 倍以上。

**符号说明**:
- $f_{codec} = 50$ Hz: [[SoundStream]] 帧率
- $N_{VQ} = 3$: [[RVQ]] 层数
- 每层码本大小 1024，bit/s = 150 x log2(1024) = 1500

### 公式3: [[Backtranslation|反向翻译训练流程]]

训练过程可形式化为：

$$
\begin{aligned}
&\text{Step 1:} \quad P \xleftarrow{\text{pretrain}} \text{Denoise}(\tilde{s} \to s) \\
&\text{Step 2:} \quad P_{bwd} \xleftarrow{\text{finetune}} (s \to t) \text{ on } \mathcal{D}_{parallel} \\
&\text{Step 3:} \quad \mathcal{D}_{synth} = \{(s_i, P_{bwd}(s_i))\}_{i=1}^{|\mathcal{D}_{audio}|} \\
&\text{Step 4:} \quad P_{fwd} \xleftarrow{\text{finetune}} (t \to s) \text{ on } \mathcal{D}_{synth} \cup \mathcal{D}_{parallel}
\end{aligned}
$$

**含义**: 先预训练 denoising 模型 P，再微调为反向模型 P_bwd（semantic -> text），转录大量无标注音频生成合成配对数据，最后训练正向模型 P_fwd（text -> semantic）。

**符号说明**:
- $P$: 预训练的 encoder-decoder 模型
- $\tilde{s}$: corrupted semantic token 序列
- $s$: 原始 semantic token 序列
- $t$: 文本（phoneme 序列）
- $\mathcal{D}_{parallel}$: 少量真实标注数据（15min-24h）
- $\mathcal{D}_{audio}$: 大量无标注音频（LibriTTS 551h）
- $\mathcal{D}_{synth}$: 合成配对数据

---

## 关键图表

### Figure 1: SPEAR-TTS 两阶段架构

![Figure 1: SPEAR-TTS Overview](https://ar5iv.labs.arxiv.org/html/2302.03540/assets/x1.png)

**说明**: SPEAR-TTS 的整体流程。S1（"reading"）将 tokenized text 映射为 [[Semantic Token]]；S2（"speaking"）将 semantic token 映射为 [[Acoustic Token]]；最后由 [[SoundStream]] decoder 解码为音频波形。两阶段解耦使 S2 可完全用无标注音频训练。

### Figure 2: S1 训练流程（预训练 + Backtranslation）

![Figure 2: Training S1](https://ar5iv.labs.arxiv.org/html/2302.03540/assets/x2.png)

**说明**: S1 的训练分四步：(1) 在纯语音 semantic token 上预训练 denoising encoder-decoder P；(2) 冻结 encoder，微调 decoder 做反向任务（semantic -> text）；(3) 用反向模型转录大量无标注语音，生成合成配对数据；(4) 在合成+真实数据上微调正向模型（text -> semantic）。这个流程使得仅 15 分钟标注数据即可训练高质量 S1。

### Figure 3: S2 Example Prompting 控制生成

![Figure 3: Example Prompting in S2](https://ar5iv.labs.arxiv.org/html/2302.03540/assets/x3.png)

**说明**: S2 的 example prompting 机制。推理时将参考音频的 semantic token（prompt 部分）和目标 semantic token 拼接，再接上参考音频的 acoustic token，模型自回归生成目标 acoustic token。这样生成的语音保持与 prompt 一致的声音特征，无需 prompt 的文本转录。

### Table 1: CER (%) — S1 在不同标注数据量下的表现

| 标注数据量 | FastSpeech2-LR | From Scratch (a) | Pretraining (b) | BT from scratch (c) | BT + Pretraining (d) |
|---|---|---|---|---|---|
| 24h | 1.99+-0.20 | 3.67+-0.21 | 2.38+-0.13 | 2.26+-0.14 | **2.06+-0.12** |
| 12h | - | 4.31+-0.28 | 2.54+-0.14 | 2.27+-0.14 | **2.03+-0.12** |
| 3h | 2.52+-0.25 | 20.1+-0.74 | 3.07+-0.15 | 2.21+-0.12 | **2.01+-0.12** |
| 2h | - | 24.7+-0.71 | 3.73+-0.17 | 2.22+-0.13 | **2.09+-0.12** |
| 1h | 2.74+-0.27 | x | 5.51+-0.21 | 2.23+-0.13 | **2.16+-0.13** |
| 30min | 3.18+-0.28 | x | 21.3+-0.43 | 2.52+-0.15 | **2.20+-0.12** |
| 15min | 4.90+-0.34 | x | x | 2.88+-0.19 | **2.21+-0.12** |

**说明**: x 表示生成不可理解的语音。从 scratch 训练在 <1h 数据时完全失败；预训练可撑到 2h；[[Backtranslation]] 是核心突破，使 15min 数据也能达到 2.21% CER。完整方案 (d) 在所有数据量上都优于 [[FastSpeech 2]]。

### Table 2: 声音多样性（bits，说话人分布熵）

| S1 训练数据 | LJSpeech (1 spk) | LibriTTS 61 spk | LibriTTS 123 spk | LibriTTS 247 spk |
|---|---|---|---|---|
| Ground-truth | 2.55 | 5.82 | 6.71 | 7.68 |
| SPEAR-TTS | 6.11 | 6.22 | 6.16 | 6.28 |
| FastSpeech2-LR | 0.66 | - | - | - |

**说明**: SPEAR-TTS 的声音多样性（6.11-6.28 bits）与 S1 训练数据的说话人数量几乎无关，即使用单说话人 [[LJSpeech]] 训练也能生成多样化语音。相比之下 [[FastSpeech 2]] 仅 0.66 bits，几乎没有声音变化。这得益于 S2 在 [[LibriLight]] 60k 小时多说话人数据上训练。

### Table 3: 声音保持能力（分类器准确率）

| CER (%) | Speaker Acc top-1 (%) | Speaker Acc top-3 (%) | Voice Diversity (bits) |
|---|---|---|---|
| 1.92 | 92.4 | 98.1 | 0.41 |

**说明**: 在 40 个未见说话人上测试，3 秒 prompt 即可实现 92.4% top-1 说话人准确率。Prompted 生成的 CER (1.92%) 甚至低于 unprompted (2.21%)，因为干净的 prompt 引导 S2 生成更干净的输出。

### Table 4: 声音相似度对比（Cosine Similarity）

| 模型 | 标注训练数据量 | Cosine Similarity |
|---|---|---|
| [[YourTTS]] | ~600h | 0.34 |
| [[VALL-E]] | 60,000h | 0.58 |
| **SPEAR-TTS** | **15 min** | **0.56** |

**说明**: SPEAR-TTS 用 15 分钟标注数据达到与 [[VALL-E]]（60,000h）几乎持平的说话人相似度（0.56 vs 0.58），是数据量差距 240,000 倍下的惊人表现。使用 [[WavLM]] 基 speaker verification 系统评估。

### Table 5: 主观评测 MOS

| 系统 | FS2-LR 15min | FS2-LR 1h | FS2-LR 24h | SPEAR-TTS 15min | Ground-truth |
|---|---|---|---|---|---|
| [[MOS]] | 1.72+-0.04 | 2.08+-0.04 | 2.11+-0.04 | **4.96+-0.02** | 4.92+-0.04 |

**说明**: SPEAR-TTS 用 15 分钟数据即达到 MOS 4.96，与真实语音（4.92）无显著差异，远超 [[FastSpeech 2]] 在 24h 数据上的 2.11。这是该论文最震撼的结果。

### Table 6: Prompted 生成 MOS — vs VALL-E

| 系统 | MOS |
|---|---|
| [[VALL-E]] | 3.35+-0.12 |
| **SPEAR-TTS (15 min)** | **4.75+-0.06** |

**说明**: 在 [[VALL-E]] demo page 的 24 个样本上对比，SPEAR-TTS 的 MOS（4.75）远超 VALL-E（3.35），且仅用 240,000 倍更少的标注数据。

### Table 7: 采样数量对质量的影响（消融）

| n_s | 1 | 2 | 3 | 5 | 10 |
|---|---|---|---|---|---|
| CER (%) | 2.10 | 1.99 | 1.93 | 1.90 | 1.90 |
| Audio Quality (DNSMOS) | 3.68 | 3.86 | 3.94 | 4.02 | 4.11 |

**说明**: 增加采样数量可同时改善 CER 和音频质量。n_s=3 是质量-计算量的最佳平衡点。

### Table 8: S1 在 LibriTTS 上的 CER（消融）

| 数据量 | From Scratch | Pretraining |
|---|---|---|
| 551h | 2.04 | 2.01 |
| 241h | 2.08 | 1.92 |
| 54h | 2.61 | 2.13 |

**说明**: 数据充足时（551h）预训练几乎无收益；但数据减少到 54h 时预训练从 2.61% 降至 2.13%，收益显著。

### Table 9: Transformer 架构参数

| 配置 | Embed. dim | FFN dim | Head dim | # Heads |
|---|---|---|---|---|
| T5-small | 256 | 512 | 64 | 6 |
| T5-base | 768 | 2048 | 64 | 12 |
| T5-large | 1024 | 2816 | 64 | 16 |

**说明**: S1 使用 T5-large 配置；S2 使用自定义 12 层（接近 T5-base 规模但参数不同）。

### Table 10: Phoneme vs Grapheme 输入（消融）

| 标注数据量 | 24h | 12h | 3h | 2h | 1h | 30min | 15min |
|---|---|---|---|---|---|---|---|
| [[Phoneme]] | 2.06 | 2.03 | 2.01 | 2.09 | 2.16 | 2.20 | 2.21 |
| Grapheme | 1.79 | 1.79 | 2.13 | 2.27 | 2.46 | 2.71 | 3.45 |

**说明**: 低资源场景下 [[Phoneme]] 显著优于 grapheme（15min: 2.21% vs 3.45%）；但数据充足时 grapheme 反超（24h: 1.79% vs 2.06%），因为 [[G2P]] 引入了转换误差。

### Table 11: S2 训练数据量影响（消融）

| 下采样倍数 | 1x | 2x | 5x | 10x |
|---|---|---|---|---|
| CER (%) | 1.99 | 1.99 | 2.36 | 2.92 |

**说明**: S2 在完整 [[LibriLight]] 和 2x 下采样（~30,000h）时性能持平；5x 起开始退化。S2 对数据量需求为 10,000h 级别。

### Table 12: 主观评测用的 20 个句子

论文附录列出了完整的 20 个评测句子，来自 2022 年出版的有声书 "Predecessors of Cleopatra"（与 LJSpeech 同一说话人），长度 3-11 秒，总计 133 秒。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LibriLight]] unlab-60k | ~60,000h | 英文有声书，>7000 说话人，16kHz | SoundStream/w2v-BERT/k-means 训练，S2 训练，S1 预训练 |
| [[LJSpeech]] | 24h | 单说话人英文，子集：15min-24h | S1 标注数据（文本-语音配对） |
| [[LibriTTS]] | 551h | 多说话人英文 | Backtranslation 音频来源（忽略转录），bandwidth extension |
| [[LibriSpeech]] test-clean | ~3h (过滤后 2007 句) | <=10s 限制 | CER 评估 |

### 实现细节

- **S1 Backbone**: [[T5]]-Large encoder-decoder（24 层）
- **S2 Backbone**: 12 层 decoder-only [[Transformer]]（768 dim, 12 heads）
- **优化器**: [[Adafactor]] with inverse square-root LR decay
- **正则化**: [[Label Smoothing]] = 0.1, dropout 0.5（预训练）/ 0.1-0.5（微调）
- **Batch Size**: 256（预训练）
- **训练步数**: 1M updates（预训练）
- **S1 推理**: [[Beam Search]]（beam = 10）
- **S2 推理**: Temperature Sampling（T = 0.75），n_s = 3 采样选最优
- **文本前端**: [[G2P]] (g2p_en) -> 去除重音 -> 47-token 词表（39 CMU phonemes + 空白 + 标点）

### 可视化结果

- S2 即使 S1 用单说话人数据训练，也能生成多样化声音（Table 2 中单说话人 entropy 6.11 bits vs GT 2.55 bits）
- Example prompting 的声音稳定性极高（diversity 仅 0.41 bits），说明同一 prompt 下的重复生成声音一致
- 合成语音检测器可达 82.5% 准确率（SoundStream 重合成 GT vs SPEAR-TTS 输出）

---

## 批判性思考

### 优点
1. **数据效率极高**: 15 分钟标注数据 + 大量无标注语音即可达到 SOTA 质量，打开了低资源语言 TTS 的大门
2. **优雅的架构设计**: 两阶段解耦使语音质量完全由 S2 决定（无标注数据训练），S1 只需学习文本到语义的映射
3. **Backtranslation 在语音领域的成功迁移**: 将 NLP 的经典技术（预训练 + 反向翻译）引入语音，效果惊艳
4. **无需 prompt 转录**: 与 [[VALL-E]] 不同，prompting 时不需要参考音频的文本，实用性更强
5. **完整的消融实验**: 12 张表覆盖了架构、数据量、输入表示等多维度消融

### 局限性
1. **仅验证英语**: 所有实验在英语上进行，未验证跨语言泛化（尤其是声调语言如中文）
2. **16kHz 采样率限制**: [[LibriLight]] 为 16kHz，需额外的 bandwidth extension 模块才能达到 24kHz，增加了系统复杂度
3. **推理速度未报告**: 论文未提及 [[RTF]] 或推理延迟，两阶段级联 + AR 采样可能较慢
4. **S2 数据需求仍大**: 虽然 S1 只需 15min 标注，S2 仍需 10,000h 级别无标注音频（Table 11）
5. **未开源**: 代码和模型均未公开，难以复现
6. **MOS 对比公平性存疑**: 与 [[VALL-E]] 的 MOS 对比（Table 6）使用了 VALL-E demo page 样本而非重新跑，条件不完全一致

### 潜在改进方向
1. **多语言扩展**: 用多语言 SSL 模型（如 [[XEUS]]）替代 [[w2v-BERT]]，将框架推广到低资源语言
2. **NAR S2**: 将 S2 改为 [[SoundStorm]] 等 NAR 模型以加速推理
3. **流式推理**: 当前为离线生成，改为 chunk-based 流式可降低延迟
4. **端到端微调**: 两阶段分开训练可能有信息损失，联合微调可能进一步提升

### 可复现性评估
- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（超参数详尽）
- [x] 数据集可获取（LibriLight/LJSpeech/LibriTTS 均公开）

---

## 关联笔记

### 基于
- [[AudioLM]]: SPEAR-TTS 的直接前身，提出 semantic + acoustic token 层级和 example prompting
- [[w2v-BERT]]: 用于提取 semantic token 的自监督模型
- [[SoundStream]]: 提供 acoustic token 的神经音频编解码器

### 对比
- [[FastSpeech 2]]: 低资源场景下的 NAR TTS 基线
- [[VALL-E]]: 同期最强零样本 TTS，但需 60,000h 标注数据
- [[YourTTS]]: 零样本 TTS 基线，speaker similarity 远低于 SPEAR-TTS

### 方法相关
- [[Semantic Token]]: 语义 token，SPEAR-TTS 的核心中间表示
- [[Acoustic Token]]: 声学 token，保留高保真声学细节
- [[Backtranslation]]: S1 训练的核心技术，大幅减少标注需求
- [[RVQ]]: SoundStream 中的残差向量量化
- [[k-means]]: 将 w2v-BERT 连续特征离散化为 semantic token
- [[Zero-shot TTS]]: SPEAR-TTS 支持的零样本语音合成任务

### 硬件/数据相关
- [[LibriLight]]: 60,000h 无标注英文音频，S2 训练和 S1 预训练
- [[LJSpeech]]: 单说话人标注数据，S1 微调
- [[LibriTTS]]: 551h 多说话人，用于 backtranslation
- [[LibriSpeech]]: CER 评测

---

## 速查卡片

> [!summary] SPEAR-TTS
> - **核心**: 两阶段 TTS（text->semantic->acoustic），通过 backtranslation 仅需 15min 标注数据
> - **方法**: S1 预训练+反向翻译学 text->semantic；S2 纯无标注音频学 semantic->acoustic；3秒 prompt 克隆声音
> - **结果**: CER 1.92%（15min 数据），MOS 4.96 vs GT 4.92，Speaker Sim 0.56 vs VALL-E 0.58（数据量差 240,000x）
> - **代码**: 未开源（仅有 demo page）

---

*笔记创建时间: 2026-05-25*
