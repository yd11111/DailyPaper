---
title: "NaturalSpeech 3: Zero-Shot Speech Synthesis with a Factorized Codec and Diffusion Models"
method_name: "NaturalSpeech3"
authors: [Zeqian Ju, Yuancheng Wang, Kai Shen, Xu Tan, Detai Xin, Dongchao Yang, Yanqing Liu, Yichong Leng, Kaitao Song, Siliang Tang, Zhizheng Wu, Tao Qin, Xiang-Yang Li, Wei Ye, Shikun Zhang, Jiang Bian, Lei He, Jinyu Li, Sheng Zhao]
year: 2024
venue: ICML 2024
tags: [zero-shot-tts, factorized-codec, diffusion-tts, speech-disentanglement, attribute-control, discrete-diffusion, neural-codec]
image_source: online
arxiv_html: https://arxiv.org/html/2403.03100v3
created: 2026-05-25
---

# 论文笔记：NaturalSpeech 3: Zero-Shot Speech Synthesis with a Factorized Codec and Diffusion Models

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft Research, Microsoft Azure, USTC, CUHK Shenzhen, Zhejiang University, University of Tokyo, Peking University |
| 日期 | March 2024 (v3: April 2024) |
| 项目主页 | [speechresearch.github.io/naturalspeech3](https://speechresearch.github.io/naturalspeech3) |
| 对比基线 | [[VALL-E]], [[NaturalSpeech2]], [[Voicebox]], [[Mega-TTS 2]], [[UniAudio]], [[StyleTTS 2]], [[HierSpeech++]] |
| 链接 | [arXiv](https://arxiv.org/abs/2403.03100) / [FACodec HF](https://huggingface.co/spaces/amphion/naturalspeech3_facodec) |

---

## 一句话总结

> 通过 FACodec 将语音分解为 content/prosody/timbre/acoustic detail 四个解耦子空间，再用分治式离散扩散模型逐属性生成，首次在多说话人数据集上达到人类水平的零样本 TTS 质量。

---

## 核心贡献

1. **FACodec（Factorized Audio Codec）**: 提出带有 [[Factorized Vector Quantization|FVQ]] 的神经语音编解码器，通过信息瓶颈、监督损失、[[Gradient Reversal Layer|梯度反转]] 和 detail dropout 四种技术，将语音波形分解为 content、prosody、timbre、acoustic detail 四个独立子空间
2. **Factorized Diffusion Model**: 设计分治式离散扩散模型，按顺序为每个属性子空间生成离散 token，每个子模块仅需建模单一维度的变化，大幅降低建模复杂度
3. **人类水平零样本 TTS**: 在 LibriSpeech test-clean 上首次实现与人类录音持平的自然度（CMOS -0.08）、相似度（Sim-O 0.67 vs GT 0.68）和可懂度（WER 1.81 vs GT 1.94）；可扩展到 1B 参数和 200K 小时训练数据

---

## 问题背景

### 要解决的问题

现有大规模 TTS 系统虽然取得了显著进展，但在语音质量、说话人相似度和韵律方面仍与人类水平存在差距。语音信号中包含多种交织的属性（内容、韵律、音色、声学细节），这使得直接生成高质量语音非常困难。

### 现有方法的局限

- **原始波形/Mel 频谱图**: 混杂了所有属性，生成模型需要同时建模所有维度的变化
- **RVQ-based Codec**（如 [[EnCodec]]、[[SoundStream]]）: 虽然将语音离散化，但 RVQ 各层按照粗到细的层级分解，并非真正的属性解耦——第 1 层不仅包含语义，也包含韵律和部分声学信息
- **[[SpeechTokenizer]]**: 用 [[HuBERT]] 蒸馏第 1 层 RVQ 来提取语义，但仍未完全解耦所有属性
- 单一生成模型同时处理所有属性，增加了建模难度

### 本文的动机

**分治法（Divide and Conquer）**: 既然语音包含多个复杂属性，就先将它们分解到独立子空间，再分别用专门的生成模型处理每个属性。这样每个子模型只需建模一个维度的变化，降低了建模复杂度；同时音色直接从参考音频提取（无需生成），减轻了扩散模型的负担。

---

## 方法详解

### 模型架构

NaturalSpeech 3 由两个核心组件组成：

- **FACodec**（Factorized Audio Codec）: 将语音波形编码为四个解耦属性子空间的离散 token
- **Factorized Diffusion Model**: 按序列依次生成各属性的离散 token

五个被建模的属性：
1. **Duration**（时长）: 通过外部对齐工具显式获得（NAR 设计需要）
2. **Prosody**（韵律）: 由 FACodec 的 $\text{FVQ}_p$ 隐式学习
3. **Content**（内容）: 由 FACodec 的 $\text{FVQ}_c$ 隐式学习
4. **Acoustic Detail**（声学细节）: 由 FACodec 的 $\text{FVQ}_d$ 隐式学习
5. **Timbre**（音色）: 由 FACodec 的 Timbre Extractor 提取为全局向量

### 核心模块

#### 模块1: FACodec（Factorized Audio Codec）

**设计动机**: 利用 [[Vector Quantization|向量量化]] 的属性分解能力，将复杂的语音波形解耦为独立子空间，使后续的生成模型只需关注单一属性。

**具体实现**:

- **Speech Encoder**: 卷积块 + 下采样（下采样率 200，16kHz 采样率下每帧 12.5 ms），生成 pre-quantization latent $\mathbf{h}$
- **Timbre Extractor**: [[Transformer]] 编码器，将 $\mathbf{h}$ 转换为全局音色向量 $\mathbf{h}_t$
- **三组 FVQ**:
  - $\text{FVQ}_p$（prosody）: 捕获韵律信息（F0 等）
  - $\text{FVQ}_c$（content）: 捕获语言内容信息（phoneme 等）
  - $\text{FVQ}_d$（acoustic detail）: 捕获声学细节（音质等）
- **Speech Decoder**: 镜像编码器结构（但参数量大得多），prosody + content + detail 的表示先相加，再通过 [[Conditional Layer Normalization|条件层归一化]] 融合 timbre 向量 $\mathbf{h}_t$，输出 $\mathbf{z}$ 解码为波形

**四种解耦技术**:

1. **信息瓶颈（Information Bottleneck）**: 编码器输出先投影到 8 维低维空间再量化，限制每个 code embedding 的信息容量，迫使不同 FVQ 捕获不同属性；量化后再投影回原始维度
2. **监督损失（Supervision）**:
   - Prosody: $\mathbf{z}_p$ 预测逐帧 F0（归一化 z-score）
   - Content: $\mathbf{z}_c$ 预测帧级 phoneme 标签
   - Timbre: $\mathbf{h}_t$ 预测 speaker ID
3. **[[Gradient Reversal Layer|梯度反转]]（Gradient Reversal + Adversarial Classifiers）**:
   - Prosody FVQ: phoneme-GRL 去除内容信息
   - Content FVQ: F0-GRL 去除韵律信息
   - Detail FVQ: phoneme-GRL + F0-GRL 同时去除
   - Speaker-GRL 作用在 $\mathbf{z}_p + \mathbf{z}_c + \mathbf{z}_d$ 上，消除残留音色
4. **Detail Dropout**: 训练时以概率 $p$ 随机 mask $\mathbf{z}_d$，迫使 decoder 仅用 prosody + content + timbre 也能重建语音（质量稍低），增强解耦效果

#### 模块2: Factorized Diffusion Model

**设计动机**: 利用分治策略，将复杂的语音生成分解为多个独立子任务，每个子任务在对应属性的离散 token 空间中做 [[Discrete Diffusion|离散扩散]]。

**生成流水线**（按顺序）:

1. **Duration Diffusion**: 生成各 phoneme 的时长 → [[Length Regulator]] 生成帧级 phoneme 条件 $\mathbf{c}_{ph}$
2. **Prosody Diffusion**: 以 prosody prompt + $\mathbf{c}_{ph}$ 为条件，生成 $\mathbf{z}_p$
3. **Content Diffusion**: 以 content prompt + $\mathbf{z}_p$ + $\mathbf{c}_{ph}$ 为条件，生成 $\mathbf{z}_c$
4. **Acoustic Detail Diffusion**: 以 detail prompt + $\mathbf{z}_p$ + $\mathbf{z}_c$ + $\mathbf{c}_{ph}$ 为条件，生成 $\mathbf{z}_d$

**上下文学习（In-Context Learning）**: 通过 partial noising 实现零样本——将 speech prompt 通过 FACodec 分解为各属性 prompt，推理时目标序列与 prompt 拼接（prompt 无噪声 + 目标有噪声），逐步去噪生成。

**Timbre 不需要生成**: 直接从参考音频通过 FACodec 的 Timbre Extractor 提取 $\mathbf{h}_t$，这是属性分解带来的关键优势之一。

---

## 关键公式

### 公式1: [[Discrete Diffusion|离散扩散前向过程]] — Masking Schedule

$$
\mathbf{X}_t = \mathbf{X} \odot \mathbf{M}_t
$$

**含义**: 在时间步 $t$，通过 mask $\mathbf{M}_t$ 将部分 token 替换为 [MASK]，模拟加噪过程。

**符号说明**:
- $\mathbf{X} = [x_i]_{i=1}^N$: 目标离散 token 序列（长度 $N$）
- $\mathbf{M}_t = [m_{t,i}]_{i=1}^N$: 二值 mask，$m_{t,i} \sim \text{Bernoulli}(\sigma(t))$
- $m_{t,i} = 1$: $x_i$ 被替换为 [MASK]；$m_{t,i} = 0$: 保持不变

### 公式2: [[Discrete Diffusion|Masking 调度函数]]

$$
\sigma(t) = \sin\left(\frac{\pi t}{2T}\right), \quad t \in (0, T]
$$

**含义**: 控制 mask 比例随时间单调递增的调度函数。$t=0$ 时 $\sigma=0$（原始序列），$t=T$ 时 $\sigma=1$（完全 mask）。

**符号说明**:
- $t$: 扩散时间步
- $T$: 总扩散步数
- $\sigma(t) \in (0, 1]$: mask 概率，单调递增

### 公式3: [[Discrete Diffusion|训练损失]] — Masked Token Prediction

$$
\mathcal{L}_{\text{mask}} = \mathbb{E}_{\mathbf{X} \in \mathcal{D},\, t \in [0,T]} \left[ -\sum_{i=1}^{N} m_{t,i} \cdot \log p_\theta(x_i \mid \mathbf{X}_t, \mathbf{X}^p, \mathbf{C}) \right]
$$

**含义**: 仅对被 mask 的位置计算负对数似然，训练模型预测被 mask 的 token。

**符号说明**:
- $\mathcal{D}$: 训练数据集
- $\mathbf{X}^p$: 属性 prompt token 序列
- $\mathbf{C}$: 条件（如 phoneme 编码）
- $p_\theta(x_i \mid \mathbf{X}_t, \mathbf{X}^p, \mathbf{C})$: 模型对第 $i$ 个 token 的预测概率

### 公式4: [[Discrete Diffusion|反向采样]] — Reverse Transition

$$
p(\mathbf{X}_{t-\Delta t} \mid \mathbf{X}_t, \mathbf{X}^p, \mathbf{C}) = \mathbb{E}_{\hat{\mathbf{X}}_0 \sim p_\theta(\mathbf{X}_0 \mid \mathbf{X}_t, \mathbf{X}^p, \mathbf{C})} \left[ q(\mathbf{X}_{t-\Delta t} \mid \hat{\mathbf{X}}_0, \mathbf{X}_t) \right]
$$

**含义**: 反向过程分两步——先预测 $\hat{\mathbf{X}}_0$，再根据当前 mask 状态重新选择要保留/重新 mask 的 token。

**符号说明**:
- $\hat{\mathbf{X}}_0$: 模型预测的无噪声 token 序列
- $q(\mathbf{X}_{t-\Delta t} \mid \hat{\mathbf{X}}_0, \mathbf{X}_t)$: 基于预测的后验 re-masking 分布
- 推理时：根据预测置信度 $p_\theta(\hat{x}_i \mid \mathbf{X}_t, \mathbf{X}^p, \mathbf{C})$ 排序，将置信度最低的 $\lfloor N \cdot \sigma(t - \Delta t) \rfloor$ 个 token 重新 mask

### 公式5: [[Classifier-Free Guidance|CFG]] 推理增强

$$
g_{\text{cfg}} = g_{\text{cond}} + \alpha \cdot (g_{\text{cond}} - g_{\text{uncond}})
$$

$$
g_{\text{final}} = \frac{\text{std}(g_{\text{cond}})}{\text{std}(g_{\text{cfg}})} \cdot g_{\text{cfg}}
$$

**含义**: 训练时以概率 $p_{\text{cfg}} = 0.15$ 丢弃 prompt，推理时用有条件/无条件 logit 的差值增强生成引导，再用标准差重缩放避免分布偏移。

**符号说明**:
- $g_{\text{cond}} = g(\mathbf{X} \mid \mathbf{X}^p)$: 有 prompt 条件的 logit
- $g_{\text{uncond}} = g(\mathbf{X})$: 无 prompt 条件的 logit
- $\alpha$: 引导强度（guidance scale = 1.0）

---

## 关键图表

### Figure 1: Overview / 系统概览 + 扩展性

![Figure 1](https://speechresearch.github.io/naturalspeech3/pics/overview.png)

**说明**: (a) NaturalSpeech 3 整体架构概览——FACodec 将语音分解为 content / prosody / timbre / acoustic detail 四个子空间，Factorized Diffusion Model 按序列为各子空间生成离散 token，最后通过 FACodec Decoder 重建波形。(b) 数据和模型扩展实验：从 1K→60K→200K 小时数据持续提升，从 500M→1B 参数持续提升。

### Figure 2: FACodec 框架 / 属性分解

![Figure 2](https://ar5iv.labs.arxiv.org/html/2403.03100/assets/x2.png)

**说明**: FACodec 的详细框架——Speech Encoder 将波形编码为 latent $\mathbf{h}$，Timbre Extractor 提取全局音色向量 $\mathbf{h}_t$，三组 FVQ 分别量化 prosody / content / acoustic detail。解耦通过信息瓶颈（8 维投影）、监督损失（F0 / phoneme / speaker ID）、梯度反转和 detail dropout 四种技术实现。Decoder 将三路 $\mathbf{z}$ 相加后通过条件层归一化融合 timbre 重建波形。

### Figure 3: Factorized Diffusion Model / 分解扩散模型

![Figure 3](https://ar5iv.labs.arxiv.org/html/2403.03100/assets/x3.png)

**说明**: 分解扩散模型的详细框架，包含 5 个模块：(1) Phoneme Encoder 编码文本输入，(2) Duration Diffusion + [[Length Regulator]] 生成帧级条件，(3) Prosody Diffusion 生成韵律 token，(4) Content Diffusion 生成内容 token，(5) Detail Diffusion 生成声学细节 token。模块 2-5 共享相同的离散扩散公式，按顺序依次生成，每步以前一步的输出为额外条件。

### Table 1: LibriSpeech test-clean 主实验结果

| System | Training Data | Sim-O | Sim-R | WER | CMOS | SMOS |
|--------|---------------|-------|-------|-----|------|------|
| Ground Truth | - | 0.68 | - | 1.94 | +0.08 | 3.85 |
| VALL-E | Librilight | 0.47 | 0.51 | 6.11 | -0.60 | 3.46 |
| NaturalSpeech 2 | Librilight | 0.55 | 0.62 | 1.94 | -0.18 | 3.65 |
| Voicebox (self-collected 60kh) | Self-Collected | 0.64 | 0.67 | 2.03 | -0.23 | 3.69 |
| Voicebox (Librilight) | Librilight | 0.48 | 0.50 | 2.14 | -0.32 | 3.52 |
| Mega-TTS 2 | Librilight | 0.53 | - | 2.32 | -0.20 | 3.63 |
| UniAudio | Mixed (165kh) | 0.57 | 0.68 | 2.49 | -0.25 | 3.71 |
| StyleTTS 2 | LT+V+LJ | 0.38 | - | 2.49 | -0.21 | 3.07 |
| HierSpeech++ | LT+LL+EX+MS+NI | 0.51 | - | 6.33 | -0.41 | 3.50 |
| **NaturalSpeech 3** | **Librilight** | **0.67** | **0.76** | **1.81** | **0.00** | **4.01** |

**表格说明**: NaturalSpeech 3 在所有客观和主观指标上全面超越所有基线。Sim-O 0.67 接近真实录音 0.68；WER 1.81 甚至低于真实录音 1.94；SMOS 4.01 大幅领先 Voicebox 3.69（+0.32）。

### Table 2: RAVDESS 情感韵律评估

| System | Avg MCD | MCD-Acc | CMOS | SMOS |
|--------|---------|---------|------|------|
| Ground Truth | 0.00 | 1.00 | +0.17 | 4.42 |
| VALL-E | 5.03 | 0.34 | -0.55 | 3.80 |
| NaturalSpeech 2 | 4.56 | 0.25 | -0.22 | 4.04 |
| Voicebox | 4.88 | 0.34 | -0.34 | 3.92 |
| Mega-TTS 2 | 4.44 | 0.39 | -0.20 | 4.51 |
| StyleTTS 2 | 4.50 | 0.40 | -0.25 | 3.98 |
| HierSpeech++ | 6.08 | 0.30 | -0.37 | 3.87 |
| **NaturalSpeech 3** | **4.28** | **0.52** | **0.00** | **4.72** |

**表格说明**: 在 RAVDESS 情感语音上，NaturalSpeech 3 的 MCD 最低（4.28），情感分类准确率 MCD-Acc 最高（0.52），SMOS 4.72 大幅领先，证明属性分解对韵律/情感保持的优势。

### Table 3: 消融实验 — 分解与 CFG

| System | Sim-O / Sim-R | WER | CMOS | SMOS |
|--------|---------------|-----|------|------|
| NaturalSpeech 3 | 0.67 / 0.76 | 1.81 | 0.00 | 4.01 |
| - factorization | 0.55 / 0.61 | 2.49 | -0.25 | 3.59 |
| - cfg | 0.64 / 0.72 | 1.81 | -0.06 | 3.80 |

**关键发现**: 去掉属性分解（改用 SoundStream token + 统一生成）导致 Sim-O 下降 0.12、SMOS 下降 0.42，证明分解是质量提升的核心原因。去掉 [[Classifier-Free Guidance|CFG]] 导致 SMOS 下降 0.21。

### Table 4: 韵律表示消融

| System | MCD Avg | MCD-Acc |
|--------|---------|---------|
| NaturalSpeech 3 | 4.28 | 0.52 |
| Mel 20 Bins | 4.34 | 0.46 |

**关键发现**: FACodec 隐式学到的韵律表示优于手工设计的 "Mel 前 20 bins" 特征，MCD-Acc 提升 6 个百分点。

### Table 5: Codec 重建质量对比

| Model | SR | Hop | N | Bandwidth | PESQ | STOI | MSTFT | MCD |
|-------|-----|-----|---|-----------|------|------|-------|-----|
| EnCodec | 24kHz | 320 | 8 | 6.0 kbps | 3.28 | 0.94 | 0.99 | 2.70 |
| HiFi-Codec | 16kHz | 320 | 4 | 2.0 kbps | 3.17 | 0.93 | 0.98 | 3.05 |
| DAC | 16kHz | 320 | 9 | 4.5 kbps | 3.52 | 0.95 | 0.97 | 2.65 |
| SoundStream | 16kHz | 200 | 6 | 4.8 kbps | 3.03 | 0.90 | 1.07 | 3.38 |
| **FACodec** | **16kHz** | **200** | **6** | **4.8 kbps** | **3.47** | **0.95** | **0.93** | **2.59** |

**关键发现**: FACodec 在相同码率（4.8 kbps、6 codebook）下全面超越 SoundStream（PESQ +0.44, STOI +0.05, MCD -0.79），并接近 DAC（PESQ 3.47 vs 3.52，但 DAC 用了 9 codebook / 4.5 kbps）。

### Table 6: 扩展性验证 — VALL-E + FACodec

| System | Sim-O / Sim-R | WER | CMOS | SMOS |
|--------|---------------|-----|------|------|
| VALL-E + FACodec | 0.57 / 0.65 | 5.60 | +0.24 | 3.61 |
| VALL-E | 0.47 / 0.51 | 6.11 | 0.00 | 3.46 |

**关键发现**: 将 FACodec 的分解框架应用于 [[VALL-E]] 的 AR 生成（AR 生成 prosody code，NAR 生成 content + detail），所有指标均提升，证明属性分解不限于 NAR 范式。

### Table 7: 数据扩展实验

| Data | Sim-O | WER |
|------|-------|-----|
| 1K hours | 0.64 | 3.94 |
| 60K hours | 0.72 | 3.03 |
| 200K hours | 0.73 | 2.11 |

**关键发现**: 即使只用 1K 小时数据，得益于属性分解，Sim-O 已达 0.64。数据扩展到 200K 小时持续带来提升（WER 3.94 → 2.11）。

### Table 8: 模型扩展实验

| Model Size | Sim-O | WER |
|------------|-------|-----|
| 500M | 0.73 | 2.11 |
| 1B | 0.78 | 1.71 |

**关键发现**: Transformer 层数从 12→24（500M→1B），Sim-O 提升 0.05，WER 下降 0.40，模型扩展持续有效。

### Table 9: 完整评测结果（含 UTMOS 和更强 ASR）

| System | Sim-O | Sim-R | WER | WER* | UTMOS | CMOS | SMOS |
|--------|-------|-------|-----|------|-------|------|------|
| Ground Truth | 0.68 | - | 1.94 | 0.68 | 4.14 | +0.08 | 3.85 |
| VALL-E | 0.47 | 0.51 | 6.11 | 4.87 | 3.68 | -0.60 | 3.46 |
| NaturalSpeech 2 | 0.55 | 0.62 | 1.94 | 1.24 | 3.88 | -0.18 | 3.65 |
| Voicebox (self-collected) | 0.64 | 0.67 | 2.03 | 1.81 | 3.82 | -0.23 | 3.69 |
| Mega-TTS 2 | 0.53 | - | 2.32 | 2.17 | 4.02 | -0.20 | 3.63 |
| UniAudio | 0.57 | 0.68 | 2.49 | 1.81 | 3.79 | -0.25 | 3.71 |
| StyleTTS 2 | 0.38 | - | 2.49 | 1.58 | 3.94 | -0.21 | 3.07 |
| HierSpeech++ | 0.51 | - | 6.33 | 4.97 | 3.80 | -0.41 | 3.50 |
| **NaturalSpeech 3** | **0.67** | **0.76** | **1.81** | **1.13** | **4.30** | **0.00** | **4.01** |

**表格说明**: 扩展版评测包含 UTMOS（4.30，超越真实录音 4.14）和更强的 Conformer Transducer XL ASR（WER* 1.13，同样超越真实录音 0.68）。

### Table 10: 延迟分析

| Models | NFE | RTF | Sim-O | Sim-R | UTMOS |
|--------|-----|-----|-------|-------|-------|
| NaturalSpeech 2 | 150 | 0.366 | 0.55 | 0.62 | 3.87 |
| VALL-E | - | 4.520 | 0.47 | 0.51 | 3.67 |
| NaturalSpeech 3 | 60 | 0.296 | 0.67 | 0.76 | 4.30 |
| NaturalSpeech 3 one-step | 15 | 0.067 | 0.66 | 0.75 | 4.01 |

**关键发现**: NaturalSpeech 3 的 60 步 NFE 对应 RTF 0.296（比 NaturalSpeech 2 的 150 步还快）。one-step（每个 diffusion 仅 1 步迭代）RTF 降至 0.067，质量几乎无损（Sim-O 0.66 vs 0.67）。

### Table 11: Duration 设计消融

| System | Sim-O | Sim-R | WER | UTMOS |
|--------|-------|-------|-----|-------|
| NaturalSpeech 3 | 0.67 | 0.76 | 1.94 | 4.30 |
| Generation ablation | 0.62 | 0.73 | 1.94 | 4.18 |
| Objective ablation | 0.62 | 0.72 | 2.38 | 4.13 |
| Conditioning ablation | 0.62 | 0.72 | 2.49 | 4.11 |
| Prompting ablation | 0.61 | 0.71 | 2.83 | 4.08 |

**关键发现**: Duration diffusion 的每个设计（生成方式、训练目标、条件设计、prompt 机制）都贡献了性能提升，去掉 prompting 影响最大（WER 1.94 → 2.83）。

### Table 12: 逐情感 MCD 得分（RAVDESS）

| System | neutral | calm | happy | sad | angry | fearful | disgust | surprised |
|--------|---------|------|-------|-----|-------|---------|---------|-----------|
| Ground Truth | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |
| VALL-E | 3.97 | 4.75 | 4.83 | 5.51 | 5.19 | 5.29 | 5.45 | 5.29 |
| Voicebox | 3.93 | 4.90 | 4.96 | 4.93 | 5.01 | 5.03 | 5.34 | 4.89 |
| NaturalSpeech 2 | 2.77 | 3.51 | 4.85 | 4.88 | 5.42 | 5.23 | 5.31 | 4.52 |
| Mega-TTS 2 | 3.28 | 4.39 | 4.44 | 4.67 | 4.21 | 5.00 | 5.42 | 4.14 |
| StyleTTS 2 | 3.41 | 4.38 | 4.40 | 4.64 | 4.80 | 4.69 | 5.10 | 4.57 |
| HierSpeech++ | 5.54 | 6.55 | 5.78 | 5.84 | 6.37 | 6.17 | 6.74 | 5.62 |
| **NaturalSpeech 3** | **3.23** | **4.32** | **4.26** | **4.41** | **4.64** | **4.25** | **4.80** | **4.45** |

**表格说明**: NaturalSpeech 3 在 8 种情感中 6 种取得最低 MCD（最佳韵律保持），尤其在 fearful（4.25 vs 次优 4.69）和 disgust（4.80 vs 次优 5.10）情感上优势明显。

### Table 13: Codec 完整重建质量对比（含更多配置）

| Model | SR | H | N | Bandwidth | PESQ | STOI | MSTFT | MCD |
|-------|-----|---|---|-----------|------|------|-------|-----|
| EnCodec (official) | 24kHz | 320 | 8 | 6.0 kbps | 3.28 | 0.94 | 0.99 | 2.70 |
| EnCodec (reproduced) | 16kHz | 320 | 10 | 5.0 kbps | 3.10 | 0.92 | 0.97 | 3.10 |
| HiFi-Codec | 16kHz | 320 | 4 | 2.0 kbps | 3.17 | 0.93 | 0.98 | 3.05 |
| DAC | 16kHz | 320 | 9 | 4.5 kbps | 3.52 | 0.95 | 0.97 | 2.65 |
| SoundStream (6 VQ) | 16kHz | 200 | 6 | 4.8 kbps | 3.03 | 0.90 | 1.07 | 3.38 |
| SoundStream (12 VQ) | 16kHz | 200 | 12 | 9.6 kbps | 3.45 | 0.94 | 0.92 | 2.76 |
| **FACodec** | **16kHz** | **200** | **6** | **4.8 kbps** | **3.47** | **0.95** | **0.93** | **2.59** |

**表格说明**: 扩展版对比增加了 SoundStream 12-VQ 配置（9.6 kbps），FACodec 在仅 4.8 kbps 下超越了 SoundStream 12-VQ 的 9.6 kbps（PESQ 3.47 vs 3.45, MCD 2.59 vs 2.76）。

### Table 14: 零样本语音转换结果

| Models | Sim-O | WER |
|--------|-------|-----|
| Ground Truth | - | 3.25 |
| YourTTS | 0.72 | 10.1 |
| Make-A-Voice (VC) | 0.68 | 6.20 |
| LM-VC | 0.82 | 4.91 |
| UniAudio | 0.87 | 4.80 |
| **FACodec** | **0.86** | **3.46** |

**表格说明**: FACodec 的音色解耦直接支持零样本语音转换。WER 3.46 大幅领先 UniAudio 4.80（+1.34），甚至接近真实录音 3.25，说明 FACodec 在转换音色的同时完好保留了语言内容。

### Table 15: 信息瓶颈消融（语音转换）

| System | Sim-O |
|--------|-------|
| w. information bottleneck | 0.86 |
| w.o. information bottleneck | 0.73 |

**关键发现**: 去掉信息瓶颈，Sim-O 从 0.86 下降到 0.73，证明 8 维投影对于音色解耦至关重要。

### Table 16: Acoustic Detail 量化器消融

| System | Codebook Number | PESQ | STOI | MSTFT | MCD |
|--------|-----------------|------|------|-------|-----|
| FACodec | 6 | 3.47 | 0.95 | 0.93 | 2.59 |
| - acoustic detail quantizers | 3 | 3.09 | 0.92 | 1.08 | 3.12 |
| SoundStream | 6 | 3.03 | 0.90 | 1.07 | 3.38 |

**关键发现**: 去掉 acoustic detail 的 3 个量化器（6→3 codebook），重建质量大幅下降（PESQ 3.47→3.09），接近 SoundStream 6-VQ（3.03），证明 acoustic detail 子空间对重建质量的重要性。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Librilight]] | 60K hours, ~7000 speakers | 无标签有声书，内部 ASR 转录 + G2P + 对齐 | 训练 |
| [[LibriSpeech]] test-clean | 5.4 hours, 40 speakers | 标准 TTS 评测集 | 测试 |
| [[RAVDESS]] | 24 actors, 8 emotions | 情感韵律语音 | 韵律评测 |

### 实现细节

- **FACodec**: 下采样率 200，帧率 80 Hz（16kHz），codebook 大小 1024，6 个 codebook（prosody 1 + content 2 + detail 3），信息瓶颈投影维度 8
- **Phoneme Encoder**: 6 层 [[Transformer]]，8 头注意力，512 dim，filter 2048，kernel 9（1D conv）
- **Prosody/Content/Detail Diffusion**: 共享 12 层 Transformer，8 头，1024 dim，filter 2048，kernel 3，[[Conditional Layer Normalization]] 注入 diffusion 时间步
- **Duration Diffusion**: 6 层 Transformer，8 头，1024 dim
- **优化器**: [[AdamW]]，lr=1e-4，$\beta_1=0.9$，$\beta_2=0.98$，5K warmup，inverse sqrt schedule
- **训练**: 8 x A100 80GB，batch 10K frames/GPU，1M steps
- **推理**: 每个 diffusion 4 步迭代，top-k=20，温度退火 1.5→0，总 NFE=60（phoneme prosody 4x2 + duration 4 + prosody 4x2 + content 4x2 + detail 4x2）

### 评测指标

| 指标 | 工具 | 说明 |
|------|------|------|
| [[SIM-O]] / [[SIM-R]] | WavLM-TDCNN | 与原始/重合成 prompt 的说话人余弦相似度 |
| [[UTMOS]] | UTokyo | 自动 MOS 预测 |
| [[WER]] | CTC-based HuBERT + Conformer Transducer XL | 可懂度 |
| [[MCD]] | Mel-Cepstral Distortion | 韵律相似度 |
| CMOS | 12 名母语评审，20 句 | 自然度对比评分 |
| SMOS | 12 名母语评审，10 句 | 相似度主观评分 |

### 可视化结果

NaturalSpeech 3 支持属性操控：不同 prompt 控制不同属性（如用 A 的音色 + B 的韵律 + C 的内容），实现了灵活的语音合成控制。Demo 见 [speechresearch.github.io/naturalspeech3](https://speechresearch.github.io/naturalspeech3)。

---

## 批判性思考

### 优点
1. **分治思想优雅**: 将复杂的语音生成分解为多个简单子问题，每个子模型建模难度大幅降低
2. **解耦技术全面**: 信息瓶颈 + 监督损失 + 梯度反转 + detail dropout 四管齐下，消融实验充分验证
3. **评测全面且公平**: 同一训练数据（Librilight）下对比多个强基线，主客观指标齐全，12 名母语评审
4. **可扩展性强**: 属性分解框架不限于 NAR，可嫁接到 [[VALL-E]] 等 AR 模型（Table 6）
5. **FACodec 开源**: 发布 checkpoint 和 HuggingFace demo，已被 [[Amphion]] 框架集成

### 局限性
1. **非端到端训练**: FACodec 和 Factorized Diffusion Model 分开训练，量化误差可能在生成阶段被放大
2. **流式能力缺失**: 全文未提及 streaming / 流式推理，60 步 NFE 的 RTF=0.296 虽然 <1 但离低延迟流式仍有差距；one-step RTF=0.067 可行但需额外训练
3. **依赖外部对齐工具**: Duration 通过内部对齐工具获取（非开源），限制了完全端到端的训练和外部复现
4. **训练成本高**: 8 x A100 80GB、1M steps（具体时间未报告），200K 小时内部数据不可复现
5. **评测集单一**: 主要在 LibriSpeech（有声书，低噪声）和 RAVDESS（实验室录制情感语音）上评测，缺乏 in-the-wild / 自然对话等更挑战性场景的验证
6. **与后续更强基线未对比**: 发布于 2024.03，未与 [[CosyVoice]]、[[F5-TTS]] 等同期/后续工作对比

### 潜在改进方向
1. 端到端联合训练 FACodec + Diffusion（消除 quantization gap）
2. 流式推理（chunk-based factorized diffusion）
3. 去掉对外部 phoneme / alignment 的依赖（类似 [[F5-TTS]] 的 text-audio 对齐学习）
4. 在更多 in-the-wild 数据集上验证（如 [[Emilia]]、[[Seed-TTS-eval]]）

### 可复现性评估
- [x] 代码开源（FACodec 部分，via Amphion）
- [x] 预训练模型（FACodec checkpoint on HuggingFace）
- [ ] 训练细节完整（Diffusion 部分训练代码未开源）
- [ ] 数据集可获取（Librilight 可获取，但 200K 小时内部数据不可获取）

---

## 关联笔记

### 基于
- [[NaturalSpeech]]: NS 系列第一代，单说话人人类水平质量（LJSpeech）
- [[NaturalSpeech2]]: NS 系列第二代，latent diffusion + RVQ codec 零样本 TTS
- [[SoundStream]]: FACodec 的架构基础（同样的编码器/解码器结构 + RVQ）

### 对比
- [[VALL-E]]: AR 离散 token TTS 范式代表，本文直接对比
- [[Voicebox]]: Meta 的 Flow Matching TTS，强基线
- [[HierSpeech++]]: 层级语音合成，对比基线之一
- [[StyleTTS 2]]: 风格控制 TTS，对比基线之一

### 方法相关
- [[Factorized Vector Quantization]]: FACodec 的核心量化方法
- [[Discrete Diffusion]]: 离散空间的扩散/去噪生成
- [[Gradient Reversal Layer]]: 对抗解耦的关键技术
- [[Classifier-Free Guidance]]: 推理时的引导增强
- [[Conditional Layer Normalization]]: 条件信息注入方式
- [[Length Regulator]]: NAR TTS 中 phoneme→frame 的扩展

### 硬件/数据相关
- [[Librilight]]: 60K 小时无标签有声书训练数据
- [[LibriSpeech]]: 标准 TTS 评测基准
- [[RAVDESS]]: 情感语音评测集

---

## 速查卡片

> [!summary] NaturalSpeech 3
> - **核心**: FACodec 属性分解（content/prosody/timbre/detail）+ 分治式离散扩散
> - **方法**: 信息瓶颈 + 监督损失 + 梯度反转实现解耦；4 个 diffusion 子模块按序生成
> - **结果**: LibriSpeech Sim-O 0.67 / WER 1.81 / UTMOS 4.30，首个多说话人人类水平零样本 TTS
> - **代码**: [Amphion FACodec](https://huggingface.co/spaces/amphion/naturalspeech3_facodec)

---

*笔记创建时间: 2026-05-25*
