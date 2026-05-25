---
title: "FastSpeech: Fast, Robust and Controllable Text to Speech"
method_name: "FastSpeech"
authors: [Yi Ren, Yangjun Ruan, Xu Tan, Tao Qin, Sheng Zhao, Zhou Zhao, Tie-Yan Liu]
year: 2019
venue: NeurIPS 2019
tags: [non-autoregressive-tts, parallel-generation, duration-predictor, feed-forward-transformer, knowledge-distillation, controllable-tts]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/1905.09263
created: 2026-05-25
---

# 论文笔记：FastSpeech: Fast, Robust and Controllable Text to Speech

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Zhejiang University, Microsoft Research, Microsoft STC Asia |
| 日期 | May 2019 (NeurIPS 2019) |
| 项目主页 | [Demo Samples](https://speechresearch.github.io/fastspeech/) |
| 对比基线 | [[Tacotron 2]] / [[Transformer]] TTS / Merlin |
| 链接 | [arXiv](https://arxiv.org/abs/1905.09263) |

---

## 一句话总结

> 首个基于 [[Feed-Forward Transformer]] 的并行 [[Mel-Spectrogram]] 生成 TTS 模型，通过 [[Duration Predictor]] + [[Length Regulator]] 实现 270x 加速、零跳词/重复错误和平滑语速控制。

---

## 核心贡献

1. **并行 Mel 生成**: 提出 Feed-Forward Transformer (FFT) 架构，完全 [[Non-Autoregressive|非自回归]]，并行生成 [[Mel-Spectrogram]]，mel 生成速度提升 **270x**，端到端提升 **38x**
2. **鲁棒性突破**: 通过 [[Duration Predictor]] 提供硬对齐（hard alignment），在 50 个 hard case 上实现 **0% 错误率**（Tacotron 2 为 24%，Transformer TTS 为 34%）
3. **可控性**: [[Length Regulator]] 通过调节 $\alpha$ 参数实现 0.5x-1.5x 平滑语速控制，并可在词间插入停顿控制韵律

---

## 问题背景

### 要解决的问题

当时主流的自回归 TTS 模型（[[Tacotron 2]]、Transformer TTS）存在三个核心问题：
1. **推理慢**: mel-spectrogram 逐帧生成，序列长度达数百到数千帧，无法并行
2. **鲁棒性差**: soft attention 机制导致错误传播（error propagation），产生跳词（word skipping）和重复（word repeating）
3. **可控性弱**: 缺乏显式的文本-语音对齐，难以控制语速和韵律

### 现有方法的局限

- [[Tacotron 2]]、Transformer TTS 等 AR 模型质量高但速度慢，且 attention 不稳定
- Parallel WaveNet / ClariNet / [[WaveGlow]] 等并行声码器只加速了 waveform 生成，mel-spectrogram 仍然是自回归生成的瓶颈
- 同期工作 Peng et al. [17] 虽然也并行生成 mel，但使用 encoder-decoder + attention 结构，参数量是教师模型的 2-3 倍

### 本文的动机

mel-spectrogram 序列虽长，但与 [[Phoneme]] 序列之间存在确定性的时长对应关系。如果能提前预测每个音素的持续时长，就可以把音素序列"展开"成帧序列后并行生成 mel，从根本上消除自回归的速度和鲁棒性瓶颈。

---

## 方法详解

### 模型架构

FastSpeech 采用 **Feed-Forward Transformer (FFT)** 架构：
- **输入**: [[Phoneme]] 序列（通过 [[G2P]] 从文本转换）
- **Backbone**: 两组堆叠的 FFT blocks，中间用 [[Length Regulator]] 连接
- **核心模块**: [[Duration Predictor]] 预测音素时长 + [[Length Regulator]] 展开序列
- **输出**: 80 维 [[Mel-Spectrogram]]，再经 [[WaveGlow]] 声码器生成波形
- **总参数**: 30.1M（与教师模型 30.7M 相当）

### 核心模块

#### 模块1: Feed-Forward Transformer (FFT)

**设计动机**: 替代传统 encoder-decoder + attention 结构，避免 soft attention 带来的对齐不稳定问题。

**具体实现**:
- 音素侧 N=6 个 FFT blocks + mel 侧 N=6 个 FFT blocks，中间由 [[Length Regulator]] 桥接
- 每个 FFT block 包含：
  - Multi-head [[Self-Attention]]（2 heads，hidden=384）用于跨位置信息交互
  - **2 层 1D 卷积网络**（kernel=3，中间维度 1536）替代标准 Transformer 的 position-wise FFN
  - 动机：语音序列中**相邻隐状态关联性更强**，1D Conv 比全连接更适合捕捉局部模式
  - 每个子层后接残差连接 + [[Layer Normalization]] + Dropout(0.1)

#### 模块2: Length Regulator

**设计动机**: 解决 [[Phoneme]] 序列（长度 $n$）与 [[Mel-Spectrogram]] 序列（长度 $m \gg n$）之间的长度不匹配问题。

**具体实现**:
- 根据 [[Duration Predictor]] 预测的音素时长 $\mathcal{D} = [d_1, d_2, \ldots, d_n]$，将音素侧隐状态按时长复制展开
- 通过超参数 $\alpha$ 控制语速：$\alpha > 1$ 变慢，$\alpha < 1$ 变快
- 可通过增大空格字符的 duration 在词间插入停顿

#### 模块3: Duration Predictor

**设计动机**: 为 [[Length Regulator]] 提供准确的音素时长，同时保证硬对齐（hard alignment）消除跳词/重复。

**具体实现**:
- 2 层 1D 卷积（kernel=3，filter=256）+ ReLU + [[Layer Normalization]] + Dropout
- 最后一个线性层输出标量（预测时长）
- 在**对数域**预测时长（使分布更接近高斯，更易训练）
- 训练时用 MSE loss 与 ground-truth 时长联合训练；推理时使用预测时长
- **Ground-truth 时长提取**：从训练好的自回归 Transformer TTS 教师模型的 encoder-decoder attention 中提取

---

## 关键公式

### 公式1: [[Length Regulator|长度调节]]

$$
\mathcal{H}_{mel} = \mathcal{LR}(\mathcal{H}_{pho}, \mathcal{D}, \alpha)
$$

**含义**: 将音素侧隐状态序列根据时长信息展开为 mel 侧序列。

**符号说明**:
- $\mathcal{H}_{pho} = [h_1, h_2, \ldots, h_n]$: 音素侧 FFT 输出隐状态序列，长度 $n$
- $\mathcal{D} = [d_1, d_2, \ldots, d_n]$: 音素时长序列，满足 $\sum_{i=1}^{n} d_i = m$（mel 帧数）
- $\alpha$: 语速控制因子（1.0=正常，>1 慢速，<1 快速）
- $\mathcal{H}_{mel}$: 展开后的 mel 侧隐状态序列

### 公式2: [[Focus Rate|注意力聚焦率]]

$$
F = \frac{1}{S} \sum_{s=1}^{S} \max_{1 \leq t \leq T} a_{s,t}
$$

**含义**: 衡量教师模型 attention 矩阵的对角性程度，用于选择最佳 attention head 提取时长。

**符号说明**:
- $S$: ground-truth mel-spectrogram 长度
- $T$: [[Phoneme]] 序列长度
- $a_{s,t}$: attention 矩阵中第 $s$ 行第 $t$ 列的权重
- $F$ 值越大说明 attention 越接近对角线（单调对齐）

### 公式3: [[Duration Predictor|音素时长提取]]

$$
d_i = \sum_{s=1}^{S} [\arg\max_t \; a_{s,t} = i]
$$

**含义**: 从教师模型的 attention 矩阵中提取每个音素对应的 mel 帧数作为 ground-truth 时长。

**符号说明**:
- $d_i$: 第 $i$ 个音素的时长（对应的 mel 帧数）
- $[\cdot]$: Iverson 括号，条件为真时取 1
- 对于每一帧 $s$，找到 attention 权重最大的音素位置，统计指向音素 $i$ 的帧数

---

## 关键图表

### Figure 1: Overall Architecture / 系统概览

#### (a) Feed-Forward Transformer 整体结构

![Figure 1a: FFT overall](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x1.png)

**说明**: FastSpeech 整体架构。左侧为音素侧 N 个 FFT blocks，中间为 [[Length Regulator]]，右侧为 mel 侧 N 个 FFT blocks，最终输出线性层生成 80 维 mel-spectrogram。

#### (b) FFT Block 内部结构

![Figure 1b: FFT Block](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x2.png)

**说明**: 每个 FFT block 由 Multi-Head [[Self-Attention]] + 2 层 1D Conv 组成，各子层后接 Add & Norm。1D Conv 替代标准 FFN 以更好捕捉局部时序特征。

#### (c) Length Regulator

![Figure 1c: Length Regulator](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x3.png)

**说明**: [[Length Regulator]] 根据 [[Duration Predictor]] 输出的时长将音素隐状态复制展开。示例：$\mathcal{D}=[2,2,3,1]$ 时 $h_1$ 复制 2 次、$h_3$ 复制 3 次等。

#### (d) Duration Predictor

![Figure 1d: Duration Predictor](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x4.png)

**说明**: [[Duration Predictor]] 由 2 层 Conv1D(kernel=3) + ReLU + LayerNorm + Dropout 堆叠，最终线性层输出标量时长。训练时 MSE loss 仅在训练阶段有效。

### Figure 2: Inference Speedup / 推理速度对比

#### (a) FastSpeech 推理时间

![Figure 2a: FastSpeech inference time](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x5.png)

#### (b) Transformer TTS 推理时间

![Figure 2b: Transformer TTS inference time](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x6.png)

**说明**: FastSpeech 的推理时间随 mel 长度几乎不增长（并行生成），而 Transformer TTS 的推理时间随序列长度线性增长。体现了 [[Non-Autoregressive|非自回归]] 的核心优势。

### Figure 3: Voice Speed Control / 语速控制

#### (a) 1.5x 速度

![Figure 3a: 1.5x speed](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x7.png)

#### (b) 1.0x 正常速度

![Figure 3b: 1.0x speed](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x8.png)

#### (c) 0.5x 速度

![Figure 3c: 0.5x speed](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x9.png)

**说明**: 通过 [[Length Regulator]] 的 $\alpha$ 参数实现 0.5x-1.5x 平滑语速调节。mel-spectrogram 可以看到明显的时间拉伸/压缩，但**音高保持稳定不变**。

### Figure 4: Break Control / 停顿控制

#### (a) 原始 mel-spectrogram

![Figure 4a: Original](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x10.png)

#### (b) 插入停顿后

![Figure 4b: With breaks](https://ar5iv.labs.arxiv.org/html/1905.09263/assets/x11.png)

**说明**: 在 "deeply" 和 "especially" 后增大空格字符的 duration，成功插入停顿（红框标注）。展示了 [[Duration Predictor]] 提供的显式对齐对韵律控制的价值。

### Table 1: 音频质量 (MOS)

| Method | MOS |
|--------|-----|
| GT | 4.41 +- 0.08 |
| GT (Mel + WaveGlow) | 4.00 +- 0.09 |
| Tacotron 2 (Mel + WaveGlow) | 3.86 +- 0.09 |
| Merlin (WORLD) | 2.40 +- 0.13 |
| Transformer TTS (Mel + WaveGlow) | 3.88 +- 0.09 |
| **FastSpeech (Mel + WaveGlow)** | **3.84 +- 0.08** |

**说明**: FastSpeech 的 [[MOS]] 为 3.84，与 [[Tacotron 2]]（3.86）和 Transformer TTS（3.88）几乎持平，证明并行生成不牺牲质量。质量上限受 [[WaveGlow]] 声码器制约（GT Mel+WaveGlow 仅 4.00）。

### Table 2: 推理延迟对比

| Method | Latency (s) | Speedup |
|--------|-------------|---------|
| Transformer TTS (Mel) | 6.735 +- 3.969 | / |
| **FastSpeech (Mel)** | **0.025 +- 0.005** | **269.40x** |
| Transformer TTS (Mel + WaveGlow) | 6.895 +- 3.969 | / |
| **FastSpeech (Mel + WaveGlow)** | **0.180 +- 0.078** | **38.30x** |

**说明**: mel 生成加速 **269x**；加上 [[WaveGlow]] 声码器后端到端加速 **38x**。评测条件：1 NVIDIA V100 GPU，batch size 1，平均生成 mel 长度约 560 帧。

### Table 3: 鲁棒性测试（50 个 Hard Sentences）

| Method | Repeats | Skips | Error Sentences | Error Rate |
|--------|---------|-------|-----------------|------------|
| Tacotron 2 | 4 | 11 | 12 | 24% |
| Transformer TTS | 7 | 15 | 17 | 34% |
| **FastSpeech** | **0** | **0** | **0** | **0%** |

**说明**: FastSpeech 在包含单字母、拼读、重复数字、长句等 50 个困难样本上实现 **零错误**，彻底消除了 AR 模型因 attention 漂移导致的跳词/重复问题。

### Table 4: 消融实验 (CMOS)

| 配置 | CMOS | 说明 |
|------|------|------|
| FastSpeech (full) | 0 (reference) | 完整模型 |
| w/o 1D Conv in FFT block | -0.113 | 用标准 FFN 替代 1D Conv，局部特征建模变弱 |
| w/o Sequence-level KD | -0.325 | 不使用 [[Knowledge Distillation|知识蒸馏]]，质量下降显著 |

**关键发现**: [[Knowledge Distillation|序列级知识蒸馏]]对质量贡献最大（CMOS -0.325），1D Conv 替代 FFN 也有明确收益（CMOS -0.113）。

### Table 5: 完整超参数对比（附录）

| Hyperparameter | Transformer TTS | FastSpeech |
|---|---|---|
| Phoneme Embedding Dim | 384 | 384 |
| Pre-net Layers | 3 | / |
| Encoder/Phoneme FFT Layers | 6 | 6 |
| Encoder/Phoneme FFT Hidden | 384 | 384 |
| Conv1D Kernel | 3 | 3 |
| Conv1D Filter Size | 1024 | 1536 |
| Attention Heads | 2 | 2 |
| Decoder/Mel FFT Layers | 6 | 6 |
| Duration Predictor Conv Kernel | / | 3 |
| Duration Predictor Filter Size | / | 256 |
| Dropout | 0.1 | 0.1 |
| Batch Size | 64 (16x4 GPUs) | 64 (16x4 GPUs) |
| **Total Parameters** | **30.7M** | **30.1M** |

**说明**: FastSpeech 参数量（30.1M）与教师模型（30.7M）几乎相同，但推理速度快两个数量级。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LJSpeech]] | ~24h, 13100 clips | 单说话人英文有声书 | 训练/验证/测试 (12500/300/300) |

### 实现细节

- **Backbone**: Feed-Forward Transformer (6+6 FFT blocks)
- **优化器**: Adam ($\beta_1=0.9$, $\beta_2=0.98$, $\epsilon=10^{-9}$)
- **学习率**: Transformer warmup schedule (per Vaswani et al.)
- **Batch Size**: 64 (16 per GPU x 4 GPUs)
- **训练轮数**: ~80k steps（教师模型和 FastSpeech 各 80k steps）
- **硬件**: 4 NVIDIA V100 GPUs
- **声码器**: 预训练 [[WaveGlow]]
- **训练流程**:
  1. 先训练自回归 Transformer TTS 教师模型 (80k steps)
  2. 用教师模型提取 attention alignment，得到 ground-truth 音素时长
  3. 用教师模型对训练集做 [[Knowledge Distillation|序列级知识蒸馏]]（source text + teacher-generated mel 作为训练对）
  4. 训练 FastSpeech (80k steps)，[[Duration Predictor]] 联合训练

### 可视化结果

- 语速控制（Figure 3）：0.5x-1.5x 速度变化平滑，音高保持稳定
- 停顿控制（Figure 4）：通过调整空格 duration 成功在指定位置插入停顿
- 推理时间（Figure 2）：FastSpeech 推理时间几乎不随序列长度增长

---

## 批判性思考

### 优点
1. **开创性工作**: 首次在 TTS 中实现完全非自回归的 mel 生成，开辟了 [[Non-Autoregressive|NAR TTS]] 研究方向，后续 [[FastSpeech 2]]、[[VITS]] 等重要工作都受其影响
2. **三位一体的改进**: 同时解决速度、鲁棒性、可控性三个问题，而非单点突破
3. **简洁有效的设计**: [[Length Regulator]] 机制简单直觉（按时长复制隐状态），但效果极佳
4. **实验充分**: 除常规 MOS 外，专门设计了 50 个 hard case 的鲁棒性测试和消融实验

### 局限性
1. **依赖教师模型**: 需要先训练一个自回归 Transformer TTS 作为教师，然后提取 attention alignment 和做知识蒸馏，训练流程复杂（[[FastSpeech 2]] 后来用 [[Forced Alignment|MFA]] 替代了教师模型）
2. **单说话人**: 仅在 [[LJSpeech]] 单说话人数据上验证，未涉及多说话人或零样本场景
3. **质量略低于 AR**: MOS 3.84 vs Transformer TTS 3.88，存在一定质量差距（虽在置信区间内重叠）
4. **声码器瓶颈**: 依赖外部 [[WaveGlow]] 声码器，GT Mel+WaveGlow 仅 4.00 vs GT 4.41，声码器引入了质量损失
5. **Duration 建模过于简化**: 将音素时长建模为标量复制次数，无法捕捉帧级的声学变化细节

### 潜在改进方向
1. 去除教师模型依赖 -- 后续 [[FastSpeech 2]] 通过 [[Forced Alignment|MFA]] 直接提取时长解决
2. 端到端并行合成 -- 后续 [[VITS]] 实现了端到端 TTS（不需要外部声码器）
3. 更精细的时长/韵律建模 -- [[Duration Predictor]] 可扩展为预测 pitch/energy 等（[[FastSpeech 2]] 已做）
4. 多说话人/零样本 -- 后续 [[VALL-E]]、[[CosyVoice]] 等在零样本场景大放异彩

### 可复现性评估
- [x] 代码开源（社区多个实现，如 ESPnet、ming024/FastSpeech2）
- [ ] 官方预训练模型（未提供）
- [x] 训练细节完整（超参数、训练步数、GPU 配置均详细说明）
- [x] 数据集可获取（[[LJSpeech]] 公开可下载）

---

## 关联笔记

### 基于
- [[Transformer]]: 核心 backbone 架构（Vaswani et al. 2017）
- [[Tacotron 2]]: 对比的 AR TTS 基线（Shen et al. 2018）
- [[WaveGlow]]: 使用的并行声码器（Prenger et al. 2019）

### 对比
- [[Tacotron 2]]: AR TTS 基线，MOS 3.86 vs FastSpeech 3.84
- [[WaveNet]]: 开创性的自回归声码器

### 后续工作
- [[FastSpeech 2]]: 改进版，去除教师模型依赖，增加 pitch/energy 预测
- [[VITS]]: 端到端 NAR TTS，整合声码器
- [[Non-Autoregressive|NAR TTS 范式]]: FastSpeech 开辟的研究方向

### 方法相关
- [[Duration Predictor]]: 核心组件 -- 预测音素时长
- [[Length Regulator]]: 核心组件 -- 按时长展开序列
- [[Knowledge Distillation]]: 训练技巧 -- 用教师模型生成的 mel 作为训练目标
- [[Attention Alignment]]: 从教师模型提取时长的关键
- [[Feed-Forward Transformer]]: FFT 架构（1D Conv 替代 FFN）
- [[Mel-Spectrogram]]: 中间表示（80 维）
- [[Phoneme]]: 输入表示
- [[G2P]]: 文本到音素转换
- [[Focus Rate]]: 选择最佳 attention head 的指标
- [[Layer Normalization]]: FFT block 中的归一化

### 硬件/数据相关
- [[LJSpeech]]: 训练/评测数据集（~24h 单说话人英文）
- [[MOS]]: 主观评测指标
- [[CMOS]]: 消融实验评测指标

---

## 速查卡片

> [!summary] FastSpeech: Fast, Robust and Controllable Text to Speech
> - **核心**: 首个完全非自回归并行 mel 生成 TTS，开创 NAR TTS 研究方向
> - **方法**: Feed-Forward Transformer + Duration Predictor + Length Regulator + 教师模型知识蒸馏
> - **结果**: MOS 3.84（接近 AR），mel 生成 270x 加速，hard case 0% 错误率，支持语速/停顿控制
> - **代码**: 社区实现（ESPnet / ming024/FastSpeech2）

---

*笔记创建时间: 2026-05-25*
