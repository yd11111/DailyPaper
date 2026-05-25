---
title: "CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training"
method_name: "CosyVoice3"
authors: [Zhihao Du, Changfeng Gao, Yuxuan Wang, Fan Yu, Tianyu Zhao, Hao Wang, Xiang Lv, Hui Wang, Chongjia Ni, Xian Shi, Keyu An, Guanrou Yang, Yabin Li, Yanni Chen, Zhifu Gao, Qian Chen, Yue Gu, Mengzhe Chen, Yafeng Chen, Shiliang Zhang, Wen Wang, Jieping Ye]
year: 2025
venue: arXiv
tags: [tts, zero-shot-tts, speech-tokenizer, post-training, multilingual, scaling, reinforcement-learning, flow-matching]
image_source: online
arxiv_html: https://arxiv.org/html/2505.17589v2
created: 2026-05-25
---

# 论文笔记：CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Speech Team, Tongyi Lab, Alibaba Group |
| 日期 | May 2025 |
| 项目主页 | [cosyvoice3 demo](https://funaudiollm.github.io/cosyvoice3) |
| 对比基线 | [[CosyVoice 2]], [[F5-TTS]], [[MaskGCT]], [[Seed-TTS]], [[Spark-TTS]], [[FireRedTTS]], [[Qwen2.5-Omni]] |
| 链接 | [arXiv](https://arxiv.org/abs/2505.17589) / [Code](https://github.com/FunAudioLLM/CosyVoice) |

---

## 一句话总结

> 阿里通义 CosyVoice 系列第三代：通过监督多任务 [[Speech Tokenizer]]、可微分奖励优化 (DiffRO) 后训练、百万小时数据 + 1.5B 参数 scaling，实现 9 语言 18 方言的 in-the-wild 零样本语音合成 SOTA。

---

## 核心贡献

1. **监督多任务 Speech Tokenizer**: 基于 [[MinMo]] 大规模语音理解模型，通过 [[ASR]]/SER/LID/AED/SA 五任务联合训练 + [[FSQ]] 量化，生成 25 Hz 语义 token，同时编码内容和副语言信息
2. **DiffRO 后训练**: 提出 [[DiffRO]]（Differentiable Reward Optimization），用 [[Gumbel-Softmax]] 使 LLM 输出的离散 token 采样可微，通过 ASR 后验概率等作为奖励信号直接反向传播优化，通用适用于 LLM-based TTS
3. **数据 + 模型 Scaling**: 训练数据从约 10K 小时扩至 100 万小时（9 语言 + 18 中国方言），LLM 从 0.5B 扩至 1.5B、CFM 从 100M 扩至 300M（采用 [[DiT]] backbone）
4. **CV3-Eval 多语言评测集**: 发布覆盖 9 语言跨语言/情感/方言/困难样本的标准化 benchmark
5. **Pronunciation Inpainting + 自训练 TN**: 解决 BPE tokenizer 发音不可控和文本归一化覆盖不足的工程问题

---

## 问题背景

### 要解决的问题

[[CosyVoice 2]] 虽然实现了人类水平合成质量和超低延迟流式推理，但在 in-the-wild 场景下面临五大局限：
- 语言覆盖不足（仅中英日韩）
- 领域多样性差（缺少导航/金融/电商等场景）
- 数据量受限（约 170K 小时）
- 文本格式限制（难以处理数字/缩写/特殊符号）
- 缺乏后训练对齐机制

### 现有方法的局限

- **LLM-based 离散 token 方法**（[[VALL-E]]、[[MaskGCT]]）：内容一致性（Content Consistency）是核心挑战，尤其是多语言、罕见词、绕口令场景
- **Diffusion-based 方法**（[[F5-TTS]]、E2 TTS）：语种覆盖受限，说话人相似度偏低
- **RL for TTS**（F5R-TTS）：需要完整走通 CFM + Vocoder 的前向过程，计算量大；且合成语音高度相似导致奖励信号区分度低

### 本文的动机

- 在 [[CosyVoice 2]] 的 LLM + chunk-aware [[Flow Matching]] 架构上，通过更好的 speech tokenizer（从 ASR-only 到多任务监督）+ 后训练（DiffRO）+ 大规模 scaling，系统性地突破上述局限
- 将 LLM 对齐领域的后训练范式迁移到语音生成，且提出的 DiffRO 方法具有通用性

---

## 方法详解

### 模型架构

CosyVoice 3 沿用 [[CosyVoice 2]] 的核心架构，包含两个主要模块：

- **Text-to-Speech LLM**: 自回归预测 speech token 序列，参数量 0.5B 或 1.5B
- **Chunk-aware CFM 模型**: [[Flow Matching|条件 Flow Matching]]，从 speech token → Mel 频谱，参数量 100M→300M（1.5B 版本用 [[DiT]] backbone）
- **Vocoder**: Mel → waveform（沿用 [[HiFi-GAN]] 系声码器）

与 CosyVoice 2 的关键区别：
- 文本编码器和 length regularization 模块被移除
- 帧率不匹配（speech token 25 Hz vs CFM 输出帧率）通过简单插值解决
- CFM 升级为 [[DiT]] backbone（300M 参数）

### 核心模块

#### 模块 1: 监督多任务 Speech Tokenizer

**设计动机**: [[CosyVoice 2]] 的 tokenizer 仅在 [[SenseVoice]]-Large 的 ASR 任务上训练 [[FSQ]]，编码的副语言信息有限。CosyVoice 3 转向基于 [[MinMo]]（140 万小时预训练的多模态语音理解模型）的多任务训练。

**架构**:
- 输入语音 $X$ → Voice Encoder₁（12 层 Transformer + [[RoPE]]）→ 中间表示 $H$
- $H$ → [[FSQ]] 模块量化 → $\hat{H}$
- $\hat{H}$ → Voice Encoder₂ → [[MinMo]] LLM → 文本 token 预测

**训练任务（53 万小时数据）**:
- [[ASR]]（36.5 万小时，多语种）
- LID 语言识别（8.5 万小时）
- SER 语音情感识别（4.8 万小时）
- AED 音频事件检测（2.1 万小时）
- SA 说话人分析（1.1 万小时）

**输出**: 25 Hz speech token 序列，码本大小 $Q = (2K+1)^D$

#### 模块 2: DiffRO（Differentiable Reward Optimization）

**设计动机**: 现有 TTS RL 方法（如 F5R-TTS 的 [[GRPO]]）存在两个问题：(1) 需要前向走完整个 CFM + Vocoder 计算链，开销极大；(2) 合成语音高度相似，奖励信号区分度低。

**核心思路**: 训练一个 ASR-like 的 Token2Text 模型，直接在 speech token 空间计算奖励，绕过 CFM 和 Vocoder。用 [[Gumbel-Softmax]] 使 LLM 的离散采样可微。

**具体流程**:
1. LLM 预测 speech token 的 logits
2. 用 [[Gumbel-Softmax]] 采样得到可微的 token embedding
3. 送入 Token2Text 模型计算 ASR 后验概率作为奖励
4. 通过 KL 散度正则化防止策略偏离参考模型
5. 反向传播优化 LLM 参数

#### 模块 3: Pronunciation Inpainting

**动机**: LLM-based TTS 使用 BPE tokenizer，无法像 phoneme-based 方法精确控制发音。

**方案**: 扩展词表支持词与音素混合序列输入。构建辅助数据：中文单音字替换为拼音，英文单音词替换为 CMU 发音词典音素。训练时模型学习混合序列，推理时用户可对多音字/多音词指定发音。

#### 模块 4: 自训练 Text Normalization

传统规则 TN 覆盖有限。三种方法构建辅助训练数据：
1. 规则 TN 处理 → CosyVoice 2 合成配对
2. Qwen-Max 做 TN → CosyVoice 2 合成配对
3. Qwen-Max 做逆 TN（从已有文本-音频对生成非标准文本）

#### 模块 5: 指令式语音生成

指令跟随数据从 1,500 扩至 5,000 小时，涵盖 100+ 种说话风格（情感/语速/角色/方言等）。支持自然语言指令和细粒度标注（`[laughter]`、`<strong>XXX</strong>` 等）。

#### 模块 6: 说话人微调能力迁移

- **单语→多语**: 辅助数据用录音棚质量的单语数据 + 随机选择的说话人，覆盖所有语言，指令中指定说话人和语言
- **指令生成能力迁移**: 训练数据部分标注说话人 ID，随机 mask 说话人/风格指令防止灾难性遗忘

---

## 关键公式

### 公式 1: [[FSQ|Finite Scalar Quantization]]

$$
\bar{H} = \operatorname{ROUND}(\operatorname{Proj}_{down}(H))
$$

$$
\hat{H} = \operatorname{Proj}_{up}(\bar{H})
$$

**含义**: 将中间表示 $H$ 投影到 $D$ 维低秩空间后量化到 $[-K, K]$ 整数，再投影回原空间。训练时用直通估计（Straight-Through Estimation）近似梯度。

**符号说明**:
- $H$: Voice Encoder₁ 的输出特征
- $\operatorname{Proj}_{down}$: 降维线性投影
- $\operatorname{ROUND}$: 有界取整操作
- $D$: FSQ 低秩维度
- $K$: 每维量化上界

### 公式 2: Speech Token 索引

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j}(2K+1)^j
$$

**含义**: 将量化后的 $D$ 维向量编码为 $(2K+1)$ 进制数的标量索引，得到 speech token $\mu_i$

**符号说明**:
- $\mu_i$: 第 $i$ 帧的 speech token 索引
- $\bar{h}_{i,j}$: 量化后向量的第 $j$ 维
- $(2K+1)^D$: 总码本大小 $Q$

### 公式 3: [[Gumbel-Softmax]] 采样

$$
\tilde{\mu}_t = \operatorname{GumbelSoftmax}\; P_{\pi_\theta}(\mu_t | \mu_{1:t-1}; Y)
$$

**含义**: 用 Gumbel-Softmax 重参数化使 LLM 的离散 token 采样可微分

**符号说明**:
- $\pi_\theta$: LLM 策略（参数 $\theta$）
- $Y$: 输入文本
- $\tilde{\mu}_t$: 可微采样得到的 token embedding

### 公式 4: [[ASR]] 奖励

$$
R_{ASR}(Y) = \log P_{ASR}(\tilde{Y}_n = Y_n | Y_{1:n-1}; \tilde{\mu}_{1:T})
$$

**含义**: 用 Token2Text 模型的后验概率作为内容一致性奖励

**符号说明**:
- $P_{ASR}$: Token2Text 模型的 ASR 后验概率
- $\tilde{Y}_n$: 预测的第 $n$ 个文本 token
- $Y_n$: 真实文本 token
- $\tilde{\mu}_{1:T}$: Gumbel-Softmax 采样的 speech token 序列

### 公式 5: DiffRO 优化目标

$$
\pi_\theta^* = \max_{\pi_\theta} \mathbb{E}[R(Y)] - \beta D_{KL}[\pi_\theta(\mu|Y) \| \pi_{ref}(\mu|Y)]
$$

**含义**: 最大化奖励的同时通过 KL 散度正则化防止策略偏离参考模型，$\beta$ 控制正则化强度

**符号说明**:
- $R(Y)$: 奖励函数
- $\beta$: KL 正则化系数
- $\pi_{ref}$: 参考策略（预训练模型）
- $D_{KL}$: KL 散度

### 公式 6: Token 级 KL 散度

$$
D_{KL}[\pi_\theta(\mu|Y) \| \pi_{ref}(\mu|Y)] = \sum_{t=1}^{T}\sum_{k=0}^{Q} P_{\pi_\theta}(\mu_t=k) \log\frac{P_{\pi_\theta}(\mu_t=k)}{P_{\pi_{ref}}(\mu_t=k)}
$$

**含义**: 在每个时间步 $t$ 的 token 输出 logits 上计算 KL 散度（而非序列级后验），计算量更小且更稳定

**符号说明**:
- $T$: speech token 序列长度
- $Q = (2K+1)^D$: 码本大小

### 公式 7: 多任务奖励（MTR）

$$
R_{MTR}(Y, \{A_i\}_{i=1}^K) = \sum_i \log P_{\text{task}_i}(\tilde{A}_i = A_i | \tilde{\mu})
$$

**含义**: 除 ASR 外，还可加入 SER、MOS 预测、AED 等多个音频理解任务的奖励信号，提升情感保真度等

**符号说明**:
- $A_i$: 第 $i$ 个任务的标签（如情感标签）
- $P_{\text{task}_i}$: 第 $i$ 个任务模型的后验概率

### 公式 8: 音量归一化

$$
\text{normalized\_wav} = \frac{\text{raw\_wav}}{\max(\text{raw\_wav})} \times 0.6
$$

**含义**: 数据管线中的音量标准化步骤，峰值归一化到 0.6

---

## 关键图表

### Figure 1: Content Consistency & Speaker Similarity 性能概览

![Figure 1a](https://arxiv.org/html/2505.17589v2/x1.png)

**(a) Content Consistency**: CosyVoice 3 与竞品模型在 9 语言上的 [[CER]]/[[WER]] 对比。100.00 表示对应模型不支持该语言。

![Figure 1b](https://arxiv.org/html/2505.17589v2/x2.png)

**(b) Speaker Similarity**: 基于 [[WavLM]] embedding 的余弦相似度对比。0.00 表示模型不支持该语言。

**说明**: CosyVoice 3 是唯一覆盖全部 9 语言的系统，在内容一致性和说话人相似度两个维度均全面领先。

### Figure 2: 系统架构与训练流程

![Figure 2](https://arxiv.org/html/2505.17589v2/x3.png)

**说明**: (a) 监督多任务 [[Speech Tokenizer]] 训练过程，Voice Encoder 中间插入 [[FSQ]] 模块，联合 [[ASR]]/LID/SER/AED/SA 五任务训练。(b) CosyVoice 3 完整训练管线：大规模预训练 → DiffRO 后训练 → 持续预训练 → 多说话人微调。虚线框模块仅训练时使用。

### Figure 3: 数据分布

![Figure 3a](https://arxiv.org/html/2505.17589v2/x4.png)

**(a) Minority Languages**: 7 种小语种（日/俄/法/德/西/韩/意）的数据占比分布。

![Figure 3b](https://arxiv.org/html/2505.17589v2/x5.png)

**(b) Chinese Dialects**: 19 种中国口音/方言的数据占比分布。

**说明**: 百万小时训练数据覆盖电商/导航/金融/教育等领域，对话/演讲/歌唱等风格，以及标准文本和非标准文本（数字/缩写等）格式。

### Figure 4: MOS 主观评测

![Figure 4](https://arxiv.org/html/2505.17589v2/x6.png)

**说明**: 200 句中英文零样本克隆的 [[MOS]] 评分（10 名母语评估者，1-5 分，0.5 步长）。CosyVoice 3-0.5B 英文已达人类水平，1.5B 超过人类录音。中文所有版本均 >4.45。

### Figure 5: SFT 模型内容一致性

![Figure 5](https://arxiv.org/html/2505.17589v2/x7.png)

**说明**: CosyVoice 3 和 [[CosyVoice 2]] 的说话人微调（SFT）模型在 [[Seed-TTS-eval]] 设定下的 [[WER]]/[[CER]] 对比。基座模型的改进（数据量/token 质量）直接传导到微调模型。

### Figure 6: 单语→多语能力迁移

![Figure 6](https://arxiv.org/html/2505.17589v2/x8.png)

**说明**: 将单语说话人转为多语种说话人的 [[CER]]/[[WER]] 结果。中/英/德/西/法/意/俄 CER/WER < 4%，日语 ~9%（汉字→假名转换错误），韩语 ~6%（数据量不足）。

### Table 1: 预训练数据中的 100 种说话风格

| 类别 | 风格 |
|------|------|
| 情感/特质 | adventurous, ambitious, angry, artistic, bold, calm, charming, cheerful, confident, curious, determined, happy, hopeful, humble, joyful, mysterious, optimistic, passionate, sad, serious, surprised, wise 等 62 种 |
| 语速/音量 | fast, loud, slow, soft |
| 角色 | adventurer, alchemist, chef, detective, doctor, girl, knight, poet, robot, scholar, warrior, witch 等 20 种 |
| 方言/口音 | 安徽方言、广东方言、重庆方言、四川方言、上海方言、天津方言 等 10 种中国方言 + 中式英语口音、印度英语口音、俄语英语口音 |

**说明**: 指令跟随数据涵盖 100+ 种说话风格类型，支持自然语言指令控制。

### Table 2: 说话人微调指令示例

| 示例 |
|------|
| 你是说话人小明。请讲法语。 |
| You are Speaker B. Please speak German. |

### Table 3: Speech Tokenizer 训练数据

| 任务 | 时长 |
|------|------|
| ASR（多语种） | 365K 小时 |
| LID 语言识别 | 85K 小时 |
| SER 情感识别 | 48K 小时 |
| AED 音频事件检测 | 21K 小时 |
| SA 说话人分析 | 11K 小时 |
| **总计** | **530K 小时** |

**说明**: 总计 53 万小时监督多任务数据用于训练 tokenizer，其中 ASR 覆盖中/英/日/韩/俄/法/德。

### Table 4: SEED-TTS-Eval 零样本 TTS 性能

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ | test-hard SS↑ |
|-------|-------------|------------|-------------|------------|---------------|--------------|
| Human | 1.26 | 0.755 (0.775) | 2.14 | 0.734 (0.742) | - | - |
| Vocoder Resyn. | 1.27 | 0.720 | 2.17 | 0.700 | - | - |
| [[MaskGCT]] | 2.27 | 0.774 (0.752) | 2.62 | 0.714 (0.730) | 10.27 | 0.748 (0.720) |
| E2 TTS | 1.97 | 0.730 | 2.19 | 0.710 | - | - |
| [[F5-TTS]] (32 NFE) | 1.56 | 0.741 (0.794) | 1.83 | 0.647 (0.742) | 8.67 | 0.713 (0.762) |
| F5R-TTS | 1.37 | 0.754 | - | - | 8.79 | 0.718 |
| [[Seed-TTS]] | 1.12 | 0.796 | 2.25 | 0.762 | 7.59 | 0.776 |
| [[FireRedTTS]] | 1.51 | 0.635 (0.653) | 3.82 | 0.460 (0.526) | 17.45 | 0.621 (0.639) |
| [[Qwen2.5-Omni]]-7B | 1.70 | 0.752 | 2.72 | 0.632 | 7.97 | 0.747 |
| Qwen2.5-Omni-7B_RL | 1.42 | 0.754 | 2.33 | 0.641 | 6.54 | 0.752 |
| [[CosyVoice]] | 3.63 | 0.723 (0.775) | 4.29 | 0.609 (0.699) | 11.75 | 0.709 (0.755) |
| [[CosyVoice 2]] | 1.45 | 0.748 (0.806) | 2.57 | 0.652 (0.736) | 6.83 | 0.724 (0.776) |
| [[Spark-TTS]] | 1.20 | 0.672 | 1.98 | 0.584 | - | - |
| **CosyVoice 3-0.5B** | 1.16 | 0.780 (0.840) | 2.02 | 0.718 (0.790) | 6.08 | 0.758 (0.815) |
| **CosyVoice 3-0.5B_RL** | **0.75** | 0.774 (0.836) | 1.76 | 0.695 (0.783) | **5.09** | 0.750 (0.809) |
| **CosyVoice 3-1.5B** | 1.12 | 0.781 (0.837) | 2.21 | 0.720 (0.789) | 5.83 | 0.758 (0.816) |
| **CosyVoice 3-1.5B_RL** | **0.71** | 0.775 (0.836) | **1.45** | 0.695 (0.784) | 5.66 | 0.750 (0.810) |

**说明**: SS 括号外为 [[WavLM]]-based，括号内为 [[ERes2Net]]-based。CosyVoice 3 相比 CosyVoice 2 在 test-zh 上 CER 相对降低 51%，test-en 上 WER 相对降低 44%。DiffRO 后训练贡献 12-35% 的相对提升。

### Table 5: CV3-Eval 多语言语音克隆 CER(%)/WER(%)

| Model | zh | en | ja | ko | de | es | fr | it | ru |
|-------|----|----|----|----|----|----|----|----|-----|
| [[F5-TTS]] | 5.47 | 8.90 | - | - | - | - | - | - | - |
| [[Spark-TTS]] | 5.15 | 11.0 | - | - | - | - | - | - | - |
| GPT-SoVITS | 7.34 | 12.5 | - | - | - | - | - | - | - |
| [[CosyVoice 2]] | 4.08 | 6.32 | 9.13 | 19.7 | - | - | - | - | - |
| + DiffRO | 3.00 | 4.72 | 6.36 | 5.14 | - | - | - | - | - |
| **CosyVoice 3-0.5B** | 3.89 | 5.24 | 10.4 | 12.8 | 7.41 | 4.25 | 12.9 | 6.68 | 6.77 |
| + DiffRO | 2.89 | 3.68 | 5.15 | 4.02 | 4.51 | 2.99 | 8.56 | 2.94 | 3.79 |
| **CosyVoice 3-1.5B** | 3.91 | 4.99 | 7.57 | 5.69 | 6.43 | 4.47 | 11.8 | 10.5 | 6.64 |
| + DiffRO | 3.01 | 3.71 | 5.27 | 4.01 | 3.93 | 3.26 | 8.09 | 2.72 | 4.11 |

**说明**: CosyVoice 3 是唯一覆盖全部 9 语言的系统。DiffRO 在各语言均带来显著提升，韩语改进最大（0.5B: 12.8→4.02，68.6% 相对提升）。

### Table 6: CV3-Eval 困难样本结果

| Model | hard-zh WER↓ | hard-zh SS↑ | hard-zh DNSMOS↑ | hard-en WER↓ | hard-en SS↑ | hard-en DNSMOS↑ |
|-------|------------|-----------|---------------|------------|-----------|---------------|
| [[CosyVoice 2]] | 12.58 | 72.6 | 3.81 | 11.96 | 66.7 | 3.95 |
| + DiffRO | 10.66 | 71.7 | 3.81 | 10.25 | 62.4 | 3.97 |
| CosyVoice 3-0.5B | 14.15 | 78.6 | 3.75 | 9.04 | 75.9 | 3.92 |
| + DiffRO | 8.26 | 77.8 | 3.80 | 7.60 | 73.9 | 3.95 |
| CosyVoice 3-1.5B | 9.77 | 78.5 | 3.79 | 10.55 | 76.1 | 3.95 |
| + DiffRO | **9.06** | **78.2** | **3.81** | **7.56** | **74.6** | **3.95** |

**说明**: 困难样本（罕见词、绕口令、领域术语）上 CosyVoice 3 + DiffRO 大幅领先 CosyVoice 2。说话人相似度提升尤为显著（66.7→74.6）。

### Table 7: CV3-Eval 跨语言语音克隆 WER(%)

| Model | to-zh(en) | to-zh(ja) | to-zh(ko) | to-en(zh) | to-en(ja) | to-en(ko) | to-ja(zh) | to-ja(en) | to-ja(ko) | to-ko(zh) | to-ko(en) | to-ko(ja) |
|-------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|-----------|
| [[CosyVoice 2]] | 13.5 | 48.1 | 7.70 | 6.47 | 17.1 | 11.2 | 13.1 | 14.9 | 5.86 | 24.8 | 21.9 | 21.5 |
| CV3-0.5B | 8.48 | 6.86 | 5.24 | 4.99 | 6.83 | 5.86 | 18.3 | 16.8 | 4.99 | 41.0 | 20.4 | 12.8 |
| + DiffRO | 5.16 | 3.22 | 1.03 | 3.40 | 4.41 | 4.78 | 7.91 | 7.25 | 3.29 | 16.9 | 11.6 | 8.2 |
| CV3-1.5B | 8.01 | 6.78 | 3.30 | 4.32 | 5.39 | 5.94 | 13.7 | 13.4 | 4.19 | 31.6 | 14.0 | 10.5 |
| + DiffRO | **5.09** | **3.05** | **1.06** | **2.98** | **4.20** | **4.19** | **7.08** | **6.80** | **3.93** | **14.4** | **5.87** | **7.92** |

**说明**: CosyVoice 2 的 ja→zh 严重退化（48.1% WER）通过将日文汉字统一转假名解决。CosyVoice 3-1.5B + DiffRO 在全部 12 个跨语言条件下均最优。半数条件下 DiffRO 带来 >50% 相对 WER 提升。

### Table 8: 中英跨语言克隆详细结果

| Model | en2zh WER↓ | en2zh SS↑ | en2zh MOS↑ | zh2en WER↓ | zh2en SS↑ | zh2en MOS↑ |
|-------|----------|---------|----------|----------|---------|----------|
| [[F5-TTS]] | 11.6 | 64.2 | 3.77 | 5.57 | 64.7 | 3.77 |
| [[Spark-TTS]] | 12.4 | 48.4 | 3.65 | 7.36 | 56.7 | 3.61 |
| [[CosyVoice 2]] | 13.5 | 63.3 | 3.87 | 6.47 | 64.3 | 3.75 |
| CosyVoice 3-0.5B | 8.48 | 67.4 | 3.82 | 4.99 | 67.8 | 3.75 |
| **CosyVoice 3-1.5B** | **8.01** | **66.9** | **3.83** | **4.32** | **66.4** | **3.77** |

**说明**: CosyVoice 3 在跨语言 WER 和说话人相似度上全面超越 F5-TTS 和 CosyVoice 2，MOS 持平或略优。

### Table 9: CV3-Eval 情感克隆准确率

| Model | 文本相关 happy | 文本相关 sad | 文本相关 angry | 文本无关 happy | 文本无关 sad | 文本无关 angry |
|-------|-------------|------------|-------------|-------------|------------|-------------|
| [[F5-TTS]] | 0.92 | 0.52 | 0.72 | 0.80 | 0.28 | 0.64 |
| [[Spark-TTS]] | 0.80 | 0.56 | 0.50 | 0.50 | 0.60 | 0.36 |
| GPT-SoVITS | 0.88 | 0.54 | 0.50 | 0.48 | 0.40 | 0.30 |
| [[CosyVoice 2]] | 0.84 | 0.72 | 0.58 | 0.56 | 0.44 | 0.38 |
| CosyVoice 3-0.5B | 0.92 | 0.70 | 0.72 | 0.64 | 0.42 | 0.58 |
| CosyVoice 3-1.5B | 0.86 | 0.64 | 0.72 | 0.64 | 0.44 | 0.48 |
| + DiffRO-EMO | **0.98** | 0.68 | **0.84** | **0.98** | 0.50 | **0.68** |

**说明**: "文本无关"情感克隆远难于"文本相关"，表明 TTS 系统主要从文本语义推断情感。DiffRO-EMO（情感奖励）在 happy 和 angry 上大幅提升，但 sad 仍具挑战。

### Table 10: Tokenizer 上游识别任务对比

| 方法 | C.V. EN | C.V. CN | C.V. JA | C.V. KO | Fleurs EN | Fleurs CN |
|------|---------|---------|---------|---------|-----------|-----------|
| [[SenseVoice]] | 7.70 | 8.67 | - | - | 4.57 | 6.98 |
| [[MinMo]] | 7.36 | 8.56 | - | - | 4.43 | 6.71 |
| VQ-SenseVoice | 18.26 | 11.56 | - | - | 7.65 | 5.03 |
| FSQ-SenseVoice | 10.67 | 7.29 | - | - | 6.58 | 4.43 |
| FSQ-MinMo | 11.36 | 9.21 | 13.90 | 9.78 | 4.46 | **3.35** |

**说明**: [[FSQ]]-MinMo 在 Fleurs CN 上甚至超过未量化的 MinMo（3.35 vs 6.71），表明 FSQ 量化的信息瓶颈对特定任务有正则化效果。VQ（传统 VQ）退化严重（WER 翻倍），FSQ 显著优于 VQ。

### Table 11: AIR-Bench 副语言任务准确率

| 方法 | LID | Gender | Age | Emotion | Vocal Sound | Sound Question |
|------|-----|--------|-----|---------|-------------|----------------|
| [[MinMo]] | 99.2 | 84.8 | 70.1 | 62.4 | 90.7 | 59.1 |
| FSQ-MinMo | 99.2 | 72.8 | 41.8 | **68.4** | 61.3 | 57.7 |

**说明**: FSQ 量化后 LID 无损、情感识别反而提升（62.4→68.4），但性别/年龄/声音事件退化，说明 FSQ 瓶颈选择性保留语义+情感信息，丢弃部分声学细节。

### Table 12: 不同 Tokenizer 的下游 TTS 性能

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ | test-hard SS↑ |
|-------|-------------|------------|-------------|------------|---------------|--------------|
| **3000h 数据** | | | | | | |
| SoundStream (1st VQ) | 14.19 | 0.457 | 25.34 | 0.301 | 27.05 | 0.455 |
| [[HuBERT]] | 18.68 | 0.716 | 6.50 | 0.609 | 33.83 | 0.699 |
| w2v-BERT 2.0 | 2.62 | 0.381 | 6.72 | 0.261 | 23.89 | 0.374 |
| [[CosyVoice 2]] | 1.92 | 0.668 | 7.21 | 0.535 | 15.99 | 0.645 |
| CosyVoice 3-0.5B | **1.68** | **0.710** | **6.60** | **0.614** | 27.60 | **0.679** |
| **170Kh 数据** | | | | | | |
| [[CosyVoice 2]] | 1.45 | 0.806 | 2.57 | 0.736 | 6.83 | 0.776 |
| CosyVoice 3-0.5B | **1.27** | **0.815** | **2.46** | **0.747** | 6.96 | **0.787** |

**说明**: 关键发现：(1) SoundStream 的声学 token 缺乏语义信息，内容一致性极差；(2) [[HuBERT]] 的 [[Semantic Token]] 说话人相似度强但中文 CER 高（语言特异性限制）；(3) 监督 tokenizer 在 CER 和 SS 上同时领先；(4) 3K→170K 小时 scaling 带来 63-75% 相对提升。

### Table 13: Pronunciation Inpainting 纠正率

| 方法 | zh 错误数 | zh 纠正数 | zh 纠正率(%) | en 错误数 | en 纠正数 | en 纠正率(%) |
|------|----------|----------|------------|----------|----------|------------|
| RepAll + MixPhn | 13 | 9 | 69.2 | 11 | 8 | 72.7 |
| RepMono + MixPhn | 15 | 15 | **100** | 9 | 9 | **100** |
| RepMono + CatPhn | 15 | 13 | 86.7 | 8 | 8 | 100 |

**说明**: "RepAll"（全替换为音素）引入 G2P 误差，"RepMono"（仅替换单音字/单音词）确保准确性。RepMono + MixPhn 中英文均达 100% 发音纠正率。

### Table 14: 指令式语音生成结果

| Model | Expresso WER↓ | Expresso SIM↑ | Expresso MOS↑ | Internal WER↓ | Internal SIM↑ | Internal MOS↑ |
|-------|-------------|-------------|-------------|-------------|-------------|-------------|
| GroundTruth | 10.0 | 100 | 3.65 | 8.98 | 100 | 3.47 |
| [[CosyVoice 2]] | 9.42 | 60.98 | 3.54 | 7.75 | 72.99 | 3.53 |
| CosyVoice 3-0.5B | 13.72 | 67.82 | 3.56 | 7.30 | 80.45 | 3.51 |
| CosyVoice 3-1.5B | 13.43 | 68.25 | 3.56 | **7.31** | **81.06** | 3.51 |

**说明**: 风格相似度（SIM）相对 CosyVoice 2 提升约 11%（72.99→81.06）。Expresso 上 WER 偏高，作者归因于 ASR 模型对标准发音的偏好（表达性语音含更多非标准发音）。歌唱尚未纳入支持。

---

## 实验

### 数据集

| 数据集 | 规模 | 用途 |
|--------|------|------|
| 多任务监督数据 | 530K 小时 | Speech Tokenizer 训练 |
| TTS 预训练数据 | 100 万小时（9 语言 + 18 方言） | LLM + CFM 训练 |
| [[Seed-TTS-eval]] | 中/英/困难测试集 | 零样本评测 |
| [[CV3-Eval]] | 9 语言 × 500 样本 + 跨语言 + 情感 + 方言 | 多语言零样本评测 |
| [[Common Voice]] + [[Fleurs]] | 多语种 | 跨语言测试集来源 |
| Expresso | 8 种风格 × 3000 样本 | 指令式 TTS 评测 |

### 数据管线（六步）

1. **语音检测与分割**: 说话人日志 + [[VAD]] + 音频事件检测 → <30s 片段
2. **降噪**: [[MossFormer2]] 降噪 + 帧能量分析筛除截断语句
3. **ASR 转录**: Faster-[[Whisper]] Large-V3 语种识别 → 三模型（Whisper / NVIDIA Canary-1B / [[SeamlessM4T]]-V2-large）交叉验证，平均配对 WER < 15% 则采纳
4. **标点调整**: [[MFA|Montreal Forced Aligner]] 获取词间停顿，≥300ms 加逗号，≤50ms 删标点
5. **音量标准化**: 峰值归一化至 0.6
6. **异常过滤**: 计算音频-文本长度比，去除最小 1% 和最大 5%

### 实现细节

- **Text-to-Speech LLM**: 0.5B / 1.5B 参数
- **CFM 模型**: 100M → 300M 参数（[[DiT]] backbone）
- **Speech Token 帧率**: 25 Hz
- **训练数据**: 100 万小时
- **Tokenizer 训练数据**: 53 万小时（5 任务）
- **内容一致性评测 ASR**: [[Whisper]]-large V3（英文）/ [[Paraformer]]（中文）
- **说话人相似度**: [[ERes2Net]] + [[WavLM]]-based speaker embedding cosine similarity
- **音频质量**: [[DNSMOS]]

### 10 个对比基线

- **NAR**: [[MaskGCT]], E2 TTS, [[F5-TTS]], F5R-TTS
- **AR**: [[Seed-TTS]], [[FireRedTTS]], [[Qwen2.5-Omni]], [[CosyVoice]], [[CosyVoice 2]], [[Spark-TTS]]

---

## 批判性思考

### 优点

1. **系统性工程**：从 tokenizer → 后训练 → 数据管线 → 评测集，全链路闭环，工业可复现性极强
2. **DiffRO 通用性**：在 speech token 空间做 RL 绕过 CFM/Vocoder，计算开销低且理论上可迁移到任何 LLM-based TTS
3. **多任务 tokenizer 设计理念独到**：通过信息瓶颈（[[FSQ]]）强制 token 编码多任务信号，Table 11 显示情感识别反而提升，体现量化正则化效果
4. **评测覆盖全面**：9 语言 + 跨语言 + 情感 + 方言 + 困难样本 + 说话人微调，维度远超同类论文
5. **实事求是**：明确报告 RL 的 reward hacking 问题（SS 下降）、歌唱/音色编辑的局限

### 局限性

1. **无法通过文本指令控制音色（timbre）**: 限制了角色扮演等场景，作者在 Limitations 中承认
2. **歌唱生成不佳**: 训练数据和 tokenizer 均未针对歌唱优化
3. **RL 的 reward hacking**: DiffRO 提升内容一致性但轻微降低说话人相似度，多任务奖励平衡仍是开放问题
4. **1.5B vs 0.5B 优势不明确**: 在部分困难样本上 1.5B 反而不如 0.5B，作者归因于困难样本数据不足，但这意味着 scaling 效率存疑
5. **数据管线不开源**: 100 万小时训练数据和管线工具未开源，纯学术团队难以复现
6. **延迟/RTF 未报告**: 作为 CosyVoice 2 的续作，流式延迟是否受 1.5B 模型影响未提及

### 潜在改进方向

1. 将 DiffRO 的多任务奖励扩展到 speaker similarity reward，解决 reward hacking
2. 加入歌唱数据做联合训练（Codec + LLM 层面）
3. 探索 timbre 编辑的文本指令方案（结合 voice conversion 技术）
4. 在更多低资源语言上验证 supervised tokenizer 的跨语言泛化

### 可复现性评估

- [x] 代码开源（CosyVoice GitHub 仓库）
- [ ] 预训练模型（未明确说明是否发布 CosyVoice 3 权重）
- [ ] 训练细节完整（架构描述完整，但百万小时数据来源未公开）
- [ ] 数据集可获取（CV3-Eval 拟发布，训练数据不可获取）

---

## 关联笔记

### 基于
- [[CosyVoice 2]]: 直接前作，沿用 LLM + chunk-aware flow matching 架构
- [[CosyVoice]]: 系列初代，提出 supervised semantic token
- [[MinMo]]: Speech tokenizer 的基座模型（140 万小时预训练的多模态语音理解 LLM）

### 对比
- [[F5-TTS]]: NAR 代表，纯 Flow Matching、无 phoneme
- [[MaskGCT]]: NAR 代表，masked generative codec transformer
- [[Seed-TTS]]: AR 代表，内容一致性和说话人相似度的主要竞争对手
- [[Spark-TTS]]: 单 LLM + BiCodec 方案
- [[Qwen2.5-Omni]]: 阿里另一条线路，Omni 模型中的 TTS 能力

### 方法相关
- [[FSQ]]: Finite Scalar Quantization，核心量化方法
- [[DiffRO]]: Differentiable Reward Optimization，核心后训练方法
- [[Flow Matching]]: CFM 解码器的生成范式
- [[DiT]]: 1.5B 版本 CFM 的 backbone
- [[Gumbel-Softmax]]: DiffRO 中使离散采样可微的技巧
- [[Speech Tokenizer]]: 离散 speech token 方案
- [[ERes2Net]]: 说话人验证 encoder

### 硬件/数据相关
- [[Seed-TTS-eval]]: 主要评测基准
- [[CV3-Eval]]: 本文提出的多语言评测集
- [[Common Voice]]: 测试集来源
- [[Fleurs]]: 测试集来源

---

## 速查卡片

> [!summary] CosyVoice 3
> - **核心**: 监督多任务 speech tokenizer + DiffRO 后训练 + 百万小时 scaling，实现 9 语言 in-the-wild 零样本 TTS SOTA
> - **方法**: LLM (0.5B/1.5B) AR 预测 speech token → DiT-based CFM (300M) → Vocoder；DiffRO 用 Gumbel-Softmax + Token2Text 奖励做可微 RL
> - **结果**: SEED-TTS-eval test-zh CER 0.71%（CosyVoice 2: 1.45%，↓51%）；test-en WER 1.45%（↓44%）；MOS 中文 4.50+ / 英文达人类水平
> - **代码**: [FunAudioLLM/CosyVoice](https://github.com/FunAudioLLM/CosyVoice)

---

*笔记创建时间: 2026-05-25*
