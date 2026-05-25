---
title: "Mega-TTS 2: Boosting Prompting Mechanisms for Zero-Shot Speech Synthesis"
method_name: "MegaTTS2"
authors: [Ziyue Jiang, Jinglin Liu, Yi Ren, Jinzheng He, Zhenhui Ye, Shengpeng Ji, Qian Yang, Chen Zhang, Pengfei Wei, Chunfeng Wang, Xiang Yin, Zejun Ma, Zhou Zhao]
year: 2024
venue: ICLR 2024
tags: [zero-shot-tts, multi-reference-prompting, prosody-transfer, timbre-disentanglement, language-model-prosody]
image_source: online
arxiv_html: https://arxiv.org/html/2307.07218v4
created: 2026-05-25
---

# 论文笔记：Mega-TTS 2: Boosting Prompting Mechanisms for Zero-Shot Speech Synthesis

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Zhejiang University, ByteDance |
| 日期 | July 2023 (ICLR 2024) |
| 项目主页 | https://boostprompt.github.io/boostprompt/ |
| 对比基线 | [[VALL-E]], [[FastSpeech 2]] + GAN |
| 链接 | [arXiv](https://arxiv.org/abs/2307.07218) / Code: N/A |

---

## 一句话总结

> 通过解耦 prosody 与 timbre 并引入多句参考机制（MRTE + P-LLM），实现 zero-shot TTS 在 10s~5min prompt 下超越 fine-tuning 基线。

---

## 核心贡献

1. **Prosody-Timbre 解耦**: 通过 compressive acoustic autoencoder 将语音分解为 content、timbre、prosody 三个独立子空间，prosody 用紧凑 VQ codebook 编码
2. **多句参考提取机制**: 设计 [[MRTE]]（Multi-Reference Timbre Encoder）和 P-LLM（Prosody Latent Language Model），支持最长 300 秒多句 prompt
3. **Prosody Interpolation**: 提出韵律插值技术，在保持目标 timbre 的同时实现细粒度风格迁移

---

## 问题背景

### 要解决的问题
[[Zero-shot TTS]] 的 prompting 机制存在两个核心缺陷：(1) 训练时只用单句 prompt，推理时给更多参考音频反而性能下降（如 [[VALL-E]] 超过 20s prompt 后 WER 暴增至 8.77%）；(2) prosody 和 timbre 在 prompt 中耦合，无法独立控制风格迁移。

### 现有方法的局限
- [[VALL-E]]、[[NaturalSpeech 2]]、[[Voicebox]] 等 in-context learning 方法均以单句 prompt 训练，缺乏多句提取策略
- Fine-tuning 方法（如 AdaSpeech）虽能利用更多数据，但需要额外训练步骤且计算开销大
- 现有 prosody transfer 方法（CopyCat、Daft-Exprt）学到的是与说话人和文本高度耦合的 utterance-level 表示，无法实现跨说话人风格迁移

### 本文的动机
将语音分解为 content $z_c$、timbre $z_t$、prosody $z_{pd}$ 三个正交维度，分别设计提取机制，使得 timbre 和 prosody 的多句 prompt 可以独立扩展且互不干扰。

---

## 方法详解

### 模型架构

MegaTTS2 采用 **两阶段训练** 架构：
- **输入**: [[Phoneme]] 序列 + 多段参考 [[Mel-Spectrogram]]
- **Stage 1**: Compressive Acoustic Autoencoder（content encoder $E_c$ + VQ prosody encoder $E_p$ + [[MRTE]] $E_t$ + GAN mel decoder $D$）
- **Stage 2**: P-LLM 自回归韵律建模
- **输出**: Mel-spectrogram → [[HiFi-GAN]] V1 vocoder → waveform
- **总参数**: 367M（autoencoder） + P-LLM

### 核心模块

#### 模块1: 语音分解 (Decomposition)

**设计动机**: 利用 corpus partition 的 [[Mutual Information|互信息]] 假设实现无监督 prosody-timbre 解耦。

**核心假设**: 给定说话人 $S$ 的语音集合 $Y = \{y_1, \ldots, y_n\}$，将其分为目标 $y_t$ 和其余 $\tilde{y}$：

$$
I(y_t; \tilde{y}) = H(z_t) + H(g)
$$

即目标语音与同说话人其余语音之间的互信息 **仅包含** timbre $z_t$ 和全局风格 $g$，不包含 content 和细粒度 prosody。

**分解逻辑**:
- $E_t(\tilde{y})$ 从其余语音中提取：只能得到 timbre $z_t$ 和全局风格 $g$
- $E_c$ 仅接收 phoneme 序列：只传递 content $z_c$
- $E_p$ 带 [[Information Bottleneck|信息瓶颈]] $B(\cdot)$：已知 $z_c$ 和 $z_t$ 后，优先移除 content/timbre，仅保留细粒度 prosody $z_p$

#### 模块2: VQ Prosody Encoder

**设计动机**: 利用 [[Vector Quantization|VQ]] 的信息瓶颈特性压缩 prosody 表示，同时使其可被 LM 建模。

**具体实现**:
- 两层卷积栈对 mel-spectrogram 做时间维度压缩，压缩率 $r=8$
- VQ 层将连续特征量化为离散 prosody codes $\mathbf{u} = \{u_1, u_2, \ldots, u_n\}$
- Codebook 大小 1024，embedding 维度 256
- 使用 CVQ-VAE 动态初始化解决 codebook collapse
- 信息瓶颈 $B(\cdot)$ = 时间压缩 + VQ 量化

#### 模块3: Multi-Reference Timbre Encoder ([[MRTE]])

**设计动机**: 从多段参考语音中提取细粒度 timbre，通过 attention 机制实现任意长度 prompt 的聚合。

**具体实现**:
- 将多段参考 mel-spectrogram $\tilde{y}$ 沿时间轴拼接
- Mel encoder（5层卷积栈 × 2）下采样 $d=16$ 倍 → $z_t$
- Timbre-to-content attention：$z_c$ 作 query，$z_t$ 作 key/value
- 通过 [[Duration Predictor|length regulator]] 上采样至目标 mel 长度
- 训练时从 $\tilde{y}$ 中随机采样 2000 帧

#### 模块4: Prosody Latent Language Model (P-LLM)

**设计动机**: 捕获说话人的韵律模式（pitch/energy 分布、节奏习惯），使零样本 TTS 在多句 prompt 下持续提升。

**具体实现**:
- 从多段语音 $\{s_1, \ldots, s_n\}$ 提取 prosody codes 和 content
- 沿时间轴拼接：$z'_p = \text{Concat}(z_{p1}, \ldots, z_{pn})$，$z'_c = \text{Concat}(z_{c1}, \ldots, z_{cn})$
- 以 $z'_c$（经 duration 扩展 + max pooling 压缩 $r$ 倍）为条件，自回归预测 prosody codes
- 12 层 Transformer decoder，1024 hidden，16 heads
- Batch size = 1 以最大化每个 batch 的 prosody codes 数量
- 混合说话人时使用 speaker-level attention mask
- 每段语音加 start/end token 避免句间过渡问题
- Duration model 也用同架构 phoneme-level 自回归建模（MSE loss）

#### 模块5: Prosody Interpolation

**设计动机**: 实现可控的跨说话人风格迁移（如将悲伤语调转为开心语调同时保持目标音色）。

**具体实现**:
- 取辅助说话人 A 的韵律分布和目标说话人 B 的韵律分布
- 在 P-LLM 的输出概率层做加权插值（soft mixing），权重 $\gamma$ 控制风格迁移程度
- 比直接替换 prosody 更平滑、更细粒度

---

## 关键公式

### 公式1: [[Mutual Information|互信息]] 分解假设

$$
I(y_t; \tilde{y}) = H(z_t) + H(g)
$$

**含义**: 目标语音与同说话人其余语音的互信息仅由 timbre 熵 $H(z_t)$ 和全局风格熵 $H(g)$ 构成，不含 content 或细粒度 prosody。

**符号说明**:
- $y_t$: 目标 mel-spectrogram
- $\tilde{y}$: 同说话人的其余语音
- $z_t$: timbre 隐变量
- $g$: 全局风格信息（兼含 timbre + prosody 的全局成分）

### 公式2: Mel 重建

$$
y = D(z_c, z_{pd}, z_t, g)
$$

**含义**: Mel decoder 接收 content、prosody+duration、timbre、全局风格四路输入重建 mel-spectrogram。

**符号说明**:
- $z_c$: content hidden states（来自 phoneme encoder $E_c$）
- $z_{pd} = (z_p, z_d)$: prosody（pitch/energy）+ duration
- $z_t$: timbre hidden states（来自 [[MRTE]]）
- $g$: global style embedding
- $D$: GAN-based mel decoder

### 公式3: Stage 1 训练损失

$$
\mathcal{L} = \mathcal{L}_{\text{rec}} + \mathcal{L}_{\text{VQ}} + \mathcal{L}_{\text{Adv}}
$$

**含义**: 第一阶段联合训练重建、VQ 量化、对抗三项损失。

**符号说明**:
- $\mathcal{L}_{\text{rec}} = \|y_t - \hat{y}_t\|^2$: mel 重建 [[Mel Loss|L2 损失]]
- $\mathcal{L}_{\text{VQ}}$: [[Vector Quantization|VQ]] codebook commitment loss
- $\mathcal{L}_{\text{Adv}}$: LSGAN-style [[Adversarial Loss|对抗损失]]

### 公式4: P-LLM 自回归预测

$$
p(\mathbf{u}' \mid z'_c; \theta) = \prod_{l=0}^{L} p(\mathbf{u}'_l \mid \mathbf{u}'_{<l}, z'_c; \theta)
$$

**含义**: P-LLM 以 content 为条件，自回归地预测 prosody codes 序列。

**符号说明**:
- $\mathbf{u}'$: 多段语音拼接后的 prosody code 序列
- $z'_c$: 多段 content 拼接（经 duration 扩展 + $r$ 倍 max pooling 压缩）
- $\theta$: P-LLM 参数
- $L$: prosody code 序列总长度

### 公式5: Prosody Interpolation

$$
p(\hat{\mathbf{u}}) = \prod_{t=0}^{T} \Big( (1-\gamma) \cdot p(\hat{\mathbf{u}}_t \mid \hat{\mathbf{u}}_{<t}, \mathbf{u}_b, \text{Concat}(z_{cb}, \hat{z}_c); \theta) + \gamma \cdot p(\hat{\mathbf{u}}_t \mid \hat{\mathbf{u}}_{<t}, \mathbf{u}_a, \text{Concat}(z_{ca}, \hat{z}_c); \theta) \Big)
$$

**含义**: 在概率空间对目标说话人 B 和辅助说话人 A 的韵律输出做加权插值，实现细粒度风格迁移。

**符号说明**:
- $\mathbf{u}_a$: 辅助说话人（提供目标风格，如 happy）的 prosody codes
- $\mathbf{u}_b$: 目标说话人（提供原始风格）的 prosody codes
- $z_{ca}, z_{cb}$: 各自的 content
- $\hat{z}_c$: 目标待合成句子的 content
- $\gamma \in [0, 1]$: 插值权重，控制风格迁移强度

---

## 关键图表

### Figure 1: Overall Architecture / 系统总览

![Figure 1: Mega-TTS 2 总体架构](https://arxiv.org/html/2307.07218v4/x1.png)

**说明**: MegaTTS2 总体架构。左侧为 Stage 1 的 acoustic autoencoder：Content Encoder $E_c$ 处理 phoneme 序列，VQ Encoder $E_p$ 从目标 mel 提取量化 prosody codes，[[MRTE]] $E_t$ 从多段参考 mel 提取 timbre，三路信息经拼接（C）、上采样（U）、重复（R）操作后送入 GAN-based Mel Decoder。右侧为 Stage 2 的 P-LLM 自回归韵律建模。

### Figure 2: Prosody Interpolation / 韵律插值

![Figure 2: Prosody Interpolation](https://arxiv.org/html/2307.07218v4/x2.png)

**说明**: Prosody interpolation 示意图。取两个说话人的 P-LLM 输出概率分布，以权重 $\gamma$ 做 soft mixing，生成融合韵律的 prosody codes，配合目标 timbre 实现风格迁移。

### Figure 3: WER 和 SIM 对比 / Few-Shot Fine-Tuning Comparison

![Figure 3a: WER Comparison](https://arxiv.org/html/2307.07218v4/x3.png)

![Figure 3b: SIM Comparison](https://arxiv.org/html/2307.07218v4/x4.png)

**说明**: 随 prompt 数据量增加的 [[WER]] 和 SIM 变化。MegaTTS2（零样本）的 WER 始终低于 [[VALL-E]] 和 fine-tuning baseline，SIM 在 60s+ 时追平/超过 fine-tuning baseline。[[VALL-E]] 在 20s 以上 prompt 时性能急剧恶化。

### Figure 4: Pitch Distribution Visualization / 音高分布可视化

![Figure 4a: Before Prosody Transfer](https://arxiv.org/html/2307.07218v4/x5.png)

![Figure 4b: After Prosody Transfer](https://arxiv.org/html/2307.07218v4/x6.png)

**说明**: 韵律迁移前后的 pitch 分布对比。将 surprised 语调迁移到 LibriSpeech 说话人 '2300'。迁移后 pitch 分布的 $\sigma$（标准差）、$\gamma$（偏度）、$\kappa$（峰度）与目标风格更接近，证明插值机制有效。

### Figure 5: MRTE 和 VQ Encoder 结构细节

![Figure 5a: MRTE 结构](https://arxiv.org/html/2307.07218v4/x7.png)

![Figure 5b: VQ Encoder 结构](https://arxiv.org/html/2307.07218v4/x8.png)

**说明**: (a) [[MRTE]] 由两组 5 层卷积栈 + 下采样模块组成，下采样因子 $d=16$，输出作为 timbre-to-content attention 的 key/value。(b) VQ Encoder 使用 max pooling（stride 8）压缩时间维度后接 VQ 层，codebook 大小 1024。

### Table 1: Zero-Shot TTS 主实验结果 (LibriSpeech test-clean)

| Model | WER↓ | SIM↑ | DTW↓ | QMOS↑ | SMOS↑ | RTF | #Params | Method |
|---|---|---|---|---|---|---|---|---|
| GT | 1.98% | - | - | 4.43±0.09 | 4.26±0.11 | - | - | - |
| Baseline-10s | 4.74% | 0.895 | 35.12 | 3.97±0.08 | 3.76±0.13 | 0.089 | 467M | Fine-tune |
| Baseline-60s | 4.18% | 0.923 | 31.08 | 4.01±0.09 | 3.92±0.10 | - | - | Fine-tune |
| Baseline-300s | 3.11% | 0.934 | 29.80 | 4.08±0.07 | 4.03±0.08 | - | - | Fine-tune |
| [[VALL-E]]-3s | 5.83% | 0.885 | 36.59 | 3.89±0.10 | 3.70±0.11 | 1.471 | 478M | Zero-shot |
| [[VALL-E]]-10s | 5.54% | 0.893 | 34.10 | 3.92±0.11 | 3.74±0.10 | 1.775 | - | Zero-shot |
| [[VALL-E]]-20s | 8.77% | 0.805 | 43.02 | 3.41±0.12 | 3.25±0.14 | 2.104 | - | Zero-shot |
| **Ours-3s** | **2.46%** | 0.898 | 34.39 | 3.99±0.06 | 3.75±0.10 | 0.302 | 473M | Zero-shot |
| **Ours-10s** | **2.28%** | 0.905 | 32.30 | 4.05±0.08 | 3.79±0.09 | 0.334 | - | Zero-shot |
| **Ours-60s** | **2.24%** | 0.926 | 30.55 | 4.11±0.09 | 3.95±0.09 | 0.413 | - | Zero-shot |
| **Ours-300s** | **2.23%** | 0.932 | 29.95 | 4.12±0.10 | 4.01±0.09 | 0.923 | - | Zero-shot |

**说明**: MegaTTS2 在所有 prompt 长度下 WER 均低于 [[VALL-E]] 和 fine-tuning baseline。在 10s prompt 时自然度（QMOS 4.05）已超越 fine-tuning baseline（3.97），60s 时 SIM（0.926）追平 fine-tuning。VALL-E 超过 20s 后性能崩溃（WER 8.77%），MegaTTS2 则持续受益于更长 prompt。RTF 0.302~0.923，远低于 VALL-E 的 1.47~2.10。

### Table 2: Prosody Transfer 实验结果

| Model | WER | SIM-AB | DE↓ | σ | γ | κ | QMOS↑ | SMOS-AB |
|---|---|---|---|---|---|---|---|---|
| GT (ESD) | 4.38% | - | - | 74.91 | 0.707 | 0.024 | 4.18±0.08 | 4.22/2.39 |
| CopyCat | 5.29% | 0.843/0.740 | 37.2 | 59.74 | 0.889 | 0.859 | 3.72±0.11 | 3.53/3.19 |
| Daft-Exprt | 4.89% | 0.901/0.633 | 36.5 | 67.20 | 0.851 | 0.427 | 3.90±0.07 | 3.81/2.90 |
| **Ours** | **4.82%** | **0.920/0.513** | **32.8** | **72.62** | **0.664** | **0.197** | **3.92±0.08** | **3.87/2.64** |

**说明**: MegaTTS2 在 prosody transfer 任务中 pitch 分布的 $\sigma$（72.62 vs GT 74.91）、$\gamma$（0.664 vs GT 0.707）、$\kappa$（0.197 vs GT 0.024）均最接近 ground truth，同时保持最高 timbre 保真度（SIM 0.920）。SIM-AB 中的第二个数字（0.513）代表与辅助说话人 A 的相似度——越低说明 timbre 泄漏越少。

### Table 3: Timbre/Prosody Prompt 长度消融

| Setting | WER↓ | SIM↑ | DTW↓ | CMOS-Q | CMOS-S |
|---|---|---|---|---|---|
| Ours-10s | 2.28% | 0.905 | 32.30 | 0.000 | 0.000 |
| w/ 60s Timbre | 2.26% | 0.922 | 32.23 | +0.128 | +0.241 |
| w/ 300s Timbre | 2.25% | 0.930 | 32.08 | +0.162 | +0.353 |
| w/ 60s Prosody | 2.27% | 0.906 | 30.74 | +0.014 | +0.154 |
| w/ 300s Prosody | 2.24% | 0.908 | 30.25 | +0.017 | +0.196 |

**关键发现**: 增加 timbre prompt 长度 → SIM 显著提升（0.905→0.930）、DTW 不变；增加 prosody prompt 长度 → DTW 显著降低（32.30→30.25）、SIM 不变。证明两种机制独立生效，互不干扰。

### Table 4: VQ Encoder / MRTE 消融

| Setting | WER↓ | SIM↑ | DTW↓ | CMOS-Q | CMOS-S |
|---|---|---|---|---|---|
| Ours-10s | 2.28% | 0.905 | 32.30 | 0.000 | 0.000 |
| Ours-300s | 2.23% | 0.932 | 29.95 | +0.144 | +0.493 |
| w/o [[MRTE]] | 5.57% | 0.841 | 36.07 | -0.458 | -0.619 |
| w/ [[VAE]] | 2.31% | 0.896 | 35.18 | -0.038 | -0.127 |
| w/ [[VAE]]+LDM | 2.25% | 0.907 | 32.98 | +0.007 | -0.005 |

**关键发现**: 移除 MRTE 导致灾难性下降（WER 5.57%、SIM 0.841），因为 timbre 信息被 VQ codebook 吸收导致 P-LLM 负担过重。将 VQ 替换为 VAE+LDM 可恢复到 Ours-10s 水平，但仍劣于多句 prompt 方案，说明 VQ 的离散化 + P-LLM 是更优的 prosody 建模范式。

### Table 5: 详细超参数配置

| 组件 | 参数 | 值 |
|---|---|---|
| **VQ Prosody Encoder** | Encoder Layers / Hidden / Kernel | 3×2 / 384 / 5 |
| | VQ Embedding Size / Channel | 1024 / 256 |
| **Content Encoder** | Layers / Hidden / Kernel / Filter | 8 / 512 / 5 / 1024 |
| **[[MRTE]]** | Layers / Q-Hidden / K-Hidden / Stride | 5×2 / 512 / 256 / 16 |
| **Mel Decoder** | Layers / Hidden / Kernel | 4 / 512 / 5 |
| **P-LLM** | Layers / Hidden / Heads | 12 / 1024 / 16 |
| **Multi-Length Disc.** | #Disc / Window / Conv Layers / Hidden | 3 / {32,64,128} / 3 / 192 |

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LibriLight\|Libri-Light]] | 60K hours | LibriVox 有声书，16kHz，DNN-HMM ASR 转写 | 训练 |
| [[LibriSpeech]] test-clean | 20 speakers × 400s | 300s prompt + 100s target | 测试 (zero-shot) |
| [[ESD]] | 情感数据集 | surprised/happy/sad 等情感 | 测试 (prosody transfer) |

### 实现细节

- **Stage 1**: 4× NVIDIA A100, batch size 48/GPU, 600K steps
- **Stage 2**: 8× NVIDIA A100, batch size 4000 tokens/GPU, 300K steps
- **优化器**: Adam ($\beta_1=0.9$, $\beta_2=0.999$, $\epsilon=10^{-9}$)
- **Vocoder**: 预训练 [[HiFi-GAN]] V1
- **P-LLM Sampling**: Top-k ($k=10$)
- **ASR for WER**: [[HuBERT]]-Large fine-tuned on [[LibriSpeech]] 960h (WER: 1.9% test-clean)
- **Speaker SIM**: [[WavLM]] fine-tuned for speaker verification (EER: 0.84% Vox1-O)
- **Alignment**: [[MFA|Montreal Forced Aligner]]
- **评测**: 50 samples/dataset, ≥20 raters/sample via Amazon Mechanical Turk

### 可视化结果

- Prompt 从 3s→300s 时，MegaTTS2 的 WER 稳定下降（2.46%→2.23%）、SIM 持续上升（0.898→0.932）
- [[VALL-E]] 在 20s prompt 时出现性能崩溃（WER 从 5.54% 跳到 8.77%），说明单句训练范式无法支持长 prompt
- Prosody transfer 后的 pitch 分布形状（$\sigma$, $\gamma$, $\kappa$）与目标 GT 高度吻合

---

## 批判性思考

### 优点
1. **解耦思路优雅**: corpus partition + information bottleneck 实现了无监督的 prosody-timbre 分离，设计动机清晰
2. **实验说服力强**: 消融实验（Table 3）定量证明 timbre/prosody 两路 prompt 独立生效（SIM vs DTW），不是在做 claim 而是有数据支撑
3. **对比公平充分**: 不仅对比零样本方法（VALL-E），还对比 fine-tuning baseline，证明零样本可超越传统范式
4. **工程价值高**: RTF 0.3~0.9 远优于 VALL-E 的 1.5~2.1，有实用潜力

### 局限性
1. **假设约束强**: Eq.1 的互信息假设要求同说话人的 prosody 在不同句子间差异大于 timbre，对情感一致的说话人（如情感数据集）可能不成立
2. **代码未开源**: 无法复现和验证
3. **评测覆盖有限**: 仅在 [[LibriSpeech]]（有声书英文）上做 zero-shot 测试，缺少中文、多语种、嘈杂环境评测
4. **与后续 SOTA 差距**: 论文发表后 [[CosyVoice]]、[[F5-TTS]]、[[Seed-TTS]] 等已超越此方法，特别是无需 phoneme 对齐的方法更简洁
5. **Vocoder 限制**: 依赖预训练 [[HiFi-GAN]] V1（较旧），采样率仅 16kHz，音质上限受限

### 潜在改进方向
1. 用 [[Conditional Flow Matching|CFM]] 替代 VQ+AR 做 prosody 建模，避免自回归的累积误差
2. 去掉 [[Phoneme]] 依赖和 [[Forced Alignment|forced alignment]]，像 [[F5-TTS]] 那样直接从文本+参考音频学习
3. 将 [[MRTE]] 机制迁移到端到端 codec-based TTS（如 [[VALL-E 2]]）

### 可复现性评估
- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（超参数、硬件、训练步数均详细报告）
- [x] 数据集可获取（Libri-Light 公开）

---

## 关联笔记

### 基于
- [[MegaTTS]]: Mega-TTS 第一代，本文的直接前置工作
- [[FastSpeech 2]]: Fine-tuning baseline 的 TTS backbone

### 对比
- [[VALL-E]]: 主要零样本对比基线，离散 codec token AR 建模，单句 prompt 训练
- [[NaturalSpeech 2]]: 同期 SOTA，latent diffusion 范式
- [[Voicebox]]: Meta 的 Flow Matching mask-and-infill 方法

### 方法相关
- [[MRTE]]: 本文核心模块，Multi-Reference Timbre Encoder
- [[Vector Quantization]]: VQ 信息瓶颈是 prosody 编码的核心
- [[Information Bottleneck]]: 解耦策略的理论基础
- [[HiFi-GAN]]: 使用的 vocoder
- [[Forced Alignment]]: 使用 [[MFA]] 获取 phoneme-mel 对齐
- [[Prosody Transfer]]: 韵律迁移任务

### 硬件/数据相关
- [[LibriLight]]: 60K hours 训练数据
- [[LibriSpeech]]: 测试集来源
- [[ESD]]: 情感韵律迁移测试集

---

## 速查卡片

> [!summary] Mega-TTS 2
> - **核心**: 通过 prosody-timbre 解耦 + 多句参考机制实现 zero-shot TTS 超越 fine-tuning
> - **方法**: VQ prosody encoder + MRTE + P-LLM 自回归韵律建模 + prosody interpolation
> - **结果**: 10s prompt 下 WER 2.28%（vs VALL-E 5.54%），300s prompt 下 SIM 0.932 追平 fine-tuning baseline
> - **代码**: 未开源

---

*笔记创建时间: 2026-05-25*
