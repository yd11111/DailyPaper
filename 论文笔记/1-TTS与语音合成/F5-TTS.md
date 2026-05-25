---
title: "F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching"
method_name: "F5-TTS"
authors: [Yushen Chen, Zhikang Niu, Ziyang Ma, Keqi Deng, Chunhui Wang, Jian Zhao, Kai Yu, Xie Chen]
year: 2024
venue: arXiv
tags: [zero-shot-tts, flow-matching, dit, non-autoregressive, filler-token, sway-sampling, multilingual]
image_source: online
arxiv_html: https://arxiv.org/html/2410.06885
created: 2026-05-25
---

# 论文笔记：F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Shanghai Jiao Tong University (上海交通大学) |
| 日期 | October 2024 (v1); May 2025 (v3) |
| 项目主页 | [SWivid.github.io/F5-TTS](https://SWivid.github.io/F5-TTS/) |
| 对比基线 | [[E2 TTS]] / [[CosyVoice]] / [[VALL-E]] / [[Voicebox]] / [[NaturalSpeech 3]] / [[MaskGCT]] / [[FireRedTTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2410.06885) / [Code](https://github.com/SWivid/F5-TTS) |

---

## 一句话总结

> 基于 [[Flow Matching]] + [[DiT]] 的全 NAR 零样本 TTS，去掉 phoneme/duration/对齐模块，用 filler token 填充 + ConvNeXt 文本精炼 + Sway Sampling 推理策略，RTF 0.15，100K 小时多语种数据训练。

---

## 核心贡献

1. **去除对齐依赖的 E2 TTS 范式改进**: 沿用 [[E2 TTS]] 的 filler token 填充思路，但用 [[ConvNeXt]] V2 对字符序列做预处理，解决 E2 TTS 的语义-声学特征"深度纠缠"导致的收敛慢和鲁棒性差问题
2. **Sway Sampling 推理策略**: 提出在推理时对 ODE 求解器的时间步采样做非均匀重分配（偏向早期步），在不重新训练的情况下显著提升自然度、可懂度和说话人相似度，且可直接迁移至其他 CFM 模型
3. **开源完整系统**: 335.8M 参数，在 Emilia ~95K 小时多语种数据上训练，RTF 0.15（16 NFE），开源代码和 checkpoint，成为零样本 TTS 领域的标杆基线

---

## 问题背景

### 要解决的问题
在 NAR TTS 中如何**不依赖显式 phoneme alignment 和 duration predictor** 实现高质量零样本语音合成，同时保持高鲁棒性和快推理速度。

### 现有方法的局限
- **AR 模型**（[[VALL-E]]、MELLE）：推理延迟高，受 speech tokenizer 质量制约，存在 exposure bias
- **NAR 模型需对齐**：[[NaturalSpeech 3]]、[[Voicebox]] 依赖 frame-wise phoneme alignment；Matcha-TTS 用 MAS；但刚性对齐会**限制自然度上限**
- **[[E2 TTS]]**：虽然去掉了 phoneme 和 duration predictor，但直接拼接填充字符和语音序列，导致"有效信息长度差距大"，语义与声学特征深度纠缠，收敛慢（E2 TTS 在 800K 步仍有 7% 样本彻底失败，WER>50%）

### 本文的动机
- **ConvNeXt 预处理**可以在拼接前独立精炼文本表示，避免语义-声学特征的长度差距直接暴露给 Transformer
- **Sway Sampling**利用了 CFM 模型"早期步构建语音轮廓、晚期步细化细节"的特性，让 ODE 求解器在早期步获得更多计算预算

---

## 方法详解

### 模型架构

[[F5-TTS]] 采用 **mask-and-infill 范式** 的 NAR 架构：

- **输入**: 字符序列（用 filler token $\langle F \rangle$ 填充到与 Mel 等长） + 参考音频 Mel + 噪声化目标 Mel
- **文本精炼**: 4 层 [[ConvNeXt]] V2（512 dim, 1024 FFN）处理填充后的字符序列 $z$
- **Backbone**: 22 层 latent [[DiT]]（adaLN-zero），1024 embed dim, 2048 FFN dim, 16 heads
- **位置编码**: [[RoPE]] 用于自注意力（替代 ALiBi）；卷积位置编码用于拼接输入；正弦位置编码加在字符序列 $z$ 上
- **时间步条件**: flow step $t$ 通过正弦嵌入 + adaLN-zero 注入（不拼接到输入序列）
- **声码器**: [[Vocos]]（预训练，Mel→Waveform）
- **总参数**: 335.8M（DiT 主体）+ ConvNeXt 部分
- **不使用**: phoneme、[[Duration Predictor]]、[[Forced Alignment]]、文本编码器、U-Net skip connection

### 核心模块

#### 模块1: Filler Token 填充 + ConvNeXt V2 文本精炼

**设计动机**: [[E2 TTS]] 直接将短字符序列（$M$ 个字符）和长 Mel 序列（$N$ 帧, $N \gg M$）拼接，大量 filler token 稀释了文本信息。ConvNeXt 的局部卷积可以在拼接前让字符信息"扩散"到邻近 filler token 位置，**缩小语义-声学的有效信息长度差距**。

**具体实现**:
- 字符序列填充: $z = (c_1, c_2, \ldots, c_M, \underbrace{\langle F\rangle, \ldots, \langle F\rangle}_{N-M})$
- 加绝对正弦位置编码后送入 4 层 ConvNeXt V2
- 精炼后的 $z'$ 再与 Mel 拼接送入 DiT
- 中文通过 jieba 分词 + pypinyin 转全拼音，词表共 2546 个符号

#### 模块2: DiT with adaLN-zero

**设计动机**: 用 [[DiT]] 的 adaLN-zero 机制优雅地注入 flow step $t$ 条件，避免像 E2 TTS 那样将 $t$ 作为额外 token 拼到输入序列尾部。

**具体实现**:
- 22 层 Transformer block，每层 adaLN-zero 根据 $t$ 的嵌入动态调制 LayerNorm 参数
- 输入是 ConvNeXt 精炼后的文本表示 $z'$ 和 masked/noised Mel 的拼接
- 使用 [[RoPE]] 替代 ALiBi，卷积位置编码替代标准绝对位置编码
- 训练时随机 mask 70%~100% 的 Mel 帧

#### 模块3: Sway Sampling（推理时）

**设计动机**: [[Flow Matching|CFM]] 模型在早期步（$t \to 0$）从纯噪声"勾勒语音轮廓"，晚期步精炼细节。让 ODE 求解器在早期步分配更多计算（更小的步长），可以获得"更精确的初始积分起点"。

**具体实现**:
- 定义非线性映射 $f_{sway}$ 将均匀采样 $u \sim \mathcal{U}(0,1)$ 映射为非均匀时间步 $t$
- 系数 $s < 0$ 时偏向早期步，$s > 0$ 偏向晚期步，$s = 0$ 退化为均匀
- **无需重新训练**，可直接应用于任何已训练的 CFM 模型
- 默认 $s = -1$，搭配 Euler ODE solver

---

## 关键公式

### 公式1: [[Flow Matching|Conditional Flow Matching]] 训练损失

$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t, q(x_1), p(x_0)} \left\| v_\theta\big((1-t)x_0 + tx_1\big) - (x_1 - x_0) \right\|^2
$$

**含义**: 使用 Optimal Transport 路径 $\psi_t(x_0) = (1-t)x_0 + tx_1$ 的 CFM 损失，训练速度场 $v_\theta$ 预测从噪声 $x_0$ 到目标 Mel $x_1$ 的方向。

**符号说明**:
- $t \sim \mathcal{U}(0, 1)$: 流步，从 0（纯噪声）到 1（目标 Mel）
- $x_0 \sim p(x_0) = \mathcal{N}(0, I)$: 采样噪声
- $x_1 \sim q(x_1)$: 目标 Mel spectrogram
- $v_\theta$: DiT 参数化的速度场网络

### 公式2: [[Classifier-Free Guidance|CFG]] 推理速度场

$$
v_{t,\text{CFG}} = v_t(\psi_t(x_0), c) + \alpha \big( v_t(\psi_t(x_0), c) - v_t(\psi_t(x_0)) \big)
$$

**含义**: 推理时用 CFG 增强条件信号的引导强度。代价是推理时间翻倍（需要计算有条件和无条件两次前向）。

**符号说明**:
- $c$: 条件信息（参考音频 + 文本）
- $\alpha$: CFG 强度系数，默认 $\alpha = 2$
- $v_t(\psi_t(x_0))$: 无条件速度场

### 公式3: Filler Token 填充

$$
z = \big(c_1, c_2, \ldots, c_M, \underbrace{\langle F\rangle, \ldots, \langle F\rangle}_{N-M}\big)
$$

**含义**: 将 $M$ 个字符的文本序列用 filler token $\langle F \rangle$ 填充到 $N$ 帧（与 Mel 等长），形成等长拼接。

**符号说明**:
- $c_i$: 第 $i$ 个字符 token
- $M$: 文本序列长度
- $N$: Mel spectrogram 帧数，$N \gg M$
- $\langle F \rangle$: 特殊 filler token

### 公式4: Sway Sampling 映射函数

$$
f_{sway}(u; s) = u + s \cdot \left( \cos\!\left(\frac{\pi}{2} u\right) - 1 + u \right)
$$

**含义**: 将均匀采样 $u$ 映射为非均匀时间步 $t$。当 $s < 0$ 时，更多推理步分配给早期（$t$ 小的区间），让 ODE 求解器在"勾勒轮廓"阶段更精确。

**符号说明**:
- $u \sim \mathcal{U}(0, 1)$: 均匀随机采样
- $s \in [-1, \frac{2}{\pi - 2}]$: Sway 系数，保证映射单调
- $s = 0$: 退化为均匀采样
- $s = -1$: 默认值，显著偏向早期步

### 公式5: 条件生成（推理）

$$
v_t\big((1-t)x_0 + tx_1 \mid x_{\text{ref}}, z_{\text{ref} \cdot \text{gen}}\big)
$$

**含义**: 推理时的条件速度场，给定参考音频 Mel $x_{\text{ref}}$ 和拼接的参考+目标文本 $z_{\text{ref} \cdot \text{gen}}$，从噪声 $x_0$ 通过 ODE 积分生成目标 Mel。

**符号说明**:
- $x_{\text{ref}}$: 参考音频的 Mel spectrogram
- $z_{\text{ref} \cdot \text{gen}}$: 参考文本 + 目标文本的字符序列（filler token 填充后由 ConvNeXt 精炼）

### 公式6: 时长估计

推理时目标语音的帧数 $N_{\text{gen}}$ 通过参考音频的字符-帧比例估计：

$$
N_{\text{gen}} = N_{\text{ref}} \cdot \frac{|y_{\text{gen}}|}{|y_{\text{ref}}|}
$$

**含义**: 假设说话速度与参考一致，按字符数比例估计目标语音的长度。

**符号说明**:
- $N_{\text{ref}}$: 参考音频帧数
- $|y_{\text{gen}}|, |y_{\text{ref}}|$: 目标/参考文本的字符数

---

## 关键图表

### Figure 1: Overview / 系统概览

![Figure 1](https://arxiv.org/html/2410.06885v3/x1.png)

**说明**: F5-TTS 的训练（左）和推理（右）流程。训练阶段：文本转字符后用 filler token 填充到与 Mel 等长，经 ConvNeXt V2 精炼后与 masked+noised Mel 拼接，送入 DiT 做 [[Flow Matching|CFM]] 损失训练。推理阶段：给定参考音频和目标文本，通过 Sway Sampling 调制的 ODE 求解器从噪声生成目标 Mel，最后经 [[Vocos]] 声码器转波形。

### Figure 2: Small Model Ablation / 小模型消融

![Figure 2](https://arxiv.org/html/2410.06885v3/x2.png)

**说明**: 155M 参数小模型在 WenetSpeech4TTS Premium（945 小时普通话）上训练，在 Seed-TTS test-zh 上的 WER 和 SIM 随训练步数变化。关键发现：(1) F5-TTS 在 800K 步时 WER=4.17 远优于 E2 TTS 的 WER=9.63；(2) E2 TTS 有 7% 样本彻底失败（WER>50%）；(3) 去掉 ConvNeXt 的纯 adaLN DiT 变体无法学会对齐；(4) MMDiT "学得快崩得也快"，导致严重重复。

### Figure 3: Sway Sampling / Sway 采样分析

![Figure 3](https://arxiv.org/html/2410.06885v3/x3.png)

**说明**: 不同 Sway 系数 $s$ 下的概率密度函数及对应的 Seed-TTS test-zh 性能。$s$ 越负，越多推理步分配给早期流步，WER 和 SIM 持续改善。右侧"leak and override"实验证明早期步对确定语音轮廓至关重要。

### Table 1: LibriSpeech-PC test-clean 主要结果

| Model | #Param | #Data | WER(%)↓ | SIM-o↑ | RTF↓ |
|-------|--------|-------|---------|--------|------|
| Ground Truth | - | - | 2.23 | 0.69 | - |
| Vocoder Resynthesized | - | - | 2.32 | 0.66 | - |
| VALL-E 2 | - | 50K EN | 2.44 | 0.643 | 0.732 |
| MELLE | - | 50K EN | 2.10 | 0.625 | 0.549 |
| [[Voicebox]] | 330M | 60K EN | 1.9 | 0.662 | 0.64 |
| [[NaturalSpeech 3]] | 500M | 60K EN | 1.94 | 0.67 | 0.296 |
| DiTTo-TTS | 740M | 55K EN | 2.56 | 0.627 | 0.162 |
| [[MaskGCT]] | 1048M | 100K Multi | 2.634 | 0.687 | - |
| [[CosyVoice]] | ~300M | 170K Multi | 3.59 | 0.66 | 0.92 |
| [[FireRedTTS]] | ~580M | 248K Multi | 2.69 | 0.47 | 0.84 |
| [[E2 TTS]] (32 NFE) | 333M | 100K Multi | 2.95 | 0.69 | 0.68 |
| **F5-TTS (16 NFE)** | **336M** | **100K Multi** | **2.53** | **0.66** | **0.15** |
| **F5-TTS (32 NFE)** | **336M** | **100K Multi** | **2.42** | **0.66** | **0.31** |

**说明**: F5-TTS 以 336M 参数在 100K 小时多语种数据上训练，WER 2.42%（32 NFE）接近 GT 的 2.23%，RTF 0.15（16 NFE）是所有对比模型中最快的，远超 [[Voicebox]]（0.64）和 [[CosyVoice]]（0.92）。SIM-o 0.66 与 vocoder resynthesized 持平。

### Table 2: Seed-TTS test-en 结果

| Model | WER(%)↓ | SIM-o↑ | CMOS↑ | SMOS↑ |
|-------|---------|--------|-------|-------|
| Ground Truth | 2.06 | 0.73 | 0.00 | 3.91 |
| [[CosyVoice]] | 3.39 | 0.64 | 0.02 | 3.64 |
| [[FireRedTTS]] | 3.82 | 0.46 | -1.46 | 2.94 |
| [[MaskGCT]] | 2.623 | 0.717 | - | - |
| Seed-TTS_DiT | **1.733** | **0.790** | - | - |
| [[E2 TTS]] (32 NFE) | 2.19 | 0.71 | 0.06 | 3.81 |
| F5-TTS (16 NFE) | 1.89 | 0.67 | 0.16 | 3.79 |
| **F5-TTS (32 NFE)** | **1.83** | **0.67** | **0.31** | **3.89** |

**说明**: F5-TTS 32 NFE 在 CMOS（+0.31）和 SMOS（3.89）上接近甚至超越 GT，WER 1.83% 优于 GT 的 2.06%。Seed-TTS_DiT 在客观指标上领先但未公开模型。

### Table 2: Seed-TTS test-zh 结果

| Model | WER(%)↓ | SIM-o↑ | CMOS↑ | SMOS↑ |
|-------|---------|--------|-------|-------|
| Ground Truth | 1.26 | 0.76 | 0.00 | 3.72 |
| [[CosyVoice]] | 3.10 | 0.75 | -0.06 | 3.54 |
| [[FireRedTTS]] | 1.51 | 0.63 | -0.49 | 3.28 |
| [[MaskGCT]] | 2.273 | 0.774 | - | - |
| Seed-TTS_DiT | **1.178** | **0.809** | - | - |
| [[E2 TTS]] (32 NFE) | 1.97 | 0.73 | -0.04 | 3.44 |
| F5-TTS (16 NFE) | 1.74 | 0.75 | 0.02 | 3.72 |
| **F5-TTS (32 NFE)** | **1.56** | **0.76** | **0.21** | **3.83** |

**说明**: 中文场景下 F5-TTS 优势更明显，CMOS +0.21 超过 GT，SMOS 3.83 > GT 3.72。SIM-o 0.76 与 GT 持平。

### Table 3: 小模型配置

| Model | Transformer (dim,layers,heads,FFN_mult) | ConvNeXt (dim,layers,FFN_mult) | #Param | GFLOPs |
|-------|----------------------------------------|--------------------------------|--------|--------|
| F5-TTS | 768, 18, 12, 2 | 512, 4, 2 | 158M | 173 |
| F5-TTS−Conv2Text | 768, 18, 12, 2 | - | 153M | 164 |
| F5-TTS++Conv2Audio | 768, 16, 12, 2 | 512, 4, 2 | 163M | 181 |
| F5-TTS++LongSkip | 768, 18, 12, 2 | 512, 4, 2 | 159M | 175 |
| [[E2 TTS]] | 768, 20, 12, 4 | - | 157M | 293 |
| E2 TTS++Conv2Text | 768, 20, 12, 4 | 512, 4, 2 | 161M | 301 |
| MMDiT | 512, 16, 16, 2 | - | 151M | 104 |

**说明**: 参数量相近的情况下，E2 TTS 的 GFLOPs（293）几乎是 F5-TTS（173）的 1.7 倍，因为 E2 TTS 用了更多层和更宽的 FFN。

### Table 4: 输入条件消融

| Model | Common WER↓ | Common SIM↑ | GT Dur WER↓ | GT Dur SIM↑ | Text-Only WER↓ | Text-Only SIM↑ |
|-------|-------------|-------------|-------------|-------------|----------------|----------------|
| F5-TTS | **4.17** | **0.54** | **3.87** | **0.54** | 3.22 | 0.21 |
| F5-TTS++Conv2Audio | 5.78 | 0.55 | 5.28 | 0.55 | 3.78 | 0.21 |
| F5-TTS++LongSkip | 5.17 | 0.53 | 5.03 | 0.53 | 3.35 | 0.21 |
| [[E2 TTS]] | 9.63 | 0.53 | 9.48 | 0.53 | 3.48 | 0.21 |
| E2 TTS++Conv2Text | 18.10 | 0.49 | 17.94 | 0.49 | **3.06** | 0.21 |

**说明**: E2 TTS 去掉音频 prompt（Text-Only）后 WER 从 9.63 降到 3.48，反证音频 prompt 的加入导致了语义-声学纠缠。E2 TTS 加 ConvNeXt（++Conv2Text）反而更差（18.10），说明 U-Net 架构与 ConvNeXt 不兼容。

### Table 5: Sway Sampling 在 Base 模型上的效果 (s=-1)

#### LibriSpeech-PC test-clean

| Model | WER(%)↓ | SIM-o↑ | UTMOS↑ | RTF↓ |
|-------|---------|--------|--------|------|
| [[E2 TTS]] (32 NFE w/ SS) | 2.84 | 0.72 | 3.70 | 0.68 |
| [[E2 TTS]] (32 NFE w/o SS) | 2.95 | 0.69 | 3.56 | 0.68 |
| F5-TTS (32 NFE w/ SS) | **2.41** | 0.66 | **3.89** | 0.53 |
| F5-TTS (32 NFE w/o SS) | 2.84 | 0.62 | 3.70 | 0.53 |

#### Seed-TTS test-en

| Model | WER(%)↓ | SIM-o↑ | UTMOS↑ |
|-------|---------|--------|--------|
| [[E2 TTS]] (32 NFE w/ SS) | 1.98 | 0.73 | 3.57 |
| [[E2 TTS]] (32 NFE w/o SS) | 2.19 | 0.71 | 3.33 |
| F5-TTS (32 NFE w/ SS) | **1.87** | 0.66 | **3.72** |
| F5-TTS (32 NFE w/o SS) | 1.93 | 0.63 | 3.51 |

#### Seed-TTS test-zh

| Model | WER(%)↓ | SIM-o↑ | UTMOS↑ |
|-------|---------|--------|--------|
| [[E2 TTS]] (32 NFE w/ SS) | 1.77 | 0.78 | 2.87 |
| [[E2 TTS]] (32 NFE w/o SS) | 1.97 | 0.73 | 2.49 |
| F5-TTS (32 NFE w/ SS) | **1.58** | 0.75 | **2.91** |
| F5-TTS (32 NFE w/o SS) | 1.93 | 0.69 | 2.58 |

**关键发现**: Sway Sampling 在**所有测试集和两个模型**上一致提升 WER、SIM 和 UTMOS。F5-TTS 上 WER 改善幅度为 0.06~0.43 个百分点，SIM 改善 0.03~0.06，UTMOS 改善 0.19~0.33。E2 TTS 同样受益。

### Table 6: ODE 求解器对比

| Config | LP-PC WER↓ | LP-PC SIM↑ | test-en WER↓ | test-zh WER↓ | RTF↓ |
|--------|------------|------------|--------------|--------------|------|
| s=-1, 16 NFE Euler | 2.53 | 0.66 | 1.89 | 1.74 | **0.15** |
| s=-1, 32 NFE Euler | 2.42 | 0.66 | 1.83 | 1.56 | 0.31 |
| s=-1, 16 NFE midpoint | 2.43 | 0.66 | 1.88 | 1.61 | 0.26 |
| s=-1, 32 NFE midpoint | 2.41 | 0.66 | 1.87 | 1.58 | 0.53 |
| s=-1, 16 NFE Heun-3 | 2.39 | 0.65 | 1.80 | 1.55 | 0.44 |

**说明**: Euler 求解器在速度和配合 Sway Sampling 的效果上最优。16 NFE Euler 的 RTF=0.15 是所有配置中最快的，且性能仅略低于 32 NFE。

### Table 7: ELLA-V Hard Sentences 鲁棒性

| Model | WER(%)↓ | Sub.(%)↓ | Del.(%)↓ | Ins.(%)↓ |
|-------|---------|----------|----------|----------|
| StyleTTS 2 | 4.83 | 2.17 | 2.03 | 0.61 |
| [[CosyVoice]] | 8.30 | 3.47 | 2.74 | 1.93 |
| E1 TTS_DMD | 4.29 | 1.89 | 1.62 | 0.74 |
| [[E2 TTS]] | 8.58 | 3.70 | 4.82 | 0.06 |
| **F5-TTS** | **4.40** | **1.81** | **2.40** | **0.18** |

**说明**: E2 TTS 的高删除率（4.82%）说明严重的跳词问题；F5-TTS 的低插入率（0.18%）说明没有无限重复问题。F5-TTS 整体鲁棒性接近 StyleTTS 2 和 E1 TTS_DMD。

### Table 8: 声码器对比（BigVGAN vs Vocos）

| Config | LP-PC WER↓ | LP-PC SIM↑ | test-en WER↓ | test-zh WER↓ |
|--------|------------|------------|--------------|--------------|
| 32 NFE [[Vocos]] | 2.42 | 0.66 | 1.83 | 1.56 |
| 32 NFE [[BigVGAN]] | **2.11** | **0.67** | **1.62** | **1.53** |
| 16 NFE [[Vocos]] | 2.53 | 0.66 | 1.89 | 1.74 |
| 16 NFE [[BigVGAN]] | **2.21** | **0.67** | **1.65** | **1.64** |

**说明**: [[BigVGAN]] 声码器在所有配置上均优于 [[Vocos]]，WER 降低 0.1~0.31 个百分点。

### Table 9: 不同数据规模训练

#### LibriTTS 585 小时 → LibriSpeech-PC test-clean

| Update | WER(%)↓ | SIM-o↑ | UTMOS↑ |
|--------|---------|--------|--------|
| 100K | 29.5 | 0.53 | 3.78 |
| 200K | 4.58 | 0.59 | 4.07 |
| 300K | 2.71 | 0.60 | 4.11 |
| 400K | 2.44 | 0.60 | 4.11 |
| 500K | **2.20** | **0.60** | 4.10 |

#### LJSpeech 24 小时 → 集内测试

| Update | WER(%)↓ | SIM-o↑ | UTMOS↑ |
|--------|---------|--------|--------|
| 100K | 5.64 | 0.72 | 4.17 |
| 200K | **2.93** | **0.72** | **4.18** |
| 300K | 3.26 | 0.71 | 4.12 |
| 600K | 5.25 | 0.69 | 3.93 |

**说明**: F5-TTS 在 24 小时（LJSpeech）到 585 小时（LibriTTS）再到 95K 小时（Emilia）的不同数据规模下都能稳定训练，说明架构设计无需依赖 G2P 即可学会 text-speech alignment。但 LJSpeech 在 200K 步后过拟合。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Emilia]] | ~95K hours (en+zh, 过滤后) | 多语种，工业级规模 | Base 模型训练 |
| WenetSpeech4TTS Premium | 945 hours | 普通话 | 小模型消融 |
| [[LibriTTS]] / [[LibriTTS]]-R | 585 hours | 英文多说话人 | 数据规模消融 |
| [[LJSpeech]] | 24 hours | 英文单说话人 | 数据规模消融 |
| LibriSpeech-PC test-clean | 1127 samples | 4-10 秒子集（本文构建并公开） | 主评测 |
| Seed-TTS test-en | 1088 samples | Common Voice | 主评测 |
| Seed-TTS test-zh | 2020 samples | DiDiSpeech | 主评测 |

### 实现细节

- **Backbone**: 22 层 DiT (adaLN-zero), 16 heads, 1024 embed / 2048 FFN
- **ConvNeXt**: 4 层 ConvNeXt V2, 512 embed / 1024 FFN
- **优化器**: AdamW, peak LR 7.5e-5, linear warmup 20K steps, linear decay
- **Batch Size**: 307,200 audio frames (~0.91 hours per batch)
- **训练步数**: 1.2M updates
- **硬件**: 8x NVIDIA A100 80G, 训练超过 1 周
- **音频特征**: 100-dim log Mel, 24 kHz, hop length 256
- **Masking**: 随机 mask 70%~100% Mel 帧
- **CFG 训练**: masked speech 以 0.3 概率 drop，masked speech + text 以 0.2 概率 drop
- **Dropout**: 0.1 (attention + FFN)
- **推理**: EMA 权重, Euler ODE solver, CFG strength=2, Sway s=-1

### 可视化结果

项目主页展示了零样本生成、中英 code-switching、语速控制（0.7x/1.0x/1.3x）、情感迁移（6 种情感）和鲁棒性（绕口令）等定性 demo。F5-TTS 仅需指定总时长即可自动安排字符定位和节奏，无需显式 duration 控制。

---

## 批判性思考

### 优点
1. **极致简洁的架构设计**: 彻底去掉 phoneme、duration predictor、forced alignment、文本编码器，大幅降低系统复杂度，且效果不降反升
2. **Sway Sampling 是即插即用的**: 无需重新训练，可直接应用于任何 CFM 模型（在 E2 TTS 上也验证了），这是一个通用贡献
3. **完整的消融体系**: 从架构变体（6 种）到采样策略、ODE 求解器、声码器、数据规模都做了详尽消融，科学严谨
4. **RTF 0.15 的推理速度**: 在相近质量下是所有对比模型中最快的，实际部署价值大
5. **完全开源**: 代码 + checkpoint + 评测集（LibriSpeech-PC）全部公开，社区复现和二次开发友好

### 局限性
1. **SIM-o 偏低**: 0.66 的说话人相似度低于 [[E2 TTS]]（0.69-0.72）和 [[MaskGCT]]（0.687），作者未深入分析原因，可能与 ConvNeXt 过度精炼导致 prompt 声学信息丢失有关
2. **缺乏细粒度韵律控制**: 论文自己承认无法控制情感、韵律等副语言特征，而 [[CosyVoice 2]] 等已支持 instruction-based 控制
3. **Mel spectrogram 表示的效率瓶颈**: Mel 帧率（~93.75 Hz at 24kHz/256hop）远高于 [[Discrete Audio Token]] 方案（如 [[Mimi]] 12.5 Hz），序列长度是重大计算瓶颈
4. **时长估计简单**: 用字符数比例估计目标时长，没有考虑停顿、语速变化，不如 [[Duration Predictor]] 精确
5. **与 Seed-TTS_DiT 的差距**: Seed-TTS_DiT 在 WER 和 SIM 上显著领先（test-en WER 1.733 vs 1.83, SIM 0.790 vs 0.67），但 Seed-TTS 未公开，F5-TTS 作为开源方案的上限仍有提升空间

### 潜在改进方向
1. 引入 speaker adapter 或更强的 speaker encoder 改善 SIM-o
2. 用 latent codec 表示（如 [[EnCodec]] latent 或 [[DAC]] latent）替代 Mel 降低序列长度
3. 加入可控韵律/情感的条件模块

### 可复现性评估
- [x] 代码开源 (GitHub)
- [x] 预训练模型 (checkpoint 公开)
- [x] 训练细节完整 (LR, batch size, GPU, 步数都有)
- [x] 数据集可获取 ([[Emilia]] 公开数据集)
- [x] 评测集公开 (LibriSpeech-PC 由本文构建并发布)

---

## 关联笔记

### 基于
- [[E2 TTS]]: F5-TTS 的直接前身，同样用 filler token 但无 ConvNeXt，收敛慢且不鲁棒
- [[Voicebox]]: 同为 mask-and-infill 范式的 [[Flow Matching]] TTS，但依赖 phoneme alignment

### 对比
- [[CosyVoice]]: 两阶段（AR+Flow）范式，WER 偏高但 SIM 较好
- [[VALL-E]]: AR 离散 token 范式代表，RTF 0.732 远慢于 F5-TTS
- [[NaturalSpeech 3]]: 基于 factorized codec + diffusion，500M 参数
- [[MaskGCT]]: 1048M 参数，两阶段 T2S+S2A，SIM 高但模型大 3 倍
- [[FireRedTTS]]: 小红书出品，SIM-o 仅 0.47 显著偏低

### 方法相关
- [[Flow Matching]]: 核心生成范式，OT-CFM 训练
- [[DiT]]: Diffusion Transformer 架构基础
- [[ConvNeXt]]: V2 变体用于文本序列精炼
- [[Classifier-Free Guidance]]: 推理时条件增强
- [[RoPE]]: 自注意力位置编码

### 硬件/数据相关
- [[Emilia]]: 主训练数据集，~95K 小时多语种
- [[Vocos]]: 默认声码器
- [[BigVGAN]]: 备选声码器，WER 更低

---

## 速查卡片

> [!summary] F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching
> - **核心**: 去掉 phoneme/duration/对齐的纯 NAR 零样本 TTS
> - **方法**: DiT + ConvNeXt 文本精炼 + Sway Sampling 推理策略
> - **结果**: WER 2.42% / SIM-o 0.66 / RTF 0.15 (16 NFE) on LibriSpeech-PC
> - **代码**: [github.com/SWivid/F5-TTS](https://github.com/SWivid/F5-TTS)

---

*笔记创建时间: 2026-05-25*
