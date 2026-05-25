---
title: "Speak Foreign Languages with Your Own Voice: Cross-Lingual Neural Codec Language Modeling"
method_name: "VALL-E X"
authors: [Ziqiang Zhang, Long Zhou, Chengyi Wang, Sanyuan Chen, Yu Wu, Shujie Liu, Zhuo Chen, Yanqing Liu, Huaming Wang, Jinyu Li, Lei He, Sheng Zhao, Furu Wei]
year: 2023
venue: arXiv
tags: [cross-lingual-tts, speech-to-speech-translation, zero-shot-tts, codec-language-model, speaker-preservation, language-id]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2303.03926
created: 2026-05-25
---

# 论文笔记：Speak Foreign Languages with Your Own Voice: Cross-Lingual Neural Codec Language Modeling

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft |
| 日期 | March 2023 |
| 项目主页 | [VALL-E X Demo](https://www.microsoft.com/en-us/research/project/vall-e-x/) |
| 对比基线 | [[VALL-E]], [[YourTTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2303.03926) / [Code](https://github.com/microsoft/unilm) |

---

## 一句话总结

> 将 [[VALL-E]] 扩展到跨语言场景，用源语言语音 + 目标语言文本作为 prompt，零样本合成保留说话人音色的目标语言语音。

---

## 核心贡献

1. **跨语言 Codec 语言模型**: 将 [[VALL-E]] 的条件语言建模范式扩展到多语言场景，用大规模多语言（中英共 70K 小时）非干净语音数据训练 [[Cross-Lingual TTS]] 模型
2. **多语言 In-Context Learning**: 通过源语言语音 prompt 实现跨语言零样本语音合成，保留未见说话人的音色、情感和声学环境
3. **Language ID 控制口音**: 引入 [[Language ID]] 嵌入机制，有效缓解外语口音问题（L2 accent），口音评分从 2.55 提升至 4.10
4. **零样本 S2ST**: 结合改进的 [[SpeechUT]] 翻译模块，实现端到端零样本 [[S2ST]]，SMOS 从 3.06 提升至 4.12

---

## 问题背景

### 要解决的问题

跨语言语音合成（Cross-Lingual Speech Synthesis）：让一个只说一种语言的人，能用自己的声音说另一种语言。核心挑战是同时保持说话人音色和消除外语口音。

### 现有方法的局限

1. **数据稀缺**: 收集同一说话人的多语言数据极其困难
2. **模型容量不足**: 传统模型（基于 Mel-Spectrogram）难以充分转移音色、背景噪声和情感
3. **零样本能力缺失**: 之前的跨语言 TTS 需要目标说话人的数据，无法处理未见说话人
4. **口音问题严重**: 合成语音往往带有明显的外语口音（L2 accent）

### 本文的动机

[[VALL-E]] 在单语言 TTS 中展示了强大的 [[In-Context Learning]] 能力——仅需 3 秒参考音频就能复刻音色。作者认为这种能力可以自然扩展到跨语言场景：用源语言语音作为声学 prompt，目标语言文本作为内容指导，配合 [[Language ID]] 消除口音。

---

## 方法详解

### 模型架构

VALL-E X 采用 **两阶段 [[Autoregressive Model|AR]]+NAR** 架构：

- **输入**: 源语言音素序列 $\mathcal{S}^s$ + 目标语言音素序列 $\mathcal{S}^t$ + 源语言 [[Acoustic Token]] 序列 $\mathcal{A}^s$
- **Backbone**: 12 层 [[Transformer]] Decoder（attention dim=1024, FFN dim=4096）
- **核心模块**: Multi-lingual AR Codec LM ($\phi_{\text{MAR}}$) + Multi-lingual NAR Codec LM ($\phi_{\text{MNAR}}$)
- **声学量化器**: [[EnCodec]]，$L=8$ 层 [[RVQ]]，码本大小 1024，帧率 75 Hz
- **输出**: 目标语言 [[Acoustic Token]] 序列 $\mathcal{A}^t_{:,1:8}$，经 [[EnCodec]] 解码器合成波形

### 核心模块

#### 模块 1: Multi-lingual Autoregressive Codec LM ($\phi_{\text{MAR}}$)

**设计动机**: 利用 [[Autoregressive Model|自回归建模]] 逐帧生成第一层 [[Acoustic Token]]，捕获语音的全局韵律和内容结构。

**具体实现**:
- 单向 [[Transformer]] Decoder，通过 attention mask 实现自回归
- 输入为拼接的音素序列 $\mathcal{S}$ 和第一层 acoustic token 序列 $\mathcal{A}_{:,1}$
- 为每个 prompt 子序列分别计算 sinusoidal position embedding
- 训练时用 teacher forcing，推理时自回归采样直到生成 end-of-sentence token

#### 模块 2: Multi-lingual Non-Autoregressive Codec LM ($\phi_{\text{MNAR}}$)

**设计动机**: 在第一层 token 基础上，并行预测第 2-8 层 [[RVQ]] 码，补充声学细节（音色、音质等）。

**具体实现**:
- 非自回归 [[Transformer]] LM，逐层迭代生成
- 输入包含当前句音素 $\mathcal{S}$ + 同说话人另一句完整 8 层 token $\tilde{\mathcal{A}}_{:,1:8}$（作为说话人 prompt） + 前 $l-1$ 层已生成 token
- 对每个 RVQ 层使用独立的 Layer Normalization
- 训练时每步随机选取一个层计算 loss，而非累积所有层

#### 模块 3: Language ID

**设计动机**: 中文（声调语言）和英文（非声调语言）声学特征差异大，不加约束时模型倾向于混合两种语言的声学模式，导致外语口音。

**具体实现**:
- 将语言 ID 编码为稠密向量嵌入
- 加到 [[Acoustic Token]] 嵌入上（而非音素嵌入），直接引导声学生成风格
- 推理时用目标语言的 ID，训练时用各自语言的 ID

---

## 关键公式

### 公式 1: [[Autoregressive Model|AR 语言模型]]训练目标

$$
\mathcal{L}_{\text{MAR}} = -\log p_{\text{AR}}(\mathcal{A}_{:,1} \mid \mathcal{S}; \phi_{\text{MAR}}) = -\log \prod_{i=1}^{N} p(a_{i,1} \mid \langle \mathcal{S}, \mathcal{A}_{<i,1} \rangle; \phi_{\text{MAR}})
$$

**含义**: 自回归地预测第一层 acoustic token 序列，每个 token 条件于音素序列和之前的 token。

**符号说明**:
- $\mathcal{S}$: 音素序列
- $\mathcal{A}_{:,1}$: 第一层 acoustic token 序列 $\{a_{i,1}\}_{i=1}^{N}$
- $\langle \cdot \rangle$: 序列拼接
- $p(\cdot)$: softmax 概率

### 公式 2: NAR 语言模型训练目标

$$
\mathcal{L}_{\text{MNAR}} = \sum_{l=2}^{8} \log p_{\text{NAR}}(\mathcal{A}_{:,l} \mid \langle \mathcal{S}, \tilde{\mathcal{A}}_{:,1:8}, \mathcal{A}_{:,1:l-1} \rangle; \phi_{\text{MNAR}})
$$

**含义**: 非自回归地逐层生成第 2-8 层 RVQ token，每层以前面所有层的 token 为条件。

**符号说明**:
- $l \in [2, 8]$: RVQ 层索引
- $\tilde{\mathcal{A}}_{:,1:8}$: 同说话人另一句的全 8 层 token（speaker prompt）
- $\mathcal{A}_{:,1:l-1}$: 当前句已有的前 $l-1$ 层 token
- $p_{\text{NAR}}(\cdot)$: pointwise 概率（非自回归，各帧独立）

### 公式 3: 跨语言推理 — AR 阶段

$$
\hat{a}^t_{i,1} \sim p_{\text{AR}}(a^t_{i,1} \mid \langle \mathcal{S}^s, \mathcal{S}^t, \mathcal{A}^s_{:,1}, \mathcal{A}^t_{<i,1} \rangle; \phi_{\text{MAR}}), \quad i = 1, \ldots
$$

**含义**: 推理时将源语言音素、目标语言音素和源语言 acoustic token 拼接作为 prompt，自回归采样目标语言的第一层 token。

**符号说明**:
- $\mathcal{S}^s, \mathcal{S}^t$: 源/目标语言音素序列
- $\mathcal{A}^s_{:,1}$: 源语言第一层 acoustic token（作为声学 prompt）
- $\hat{a}^t_{i,1}$: 生成的目标语言第 $i$ 帧第一层 token

### 公式 4: 跨语言推理 — NAR 阶段

$$
\mathcal{A}^t_{:,l} = \operatorname{argmax}_{\mathcal{A}^t_{:,l}} p_{\text{NAR}}(\mathcal{A}^t_{:,l} \mid \langle \mathcal{S}^t, \mathcal{A}^s_{:,1:8}, \mathcal{A}^t_{:,1:l-1} \rangle; \phi_{\text{MNAR}}), \quad l = 2, \ldots, 8
$$

**含义**: 在 NAR 阶段，用源语言全 8 层 token 作为 speaker prompt，逐层 argmax 解码目标语言剩余 RVQ 层。

**符号说明**:
- $\mathcal{A}^s_{:,1:8}$: 源语言完整 8 层 acoustic token（提供音色信息）
- $\mathcal{A}^t_{:,1:l-1}$: 目标语言已生成的前 $l-1$ 层 token

### 公式 5: SpeechUT 语音侧预训练

$$
\mathcal{L}_{\text{speech}} = -\sum_{i \in \mathcal{M}} \left( \log p(s^s_i \mid \mathcal{X}^s; \theta_{enc1}) + \log p(s^s_i \mid \mathcal{X}^s; \theta_{enc1}, \theta_{enc2}) \right)
$$

**含义**: 对遮蔽位置的音素标签做预测，分别用语音编码器和语义编码器联合预测。

**符号说明**:
- $\mathcal{M}$: 遮蔽位置集合
- $\mathcal{X}^s$: 源语言语音
- $\theta_{enc1}$: 语音编码器参数
- $\theta_{enc2}$: 语义编码器参数

### 公式 6: SpeechUT 文本侧预训练

$$
\mathcal{L}_{\text{text}} = -\sum_{i=1}^{|\mathcal{S}^t|} \log p(s^t_i \mid \mathcal{S}^t_{<i}, \mathcal{S}^s; \theta_{enc2}, \theta_{dec})
$$

**含义**: 文本侧以 seq2seq 方式训练，源语言音素到目标语言音素的翻译。

**符号说明**:
- $\mathcal{S}^s, \mathcal{S}^t$: 源/目标语言音素序列
- $\theta_{dec}$: 语义解码器参数

---

## 关键图表

### Figure 1: Overall Framework / 系统概览

![Figure 1](https://ar5iv.labs.arxiv.org/html/2303.03926/assets/x1.png)

**说明**: VALL-E X 的整体框架。对于只说源语言的说话人，系统将其语音通过 [[EnCodec]] 编码为 [[Acoustic Token]]，与目标语言文本的 [[Phoneme|音素]] 序列一起作为 prompt，生成目标语言的 acoustic token 序列，再通过 [[EnCodec]] 解码器合成波形。无需同一说话人的跨语言数据。

### Figure 2: Training Illustration / 训练流程

![Figure 2](https://ar5iv.labs.arxiv.org/html/2303.03926/assets/x2.png)

**说明**: VALL-E X 的训练由两部分组成：(1) 多语言 AR Codec LM ($\phi_{\text{MAR}}$) 从音素预测第一层 acoustic token；(2) 多语言 NAR Codec LM ($\phi_{\text{MNAR}}$) 从第一层 token 预测第 2-8 层。训练数据为不同语言的配对 (音素, acoustic token) 序列。

### Figure 3: Inference Illustration / 推理流程

![Figure 3](https://ar5iv.labs.arxiv.org/html/2303.03926/assets/x3.png)

**说明**: 推理时的两阶段解码策略。AR 阶段：拼接源语言音素 + 目标语言音素 + 源语言 acoustic token 作为 prefix，自回归生成目标语言第一层 token。NAR 阶段：用源语言全 8 层 token 作为 speaker prompt，逐层解码剩余层。支持零样本跨语言 TTS 和零样本 [[S2ST]]。

### Table 1: 与之前跨语言 TTS 系统对比

| 特性 | 之前系统 | VALL-E X |
|------|----------|----------|
| 中间表示 | Mel spectrogram | [[Acoustic Token|Audio codec codes]] |
| 训练数据 | << 13K 小时 | **70K 小时** |
| 语音口音 | 外语口音 | **母语口音** |
| 说话人相似度 | 较低 | **高** |
| In-context learning | 不支持 | **支持** |
| 零样本跨语言 TTS | 不支持 | **支持** |

**说明**: VALL-E X 在多个维度全面超越传统跨语言 TTS 系统，核心优势来自大规模数据 + codec 离散表示 + in-context learning 范式。

### Table 2: 零样本跨语言 TTS 自动评估

| 配置 | ASV-Score | ASR-WER | Naturalness |
|------|-----------|---------|-------------|
| **英文 TTS (中文 prompt)** | | | |
| Baseline ([[YourTTS]]) | 0.30 ± 0.10 | 8.53 | 3.36 |
| **VALL-E X** | **0.36 ± 0.11** | **4.07** | **3.54** |
| **中文 TTS (英文 prompt)** | | | |
| VALL-E X | 0.29 ± 0.10 | 8.52 | 3.36 |

**说明**: VALL-E X 在英文跨语言 TTS 上全面超越 [[YourTTS]]：说话人相似度提升 20%（0.30→0.36），[[WER]] 降低一半（8.53→4.07），自然度提升。中文方向的 WER 较高（8.52），可能受限于中文 [[ASR]] 评估器精度和中文合成数据量不足。

### Table 3: 零样本跨语言 TTS 人工评估（50 样本，英文 TTS + 中文 prompt）

| 系统 | [[SMOS]] | [[CMOS]] (vs. Baseline) |
|------|------|------|
| Baseline ([[YourTTS]]) | 3.42 ± 0.19 | 0.00 |
| **VALL-E X** | **4.00 ± 0.20** | **+0.24** |

**说明**: 人工评估进一步验证 VALL-E X 的优势。SMOS（说话人相似度主观评分）从 3.42 提升至 4.00（+0.58），CMOS 相对偏好 +0.24。

### Table 4: 零样本 S2ST 在 EMIME 数据集上的表现

| 配置 | ASV-Score (hyp vs. src) | ASR-BLEU | Naturalness |
|------|------------------------|----------|-------------|
| **中→英 S2ST** | | | |
| Baseline (级联系统) | 0.28 ± 0.10 | 27.49 | 3.44 |
| **VALL-E X Trans** | **0.37 ± 0.10** | **30.66** | **3.54** |
| VALL-E X Trans (oracle text) | 0.39 ± 0.10 | 86.78 | 3.54 |
| **英→中 S2ST** | | | |
| VALL-E X Trans | 0.48 ± 0.11 | 34.45 | 3.41 |
| VALL-E X Trans (oracle text) | 0.47 ± 0.12 | 84.00 | 3.42 |

**说明**: VALL-E X Trans 在中→英方向显著优于级联基线：说话人相似度 0.37 vs 0.28（+32%），BLEU 30.66 vs 27.49（+3.17）。Oracle text 条件下 BLEU 达 84-87，表明 codec LM 本身的合成质量很高，瓶颈主要在翻译精度。

### Table 5: S2ST 人工评估（56 翻译对）

| 系统 | 中→英 SMOS | 中→英 MOS | 英→中 SMOS | 英→中 MOS |
|------|-----------|-----------|-----------|-----------|
| Baseline (级联) | 3.06 ± 0.14 | 3.81 ± 0.19 | - | - |
| **VALL-E X Trans** | **4.12 ± 0.13** | **3.87 ± 0.21** | **3.94 ± 0.15** | 3.48 ± 0.13 |
| Source speech | 4.91 ± 0.05 | - | 4.64 ± 0.06 | - |
| Oracle target | - | 3.92 ± 0.17 | - | 3.88 ± 0.13 |

**说明**: SMOS 从 3.06 跃升至 4.12（+1.06），说明 VALL-E X 在保留说话人身份方面大幅领先。MOS 接近 oracle target（3.87 vs 3.92），合成质量接近参考上界。

### Table 6: Language ID 消融实验

| 配置 | ASV-Score (vs. src) | ASR-BLEU | Accent Score |
|------|-------------------|----------|-------------|
| **中→英 S2ST** | | | |
| VALL-E X Trans (w/ LID) | 0.37 ± 0.10 | 30.66 | **4.10** |
| w/o Language ID | 0.41 ± 0.10 | 29.04 | 2.98 |
| w/ wrong Language ID | 0.41 ± 0.10 | 29.07 | 2.55 |
| **英→中 S2ST** | | | |
| VALL-E X Trans (w/ LID) | 0.48 ± 0.11 | 34.45 | **4.03** |
| w/o Language ID | 0.49 ± 0.11 | 30.86 | 2.35 |
| w/ wrong Language ID | 0.50 ± 0.11 | 29.70 | 2.25 |

**说明**: 这是最有洞察力的消融。去掉 [[Language ID]] 后 ASV-Score 反而上升（0.37→0.41），说明不加约束时模型更多地"复制"源说话人的声学特征（包括口音）。但 Accent Score 从 4.10 暴跌至 2.55，外语口音极其严重。这揭示了一个 trade-off：**Language ID 以牺牲少量说话人相似度为代价，大幅改善母语口音质量**。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LibriLight]] | ~60,000 小时 | 英文有声书，无标注，用 [[Kaldi]] ASR 伪标注 | VALL-E X 训练（英文） |
| [[WenetSpeech]] | 10,000+ 小时 | 中文多领域带标注语音 | VALL-E X 训练（中文） |
| [[EMIME]] | 25 双语句对 × 14 人 | 中英双语同说话人 | S2ST 评测 |
| [[LibriSpeech]] dev-clean | 5.4 小时 | 英文有声书 | 跨语言 TTS 评测（英文方向） |
| AI Challenger + OpenSub + WMT | ~73M 对 | 中英机器翻译 | SpeechUT 翻译模块训练 |
| GigaST | 翻译的 GigaSpeech 转录 | 英→中语音翻译 | ST 微调 |

### 实现细节

- **Backbone**: 12 层 [[Transformer]] Decoder，attention dim=1024，FFN dim=4096
- **优化器**: 最大学习率 5e-4，warm-up 8,000 步
- **Batch Size**: AR 120 秒/GPU，NAR 66 秒/GPU
- **训练步数**: 800K 步
- **硬件**: 32 × V100 GPU
- **最大句长**: 20 秒
- **音素化**: BigCiDian 统一 IPA 音素集（中英共用），[[Kaldi]] 做 [[Forced Alignment]]
- **声学量化**: [[EnCodec]]，8 层 [[RVQ]]，码本大小 1024，75 Hz 帧率

### S2ST 翻译模块

- 基于改进的 [[SpeechUT]]，将 clustering-based hidden unit 替换为 [[Phoneme|音素]]
- 架构：6 层 Transformer（语音编码器 + 语义编码器 + 语义解码器）
- 预训练 400K 步（32 × V100），微调 200K 步
- 微调用 [[CTC]] loss（语义编码器预测源音素）+ Cross-Entropy loss（语义解码器预测目标音素）

### 可视化结果

- **情感保持**: 使用 EmoV-DB 的情感语音作为 prompt 时，VALL-E X 成功保持情感一致性（如愤怒→愤怒语调的跨语言合成）
- **Code-Switch**: 尽管只用单语数据训练，VALL-E X 可以合成流畅的中英混合语音，保持一致音色——这是 [[In-Context Learning]] 能力的涌现

---

## 批判性思考

### 优点
1. **范式创新**: 首次将 codec language modeling 扩展到跨语言场景，简洁地利用 [[VALL-E]] 的 in-context learning 解决跨语言音色迁移
2. **Language ID 设计精巧**: 简单的嵌入加法就能有效分离语言特征与说话人特征，消融实验揭示了 trade-off 的本质
3. **训练数据高效利用**: 不需要同一说话人的跨语言数据，用两个单语语料库（LibriLight + WenetSpeech）就能实现跨语言迁移
4. **涌现能力**: Code-switch 和情感保持能力是无监督涌现的，说明大规模训练 + in-context learning 范式的强大泛化性

### 局限性
1. **仅中英双语**: 只验证了中英两种语言，未验证到更多语种（如日/韩/法/德）的泛化能力
2. **中文方向较弱**: 中文 TTS 的 WER 高达 8.52，ASV-Score 仅 0.29，明显弱于英文方向（可能因英文训练数据 6 倍于中文）
3. **评测集偏小**: EMIME 仅 25 句 × 14 人 = 350 样本，人工评估仅 50-56 样本，统计显著性可能不足
4. **推理效率未报告**: 论文未报告 RTF 或延迟数据，AR 解码 + 8 层 RVQ 的推理速度可能是实用瓶颈
5. **对比基线偏弱**: [[YourTTS]] 是 2022 年工作，并非当时最强跨语言 TTS 系统；缺少与 [[SeamlessM4T]] 等更强基线的对比

### 潜在改进方向
1. 扩展到更多语言对，验证 N 语种的 scaling 效果
2. 引入流式推理机制降低延迟
3. 用更强的 codec（如 [[DAC]]、[[SpeechTokenizer]]）替换 [[EnCodec]]
4. 中文方向可通过增加中文训练数据或 balanced sampling 改善

### 可复现性评估
- [x] 代码开源（github.com/microsoft/unilm 仓库）
- [ ] 预训练模型（论文未明确开放 checkpoint）
- [x] 训练细节完整（架构、超参、训练步数、硬件均有说明）
- [x] 数据集可获取（LibriLight、WenetSpeech 均公开）

---

## 关联笔记

### 基于
- [[VALL-E]]: 核心基础，VALL-E X 是其跨语言扩展
- [[SpeechUT]]: S2ST 翻译模块的基础预训练框架
- [[EnCodec]]: 声学量化器，将波形压缩为 8 层 [[RVQ]] 离散码

### 对比
- [[YourTTS]]: 跨语言 TTS 基线，基于 VITS + speaker embedding
- [[Translatotron]]: 端到端 S2ST 先驱，但难以保留说话人信息

### 方法相关
- [[Acoustic Token]]: 核心中间表示
- [[RVQ]]: 残差向量量化，8 层码本
- [[Language ID]]: 控制口音的关键模块
- [[Cross-Lingual TTS]]: 跨语言语音合成任务
- [[S2ST]]: 语音到语音翻译任务
- [[In-Context Learning]]: VALL-E 系列的核心能力来源
- [[CTC]]: SpeechUT 微调时的辅助损失
- [[G2P]]: 文本到音素转换
- [[Forced Alignment]]: 训练数据对齐

### 硬件/数据相关
- [[LibriLight]]: 60K 小时英文训练数据
- [[WenetSpeech]]: 10K+ 小时中文训练数据
- [[EMIME]]: 中英双语评测数据集
- [[LibriSpeech]]: 英文评测数据集

---

## 速查卡片

> [!summary] VALL-E X
> - **核心**: 跨语言 codec language model，源语音 prompt + 目标文本 → 保留音色的目标语言语音
> - **方法**: AR(第 1 层 token) + NAR(第 2-8 层 token)，Language ID 消除口音
> - **结果**: SMOS 4.00 vs 3.42 (XTTS)，4.12 vs 3.06 (S2ST)；WER 4.07 vs 8.53
> - **代码**: [microsoft/unilm](https://github.com/microsoft/unilm)

---

*笔记创建时间: 2026-05-25*
