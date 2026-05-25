---
title: "NaturalSpeech: End-to-End Text to Speech Synthesis with Human-Level Quality"
method_name: "NaturalSpeech"
authors: [Xu Tan, Jiawei Chen, Haohe Liu, Jian Cong, Chen Zhang, Yanqing Liu, Xi Wang, Yichong Leng, Yuanhao Yi, Lei He, Frank Soong, Tao Qin, Sheng Zhao, Tie-Yan Liu]
year: 2022
venue: TPAMI
tags: [tts, vae, end-to-end, human-level, flow, phoneme-pretraining, duration-modeling]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2205.04421
created: 2026-05-25
---

# 论文笔记：NaturalSpeech: End-to-End Text to Speech Synthesis with Human-Level Quality

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft Research Asia & Microsoft Azure Speech |
| 日期 | May 2022 |
| 项目主页 | https://speechresearch.github.io/naturalspeech/ |
| 对比基线 | [[FastSpeech 2]] + [[HiFi-GAN]], [[Glow-TTS]] + [[HiFi-GAN]], [[Grad-TTS]] + [[HiFi-GAN]], [[VITS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2205.04421) |

---

## 一句话总结

> 基于 [[VAE]] 的端到端 TTS 系统，通过 phoneme pre-training、differentiable durator、bidirectional prior/posterior 和 memory 机制四个模块系统性地缩小 prior-posterior gap，在 [[LJSpeech]] 上首次达到与人类录音无统计显著差异的质量（-0.01 [[CMOS]]）。

---

## 核心贡献

1. **人类水平质量的形式化定义**: 首次给出基于统计显著性（Wilcoxon signed rank test, p > 0.05）的 human-level TTS 定义，并用 [[CMOS]] 而非 [[MOS]] 作为判断标准
2. **系统性缩小 prior-posterior gap 的四模块设计**: phoneme pre-training 增强文本表示、differentiable durator 消除时长建模的 train-inference mismatch、bidirectional prior/posterior 同时简化后验增强先验、memory VAE 降低后验复杂度
3. **首次在 LJSpeech 上实现人类水平 TTS**: -0.01 CMOS（p = 0.69），28.7M 参数，RTF = 0.013

---

## 问题背景

### 要解决的问题

TTS 系统的合成语音质量能否达到人类水平？如何定义"人类水平"？如何实现？

### 现有方法的局限

作者复现并严格评测了四个主流 TTS 系统，发现均未达到人类水平：

| 系统 | CMOS vs 录音 | Wilcoxon p-value |
|------|-------------|------------------|
| [[FastSpeech 2]] + [[HiFi-GAN]] | -0.30 | 5.1e-20 |
| [[Glow-TTS]] + [[HiFi-GAN]] | -0.23 | 8.7e-17 |
| [[Grad-TTS]] + [[HiFi-GAN]] | -0.23 | 1.2e-11 |
| [[VITS]] | -0.19 | 2.9e-04 |

所有系统 p << 0.05，存在统计显著差异。

质量差距的根源分析（Appendix A，以 [[FastSpeech 2]] + [[HiFi-GAN]] 为例）：
1. **Training-inference mismatch**: 训练用 GT duration/pitch，推理用预测值
2. **One-to-many mapping**: 同一文本对应多种语音变体（语速、韵律、情感）
3. **Representation capacity 不足**: phoneme encoder 和生成模型表达力有限

### 本文的动机

用 [[VAE]] 框架统一建模 speech posterior $q(z|x)$ 和 text prior $p(z|y)$，针对 gap 的每个来源设计专门模块：phoneme pre-training 增强表示容量、differentiable durator 消除时长 mismatch、bidirectional prior/posterior 双向拉近分布、memory VAE 降低 one-to-many 难度。

---

## 方法详解

### 模型架构

NaturalSpeech 采用 **[[VAE]] + [[Normalizing Flow|Flow]] + [[GAN]]** 混合架构：

- **输入**: [[Phoneme]] 序列 $y$（通过 [[G2P|Grapheme-to-Phoneme]] 转换）
- **Posterior Encoder** $\phi$: 16 层 [[WaveNet]]，从线性频谱图提取后验分布 $q(z|x; \phi)$
- **Phoneme Encoder** $\theta_{\text{pho}}$: 6 层 [[Feed-Forward Transformer]]，预训练后微调
- **Differentiable Durator** $\theta_{\text{dur}}$: [[Duration Predictor]] + 可学习上采样层
- **Bidirectional Prior/Posterior** $\theta_{\text{bpp}}$: 4 层 [[Affine Coupling Layer]]（[[Normalizing Flow]]）
- **Waveform Decoder** $\theta_{\text{dec}}$: [[HiFi-GAN]] 架构 + Memory Bank 注意力
- **总推理参数**: 28.7M

### 核心模块

#### 模块 1: Phoneme Pre-training

**设计动机**: 利用大规模文本数据增强 [[Phoneme]] encoder 的表示能力

**具体实现**:
- 在 200M 句子（news-crawl）上做 mixed-phoneme 预训练
- 同时使用 phoneme（音素粒度，词典 182）和 sup-phoneme（[[BPE]] 合并的相邻音素，词典 30,088）作为输入
- 训练目标：masked language modeling，同时 mask 并预测 sup-phoneme token 和对应 phoneme token
- Mask ratio: 15%
- 训练：8 x A100 80G，120k steps，batch size 1024 句

#### 模块 2: Differentiable Durator

**设计动机**: 消除传统 [[Duration Predictor]] 训练时用 GT duration、推理时用预测 duration 的 mismatch

**具体实现**:
- **Duration Predictor**: 3 层 1D 卷积 + ReLU + [[Layer Normalization]] + Dropout，预测每个 phoneme 的时长 $\hat{d}$
- **Learnable Upsampling Layer**: 基于预测时长构建可微分投影矩阵，将 phoneme hidden $H_{n \times h}$ 平滑扩展到帧级 $O_{m \times h}$
  - 计算 start/end 矩阵 $S, E$（公式 6）
  - 通过 MLP + Softmax 生成注意力权重 $W$（公式 7）和辅助上下文 $C$（公式 8）
  - 帧级输出 $O = \text{Proj}(WH) + \text{Proj}(\text{Einsum}(W, C))$（公式 9）
- 输出经两个线性层得到先验分布的均值 $\mu$ 和方差 $\sigma$
- 训练全程可微分，梯度可从先验损失反传到 duration predictor

#### 模块 3: Bidirectional Prior/Posterior

**设计动机**: 利用 [[Normalizing Flow]] 的可逆性，**同时**简化后验和增强先验

**具体实现**:
- 4 层 [[Affine Coupling Layer]]，去掉 scaling 仅保留 shifting（稳定性），shifting 用 4 层 [[WaveNet]]（dilation rate 1）估计
- **Backward mapping** $f^{-1}$: 将后验 $q(z|x; \phi)$ 映射到更简单的 $q(z'|x; \phi, \theta_{\text{bpp}})$，用 $\mathcal{L}_{\text{bwd}}$（公式 1）训练
- **Forward mapping** $f$: 将先验 $p(z'|y; \theta_{\text{pri}})$ 映射到更强的 $p(z|y; \theta_{\text{pri}}, \theta_{\text{bpp}})$，用 $\mathcal{L}_{\text{fwd}}$（公式 2）训练
- 双向训练消除了传统 flow 模型"反向训练、正向推理"的 mismatch

#### 模块 4: VAE with Memory

**设计动机**: 通过 memory bank 进一步降低后验复杂度——$z$ 只需决定注意力权重而非直接编码所有声学细节

**具体实现**:
- Memory Bank $M \in \mathbb{R}^{L \times h}$，$L$ 为 bank 大小，$h$ 为隐藏维度
- 用 $z$ 作为 query，$M$ 作为 key/value，做标准 Q-K-V 注意力
- 注意力结果（而非 $z$ 本身）送入 waveform decoder 重建波形
- 隐式学习 pitch 等变异信息，无需显式 pitch predictor

---

## 关键公式

### 公式 1: [[KL Divergence|Backward KL Loss]]

$$
\mathcal{L}_{\text{bwd}}(\phi, \theta_{\text{bpp}}, \theta_{\text{pri}}) = \mathbb{E}_{z \sim q(z|x;\phi)} \Big[ \log q(z|x;\phi) - \log p\big(f^{-1}(z;\theta_{\text{bpp}})|y;\theta_{\text{pri}}\big) - \log \big|det \frac{\partial f^{-1}(z;\theta_{\text{bpp}})}{\partial z}\big| \Big]
$$

**含义**: 通过 flow 的反向映射 $f^{-1}$ 将后验 $q(z|x)$ 变换到更简单的分布 $q(z'|x)$，然后与先验 $p(z'|y)$ 做 KL 散度

**符号说明**:
- $q(z|x; \phi)$: [[Posterior Encoder]] 输出的后验分布
- $p(z'|y; \theta_{\text{pri}})$: phoneme encoder + durator 输出的先验分布
- $f^{-1}(z; \theta_{\text{bpp}})$: flow 反向映射，$z \to z'$
- $\det \frac{\partial f^{-1}}{\partial z}$: Jacobian 行列式（flow 变换的体积变化因子）

### 公式 2: [[KL Divergence|Forward KL Loss]]

$$
\mathcal{L}_{\text{fwd}}(\phi, \theta_{\text{bpp}}, \theta_{\text{pri}}) = \mathbb{E}_{z' \sim p(z'|y;\theta_{\text{pri}})} \Big[ \log p(z'|y;\theta_{\text{pri}}) - \log q\big(f(z';\theta_{\text{bpp}})|x;\phi\big) - \log \big|det \frac{\partial f(z';\theta_{\text{bpp}})}{\partial z'}\big| \Big]
$$

**含义**: 通过 flow 的正向映射 $f$ 增强先验分布的表达能力，与后验做反向 KL 散度

**符号说明**:
- $f(z'; \theta_{\text{bpp}})$: flow 正向映射，$z' \to z$
- 其余同公式 1

### 公式 3: [[VAE|Memory VAE Reconstruction Loss]]

$$
\mathcal{L}_{\text{rec}}(\phi, \theta_{\text{dec}}) = -\mathbb{E}_{z \sim q(z|x;\phi)} \big[ \log p(x | \text{Attention}(z, M, M); \theta_{\text{dec}}) \big]
$$

其中注意力机制为：

$$
\text{Attention}(Q, K, V) = \Big[ \text{softmax}\Big(\frac{QW_Q(KW_K)^T}{\sqrt{h}}\Big) VW_V \Big] W_O
$$

**含义**: 用 $z$ 查询 memory bank $M$ 得到增强表示，再通过 waveform decoder 重建波形

**符号说明**:
- $M \in \mathbb{R}^{L \times h}$: memory bank，$L$ 为 bank 大小
- $W_Q, W_K, W_V, W_O \in \mathbb{R}^{h \times h}$: 注意力投影矩阵
- $\theta_{\text{dec}}$: 包含 decoder、memory bank、注意力参数

### 公式 4: [[ELBO|End-to-End Loss]]

$$
\mathcal{L}_{\text{e2e}}(\theta_{\text{pri}}, \theta_{\text{bpp}}, \theta_{\text{dec}}) = -\mathbb{E}_{z' \sim p(z'|y;\theta_{\text{pri}})} \big[ \log p(x | \text{Attention}(f(z';\theta_{\text{bpp}}), M, M); \theta_{\text{dec}}) \big]
$$

**含义**: 从先验采样 → flow 正向映射 → memory 注意力 → 重建波形的端到端损失，确保推理路径在训练中被直接优化

### 公式 5: [[VAE|Total Loss]]

$$
\mathcal{L} = \mathcal{L}_{\text{bwd}} + \mathcal{L}_{\text{fwd}} + \mathcal{L}_{\text{rec}} + \mathcal{L}_{\text{e2e}}
$$

**含义**: 总损失由四项组成——反向 KL + 正向 KL + 重建 + 端到端

**符号说明**:
- $\theta_{\text{pri}} = [\theta_{\text{pho}}, \theta_{\text{dur}}]$: 先验参数 = phoneme encoder + durator
- $\mathcal{L}_{\text{rec}}$ 包含 [[GAN]] loss、[[Feature Matching Loss]] 和 [[Mel Loss]]（沿用 [[HiFi-GAN]]）
- $\mathcal{L}_{\text{e2e}}$ 仅用 GAN loss
- $\mathcal{L}_{\text{bwd}}$ 和 $\mathcal{L}_{\text{fwd}}$ 使用 soft-[[DTW]] 版本的 KL 处理先验/后验帧长不一致问题

### 公式 6-9: Differentiable Durator 细节

**公式 6** — Duration start/end 矩阵：

$$
S_{i,j} = i - \sum_{k=1}^{j-1} d_k, \quad E_{i,j} = \sum_{k=1}^{j} d_k - i
$$

**公式 7** — Primary attention weight $W_{m \times n \times q}$：

$$
W = \text{Softmax}\Big(\underset{10 \to q}{\text{MLP}}\big([S, E, \text{Expand}(\text{Conv1D}(\text{Proj}(H)))]\big)\Big)
$$

**公式 8** — Auxiliary context $C_{m \times n \times p}$：

$$
C = \underset{10 \to p}{\text{MLP}}\big([S, E, \text{Expand}(\text{Conv1D}(\text{Proj}(H)))]\big)
$$

**公式 9** — 帧级输出：

$$
O = \underset{qh \to h}{\text{Proj}}(WH) + \underset{qp \to h}{\text{Proj}}(\text{Einsum}(W, C))
$$

**符号说明**:
- $H_{n \times h}$: phoneme hidden sequence，$n$ 为 phoneme 数，$h$ 为隐藏维度
- $O_{m \times h}$: 帧级输出，$m$ 为帧数
- $p = 2, q = 4$: 超参数
- Proj: 线性层 $h \to h$；Conv1D 输出维度 8；拼接后维度 $1+1+8=10$

### 公式 12-13: [[DTW|Soft-DTW]] KL Loss

$$
r_{i,j} = \min^{\gamma} \begin{cases}
r_{i-1,j} + KL_{\text{pair}} + \text{warp} \\
r_{i,j-1} + KL_{\text{pair}} + \text{warp} \\
r_{i-1,j-1} + KL_{\text{pair}}
\end{cases}
$$

**含义**: 因先验和后验帧长不同，无法逐帧计算 KL，故用 soft-DTW 动态对齐后计算

**符号说明**:
- $\min^{\gamma}(a_1, \ldots, a_n) = -\gamma \log \sum_i e^{-a_i / \gamma}$: 可微 soft-min，$\gamma = 0.01$
- $\text{warp} = 0.07$: 非对角路径惩罚

### 公式 14-16: Waveform Decoder Losses

**公式 14** — [[GAN|LS-GAN Loss]]:

$$
\mathbb{E}_x[(D(x)-1)^2] + \mathbb{E}_z[D(G(z))^2]
$$

**公式 15** — [[Feature Matching Loss]]:

$$
\mathbb{E}_{(x,z)} \Big[\sum_l \frac{1}{N_l} \|D^l(x) - D^l(G(z))\|_1 \Big]
$$

**公式 16** — [[Mel Loss|Mel-Spectrogram Loss]]:

$$
\mathbb{E}_{(x,z)} \|S(x) - S(G(z))\|_1
$$

### 公式 17-18: [[Monotonic Alignment Search|MAS]] for Duration Warmup

**公式 17** — 最优对齐：

$$
A = \underset{A}{\arg\max} \sum_{i=1}^{m} \mathcal{N}\big(z'_i; \mu(y;\theta_{\text{pho}})_{A(i)}, \sigma(y;\theta_{\text{pho}})_{A(i)}\big)
$$

**公式 18** — 动态规划：

$$
Q_{i,j} = \max(Q_{i-1,j-1}, Q_{i-1,j}) + \log \mathcal{N}\big(z'_i; \mu(y;\theta_{\text{pho}})_j, \sigma(y;\theta_{\text{pho}})_j\big)
$$

**含义**: Warmup 阶段（前 1k epoch）用 MAS 提供 duration label 加速收敛，之后切换为 differentiable durator 的预测 duration

---

## 关键图表

### Figure 1: System Overview / 系统概览

![Figure 1: NaturalSpeech 系统总览](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x1.png)

**说明**: NaturalSpeech 整体架构。输入 [[Phoneme]] 序列经 pre-trained phoneme encoder 编码，differentiable durator 上采样到帧级，得到先验分布 $p(z'|y)$。训练时后验 $q(z|x)$ 从线性频谱图提取。Bidirectional prior/posterior（flow）双向连接先验和后验空间。Memory-augmented VAE decoder 利用 memory bank 生成波形。

### Figure 2(a): Differentiable Durator

![Figure 2(a): Differentiable Durator](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x2.png)

**说明**: [[Duration Predictor]] 预测每个 phoneme 的时长 $\hat{d}$，learnable upsampling layer 基于 $\hat{d}$ 构建可微投影矩阵 $W$，将 phoneme hidden 平滑扩展到帧级。整个过程可微，梯度可从重建损失反传到 duration predictor。

### Figure 2(b): Bidirectional Prior/Posterior

![Figure 2(b): Bidirectional Prior/Posterior](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x3.png)

**说明**: [[Normalizing Flow]] 模块的双向使用。Backward mapping $f^{-1}$ 简化后验（从 $z$ 空间到 $z'$ 空间），forward mapping $f$ 增强先验（从 $z'$ 空间到 $z$ 空间）。训练时两个方向都被优化。

### Figure 2(c): Phoneme Pre-training

![Figure 2(c): Phoneme Pre-training](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x4.png)

**说明**: Mixed-phoneme 预训练。同时使用 phoneme 和 sup-phoneme（[[BPE]] 合并）两种粒度的输入，mask 后同时预测两种粒度的 token。200M 句子语料，120k steps 训练。

### Figure 2(d): Memory Mechanism in VAE

![Figure 2(d): Memory Mechanism in VAE](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x5.png)

**说明**: 后验采样得到的 $z$ 作为 query 查询 memory bank $M$，注意力输出替代 $z$ 送入 decoder。这样 $z$ 只需编码"选哪些 memory"的信息，降低了后验需要承载的信息量。

### Figure 3: Gradient Flows / 梯度流

![Figure 3: Gradient Flows](https://ar5iv.labs.arxiv.org/html/2205.04421/assets/x6.png)

**说明**: 训练中的六条梯度路径：(1) $\mathcal{L}_{\text{rec}} \to \theta_{\text{dec}} \to \phi$；(2) $\mathcal{L}_{\text{bwd}} \to \theta_{\text{dur}} \to \theta_{\text{pho}}$；(3) $\mathcal{L}_{\text{bwd}} \to \theta_{\text{bpp}} \to \phi$；(4) $\mathcal{L}_{\text{fwd}} \to \theta_{\text{bpp}} \to \theta_{\text{dur}} \to \theta_{\text{pho}}$；(5) $\mathcal{L}_{\text{fwd}} \to \phi$；(6) $\mathcal{L}_{\text{e2e}} \to \theta_{\text{dec}} \to \theta_{\text{bpp}} \to \theta_{\text{dur}} \to \theta_{\text{pho}}$。推理时只用 $\theta_{\text{pho}}, \theta_{\text{dur}}, \theta_{\text{bpp}}, \theta_{\text{dec}}$，后验编码器 $\phi$ 被丢弃。

### Table 1: 现有 TTS 系统与人类录音的 MOS/CMOS 对比

| System | MOS | Wilcoxon p (MOS) | CMOS | Wilcoxon p (CMOS) |
|--------|-----|------------------|------|-------------------|
| Human Recordings | 4.52 +/- 0.11 | - | 0 | - |
| [[FastSpeech 2]] + [[HiFi-GAN]] | 4.32 +/- 0.10 | 1.0e-05 | -0.30 | 5.1e-20 |
| [[Glow-TTS]] + [[HiFi-GAN]] | 4.33 +/- 0.10 | 1.3e-06 | -0.23 | 8.7e-17 |
| [[Grad-TTS]] + [[HiFi-GAN]] | 4.37 +/- 0.10 | 0.0127 | -0.23 | 1.2e-11 |
| [[VITS]] | 4.49 +/- 0.10 | 0.2429 | -0.19 | 2.9e-04 |

**说明**: 所有现有系统在 CMOS 上 p << 0.05，与录音有统计显著差异。[[VITS]] 在 MOS 上 p = 0.24 但 CMOS 仍显著，说明 MOS 对差异不敏感。

### Table 2: NaturalSpeech MOS 对比

| System | MOS | Wilcoxon p-value |
|--------|-----|------------------|
| Human Recordings | 4.58 +/- 0.13 | - |
| **NaturalSpeech** | **4.56 +/- 0.13** | **0.7145** |

**说明**: p = 0.71 >> 0.05，MOS 无统计显著差异。

### Table 3: NaturalSpeech CMOS 对比

| System | CMOS | Wilcoxon p-value |
|--------|------|------------------|
| Human Recordings | 0 | - |
| **NaturalSpeech** | **-0.01** | **0.6902** |

**说明**: CMOS 仅 -0.01，p = 0.69 >> 0.05，无统计显著差异。**首次在 LJSpeech 上实现。**

### Table 4: NaturalSpeech 与现有系统 MOS/CMOS 对比

| System | MOS | CMOS |
|--------|-----|------|
| [[FastSpeech 2]] + [[HiFi-GAN]] | 4.32 +/- 0.15 | -0.33 |
| [[Glow-TTS]] + [[HiFi-GAN]] | 4.34 +/- 0.13 | -0.26 |
| [[Grad-TTS]] + [[HiFi-GAN]] | 4.37 +/- 0.13 | -0.24 |
| [[VITS]] | 4.43 +/- 0.13 | -0.20 |
| **NaturalSpeech** | **4.56 +/- 0.13** | **0** |

**说明**: NaturalSpeech 在 MOS 和 CMOS 上均大幅领先所有基线。相比最强基线 [[VITS]]，MOS 提升 0.13，CMOS 提升 0.20。

### Table 5: 消融实验

| 配置 | CMOS | 说明 |
|------|------|------|
| NaturalSpeech (full) | 0 | 完整模型 |
| - Phoneme Pre-training | -0.09 | 随机初始化替代预训练 |
| - Differentiable Durator | -0.12 | 退化为 hard duration expansion + [[Monotonic Alignment Search|MAS]] |
| - Bidirectional Prior/Posterior | -0.09 | 仅用 $\mathcal{L}_{\text{bwd}}$，去掉 $\mathcal{L}_{\text{fwd}}$ |
| - Memory in VAE | -0.06 | 原始 VAE 重建 |

**关键发现**: Differentiable Durator 贡献最大（-0.12），说明消除 duration 的 train-inference mismatch 是最关键的改进。四个模块都有正向贡献，验证了系统性设计的必要性。

### Table 6: 推理速度对比 (RTF)

| System | [[RTF]] |
|--------|---------|
| [[FastSpeech 2]] + [[HiFi-GAN]] | 0.011 |
| [[VITS]] | 0.014 |
| **NaturalSpeech** | **0.013** |
| [[Glow-TTS]] + [[HiFi-GAN]] | 0.021 |
| [[Grad-TTS]] (10 steps) + [[HiFi-GAN]] | 0.082 |
| [[Grad-TTS]] (1000 steps) + [[HiFi-GAN]] | 4.120 |

**说明**: NaturalSpeech RTF = 0.013，与 [[FastSpeech 2]] + [[HiFi-GAN]] 和 [[VITS]] 相当，远快于 [[Grad-TTS]]。在 V100 GPU 上测量。

### Table 7: 各组件质量差距分析（Appendix A）

| Component | Setting | Upper Bound | CMOS |
|-----------|---------|-------------|------|
| Vocoder | GT Mel -> Vocoder | Human Recordings | -0.04 |
| Mel Decoder | GT Pitch/Duration -> Mel Decoder | GT Mel | -0.15 |
| Variance Adaptor | Predicted Pitch/Duration | GT Pitch/Duration | -0.14 |
| Phoneme Encoder | Phoneme Encoder | + Pre-training | -0.12 |

**关键发现**: Mel Decoder（-0.15）和 Variance Adaptor（-0.14）是最大瓶颈，对应 NaturalSpeech 的 bidirectional prior/posterior 和 differentiable durator 设计。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LJSpeech]] | ~24h, 13,100 条 | 单说话人英文有声书，22.05 kHz | 训练/测试 |
| news-crawl | 200M 句子 | 新闻文本语料 | Phoneme pre-training |

### 实现细节

- **Phoneme Encoder**: 6 层 [[Feed-Forward Transformer]]，hidden size 192
- **Duration Predictor**: 3 层 1D 卷积
- **Bidirectional Prior/Posterior**: 4 层 [[Affine Coupling Layer]]，shifting 用 4 层 [[WaveNet]] (dilation rate 1)
- **[[Posterior Encoder]]**: 16 层 [[WaveNet]]，kernel size 5，dilation rate 1
- **Waveform Decoder**: 4 个残差卷积块（沿用 [[HiFi-GAN]]），转置卷积上采样率 [8, 8, 2, 2]
- **优化器**: [[AdamW]]，$\beta_1 = 0.8$，$\beta_2 = 0.99$
- **学习率**: $2 \times 10^{-4}$，decay $\gamma = 0.999875$ per epoch
- **Batch Size**: 动态，每 GPU 8000 speech frames（hop 256）
- **训练轮数**: 15k epochs（前 1k warmup，最后 2k tuning）
- **硬件**: 8 x NVIDIA V100 32G（pre-training 用 8 x A100 80G）
- **推理参数**: 28.7M

### 训练阶段

1. **Phoneme Pre-training**: 200M sentences, 120k steps, 8 x A100
2. **Warmup (1-1k epoch)**: 用 [[Monotonic Alignment Search|MAS]] 提供 duration label；仅用 $\mathcal{L}_{\text{bwd}}$，不用 $\mathcal{L}_{\text{fwd}}$
3. **Main Training (1k-13k epoch)**: 切换到 differentiable durator 预测 duration；启用 $\mathcal{L}_{\text{fwd}}$
4. **Tuning (13k-15k epoch)**: 微调阶段

---

## 批判性思考

### 优点

1. **严谨的质量评估方法论**: 用统计假设检验（Wilcoxon signed rank test）而非简单看 MOS 绝对值，是方法论上的重要进步
2. **系统性的问题分析**: 先诊断质量差距来源（Table 7），再针对性设计四个模块，逻辑清晰
3. **端到端可微**: differentiable durator 消除了 duration 的 train-inference mismatch，这个思路启发了后续大量工作
4. **推理高效**: 28.7M 参数，RTF 0.013，与 [[FastSpeech 2]] 相当

### 局限性

1. **仅在单说话人 [[LJSpeech]] 上验证**: 24 小时单一说话人有声书是 TTS 最简单的场景，结论能否推广到多说话人、多语种、表达性语音等场景未知
2. **无零样本能力**: 不支持 zero-shot voice cloning，这在 [[VALL-E]] 之后已成为主流需求
3. **Pre-training 成本高**: 200M 句子的 phoneme pre-training 需要 8 x A100，对复现者不友好
4. **Soft-DTW 计算开销**: KL loss 中使用 soft-DTW 对齐，时间复杂度为 $O(m \times n)$，在长序列上较慢
5. **未开源**: 代码和模型均未公开，可复现性差

### 潜在改进方向

1. **扩展到多说话人/零样本**: 作者后续在 [[NaturalSpeech 2]] 和 [[NaturalSpeech 3]] 中实现
2. **简化训练流程**: 四个模块的联合训练较复杂，可探索更简洁的架构（如 [[F5-TTS]] 的纯 flow matching 方案）
3. **去掉 phoneme 依赖**: 字素直接建模可减少前端错误（后续 [[E2 TTS]]、[[F5-TTS]] 验证了这一方向）

### 可复现性评估

- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（附录详尽）
- [x] 数据集可获取（[[LJSpeech]] 公开）

---

## 关联笔记

### 基于

- [[VITS]]: 共享 VAE + flow + GAN 的基础框架，NaturalSpeech 在此基础上增加四个模块
- [[FastSpeech 2]]: duration predictor 的基础设计来源
- [[HiFi-GAN]]: waveform decoder 和判别器的设计直接沿用

### 后续工作

- [[NaturalSpeech 2]]: 扩展到多说话人 + latent diffusion + 离散 token
- [[NaturalSpeech 3]]: 扩展到 zero-shot + factorized codec + DiT

### 对比

- [[VITS]]: 最强基线，NaturalSpeech 比 VITS 高 +0.20 CMOS
- [[Glow-TTS]]: flow-based TTS，NaturalSpeech 借鉴了 flow 模块但做了双向改进
- [[Grad-TTS]]: diffusion-based TTS，质量不如 NaturalSpeech 且推理极慢

### 方法相关

- [[VAE]]: 核心生成框架
- [[Normalizing Flow]]: bidirectional prior/posterior 模块
- [[Duration Predictor]]: differentiable durator 的基础
- [[Monotonic Alignment Search]]: warmup 阶段提供 duration label
- [[Mel-Spectrogram]]: 后验编码器输入 + decoder 损失
- [[GAN]]: waveform decoder 对抗训练
- [[KL Divergence]]: 先验-后验匹配的核心损失
- [[ELBO]]: VAE 优化目标的理论基础
- [[DTW]]: soft-DTW 用于处理帧长不一致

### 数据相关

- [[LJSpeech]]: 唯一训练和评测数据集

---

## 速查卡片

> [!summary] NaturalSpeech
> - **核心**: 首个在 LJSpeech 上达到人类水平的 TTS（-0.01 CMOS, p=0.69）
> - **方法**: VAE + Flow + GAN + 4 模块（phoneme pretrain / differentiable durator / bidirectional prior-posterior / memory VAE）
> - **结果**: MOS 4.56 vs 录音 4.58，RTF 0.013，28.7M 参数
> - **代码**: 未开源

---

*笔记创建时间: 2026-05-25*
