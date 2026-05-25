---
title: "VoxCPM: Tokenizer-Free TTS for Context-Aware Speech Generation and True-to-Life Voice Cloning"
method_name: "VoxCPM"
authors: [Yixuan Zhou, Guoyang Zeng, Xin Liu, Xiang Li, Renjie Yu, Ziyang Wang, Runchuan Ye, Weiyue Sun, Jiancheng Gui, Kehan Li, Zhiyong Wu, Zhiyuan Liu]
year: 2025
venue: arXiv
tags: [zero-shot-tts, tokenizer-free, hierarchical-modeling, flow-matching, semi-discrete, voice-cloning, context-aware]
image_source: online
arxiv_html: https://arxiv.org/html/2509.24650v1
created: 2026-05-25
---

# 论文笔记：VoxCPM: Tokenizer-Free TTS for Context-Aware Speech Generation and True-to-Life Voice Cloning

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Tsinghua SIGS (THUHCSI), THUNLP, ModelBest (面壁智能), OpenBMB |
| 日期 | September 2025 |
| 项目主页 | [Demo Page](https://openbmb.github.io/VoxCPM-demopage/) |
| 对比基线 | [[CosyVoice]], [[CosyVoice 2]], [[F5-TTS]], [[MaskGCT]], [[SparkTTS]], [[IndexTTS 2]], [[FireRedTTS]], [[DiTAR]] |
| 链接 | [arXiv](https://arxiv.org/abs/2509.24650) / [HuggingFace](https://huggingface.co/OpenBMB) |

---

## 一句话总结

> 通过可微分 FSQ 瓶颈将语义-韵律规划与细粒度声学渲染结构性分离，无需外部 tokenizer 即实现端到端 TTS 训练，0.5B 参数在开源系统中达到 SOTA。

---

## 核心贡献

1. **端到端层次化架构**: 通过内部半离散（semi-discrete）瓶颈解决连续自回归 TTS 中表达力与稳定性的矛盾，无需预训练 speech tokenizer
2. **残差学习策略**: TSLM+FSQ 负责稳定的语义-韵律骨架，RALM 补回量化丢失的声学细节，实现功能分离而不破坏端到端训练
3. **大规模训练 + 上下文感知合成**: 在 180 万小时双语数据上训练，0.5B 模型在 SEED-TTS-EVAL 和 CV3-EVAL 上取得开源 SOTA；可从纯文本推断合适的语气风格

---

## 问题背景

### 要解决的问题

现代 TTS 存在一个根本矛盾：[[Discrete Audio Token|离散 token]] 提供训练稳定性但丢失声学细节（量化失真），[[Flow Matching|连续表示]] 保留声学丰富度但在长序列上累积误差。

### 现有方法的局限

1. **离散 token 方法**（[[VALL-E]], [[CosyVoice]], [[SparkTTS]]）：[[RVQ]] 量化不可避免地引入声学保真度损失
2. **连续自回归方法**（[[DiTAR]], MELLE）：高层语义-韵律规划与底层声学渲染纠缠在同一学习目标中，导致长序列不稳定
3. **多阶段混合流水线**：LLM 生成离散 token + diffusion decoder 的级联架构无法端到端优化，存在"语义-声学鸿沟"
4. **直接使用 FSQ/VQ 做离散码本**：维度增加时码本大小指数爆炸，词表不可管理

### 本文的动机

一个有效的解决方案应当**结构性地分离**语义-韵律内容建模和细粒度声学细节，同时保持可微分性。关键洞察：可微分量化瓶颈可以将信息分裂为类离散骨架（内容稳定性）和连续残差分量（细节表达力），而非将量化作为预测目标。

---

## 方法详解

### 模型架构

VoxCPM 采用**层次化自回归**架构，生成连续语音 latent $\mathbf{Z} = \{\mathbf{z}_1, \dots, \mathbf{z}_M\}$，每个 $\mathbf{z}_i \in \mathbb{R}^{P \times D}$ 表示 $P$ 帧 [[VAE]] latent patch：

- **输入**: 文本 token $\mathbf{T} = \{t_1, \dots, t_N\}$ + 可选参考音频
- **LocEnc**: 轻量级局部音频编码器，将 [[VAE]] latent patch 压缩为紧凑声学嵌入
- **TSLM**: Text-Semantic Language Model（24 层，基于 [[MiniCPM]]-4-0.5B 初始化），捕获高层语言结构和语义-韵律表示
- **[[FSQ]]**: Finite Scalar Quantization 瓶颈层，256 维 × 9 量化级别，产生半离散语义骨架
- **RALM**: Residual Acoustic Language Model（6 层），补回量化丢失的声学细节
- **LocDiT**: 局部 [[Diffusion Transformer]] 解码器（4 层），双向 Transformer 生成高保真 latent
- **Stop Predictor**: 3 层 MLP，判断生成终止
- **Causal Audio [[VAE]]**: 独立训练，16kHz waveform → 25 Hz latent（640× 下采样）
- **总参数**: 0.5B

### 核心模块

#### 模块1: Text-Semantic Language Model (TSLM)

**设计动机**: 利用预训练文本 LM 的丰富上下文理解能力，实现从原始文本直接推断语义和韵律，而非依赖 [[Phoneme]] 序列。

**具体实现**:
- 以 [[MiniCPM]]-4-0.5B 为骨干初始化，24 层 Transformer，1024 hidden dim，4096 FFN dim
- 中文使用字级别分词（character-level segmentation），缓解 BPE Tokenizer 在 TTS 任务中的词汇稀疏问题
- 同时处理文本 token 和历史音频上下文（通过 LocEnc 提供的 $\mathbf{E}_{<i}$）
- 输出连续的语义-韵律表示，编码内容和韵律实现

#### 模块2: FSQ 瓶颈层（Semi-Discrete Representation Learning）

**设计动机**: 作为**正则化机制**约束隐状态空间，迫使 TSLM 优先保留可通过瓶颈的稳定高层信息，而非作为离散预测目标。

**具体实现**:
- 使用 256 维 × 9 级 [[FSQ]]，称为"semi-discrete"——维度远大于标准 FSQ，保留足够信息容量
- 前向传播时离散化，反向传播通过 [[Straight-Through Estimator]] 保持梯度流
- 类比于 [[RVQ]] 第一层：捕获粗粒度语义-韵律骨架
- 关键区别：FSQ 是连续数据流中的**中间可微归纳偏置**，而非 token 化的预测目标

#### 模块3: Residual Acoustic Language Model (RALM)

**设计动机**: FSQ 瓶颈必然丢失细粒度声学细节（音色纹理、环境特征等），RALM 专门补回这些残差信息。

**具体实现**:
- 6 层 Transformer，1024 hidden dim
- 条件输入三部分：TSLM 文本部分隐状态 $\mathbf{H}_{\text{text}}^{\text{TSLM}}$、FSQ 后的语音隐状态 $\mathbf{H}_{<i}^{\text{FSQ}}$、历史声学嵌入 $\mathbf{E}_{<i}$
- 最终表示通过残差连接：$\mathbf{h}_i^{\text{final}} = \mathbf{h}_i^{\text{FSQ}} + \mathbf{h}_i^{\text{residual}}$
- 形成自然的分工：TSLM+FSQ 专注内容稳定性和韵律一致性，RALM 专注声学表达力和说话人特征

#### 模块4: LocDiT 解码器

**设计动机**: 使用双向 Transformer 在每个 patch 内部拥有完整感受野，将条件信息转为高保真 latent。

**具体实现**:
- 4 层双向 Transformer
- 引入前一个 patch $\mathbf{z}_{i-1}$ 作为额外条件（outpainting 而非独立生成，参考 [[DiTAR]]）
- 训练时以 0.1 概率 mask LM guidance，推理时使用 [[Classifier-Free Guidance|CFG]]
- 最优 CFG = 2.0

#### 模块5: Causal Audio VAE

**具体实现**:
- 架构类似 [[DAC]]，使用 causal CNN 编码器和解码器
- 16kHz 单声道 → 25 Hz 帧率（stride 序列 [2, 5, 8, 8]，640× 下采样）
- 训练损失：Mel 频谱重建 + multi-period / multi-scale 判别器对抗损失 + KL 散度（权重 5e-5）
- 因果设计支持流式编解码

---

## 关键公式

### 公式1: [[Autoregressive|自回归生成分解]]

$$
p(\mathbf{Z}|\mathbf{T}) = \prod_{i=1}^{M} p(\mathbf{z}_i | \mathbf{T}, \mathbf{Z}_{<i})
$$

**含义**: 将语音 latent 序列的联合分布分解为逐 patch 的条件概率乘积，每个 patch 依赖文本和历史音频。

**符号说明**:
- $\mathbf{Z} = \{\mathbf{z}_1, \dots, \mathbf{z}_M\}$: 语音 latent 序列，共 $M$ 个 patch
- $\mathbf{T} = \{t_1, \dots, t_N\}$: 文本 token 序列
- $\mathbf{z}_i \in \mathbb{R}^{P \times D}$: 第 $i$ 个 patch，包含 $P$ 帧 $D$ 维 VAE latent

### 公式2: [[Hierarchical Modeling|层次化 Patch 生成]]

$$
\mathbf{z}_i \sim \text{LocDiT}(\mathbf{h}_i^{\text{final}}), \quad \mathbf{h}_i^{\text{final}} = \underbrace{\text{FSQ}(\text{TSLM}(\mathbf{T}, \mathbf{E}_{<i}))}_{\text{stable skeleton}} + \underbrace{\text{RALM}(\cdot)}_{\text{residual details}}
$$

**含义**: 每个 patch 通过 LocDiT 从组合条件生成，条件由 FSQ 骨架和 RALM 残差相加得到。

**符号说明**:
- $\mathbf{h}_i^{\text{final}}$: 最终条件向量，指导 LocDiT 生成
- $\mathbf{E}_{<i} = \text{LocEnc}(\mathbf{Z}_{<i})$: 历史音频上下文的紧凑嵌入

### 公式3: [[FSQ|Finite Scalar Quantization]]

$$
\mathbf{h}_{i,j}^{\text{FSQ}} = \Delta \cdot \text{clip}\left(\text{round}\left(\frac{\mathbf{h}_{i,j}^{\text{TSLM}}}{\Delta}\right), -L, L\right)
$$

**含义**: 对 TSLM 输出的每个维度做标量量化——先缩放、取整、裁剪，再还原尺度，形成半离散表示。

**符号说明**:
- $\Delta$: 量化步长
- $L$: 裁剪范围，$2L+1$ 为量化级数（本文用 9 级，$L=4$）
- $\text{round}(\cdot)$: 取整操作，前向离散化，反向通过 [[Straight-Through Estimator]] 传梯度

### 公式4: [[Residual Learning|残差声学建模]]

$$
\mathbf{h}_i^{\text{residual}} = \text{RALM}(\mathbf{H}_{\text{text}}^{\text{TSLM}}, \mathbf{H}_{<i}^{\text{FSQ}} \oplus \mathbf{E}_{<i})
$$

**含义**: RALM 利用 TSLM 的文本表示、FSQ 后的语音骨架、历史声学嵌入三路信息，预测量化损失的残差。

**符号说明**:
- $\mathbf{H}_{\text{text}}^{\text{TSLM}}$: TSLM 对文本部分的隐状态
- $\mathbf{H}_{<i}^{\text{FSQ}}$: FSQ 后的语音历史隐状态
- $\oplus$: 拼接操作

### 公式5: [[Flow Matching|Flow Matching 损失]]

$$
\mathcal{L}_{\text{FM}} = \mathbb{E}_{t, \mathbf{z}_i^0, \boldsymbol{\epsilon}} \left[ \left\| \mathbf{v}_\theta(\mathbf{z}_i^t, t, \mathbf{h}_i^{\text{final}}, \mathbf{z}_{i-1}) - \frac{d}{dt}(\alpha_t \mathbf{z}_i^0 + \sigma_t \boldsymbol{\epsilon}) \right\|^2 \right]
$$

**含义**: LocDiT 以 Flow Matching 目标训练，预测从噪声到干净 latent 的速度场。

**符号说明**:
- $\mathbf{z}_i^t = \alpha_t \mathbf{z}_i^0 + \sigma_t \boldsymbol{\epsilon}$: 时间步 $t$ 的噪声 latent
- $\boldsymbol{\epsilon} \sim \mathcal{N}(0, \mathbf{I})$: 高斯噪声
- $\mathbf{v}_\theta$: LocDiT 预测的速度场
- $\mathbf{z}_{i-1}$: 前一个 patch 作为 outpainting 条件

### 公式6: Stop 预测损失

$$
\mathcal{L}_{\text{Stop}} = \mathbb{E}_{i \sim \text{sequence}} \left[ \text{BCE}(s_\theta(\mathbf{h}_i^{\text{FSQ}}), \mathbb{1}[\text{token } i \text{ is the last}]) \right]
$$

**含义**: 二元交叉熵损失训练 Stop Predictor 判断何时停止生成。

**符号说明**:
- $s_\theta$: Stop Predictor（3 层 MLP）
- $\mathbb{1}[\cdot]$: 指示函数

### 公式7: 总训练目标

$$
\mathcal{L} = \mathcal{L}_{\text{FM}} + \lambda \mathcal{L}_{\text{Stop}}
$$

**含义**: 整个系统端到端训练，梯度通过 FSQ（via straight-through estimation）、TSLM、LocEnc 反向传播。

---

## 关键图表

### Figure 1: Overall Architecture / 系统概览

![Figure 1](https://arxiv.org/html/2509.24650v1/images/draft_1.png)

**说明**: VoxCPM 的整体架构。音频 latent 通过 LocEnc 压缩为紧凑嵌入，TSLM 基于文本和音频历史生成语义-韵律表示，经 [[FSQ]] 瓶颈产生半离散骨架，RALM 补回声学细节残差，最终 LocDiT 基于组合条件生成高保真 latent patch。整个系统端到端以 [[Flow Matching]] 目标训练。

### Figure 2: T-SNE Visualization — Zero-Shot Voice Cloning / 零样本克隆任务的隐空间分布

![Figure 2](https://arxiv.org/html/2509.24650v1/images/tsne1.png)

**说明**: 零样本语音克隆任务中的 T-SNE 可视化。每种颜色代表一位未见说话人的不同话语。TSLM-FSQ 输出形成与文本内容绑定的语义-韵律结构（按内容聚类），而 RALM 残差显示出强说话人相关变异（按说话人聚类），验证了层次化分工的有效性。

### Figure 3: T-SNE Visualization — Context-Aware TTS / 无 prompt 语音的上下文感知合成

![Figure 3](https://arxiv.org/html/2509.24650v1/images/tsne2.png)

**说明**: 无 prompt 语音的 TTS 任务中的 T-SNE 可视化。处理不同文本体裁（新闻、诗歌、对话）时，TSLM-FSQ 表示按语义类别聚类。RALM 输出在类别内部展现更大变异，体现其在细粒度声学细微差别上的作用。

### Table 1: 模型架构配置

| 模块 | 配置 |
|------|------|
| LocEnc | 4 layers, 1024 hidden dim, 4096 FFN dim |
| TSLM | 24 layers (MiniCPM-4-0.5B 初始化), 1024 hidden dim, 4096 FFN dim |
| FSQ | 256 dimensions, 9 quantization levels |
| RALM | 6 layers, 1024 hidden dim, 4096 FFN dim |
| LocDiT | 4 layers, 1024 hidden dim, 4096 FFN dim |
| Stop Predictor | 3-layer MLP, 1024 hidden dim, 2 output dim |
| Patch-size | 2 (TSLM 和 RALM 工作在 12.5Hz token rate) |
| AudioVAE | 16kHz waveform → 25Hz latents (stride [2, 5, 8, 8]) |

**说明**: 0.5B 参数的完整配置。Patch-size=2 意味着 TSLM/RALM 在 12.5 Hz token rate 下工作，LocDiT 在 25 Hz 下渲染每个 patch。

### Table 2: 训练细节

| Model | Phase | Learning Rate | Tokens/Batch | Iterations | GPUs |
|-------|-------|--------------|-------------|------------|------|
| VoxCPM | Stable | 1×10⁻⁴ | 4,096 | 400K | 40 × H100 |
| VoxCPM | Decay | 1×10⁻⁴ → 5×10⁻⁶ | 8,192 | 100K | 40 × H100 |
| VoxCPM-Emilia | Stable | 1×10⁻⁴ | 4,096 | 150K | 24 × H100 |
| VoxCPM-Emilia | Decay | 1×10⁻⁴ → 5×10⁻⁶ | 8,192 | 50K | 24 × H100 |
| VoxCPM-ablation | Stable | 1×10⁻⁴ | 4,096 | 200K | 8 × H100 |

**说明**: 使用 WSD（Warmup-Stable-Decay）学习率调度。Decay 阶段 batch 翻倍至 8192 tokens，对最终性能至关重要（Table 8 消融）。

### Table 3: SEED-TTS-EVAL 主实验结果

| Model | Params | Open | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | Hard CER↓ | Hard SIM↑ |
|-------|--------|------|---------|---------|---------|---------|-----------|-----------|
| MegaTTS3 | 0.5B | ✗ | 2.79 | 77.1 | 1.52 | 79.0 | - | - |
| DiTAR | 0.6B | ✗ | 1.69 | 73.5 | 1.02 | 75.3 | - | - |
| CosyVoice3-0.5B | 0.5B | ✗ | 2.02 | 71.8 | 1.16 | 78.0 | 6.08 | 75.8 |
| CosyVoice3-1.5B | 1.5B | ✗ | 2.22 | 72.0 | 1.12 | 78.1 | 5.83 | 75.8 |
| Seed-TTS | - | ✗ | 2.25 | 76.2 | 1.12 | 79.6 | 7.59 | 77.6 |
| MiniMax-Speech | - | ✗ | 1.65 | 69.2 | 0.83 | 78.3 | - | - |
| F5-TTS | 0.3B | ✓ | 2.00 | 67.0 | 1.53 | 76.0 | 8.67 | 71.3 |
| MaskGCT | - | ✓ | 2.62 | 71.7 | 2.27 | 77.4 | - | - |
| CosyVoice | 0.3B | ✓ | 4.29 | 60.9 | 3.63 | 72.3 | 11.75 | 70.9 |
| CosyVoice 2 | 0.5B | ✓ | 3.09 | 65.9 | 1.38 | 75.7 | 6.83 | 72.4 |
| SparkTTS | 0.5B | ✓ | 3.14 | 57.3 | 1.54 | 66.0 | - | - |
| FireRedTTS | 0.5B | ✓ | 3.82 | 46.0 | 1.51 | 63.5 | 17.45 | 62.1 |
| FireRedTTS-2 | - | ✓ | 1.95 | 66.5 | 1.14 | 73.6 | - | - |
| Qwen2.5-Omni | 7B | ✓ | 2.72 | 63.2 | 1.70 | 75.2 | 7.97 | 74.7 |
| OpenAudio-s1-mini | 0.5B | ✓ | 1.94 | 55.0 | 1.18 | 68.5 | 23.37 | 64.3 |
| IndexTTS 2 | 1.5B | ✓ | 2.23 | 70.6 | 1.03 | 76.5 | 7.12 | 75.5 |
| VibeVoice | 1.5B | ✓ | 3.04 | 68.9 | 1.16 | 74.4 | - | - |
| HiggsAudio-v2 | 3B | ✓ | 2.44 | 67.7 | 1.50 | 74.0 | 55.07 | 65.6 |
| **VoxCPM-Emilia** | **0.5B** | **✓** | **2.34** | **68.1** | **1.11** | **74.0** | **12.46** | **69.8** |
| **VoxCPM** | **0.5B** | **✓** | **1.85** | **72.9** | **0.93** | **77.2** | **8.87** | **73.0** |

**说明**: VoxCPM 在开源系统中取得最优 EN WER（1.85%）和 ZH CER（0.93%），SIM 也具竞争力（EN 72.9 / ZH 77.2）。0.5B 参数超越了 1.5B 的 IndexTTS 2 和 3B 的 HiggsAudio-v2。与闭源 DiTAR（0.6B）相比，WER 略高但 SIM 接近。

### Table 4: CV3-EVAL 结果

| Model | ZH CER↓ | EN WER↓ | Hard-ZH CER↓ | Hard-ZH SIM↑ | Hard-ZH DNSMOS↑ | Hard-EN WER↓ | Hard-EN SIM↑ | Hard-EN DNSMOS↑ |
|-------|---------|---------|--------------|-------------|-----------------|-------------|-------------|-----------------|
| F5-TTS | 5.47 | 8.90 | - | - | - | - | - | - |
| SparkTTS | 5.15 | 11.0 | - | - | - | - | - | - |
| GPT-SoVITS | 7.34 | 12.5 | - | - | - | - | - | - |
| CosyVoice 2 | 4.08 | 6.32 | 12.58 | 72.6 | 3.81 | 11.96 | 66.7 | 3.95 |
| OpenAudio-s1-mini | 4.00 | 5.54 | 18.1 | 58.2 | 3.77 | 12.4 | 55.7 | 3.89 |
| IndexTTS 2 | 3.58 | 4.45 | 12.8 | 74.6 | 3.65 | 8.78 | 74.5 | 3.80 |
| HiggsAudio-v2 | 9.54 | 7.89 | 41.0 | 60.2 | 3.39 | 10.3 | 61.8 | 3.68 |
| CosyVoice3-0.5B* | 3.89 | 5.24 | 14.15 | 78.6 | 3.75 | 9.04 | 75.9 | 3.92 |
| CosyVoice3-1.5B* | 3.91 | 4.99 | 9.77 | 78.5 | 3.79 | 10.55 | 76.1 | 3.95 |
| **VoxCPM-Emilia** | **4.47** | **5.23** | **22.2** | **62.6** | **3.47** | **10.00** | **62.6** | **3.68** |
| **VoxCPM** | **3.40** | **4.04** | **12.9** | **66.1** | **3.59** | **7.89** | **64.3** | **3.74** |

**说明**: VoxCPM 在 CV3-EVAL 上取得最优 ZH CER（3.40%）和 EN WER（4.04%），超越所有系统含闭源。Hard-EN WER 7.89% 也优于闭源 CosyVoice 3。但 Hard 子集的 SIM 和 DNSMOS 落后于 CosyVoice 3 和 IndexTTS 2。

### Table 5: 主观评测 (MOS ± 置信区间)

| Model | ZH N-MOS | ZH S-MOS | EN N-MOS | EN S-MOS |
|-------|----------|----------|----------|----------|
| MaskGCT | 3.20±0.11 | 3.77±0.11 | 3.84±0.11 | 4.00±0.10 |
| CosyVoice 2 | 3.38±0.12 | 4.01±0.10 | **4.14±0.09** | 3.97±0.10 |
| IndexTTS 2 | **4.25±0.09** | 4.05±0.09 | 4.03±0.10 | 4.16±0.09 |
| VoxCPM-Emilia | 3.79±0.12 | 3.99±0.11 | 3.91±0.10 | 4.10±0.09 |
| **VoxCPM** | **4.10±0.10** | **4.11±0.10** | **4.11±0.09** | **4.18±0.09** |

**说明**: VoxCPM 取得最高 EN S-MOS（4.18）和 ZH S-MOS（4.11），说话人相似度主观评价最优。ZH N-MOS 4.10 仅次于 IndexTTS 2 的 4.25。20 位母语者评测。

### Table 6: 消融实验 — FSQ 维度

| FSQ 配置 | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | ZH-hard CER↓ | ZH-hard SIM↑ |
|----------|---------|---------|---------|---------|--------------|-------------|
| d4s9 | 5.18 | 59.3 | 4.05 | 68.0 | 19.55 | 62.3 |
| d16s9 | 3.22 | 60.4 | 1.87 | 70.5 | 14.42 | 66.2 |
| d64s9 | 3.22 | 61.1 | 2.14 | 69.8 | 17.48 | 65.1 |
| d128s9 | 3.43 | 62.2 | 1.67 | 70.7 | 16.76 | 65.7 |
| **d256s9** | **2.98** | **62.6** | **1.77** | **70.4** | **18.19** | **64.9** |
| d1024s9 | 3.07 | 62.0 | 2.38 | 69.8 | 20.38 | 64.7 |
| w/o FSQ (d1024s∞) | 3.67 | 62.1 | 2.30 | 69.6 | 24.92 | 63.5 |

**关键发现**: 移除 FSQ（纯连续），ZH-hard CER 灾难性退化至 24.92%，验证了语义规划和声学渲染纠缠在连续空间中会导致不稳定。d256 为最优平衡点。d4 过度约束，d1024 离散化强度不足。

### Table 7: 消融实验 — RALM

| 配置 | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | ZH-hard CER↓ | ZH-hard SIM↑ |
|------|---------|---------|---------|---------|--------------|-------------|
| **默认配置** | **2.98** | **62.6** | **1.77** | **70.4** | **18.19** | **64.9** |
| w/o RALM: TSLM (24L) → LocDiT | 4.34 | 61.8 | 3.05 | 69.4 | 25.00 | 63.8 |
| w/o RALM: TSLM (30L) → LocDiT | 5.35 | 62.6 | 3.46 | 69.8 | 30.40 | 63.9 |
| w/o $\mathbf{E}_{<i}$ in RALM | 4.91 | 60.9 | 4.94 | 68.1 | 27.17 | 61.7 |
| w/o $\mathbf{h}^{\text{residual}}$: TSLM → FSQ → LocDiT | 3.86 | 58.3 | 3.05 | 67.6 | 23.65 | 61.7 |

**关键发现**: 移除 RALM（类似 [[DiTAR]] 架构），ZH-hard CER 从 18.19% 恶化至 25.00%。将 TSLM 扩展到 30 层（增加参数但无 RALM）反而更差（30.40%），证明问题是学习目标层面的，而非参数量不足。移除声学嵌入 $\mathbf{E}_{<i}$ 造成严重退化，RALM 需要细粒度声学信息输入。

### Table 8: 消融实验 — 训练阶段

| Phase | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | ZH-Hard CER↓ | ZH-Hard SIM↑ |
|-------|---------|---------|---------|---------|--------------|-------------|
| Stable | 2.05 | 69.7 | 0.99 | 75.1 | 13.22 | 68.6 |
| **Decay** | **1.85** | **72.9** | **0.93** | **77.2** | **8.87** | **73.0** |

**关键发现**: Decay 阶段在所有指标上带来一致提升。ZH-Hard CER 从 13.22% 降至 8.87%，SIM 提升 4.4 个点。WSD 调度对细化零样本语音相似度至关重要。

### Table 9: 消融实验 — CFG 值

| CFG | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | ZH-hard CER↓ | ZH-hard SIM↑ |
|-----|---------|---------|---------|---------|--------------|-------------|
| 1.0 (无 CFG) | 16.32 | 55.1 | 14.47 | 61.5 | 56.87 | 43.0 |
| 1.5 | 1.86 | 72.1 | 1.16 | 77.0 | 9.60 | 73.9 |
| **2.0** | **1.85** | **72.9** | **0.93** | **77.2** | **8.87** | **73.0** |
| 3.0 | 2.16 | 71.4 | 1.12 | 74.7 | 13.22 | 65.0 |
| 5.0 | 12.78 | 60.7 | 17.23 | 59.4 | 48.46 | 39.9 |

**关键发现**: CFG 对 VoxCPM 极为关键。无 CFG 时灾难性退化（ZH-hard CER 56.87%）。最优值 CFG=2.0，关系强非单调：≥3.0 时可懂度显著下降，5.0 时几乎不可用。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 大规模双语语料 | 180 万小时 | 中英双语，有声书/播客/访谈/广播剧；经源分离、VAD、ASR 对齐 | VoxCPM 训练 |
| [[Emilia]] | 9.5 万小时 | 公开中英数据集 | VoxCPM-Emilia 训练 |
| SEED-TTS-EVAL | - | 通用 TTS 评测（EN/ZH + Hard 子集） | 客观评测 |
| CV3-EVAL | - | 来自 CosyVoice 3 竞赛的表达性/野外语音克隆评测 | 客观评测 |

### 实现细节

- **Backbone**: [[MiniCPM]]-4-0.5B 初始化 TSLM
- **优化器**: AdamW, WSD (Warmup-Stable-Decay) 调度
- **Batch Size**: Stable 阶段 4096 tokens → Decay 阶段 8192 tokens
- **训练轮数**: 400K (stable) + 100K (decay) iterations
- **硬件**: 40 × NVIDIA H100
- **推理**: RTF 0.17 on RTX 4090
- **CFG**: 推理时 CFG = 2.0，训练时 mask probability = 0.1
- **VAE**: 独立训练，16kHz，25 Hz 帧率
- **数据增强**: 随机音素替换用于发音纠正

### 可视化结果

T-SNE 可视化（Figure 2, 3）验证了层次化分工：TSLM-FSQ 按语义/内容聚类，RALM 残差按说话人/声学特征聚类。此外，Demo Page 展示了方言克隆（四川话、河南话、粤语等）、环境录音条件克隆、数学符号朗读等能力。

---

## 批判性思考

### 优点
1. **架构洞察深刻**: FSQ 作为可微正则化而非预测目标的设计理念新颖，消融实验充分验证了每个组件的必要性
2. **消融极其充分**: 9 张表覆盖了 FSQ 维度、RALM 必要性、训练阶段、CFG 值四个维度，结论有数据支撑
3. **开源且强基线**: 0.5B 参数在 SEED-TTS-EVAL 上超越多个 1.5B-3B 开源系统和部分闭源系统，Apache 2.0 开源

### 局限性
1. **AudioVAE 仅支持 16kHz**: 在高保真场景（24kHz/44.1kHz）下不适用，限制了音质上限
2. **Hard 子集的 SIM 和 DNSMOS 落后**: CV3-Hard 上 SIM 66.1 vs CosyVoice3 的 78.6，极端难例上的说话人相似度差距较大
3. **训练数据量优势**: 180 万小时私有数据 vs Emilia 9.5 万小时对比显示数据贡献巨大，限制了方法论的独立评估
4. **多语种覆盖有限**: 仅中英双语
5. **可控性不足**: 缺乏对韵律、情感等属性的直观用户控制接口

### 潜在改进方向
1. 升级 AudioVAE 到 24kHz/44.1kHz 以提升音质上限
2. 引入显式韵律/情感控制机制（风格 token 或指令控制）
3. 扩展到更多语种，特别是低资源语言
4. 探索更大模型规模（1B+）以进一步缩小与闭源系统在 Hard 子集上的差距

### 可复现性评估
- [x] 代码开源（Apache 2.0）
- [x] 预训练模型（HuggingFace / ModelScope）
- [x] 训练细节完整（Table 1-2）
- [ ] 训练数据集可获取（180 万小时私有数据不可获取，Emilia 版本可复现）

---

## 关联笔记

### 基于
- [[MiniCPM]]: TSLM 骨干初始化
- [[DiTAR]]: LocDiT 设计参考（patch-based causal LM + 双向局部 diffusion），VoxCPM 在其基础上增加 FSQ + RALM
- [[DAC]]: AudioVAE 架构参考

### 对比
- [[CosyVoice 2]]: 离散 token 方案代表，SEED-TTS-EVAL 上 VoxCPM WER/CER 显著更优
- [[F5-TTS]]: 纯 Flow Matching NAR 方案，VoxCPM 在 WER 和 SIM 上均优
- [[IndexTTS 2]]: 1.5B 开源最强之一，VoxCPM 以 0.5B 参数达到相近水平
- [[SparkTTS]]: 单一 LLM-based TTS，VoxCPM 在所有指标上大幅领先
- [[MaskGCT]]: 非自回归离散 token 方案，VoxCPM 主观和客观指标均更优

### 方法相关
- [[FSQ]]: 核心瓶颈机制，Finite Scalar Quantization
- [[Flow Matching]]: LocDiT 训练目标
- [[Classifier-Free Guidance]]: 推理时的关键技巧
- [[RVQ]]: FSQ 对标的传统量化方案
- [[Straight-Through Estimator]]: FSQ 反向传播机制
- [[VAE]]: 音频编解码器

### 硬件/数据相关
- [[Emilia]]: 公开训练数据集（9.5 万小时）

---

## 速查卡片

> [!summary] VoxCPM: Tokenizer-Free TTS
> - **核心**: 可微 FSQ 瓶颈实现语义-韵律骨架与声学细节的结构性分离，端到端训练无需外部 tokenizer
> - **方法**: TSLM (MiniCPM-4 初始化) → FSQ 半离散化 → RALM 残差补回 → LocDiT Flow Matching 解码
> - **结果**: 开源 SOTA: EN WER 1.85%, ZH CER 0.93%, EN S-MOS 4.18; RTF 0.17 on RTX 4090
> - **代码**: [HuggingFace/OpenBMB](https://huggingface.co/OpenBMB) (Apache 2.0)

---

*笔记创建时间: 2026-05-25*
