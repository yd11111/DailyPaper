---
title: "NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers"
method_name: "NaturalSpeech2"
authors: [Kai Shen, Zeqian Ju, Xu Tan, Yanqing Liu, Yichong Leng, Lei He, Tao Qin, Sheng Zhao, Jiang Bian]
year: 2023
venue: arXiv
tags: [zero-shot-tts, latent-diffusion, speech-prompting, singing-synthesis, neural-audio-codec, non-autoregressive]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2304.09116
created: 2026-05-25
---

# 论文笔记：NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft Research Asia & Microsoft Azure Speech |
| 日期 | April 2023 |
| 项目主页 | https://speechresearch.github.io/naturalspeech2 |
| 对比基线 | [[YourTTS]], [[VALL-E]], [[Tacotron 2]], [[FastSpeech]] |
| 链接 | [arXiv](https://arxiv.org/abs/2304.09116) |

---

## 一句话总结

> 微软提出在 [[Audio Codec]] 的连续 latent 空间上做 [[Latent Diffusion]]，配合 speech prompting 实现 zero-shot 多说话人语音与歌声合成，规避了离散 token AR 方法的鲁棒性问题。

---

## 核心贡献

1. **连续 latent 表示替代离散 token**: 用 [[RVQ]] codec 的连续向量（而非离散 token）作为中间表示，避免了序列长度 R 倍膨胀和 AR 误差累积
2. **Latent Diffusion 生成**: 用 [[WaveNet]]-based diffusion model 在 latent 空间做 [[Non-Autoregressive]] 生成，彻底消除 word skipping/repeating 问题
3. **Speech Prompting 机制**: 设计 Transformer prompt encoder + Q-K-V attention + FiLM 层，在 duration/pitch predictor 和 diffusion model 中同时注入说话人信息，实现 [[In-Context Learning]]
4. **Zero-shot 歌声合成**: 首次用语音 prompt 直接做零样本歌声合成（不需要歌声 prompt）

---

## 问题背景

### 要解决的问题

大规模 zero-shot TTS 需要在多说话人、多风格数据上训练，生成未见说话人的自然语音。

### 现有方法的局限

当时的大规模 TTS 系统（[[VALL-E]]、[[AudioLM]]）把语音量化为 [[Discrete Audio Token]] 后用 [[Autoregressive Model]] 逐 token 生成，存在两个核心问题：

1. **序列过长导致误差累积**: R 层 [[RVQ]] 产生 R 倍长的离散序列，AR 生成时 error propagation 导致不稳定韵律、word skipping/repeating
2. **低码率 vs 高质量的两难**: 单层 VQ token 易于语言建模但重建质量差；多层 RVQ token 重建好但序列太长

### 本文的动机

- **连续向量**: 每帧只有一个向量（而非 R 个离散 token），序列长度不膨胀
- **Diffusion 替代 AR**: [[Non-Autoregressive]] 生成无 error propagation，天然鲁棒
- **Speech prompting**: 只需语音 prompt（不需配对文本），实现灵活的 [[In-Context Learning]]

---

## 方法详解

### 模型架构

NaturalSpeech 2 采用 **codec encoder-decoder + latent diffusion** 架构：

- **输入**: [[Phoneme]] 序列 + speech prompt $z^p$
- **Codec Encoder**: waveform $x$ → 连续隐表示 $h$ → [[RVQ]] 量化 → 连续向量 $z$
- **Prior Model**: [[Phoneme]] Encoder（[[Transformer]]）+ [[Duration Predictor]] + Pitch Predictor → 条件 $c$
- **Diffusion Model**: [[WaveNet]]-based，在 latent 空间生成 $z$
- **Codec Decoder**: $z$ → waveform $\hat{x}$
- **总参数**: 435M

### 核心模块

#### 模块 1: Neural Audio Codec（连续向量表示）

**设计动机**: 利用 [[RVQ]] 将音频编码为连续向量而非离散 token，兼顾高重建质量和短序列长度。

**具体实现**:
- Encoder: 卷积块，总下采样率 200（16kHz 音频每帧对应 12.5ms）
- [[RVQ]]: 16 层残差量化器，每层 codebook size 1024，embedding dim 256
- 量化后将所有层 embedding 求和得到连续向量 $z^i = \sum_{j=1}^{R} e^i_j$
- 大 $R$ + 大 codebook 的 [[RVQ]] 近似连续分布，同时提供存储效率（只需存 codebook + token ID）和额外的 cross-entropy 正则化

#### 模块 2: Latent Diffusion Model

**设计动机**: 在 codec latent 空间做 [[Latent Diffusion]]，避免在高维 waveform 空间建模。

**具体实现**:
- 40 层 [[WaveNet]]（dilated 1D conv, kernel 3, dilation 2, filter 1024）
- 网络预测 $\hat{z}_0$（而非 score），通过公式转换得到 score
- 训练损失 = data loss + score loss + [[RVQ]] cross-entropy loss（$\lambda_{ce-rvq} = 0.1$）
- [[RVQ]] CE loss: 对每层量化器，将预测残差与 codebook 做 L2 距离 → softmax → 与 ground-truth token ID 做交叉熵

#### 模块 3: Prior Model（Phoneme Encoder + Duration/Pitch Predictor）

**设计动机**: 提供文本条件 $c$，驱动 diffusion 生成正确内容和韵律。

**具体实现**:
- **Phoneme Encoder**: 6 层 [[Transformer]]（8 heads, 512 dim, conv FFN kernel 9）
- **[[Duration Predictor]]**: 30 层 1D conv + 10 层 Q-K-V attention（每 3 层 conv 插一层 attention）
- **Pitch Predictor**: 同架构不同参数（conv kernel 5）
- 训练时用 ground-truth duration expand phoneme → frame 级序列，加上 ground-truth pitch

#### 模块 4: Speech Prompting（In-Context Learning）

**设计动机**: 仅用语音 prompt（无需配对文本）实现 zero-shot 音色/韵律克隆。

**具体实现**:
- 训练时随机截取目标语音片段 $z^{u:v}$ 作为 prompt，剩余部分作为生成目标
- Prompt Encoder: 6 层 [[Transformer]]（与 Phoneme Encoder 同架构）
- **Duration/Pitch Predictor 中的注入**: 在 conv 层插入 Q-K-V attention，query = conv hidden，key/value = prompt hidden
- **Diffusion Model 中的注入**: 两级 attention + [[FiLM]] 层
  - 第一级: $m = 32$ 个可学习 query token attend to prompt hidden → $m$ 长度结果
  - 第二级: [[WaveNet]] hidden 作 query，$m$ 长度结果作 key/value
  - 输出通过 [[FiLM]] 层对 [[WaveNet]] hidden 做 affine transform（scale + bias）

---

## 关键公式

### 公式 1: [[Audio Codec|音频编码与量化]]

$$
h = f_{enc}(x), \quad \{e^i_j\}_{j=1}^{R} = f_{rvq}(h^i), \quad z^i = \sum_{j=1}^{R} e^i_j, \quad \hat{x} = f_{dec}(z)
$$

**含义**: 将 waveform $x$ 编码为帧级隐表示 $h$，经 [[RVQ]] 量化后各层 embedding 求和得到连续向量 $z$，最终解码回波形。

**符号说明**:
- $f_{enc}, f_{rvq}, f_{dec}$: encoder / RVQ / decoder
- $h^i$: 第 $i$ 帧的隐表示
- $e^i_j$: 第 $i$ 帧第 $j$ 层量化器的 codebook embedding
- $R = 16$: 残差量化层数

### 公式 2: [[DDPM|前向 SDE]]

$$
dz_t = -\frac{1}{2}\beta_t z_t \, dt + \sqrt{\beta_t} \, dw_t, \quad t \in [0, 1]
$$

**含义**: 前向扩散过程，逐步向 latent 加噪。

**符号说明**:
- $z_t$: 时刻 $t$ 的 noisy latent
- $\beta_t$: 噪声调度函数
- $w_t$: 标准 Wiener 过程

### 公式 3: [[DDPM|前向 SDE 解]]

$$
z_t = e^{-\frac{1}{2}\int_0^t \beta_s \, ds} \, z_0 + \int_0^t \sqrt{\beta_s} \, e^{-\frac{1}{2}\int_s^t \beta_u \, du} \, dw_s
$$

**含义**: 前向 SDE 的闭式解，条件分布 $p(z_t | z_0) \sim \mathcal{N}(\rho(z_0, t), \Sigma_t)$。

**符号说明**:
- $\rho(z_0, t) = e^{-\frac{1}{2}\int_0^t \beta_s ds} z_0$: 均值
- $\Sigma_t = I - e^{-\int_0^t \beta_s ds}$: 方差

### 公式 4: [[DDPM|逆向 SDE]]

$$
dz_t = -\frac{1}{2}(z_t + \nabla \log p_t(z_t)) \beta_t \, dt + \sqrt{\beta_t} \, d\tilde{w}_t, \quad t \in [0, 1]
$$

**含义**: 去噪过程的随机版本，从噪声恢复 latent。

### 公式 5: [[DDPM|逆向 ODE]]

$$
dz_t = -\frac{1}{2}(z_t + \nabla \log p_t(z_t)) \beta_t \, dt, \quad t \in [0, 1]
$$

**含义**: 去噪过程的确定性版本（probability flow ODE），推理时使用 Euler ODE solver。

### 公式 6: [[Latent Diffusion|Diffusion 损失]]

$$
\mathcal{L}_{diff} = \mathbb{E}_{z_0, t} \Big[ \|\hat{z}_0 - z_0\|_2^2 + \|\Sigma_t^{-1}(\rho(\hat{z}_0, t) - z_t) - \nabla \log p_t(z_t)\|_2^2 + \lambda_{ce\text{-}rvq} \mathcal{L}_{ce\text{-}rvq} \Big]
$$

**含义**: 三项损失——data loss（预测 $\hat{z}_0$ 与真值距离）、score loss（保证 score 估计准确）、RVQ cross-entropy loss（离散正则化）。

**符号说明**:
- $\hat{z}_0 = s_\theta(z_t, t, c)$: 网络预测的去噪结果
- $\lambda_{ce\text{-}rvq} = 0.1$: CE loss 权重
- $\mathcal{L}_{ce\text{-}rvq}$: 对每层量化器预测残差与 codebook 做 softmax → CE

### 公式 7: [[Latent Diffusion|总损失]]

$$
\mathcal{L} = \mathcal{L}_{diff} + \mathcal{L}_{dur} + \mathcal{L}_{pitch}
$$

**含义**: 总训练损失 = diffusion 损失 + duration L1 损失 + pitch L1 损失。

### 公式 8: [[Voice Conversion|Source-Aware Diffusion]]

$$
z_1 = z_0 + \int_0^1 -\frac{1}{2}\Big(z_t + \Sigma_t^{-1}(\rho(\hat{s}_\theta(z_t, t, c), t) - z_t)\Big)\beta_t \, dt
$$

**含义**: 语音转换的第一步——将源语音通过逆向 ODE 的反向（即前向 ODE）映射到噪声空间 $z_1$，保留源语音的内容信息。随后用目标 prompt 做标准去噪，实现音色转换。

---

## 关键图表

### Figure 1: Overview / 系统概览

![Figure 1: NaturalSpeech 2 Overview](https://ar5iv.labs.arxiv.org/html/2304.09116/assets/x1.png)

**说明**: NaturalSpeech 2 的整体架构。左侧为 [[Audio Codec]]（encoder + [[RVQ]] + decoder），右侧为 [[Latent Diffusion]] model，条件来自 [[Phoneme]] Encoder + [[Duration Predictor]] + Pitch Predictor。Speech prompt 通过 prompt encoder 注入到 duration/pitch predictor 和 diffusion model 中。

### Figure 2: Neural Audio Codec

![Figure 2: Neural Audio Codec](https://ar5iv.labs.arxiv.org/html/2304.09116/assets/x2.png)

**说明**: [[Audio Codec]] 的内部结构。Encoder 将 waveform 下采样 200 倍得到帧级隐表示，经 16 层 [[RVQ]]（每层 codebook 1024）量化后各层 embedding 求和得到连续向量 $z$，Decoder 重建波形。

### Figure 3: Speech Prompting Mechanism

![Figure 3: Speech Prompting](https://ar5iv.labs.arxiv.org/html/2304.09116/assets/x3.png)

**说明**: Speech Prompting 的两种注入方式。**左侧**：在 [[Duration Predictor]]/Pitch Predictor 中插入 Q-K-V attention（query = conv hidden, key/value = prompt hidden）。**右侧**：在 [[WaveNet]] diffusion model 中使用 32 个可学习 query token 的两级 attention + [[FiLM]] 层。

### Figure 4: WaveNet Architecture in Diffusion Model

![Figure 4: WaveNet Architecture](https://ar5iv.labs.arxiv.org/html/2304.09116/assets/x4.png)

**说明**: Diffusion model 中 [[WaveNet]] 的详细架构。40 个 block，每个 block 包含 dilated CNN（kernel 3, dilation 2）、Q-K-V attention to prompt encoder output、[[FiLM]] 层（scale/bias from attention results）。Skip outputs 平均后作为最终输出。

### Table 1: NaturalSpeech 2 与前代系统对比

| 特性 | 前代系统 (VALL-E/AudioLM) | NaturalSpeech 2 |
|------|--------------------------|-----------------|
| 表示 | [[Discrete Audio Token]] | 连续向量 |
| 生成模型 | [[Autoregressive Model]] | [[Non-Autoregressive]] / Diffusion |
| In-Context Learning | 需要文本+语音 | 仅需语音 |
| 稳定性/鲁棒性 | 差 | 好 |
| 单一声学模型 | 否 | 是 |
| 支持歌声合成 | 否 | 是 |

### Table 2: 离散 token 的两难困境

| 困境 | 单层 VQ | 多层 RVQ |
|------|---------|----------|
| 波形重建（Codec） | 难 | 易 |
| Token 生成（AR LM） | 易 | 难 |

**说明**: 这是 NaturalSpeech 2 用连续向量绕开的核心困境——单层 token 重建质量差，多层 token 序列过长。

### Table 3: CMOS 结果（对比 NaturalSpeech 2）

| 系统 | [[LibriSpeech]] | [[VCTK]] |
|------|----------------|----------|
| Ground Truth | +0.04 | -0.30 |
| [[YourTTS]] | -0.65 | -0.58 |
| NaturalSpeech 2 | 0.00 | 0.00 |

**说明**: NaturalSpeech 2 在 [[LibriSpeech]] 上与真实语音持平（CMOS +0.04），在 [[VCTK]] 上**超越**真实语音（GT 为 -0.30）。

### Table 4: 韵律相似度（合成 vs Prompt）

**[[LibriSpeech]]:**

| 系统 | Pitch Mean ↓ | Pitch Std ↓ | Pitch Skew ↓ | Pitch Kurt ↓ | Dur Mean ↓ | Dur Std ↓ | Dur Skew ↓ | Dur Kurt ↓ |
|------|-------------|-------------|--------------|--------------|------------|-----------|------------|------------|
| [[YourTTS]] | 10.52 | 7.62 | 0.59 | 1.18 | 0.84 | 0.66 | 0.75 | 3.70 |
| **NaturalSpeech 2** | **10.11** | **6.18** | **0.50** | **1.01** | **0.65** | 0.70 | **0.60** | **2.99** |

**[[VCTK]]:**

| 系统 | Pitch Mean ↓ | Pitch Std ↓ | Pitch Skew ↓ | Pitch Kurt ↓ | Dur Mean ↓ | Dur Std ↓ | Dur Skew ↓ | Dur Kurt ↓ |
|------|-------------|-------------|--------------|--------------|------------|-----------|------------|------------|
| [[YourTTS]] | 13.67 | 6.63 | 0.72 | 1.54 | 0.72 | 0.85 | 0.84 | 3.31 |
| **NaturalSpeech 2** | **13.29** | **6.41** | **0.68** | **1.27** | 0.79 | **0.76** | **0.76** | **2.65** |

**说明**: NaturalSpeech 2 在几乎所有韵律统计量上优于 [[YourTTS]]，即使 YourTTS 在训练时见过 VCTK 的 97/108 个说话人。

### Table 5: SMOS 说话人相似度

| 系统 | [[LibriSpeech]] | [[VCTK]] |
|------|----------------|----------|
| Ground Truth | 3.33 | 3.86 |
| [[YourTTS]] | 2.03 | 2.43 |
| **NaturalSpeech 2** | **3.28** | **3.20** |

**说明**: NaturalSpeech 2 的说话人相似度分别超过 [[YourTTS]] 1.25 和 0.77 分。

### Table 6: Word Error Rate

| 系统 | [[LibriSpeech]] | [[VCTK]] |
|------|----------------|----------|
| Ground Truth | 1.94 | 9.49 |
| [[YourTTS]] | 7.10 | 14.80 |
| **NaturalSpeech 2** | **2.26** | **6.99** |

**说明**: NaturalSpeech 2 的 [[WER]] 接近真实语音水平（LibriSpeech 2.26 vs GT 1.94），远优于 [[YourTTS]]。ASR 模型为 CTC-based [[HuBERT]]（Librilight 预训练 + LibriSpeech 960h 微调）。

### Table 7: 50 条困难句子鲁棒性测试

| AR/NAR | 模型 | Repeats | Skips | Error Sentences | Error Rate |
|--------|------|---------|-------|-----------------|------------|
| AR | [[Tacotron 2]] | 4 | 11 | 12 | 24% |
| AR | [[Transformer TTS]] | 7 | 15 | 17 | 34% |
| NAR | [[FastSpeech]] | 0 | 0 | 0 | 0% |
| NAR | NaturalSpeech | 0 | 0 | 0 | 0% |
| NAR | **NaturalSpeech 2** | **0** | **0** | **0** | **0%** |

**说明**: 所有 [[Non-Autoregressive]] 模型在困难句子上零错误，而 AR 模型（[[Tacotron 2]]、[[Transformer TTS]]）有 24-34% 的错误率。

### Table 8: 与 VALL-E 对比

| 系统 | [[SMOS]] | [[CMOS]] |
|------|----------|----------|
| Ground Truth | 4.09 | - |
| [[VALL-E]] | 3.53 | -0.31 |
| **NaturalSpeech 2** | **3.83** | **0.00** |

**说明**: NaturalSpeech 2 在说话人相似度上超过 [[VALL-E]] 0.3 分（SMOS），音质超过 0.31 分（CMOS）。

### Table 9: 消融实验——韵律相似度（合成 vs Prompt, LibriSpeech）

| 配置 | Pitch Mean ↓ | Pitch Std ↓ | Pitch Skew ↓ | Pitch Kurt ↓ | Dur Mean ↓ | Dur Std ↓ | Dur Skew ↓ | Dur Kurt ↓ |
|------|-------------|-------------|--------------|--------------|------------|-----------|------------|------------|
| **NaturalSpeech 2** | **10.11** | **6.18** | **0.50** | **1.01** | **0.65** | **0.70** | **0.60** | **2.99** |
| w/o diff prompt | - | - | - | - | - | - | - | - |
| w/o dur/pitch prompt | 21.69 | 19.38 | 0.63 | 1.29 | 0.77 | 0.72 | 0.70 | 3.70 |
| w/o CE loss | 10.69 | 6.24 | 0.55 | 1.06 | 0.71 | 0.72 | 0.74 | 3.85 |
| w/o query attn | 10.78 | 6.29 | 0.62 | 1.37 | 0.67 | 0.71 | 0.69 | 3.59 |

**关键发现**:
- **w/o diff prompt**: 模型完全无法收敛（"-"），说明 diffusion model 中的 speech prompt 是系统核心
- **w/o dur/pitch prompt**: pitch mean 从 10.11 恶化到 21.69，韵律克隆能力严重退化
- **w/o CE loss**: 小幅退化，RVQ CE 正则有稳定增益
- **w/o query attn**: 替换为简单 attention 后小幅退化，说明 32 query token 的 bottleneck 设计有效

### Table 10: Prompt 长度消融

**[[LibriSpeech]]:**

| Prompt 长度 | Pitch Mean ↓ | Pitch Std ↓ | Dur Mean ↓ | Dur Std ↓ |
|------------|-------------|-------------|------------|-----------|
| 3s | 10.11 | 6.18 | 0.65 | 0.70 |
| 5s | 6.96 | 4.29 | 0.69 | 0.60 |
| 10s | 6.90 | 4.03 | 0.62 | 0.45 |

**[[VCTK]]:**

| Prompt 长度 | Pitch Mean ↓ | Pitch Std ↓ | Dur Mean ↓ | Dur Std ↓ |
|------------|-------------|-------------|------------|-----------|
| 3s | 13.29 | 6.41 | 0.79 | 0.76 |
| 5s | 14.46 | 5.47 | 0.62 | 0.67 |
| 10s | 10.28 | 4.31 | 0.71 | 0.62 |

**说明**: 更长的 prompt 提供更多说话人韵律细节，相似度持续提升。5s→10s 的提升小于 3s→5s。

### Table 11: 详细模型配置

| 模块 | 配置 | 值 | 参数量 |
|------|------|-----|--------|
| Audio Codec | RVQ Blocks / Codebook Size / Dim / Hop | 16 / 1024 / 256 / 200 | 27M |
| Phoneme Encoder | Transformer 6L / 8H / 512D / Conv 2048 | kernel 9, dropout 0.2 | 72M |
| [[Duration Predictor]] | Conv1D 30L / Attn 10L / 8H / 512D | kernel 3, dropout 0.5 | 34M |
| Pitch Predictor | Conv1D 30L / Attn 10L / 8H / 512D | kernel 5, dropout 0.5 | 50M |
| Speech Prompt Encoder | Transformer 6L / 8H / 512D / Conv 2048 | kernel 9, dropout 0.2 | 69M |
| Diffusion Model ([[WaveNet]]) | 40L / Attn 13L / 8H / 512D / 32 query | dilation 2, dropout 0.2 | 183M |
| **总计** | | | **435M** |

### Table 12: 韵律相似度（合成 vs Ground Truth）

**[[LibriSpeech]]:**

| 系统 | Pitch Corr ↑ | Pitch RMSE ↓ | Dur Corr ↑ | Dur RMSE ↓ |
|------|-------------|--------------|------------|------------|
| [[YourTTS]] | 0.77 | 51.78 | 0.52 | 3.24 |
| **NaturalSpeech 2** | **0.81** | **47.72** | **0.65** | **2.72** |

**[[VCTK]]:**

| 系统 | Pitch Corr ↑ | Pitch RMSE ↓ | Dur Corr ↑ | Dur RMSE ↓ |
|------|-------------|--------------|------------|------------|
| [[YourTTS]] | 0.82 | 42.63 | 0.55 | 2.55 |
| **NaturalSpeech 2** | **0.87** | **39.83** | **0.64** | **2.50** |

### Table 13: 消融实验——韵律相似度（合成 vs Ground Truth）

| 配置 | Pitch Corr ↑ | Pitch RMSE ↓ | Dur Corr ↑ | Dur RMSE ↓ |
|------|-------------|--------------|------------|------------|
| **NaturalSpeech 2** | **0.81** | **47.72** | **0.65** | **2.72** |
| w/o diff prompt | - | - | - | - |
| w/o dur/pitch prompt | 0.80 | 55.00 | 0.59 | 2.76 |
| w/o CE loss | 0.79 | 50.69 | 0.63 | 2.73 |
| w/o query attn | 0.79 | 50.65 | 0.63 | 2.73 |

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[MLS]] (English) | 44K hours, 5490 speakers | 多说话人有声书，16kHz | 训练 |
| [[LibriSpeech]] test-clean | 40 speakers, 5.4h → 600 utterances | 未见说话人 | 评测 |
| [[VCTK]] | 108 speakers → 540 utterances | 未见说话人 | 评测 |
| 歌声数据 | ~30h | 网络爬取，去伴奏 | 歌声合成训练 |

### 实现细节

- **Codec 训练**: 8 V100 16GB, batch 200 audios/GPU, 440K steps, Adam lr 2e-4
- **Diffusion 训练**: 16 V100 32GB, batch 6K frames/GPU, 300K steps（仍欠拟合）, AdamW lr 5e-4, 32K warmup, inverse sqrt schedule
- **[[G2P]]**: grapheme-to-phoneme 转换 + 内部对齐工具获取 duration; PyWorld 提取 frame-level pitch
- **推理**: temperature $\tau = 1.2^2$, terminal $z_T \sim \mathcal{N}(0, \tau^{-1}I)$, Euler ODE solver, 150 diffusion steps
- **歌声合成推理**: 1000 diffusion steps, lr 5e-5 微调

### 评测指标详情

- **韵律相似度（vs Prompt）**: phoneme-level duration/pitch → 计算 mean, std, skewness, kurtosis → 差值
- **韵律相似度（vs GT）**: Pearson correlation + RMSE
- **[[WER]]**: CTC-based [[HuBERT]]（Librilight 预训练 + LibriSpeech 960h 微调）
- **[[CMOS]]**: 12 名母语评测员
- **[[SMOS]]**: 6 名母语评测员（说话人相似度）
- **鲁棒性**: 50 条包含特殊字符、长数字、URL 等的困难句子

### 扩展应用

#### Zero-shot 歌声合成

- 混合 ~30h 歌声数据和语音数据训练
- 用语音 prompt 即可合成歌声（无需歌声 prompt）
- 需要 ground-truth pitch/duration（来自另一歌声）

#### [[Voice Conversion]]

- Source-aware diffusion: 将源语音通过前向 ODE 映射到噪声空间 $z_1$
- Target-aware denoising: 从 $z_1$ 出发，用目标 prompt 做去噪，实现音色转换
- 保留源语音的内容和韵律

#### 语音增强

- 源语音和 prompt 都含噪声
- Source-aware diffusion 用带噪输入，target-aware denoising 用干净 prompt
- 可同时去噪和保持韵律/音色

---

## 批判性思考

### 优点
1. **概念优雅**: 用连续向量 + diffusion 一步解决了离散 token AR 方法的序列膨胀和误差累积两个核心问题
2. **鲁棒性出色**: 50 条困难句子零错误，[[WER]] 2.26 接近真实语音（1.94），远优于 AR 方法
3. **统一框架**: 同一模型支持 TTS、[[Voice Conversion]]、语音增强、歌声合成四种任务
4. **消融充分**: 4 个关键组件的消融 + prompt 长度消融，清晰证明了各模块贡献

### 局限性
1. **推理速度慢**: 150 diffusion steps（歌声 1000 steps），RTF 未报告，但必然远大于 [[VALL-E]] 等 AR 方法的实时率。论文提到未来可用 consistency model 加速，但当时未实现
2. **300K steps 仍欠拟合**: 作者明确指出模型未完全收敛，暗示可能需要更多算力
3. **评测局限**: 与 [[VALL-E]] 的对比仅用 demo page 的 16 条音频，非系统性公平对比；未与 [[Voicebox]]（同期工作）对比
4. **歌声合成依赖 GT pitch/duration**: 需要目标歌声的 pitch 和 duration 作为输入，非完全端到端
5. **无 SIM-O/SECS 客观指标**: 说话人相似度仅用主观 SMOS，缺少 [[SIM-O]] 等客观 speaker embedding 指标
6. **数据集单一**: 仅用英文 [[MLS]]（有声书），未验证多语种和 in-the-wild 数据

### 潜在改进方向
1. 用 [[Conditional Flow Matching]] 或 consistency distillation 加速推理（已在 [[NaturalSpeech 3]] 中实现）
2. 加入 [[SIM-O]] / speaker embedding cosine similarity 客观评测
3. 在 [[Seed-TTS-eval]] 等标准化评测集上对比
4. 探索无需 GT pitch/duration 的端到端歌声合成

### 可复现性评估
- [ ] 代码开源（未开源）
- [ ] 预训练模型（未开源）
- [x] 训练细节完整（配置表详尽）
- [x] 数据集可获取（[[MLS]] 公开可用）

---

## 关联笔记

### 基于
- [[NaturalSpeech]]: 前作，single-speaker TTS 达到人类水平
- [[EnCodec]]: codec 架构的基础参考
- [[SoundStream]]: 另一主要 codec 参考
- [[AudioLM]]: 提出 semantic→acoustic 两阶段生成范式

### 对比
- [[VALL-E]]: 直接对标的 zero-shot TTS 系统（离散 token + AR）
- [[YourTTS]]: 主要实验基线（端到端多说话人 TTS）
- [[Voicebox]]: 同期工作，Meta 的 [[Conditional Flow Matching]] 方法

### 后续
- [[NaturalSpeech 3]]: 继任者，用 DiT + [[Conditional Flow Matching]] 替代 WaveNet + diffusion

### 方法相关
- [[RVQ]]: 核心量化方法
- [[WaveNet]]: diffusion backbone
- [[Latent Diffusion]]: 核心生成范式
- [[Duration Predictor]]: 韵律预测组件
- [[FiLM]]: 条件注入机制
- [[In-Context Learning]]: speech prompting 的理论基础

### 数据集
- [[MLS]]: 训练数据（44K hours）
- [[LibriSpeech]]: 评测数据
- [[VCTK]]: 评测数据

---

## 速查卡片

> [!summary] NaturalSpeech 2
> - **核心**: 在 RVQ codec 的连续 latent 空间做 diffusion，配合 speech prompting 实现 zero-shot TTS
> - **方法**: Neural Audio Codec（16 层 RVQ）+ WaveNet Latent Diffusion + Speech Prompt Encoder（Transformer + FiLM）
> - **结果**: CMOS 与 GT 持平，SMOS 3.28（LibriSpeech），WER 2.26（接近 GT 1.94），50 条困难句子零错误，超越 VALL-E 0.3 SMOS
> - **规模**: 435M params, 44K hours MLS, 5490 speakers
> - **Demo**: https://speechresearch.github.io/naturalspeech2

---

*笔记创建时间: 2026-05-25*
