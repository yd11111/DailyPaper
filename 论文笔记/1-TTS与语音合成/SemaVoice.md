---
title: "SemaVoice: Semantic-Aware Continuous Autoregressive Speech Synthesis"
method_name: "SemaVoice"
authors: [Huimeng Wang, Hui Lu, Jiajun Deng, Haoning Xu, Youjun Chen, Xueyuan Chen, Zhaoqing Li, Shuhai Peng, Shiyin Kang, Xunying Liu]
year: 2026
venue: arXiv
tags: [zero-shot-tts, continuous-autoregressive, sfm-alignment, latent-diffusion, patch-wise-generation]
zotero_collection: 1-TTS与语音合成
image_source: online
arxiv_html: https://arxiv.org/html/2605.16964
created: 2026-05-19
---

# 论文笔记：SemaVoice: Semantic-Aware Continuous Autoregressive Speech Synthesis

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | The Chinese University of Hong Kong (CUHK) / Tsinghua University / SenseTime Research |
| 日期 | May 2026 |
| 项目主页 | 未公开 |
| 对比基线 | [[F5-TTS]] / [[CosyVoice 2]] / [[VoxCPM]] / [[VibeVoice]] / [[IndexTTS 2]] / [[MaskGCT]] / [[SparkTTS]] / [[FireRedTTS]] / [[HiggsAudio-v2]] / [[Qwen2.5-Omni]] |
| 链接 | [arXiv](https://arxiv.org/abs/2605.16964) / Code: 未开源 |

---

## 一句话总结

> 在 [[Continuous Autoregressive TTS|连续 AR TTS]] 的 VAE 表示中引入 [[WavLM]] 语义对齐 ($\mathcal{L}_{frame}+\mathcal{L}_{pair}$) ，配合 [[Patch-wise Diffusion Head]] 的 [[Qwen2.5-1.5B]] backbone，把 Seed-TTS 英文 WER 压到 **1.71%**。

---

## 核心贡献

1. **指出连续 AR TTS 的核心瓶颈**：VAE 学到的连续表示是为重建优化的，与 LLM 端做的"语义/韵律 planning"目标存在内在错配 (mismatch)，是 [[Error Accumulation|误差累积]] 的根因之一。
2. **SFM-guided 对齐机制**：在 σ-VAE 训练阶段额外加 frame-wise 余弦对齐 + pair-wise 自相似矩阵对齐两条路径，把 [[WavLM]]-large 的语义结构 "蒸" 进 VAE latent，**不改下游 TTS 架构和训练管线**。
3. **patch-wise 扩散 + history conditioning**：用 [[Qwen2.5-1.5B]] 做 LLM backbone，patch size = 2；轻量 [[LocDiT|Local Diffusion Transformer]] 头，把 next-patch 生成形式化为 **outpainting**（条件包含 LLM 的 $h_{i-1}$ + 上一拍 $p_{i-1}$）。消融显示去掉 history → WER 从 2.97 → 8.46，绝对决定性。
4. **150K h 双语训练**（100K Emilia + 50K 内部，中英各 25K），对照 SemaVoice-Emilia (100K) 给出干净的 scaling 曲线。
5. **新发现**：SFM 对齐的收益随表示粒度（帧率）变细呈指数放大——15 Hz 时只是 0.43 个绝对 WER 点，到 60 Hz 时变成 13.35 个点。

---

## 问题背景

### 要解决的问题
[[Zero-shot TTS]] 中，**连续 AR**（区别于 [[VALL-E]] / [[CosyVoice 2]] 那种 discrete AR）路线虽然在重建保真度上限上更高（不丢声学细节），但在 Seed-TTS 这种 hard 测评下 WER/CER 仍然落后于 cascaded discrete AR + diffusion 系统，主要表现是 **语义可懂度差 + 长句崩坏**。

### 现有方法的局限
- **[[VALL-E]] 系 discrete AR**：码率有限，重建天花板被钉死。
- **[[CosyVoice 2]] / [[Seed-TTS]] / [[BASE-TTS]] / [[FireRedTTS]] 等 cascaded（D-AR + C-NAR）**：先生成 semantic token 再用 flow/diffusion 补声学，难以在同一阶段联合捕捉长程 prosody 与精细 acoustic。
- **[[MELLE]] 那类回归式 continuous AR**：直接回归 Mel，倾向于 mean prediction，输出过平滑、低多样性。
- **[[DiTAR]] / [[VibeVoice]] / [[FELLE]] / [[VoxCPM]] 这些扩散式 continuous AR**：把生成质量提上来了，但 VAE latent 仍由纯重建驱动，**没有显式的语义约束**——LLM 端学到的语义/韵律 planning 必须从纯声学 latent 中"猜"出来，长句容易 drift。

### 本文的动机
既然 [[HuBERT]] / [[WavLM]] 这类 [[SSL Speech Representation|语音 SSL 表示]] 天生就承载语义和韵律结构，**那就让 VAE 在保留重建能力的同时，把 latent 空间向 SFM 特征对齐**——既给 LLM 一个语义友好的目标，又不丢声学细节。这是核心 insight。

---

## 方法详解

### 模型架构

SemaVoice 由两阶段组成：

- **第一阶段 ([[σ-VAE|σ-VAE]] + SFM 对齐)**：把 24 kHz 波形压到 **15 Hz** latent（1600× 时间下采样，潜在维度 32）。Encoder 是 7 段 hierarchical Transformer block + 1D depth-wise conv（替换 self-attention），mirror-symmetric decoder。训练目标 = 标准 VAE loss + 自适应权重的 SFM 对齐 loss。
- **第二阶段 (Continuous AR + LocDiT 头)**：[[Qwen2.5-1.5B]] 作为 LLM backbone，做 next-token diffusion 预测；patch size = 2 把 2 帧 latent 打成一个 token；[[LocDiT]] 是双向 [[DiT|Diffusion Transformer]] 头，在 patch 内全感受野；额外条件包含上一拍 $p_{i-1}$（history conditioning）。
- **CFG 推理**：[[Classifier-Free Guidance]] 引导尺度 $w=2.5$。

### 核心模块

#### 模块 1: SFM-Guided Alignment

**设计动机**: 闭合 [[VAE]] 重建目标和 [[Semantic-Prosodic Modeling|语义韵律建模]] 之间的差距。

**具体实现**:
- 用冻结的 [[WavLM]]-large 提取 SFM 特征 $\mathbf{e} = \mathcal{F}(\mathbf{x}) \in \mathbb{R}^{T_e \times d_e}$（音频先 resample 到 16 kHz）。
- 通过线性投影 $\mathcal{P}_e$ 把 $\mathbf{e}$ 映射到与 VAE latent $\mathbf{z}$ 同维度（$d=32$），并通过线性插值对齐时间步长得到 $\mathbf{s} \in \mathbb{R}^{T \times d}$。
- 双路对齐：
  - **Frame-wise**：逐帧余弦对齐，约束 *局部语义一致性*。
  - **Pair-wise**：把 $\mathbf{z}$ 和 $\mathbf{s}$ 的 $T\times T$ 自相似矩阵 $\mathbf{D}^z, \mathbf{D}^s$ 做 L1 对齐，约束 *全局结构关系*——这是关键，单纯逐帧对齐不够，因为 VAE 和 SFM 的绝对方向可能不一致，但相对关系应当一致。
- **梯度自适应权重** ($\lambda_{align}$)：根据 $\|\nabla_\theta \mathcal{L}_{mel}\|_2 / \|\nabla_\theta \mathcal{L}_{align}\|_2$ 比值动态平衡，避免对齐项淹没重建项。

#### 模块 2: Continuous AR with Patch-wise Diffusion (LocDiT)

**设计动机**: 长 latent 序列直接 AR 难做，且 diffusion head 不能脱离上下文独立采样。

**具体实现**:
- **Patch-based 序列降维**：连续 $L=2$ 帧 latent 合并为一个 patch $\mathbf{p}_i$，把序列长度减半，缓解 [[AR Long-Sequence Modeling|长序列建模]] 压力。
- **LLM backbone**：因果 [[Qwen2.5-1.5B]] 输入文本 token $\mathbf{c}$ + 历史 patch，输出隐状态 $\mathbf{h}_{i-1}$。
- **Diffusion head ([[LocDiT]])**：双向 [[DiT]]，输入 = (含噪 $\mathbf{p}_{i,t}$, 时间步 $t$, LLM 隐状态 $\mathbf{h}_{i-1}$, 上一拍 $\mathbf{p}_{i-1}$)；标准 [[DDPM]] $\epsilon$-prediction 目标。
- **Outpainting 形式化**：把"生成下一个 patch"重新表述为"在已生成 patch 之后做局部连续延展"，这是 history conditioning 的形象解释。
- **End-of-Speech**：LLM 同时联合预测 utterance-level 终止 token $\langle E \rangle$。
- **[[Classifier-Free Guidance|CFG]]**：训练时随机将 $\mathbf{h}_{i-1}$ 替换为 null embedding $\emptyset$；推理时按 $w=2.5$ 组合条件/无条件预测。

---

## 关键公式

### 公式 1: [[σ-VAE|σ-VAE 重参数采样]]

$$
\mathbf{z} = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon}, \quad \text{where } \boldsymbol{\epsilon} \sim \mathcal{N}(0, \mathbf{I}),\ \boldsymbol{\sigma} \sim \mathcal{N}(0, C_\sigma)
$$

**含义**: 与标准 [[VAE]] 不同，σ-VAE 让方差 $\boldsymbol{\sigma}$ 也是从预设分布采样，保证 latent 空间方差非消失，给下游 AR 一个"稳定"的目标分布。

**符号说明**:
- $\boldsymbol{\mu}$: 编码器输出的均值参数
- $\boldsymbol{\sigma}$: 来自 $\mathcal{N}(0, C_\sigma)$ 的随机方差（**不是**编码器输出）
- $\boldsymbol{\epsilon}$: 标准高斯噪声
- $\odot$: 逐元素乘

### 公式 2: VAE 多任务损失

$$
\mathcal{L}_{VAE} = \lambda_{mel} \mathcal{L}_{mel} + \lambda_{fm} \mathcal{L}_{fm} + \lambda_{adv} \mathcal{L}_{adv} + \lambda_{kl} \mathcal{L}_{kl}
$$

**含义**: VAE 训练用多分辨率 [[Mel Loss]] + 判别器 ([[Adversarial Loss]] + [[Feature Matching Loss]]) + KL 正则。

**符号说明**:
- $\lambda_{mel}=15.0$, $\lambda_{fm}=2.0$, $\lambda_{adv}=1.0$, $\lambda_{kl}=0.01$
- $\mathcal{L}_{mel}$: 多分辨率 Mel 重建
- $\mathcal{L}_{fm}, \mathcal{L}_{adv}$: 判别器 feature matching / 对抗损失
- $\mathcal{L}_{kl}$: 朝 prior 收敛的 KL 项

### 公式 3: [[SFM Alignment|SFM 对齐总损失]]

$$
\mathcal{L}_{align} = \mathcal{L}_{frame} + \mathcal{L}_{pair}
$$

**含义**: 对齐损失由局部 (frame-wise) 与全局 (pair-wise) 两项组成。

### 公式 4: Frame-wise 对齐

$$
\mathcal{L}_{frame} = \frac{1}{T} \sum_{t=1}^{T} \left(1 - \cos(\mathbf{z}_t, \mathbf{s}_t)\right)
$$

**含义**: 让每一帧 VAE latent 和投影后的 SFM 特征余弦相似度趋近 1，强制*局部语义对齐*。

**符号说明**:
- $\mathbf{z}_t \in \mathbb{R}^d$: 第 $t$ 帧 VAE latent
- $\mathbf{s}_t = \mathcal{P}_e(\mathbf{e})_t$: 第 $t$ 帧投影并时间插值后的 SFM 特征
- $\cos(\cdot,\cdot)$: 余弦相似度

### 公式 5: [[Pair-wise Structure Alignment|Pair-wise 结构对齐]]

$$
\mathcal{L}_{pair} = \frac{1}{T^2} \sum_{i,j} \left| \mathbf{D}_{i,j}^z - \mathbf{D}_{i,j}^s \right|
$$

**含义**: 对齐自相似矩阵的 L1 差，让 VAE latent 之间的相对关系与 SFM 特征之间的相对关系一致——*这是 SemaVoice 区别于纯 frame-level 蒸馏（如 [[T-Free TTS]]）的关键设计*。

**符号说明**:
- $\mathbf{D}^z, \mathbf{D}^s \in \mathbb{R}^{T\times T}$: 两个序列的内部余弦自相似矩阵
- $\mathbf{D}_{i,j}^z = \cos(\mathbf{z}_i, \mathbf{z}_j)$
- $\mathbf{D}_{i,j}^s = \cos(\mathbf{s}_i, \mathbf{s}_j)$

### 公式 6: 梯度自适应对齐权重

$$
\lambda_{align} = \alpha \cdot \frac{\|\nabla_\theta \mathcal{L}_{mel}\|_2}{\|\nabla_\theta \mathcal{L}_{align}\|_2 + \epsilon}
$$

**含义**: 用共享 encoder 参数 $\theta$ 上的梯度比值动态调权，避免对齐损失被重建损失淹没（或反之），$\alpha=0.5$ 是基础缩放系数。

### 公式 7: VAE 完整训练目标

$$
\mathcal{L}_{total} = \mathcal{L}_{VAE} + \lambda_{align} \mathcal{L}_{align}
$$

### 公式 8: [[Continuous Autoregressive TTS|连续 AR 联合分布]]

$$
p(\mathbf{z} \mid \mathbf{c}) = \prod_{i=1}^{T} p(\mathbf{z}_i \mid \mathbf{c}, \mathbf{z}_{<i})
$$

**含义**: 把 patch 级 latent 序列建模为以文本 $\mathbf{c}$ 为条件的因果分解。

### 公式 9: [[DDPM|扩散头训练目标]]

$$
\mathcal{L}_{\text{diff}} = \mathbb{E}_{\mathbf{p}_{i,0}, t, \epsilon}\left[ \|\epsilon - \epsilon_\theta(\mathbf{p}_{i,t}, t, \mathbf{h}_{i-1}, \mathbf{p}_{i-1})\|_2^2 \right]
$$

**含义**: 标准 [[DDPM]] $\epsilon$-prediction，但条件项除了 LLM 隐状态 $\mathbf{h}_{i-1}$ 外**还包含上一拍 $\mathbf{p}_{i-1}$**（history conditioning）。

**符号说明**:
- $\mathbf{p}_{i,0}$: 第 $i$ 个干净 latent patch
- $\mathbf{p}_{i,t} = \sqrt{\bar{\alpha}_t}\mathbf{p}_{i,0} + \sqrt{1-\bar{\alpha}_t}\epsilon$: 加噪 patch
- $\bar{\alpha}_t = \prod_{s=1}^t (1-\beta_s)$: 噪声时刻表累乘
- $\epsilon_\theta$: [[LocDiT]] 噪声预测网络
- $\mathbf{h}_{i-1}$: LLM 隐状态
- $\mathbf{p}_{i-1}$: 已生成的上一拍 patch

### 公式 10: LLM-based [[Classifier-Free Guidance|CFG]] 推理

$$
\hat{\epsilon} = (1 + w)\, \epsilon_\theta(\mathbf{p}_{i,t}, t, \mathbf{h}_{i-1}, \mathbf{p}_{i-1}) - w\, \epsilon_\theta(\mathbf{p}_{i,t}, t, \emptyset, \mathbf{p}_{i-1})
$$

**含义**: 把 LLM 隐状态作为可丢弃的"条件"做 CFG，**只 drop $\mathbf{h}_{i-1}$ 不 drop $\mathbf{p}_{i-1}$**——即弱化"语义 planning"但保留"局部声学 outpainting"。

**符号说明**:
- $w=2.5$: 引导强度
- $\emptyset$: null embedding（条件训练时按概率随机替换）

---

## 关键图表

### Figure 1: SemaVoice 框架总览

![Figure 1: SemaVoice framework overview](https://arxiv.org/html/2605.16964v1/x1.png)

**说明**: 
- **(a) [[σ-VAE|σ-VAE]] + SFM 对齐训练阶段**：输入 24 kHz 波形 $\mathbf{x}$，VAE encoder 输出 latent $\mathbf{z}$；并行从冻结的 [[WavLM]] 提语义特征 $\mathbf{e}$，经投影 + 时间插值得 $\mathbf{s}$。计算两个自相似矩阵 $\mathbf{D}^z, \mathbf{D}^s$ 并对齐。图里只画了对齐损失，省略了 mel/fm/adv/kl 等标准 VAE 损失。
- **(b) Continuous AR 推理流水线**：文本 $\mathbf{c}$ + 历史 patch 喂给 [[Qwen2.5-1.5B]] LLM backbone → 隐状态 $\mathbf{h}_{i-1}$ → [[LocDiT]] 扩散头（输入还包含上一拍 $\mathbf{p}_{i-1}$，做 outpainting）→ 当前 patch $\mathbf{p}_i$ → VAE decoder → 24 kHz 波形。

### Table 1: Seed-TTS 测试集客观评测

| Model | Type | Params | #Hours | EN WER↓ | EN SIM↑ | ZH CER↓ | ZH SIM↑ | Hard CER↓ | Hard SIM↑ |
|---|---|---|---|---|---|---|---|---|---|
| Ground Truth | - | - | - | 2.14 | 0.734 | 1.26 | 0.755 | - | - |
| [[Qwen2.5-Omni]] | MLLM | 7.0B | - | 2.72 | 0.632 | 1.70 | 0.752 | 7.97 | 0.747 |
| [[F5-TTS]] | C-NAR | 0.3B | 100K | 2.00 | 0.647 | 1.52 | 0.741 | 8.67 | 0.713 |
| [[MaskGCT]] | D-NAR | 1.0B | 100K | 2.62 | 0.717 | 2.27 | 0.774 | - | - |
| [[Spark-TTS\|SparkTTS]] | D-AR | 0.5B | 100K | 1.98 | 0.573 | 1.20 | 0.660 | - | - |
| [[FireRedTTS-2]] | D-AR | - | 1.4M | 1.95 | 0.665 | 1.14 | 0.732 | - | - |
| [[OpenAudio-s1]] | D-AR | 0.5B | 2.0M | 1.94 | 0.550 | 1.18 | 0.685 | 23.37 | 0.643 |
| [[HiggsAudio-v2]] | D-AR | 3.0B | 10M | 2.44 | 0.677 | 1.50 | 0.740 | 55.07 | 0.656 |
| [[CosyVoice]] | D-AR+C-NAR | 0.3B | 170K | 4.29 | 0.609 | 3.63 | 0.723 | 11.75 | 0.709 |
| [[CosyVoice 2]] | D-AR+C-NAR | 0.5B | 170K | 2.57 | 0.659 | 1.45 | 0.757 | 6.83 | 0.724 |
| [[FireRedTTS]] | D-AR+C-NAR | 0.5B | 248K | 3.82 | 0.460 | 1.51 | 0.635 | 17.45 | 0.621 |
| [[IndexTTS 2]] | D-AR+C-NAR | 1.5B | 55K | 2.23 | 0.706 | 1.03 | 0.765 | 7.12 | 0.755 |
| [[VoxCPM]]-Emilia | C-AR | 0.5B | 100K | 2.34 | 0.681 | 1.11 | 0.740 | 12.46 | 0.698 |
| [[VoxCPM]] | C-AR | 0.5B | 1.8M | 1.85 | **0.729** | **0.93** | 0.772 | 8.87 | 0.730 |
| [[VibeVoice]] | C-AR | 1.5B | - | 3.04 | 0.689 | 1.16 | 0.744 | - | - |
| **SemaVoice-Emilia** | C-AR | 1.5B | 100K | 1.91 | 0.657 | 1.32 | 0.728 | 9.37 | 0.687 |
| **SemaVoice** | C-AR | 1.5B | 150K | **1.71** | 0.694 | 1.18 | 0.754 | 8.09 | 0.711 |

**说明**:
- C/D = continuous / discrete；AR/NAR/MLLM = 自回归/非自回归/多模态 LLM。
- **EN WER 1.71% 是表中最低**，比 [[VoxCPM]] (1.85)、[[OpenAudio-s1]] (1.94)、[[F5-TTS]] (2.00) 都好。
- **SIM 没赢**：英文 0.694 略输 [[IndexTTS 2]] (0.706) 和 [[VoxCPM]] (0.729)；中文 0.754 略低于 [[CosyVoice 2]] (0.757) 和 [[IndexTTS 2]] (0.765)。
- Hard 子集 CER 8.09，输给 [[CosyVoice 2]] (6.83) 和 [[IndexTTS 2]] (7.12)，但好于 [[VoxCPM]] (8.87) 和 [[F5-TTS]] (8.67)。
- **同等数据规模 (~100K) 比较**：SemaVoice-Emilia 在 EN WER (1.91) 上是最强的连续 AR 方案，超过 [[F5-TTS]] (2.00)、[[VoxCPM]]-Emilia (2.34)、[[VibeVoice]] (3.04)。

### Table 2: Seed-TTS 测试集主观评测 (95% 置信区间)

| System | EN N-MOS | EN S-MOS | ZH N-MOS | ZH S-MOS |
|---|---|---|---|---|
| Ground Truth | 4.02 ± 0.09 | 4.53 ± 0.12 | 3.94 ± 0.10 | 4.45 ± 0.07 |
| [[CosyVoice 2]] | 3.96 ± 0.13 | 3.78 ± 0.12 | 3.73 ± 0.11 | 4.01 ± 0.15 |
| [[IndexTTS 2]] | 3.75 ± 0.11 | 3.93 ± 0.14 | 3.79 ± 0.13 | 4.07 ± 0.13 |
| SemaVoice-Emilia | 3.86 ± 0.11 | 3.69 ± 0.12 | 3.91 ± 0.12 | 3.92 ± 0.12 |
| **SemaVoice** | **3.98 ± 0.12** | 3.89 ± 0.14 | **4.07 ± 0.13** | 4.03 ± 0.11 |

**说明**:
- 12 位评分员，5 分制，20 句测试。
- 中文 N-MOS 4.07 **超过 ground truth (3.94)**——典型的"主观分饱和"，但确实达到顶级 baseline 水平。
- 英文 N-MOS 3.98 接近 GT 4.02，比 [[CosyVoice 2]] 略低 (3.96)。
- **S-MOS 是相对短板**：英文 3.89 不如 [[IndexTTS 2]] 3.93，与客观 SIM 短板一致。

### Table 3: 关键组件消融（Emilia-EN）

| 配置 | SFM Align. | History | WER↓ | SIM↑ |
|---|---|---|---|---|
| **SemaVoice** | ✓ | ✓ | **2.97** | **0.635** |
| w/o SFM Align. | × | ✓ | 3.40 | 0.625 |
| w/o History | ✓ | × | 8.46 | 0.587 |

**关键发现**:
- 去掉 [[SFM Alignment|SFM 对齐]]：WER 2.97 → 3.40 (相对 +14.5%)，SIM 略降——证明对齐确实在改善表示空间质量。
- **去掉 history conditioning：WER 暴涨 2.97 → 8.46 (相对 +185%)**，SIM 也大跌——说明 patch 之间的局部依赖才是 SemaVoice 稳定生成的命脉。**这条结果比 SFM 对齐更剧烈**，是 SemaVoice 的"防雪崩保险丝"。

### Table 4: 表示粒度对 SFM 对齐效果的影响（固定信息率）

| Freq. | Latent Dim. | SFM Align. | PESQ↑ | STOI↑ | UTMOS↑ | Gen WER↓ | Gen SIM↑ |
|---|---|---|---|---|---|---|---|
| 15 Hz | 32 | ✓ | 3.175 | 0.950 | 4.030 | **2.97** | **0.635** |
| 15 Hz | 32 | × | 3.179 | 0.949 | 4.066 | 3.40 | 0.625 |
| 30 Hz | 16 | ✓ | 3.086 | 0.953 | 4.052 | **3.24** | **0.621** |
| 30 Hz | 16 | × | 3.078 | 0.953 | 4.060 | 5.20 | 0.615 |
| 60 Hz | 8 | ✓ | 2.908 | 0.945 | 3.965 | **14.71** | **0.526** |
| 60 Hz | 8 | × | 2.880 | 0.944 | 3.965 | 28.06 | 0.464 |

**关键发现**:
- 三个 (Freq, Dim) 组合在保持总信息率不变的前提下重建质量 (PESQ/STOI/UTMOS) **基本相同**——证明信息容量是匹配的。
- 但生成 WER 随帧率上升急剧恶化（15→30→60 Hz: 2.97→3.24→14.71，开了对齐时；3.40→5.20→**28.06**，关闭对齐时）——更细粒度 = 更长序列 = AR 建模难度上升。
- **SFM 对齐的相对收益随粒度变细放大**：15 Hz 时 0.43 个 WER 点，30 Hz 时 1.96 点，60 Hz 时 13.35 点。论文最重要的"惊喜结论"——*如果未来推 streaming 或更细粒度的 codec，这个机制会更值钱*。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Emilia]] | 100K h（中英各 ~50K） | 开源大规模多语种 TTS 语料 | SemaVoice-Emilia 训练；VAE 训练（采样 20K） |
| 内部数据 | 50K h（中英各 25K） | 内部清洗 | SemaVoice 完整训练补充 |
| [[Seed-TTS-eval]] | EN / ZH / Hard 三个子集 | Hard = 复杂句 | 客观 + 主观评测 |
| [[LibriTTS]] test-clean | - | - | VAE 重建评测（PESQ/STOI/UTMOS） |

### 实现细节

- **VAE 架构**：mirror-symmetric encoder-decoder；7 hierarchical Transformer stages，self-attn 替换为 1D depth-wise conv；6 次下采样把 24 kHz 压到 **15 Hz**（1600× 时间下采样）。
- **VAE 训练**：8× A800，280K steps，全局 batch = 320 秒，cosine LR peak $1\times 10^{-4}$。
- **TTS backbone**：[[Qwen2.5-1.5B]] 初始化；patch size = 2；history conditioning。
- **TTS 训练**：8× **H200**，batch = 8192 秒；SemaVoice-Emilia 跑 150K steps，全量 SemaVoice 跑 300K steps；ablation 用 8× A800、batch=1024 秒、100K steps；ablation LR peak $7.5\times 10^{-5}$。
- **CFG 引导尺度**：$w=2.5$。
- **SFM**：[[WavLM]]-large，输入 16 kHz；冻结。
- **VAE 损失权重**：$\lambda_{mel}=15.0,\ \lambda_{fm}=2.0,\ \lambda_{adv}=1.0,\ \lambda_{kl}=0.01$，$\alpha_{align}=0.5$。
- **评测 ASR**：EN 用 [[Whisper]]-large-v3，ZH/Hard 用 [[Paraformer]]-zh；说话人相似度用 [[WavLM]]-large。

### 可视化结果

论文未提供 attention map / 频谱对比图，只有 Figure 1 一张架构图。

---

## 批判性思考

### 优点

1. **诊断准确**：把"VAE 重建目标 vs LLM 语义建模目标"的 mismatch 识别为连续 AR TTS 的核心痛点，比之前 [[VibeVoice]] / [[VoxCPM]] 那种"加更多 capacity"的解法更直击病因。
2. **Pair-wise structure alignment 是亮点**：单纯逐帧对齐会受限于 [[WavLM]] 和 VAE latent 的绝对方向差异，对齐自相似矩阵改成"对齐相对结构"是正确的设计，**这是 SemaVoice 能 work 的关键之一**。
3. **梯度自适应权重** (Eq. 6) 避免了对齐项手调系数，工程上很实用。
4. **正交性强**：SFM 对齐只动 VAE 训练，不改下游 TTS pipeline，**可以无成本接到任何 continuous AR TTS 上**——和 [[VoxCPM]] / [[DiTAR]] / [[VibeVoice]] 都兼容。
5. **粒度 scaling 实验** (Table 4) 是论文最有 insight 的部分，给出可推广的结论：粒度越细 SFM 对齐越值钱。

### 局限性

1. **未开源**：截至 v1 没看到 GitHub / 模型 / demo 主页，复现有难度（虽然方法描述足够清楚）。
2. **SIM 不是 SOTA**：英文 SIM 0.694 < [[VoxCPM]] 0.729 / [[IndexTTS 2]] 0.706，主观 S-MOS 也只是 mid-pack。说明 SFM 对齐主要受益的是**语义可懂度**，对**说话人相似度**帮助有限——合理，因为 [[WavLM]] 是混合 content+speaker 的 SSL 表示，对齐到它的"语义部分"不一定增强 speaker 信息。
3. **没有流式**：论文没提 streaming / first-packet latency / RTF，[[Qwen2.5-1.5B]] + diffusion head 的成本明显高于 [[F5-TTS]]，实际部署不友好。
4. **Hard 子集落后**：8.09 CER 输给 [[CosyVoice 2]] 6.83 和 [[IndexTTS 2]] 7.12，说明 SFM 对齐对极端长/复杂句的帮助还不够，可能需要叠加 LLM 端的更强 planning。
5. **WavLM 的局限就是 SemaVoice 的局限**：方法效果上限受 [[SSL Speech Representation|SSL]] 表示质量制约，多语种、低资源语言、噪声环境下未验证。
6. **VAE 帧率固定 15 Hz**：和 [[Mimi]] (12.5 Hz)、[[CosyVoice 2]] tokenizer 等对手相比并不极致，Table 4 表明 60 Hz 时整体性能崩溃，但生产中多数对手在更高帧率上也能 work——意味着 SemaVoice 的 latent 设计仍偏保守。

### 潜在改进方向

1. **替换 SFM**：用 [[XEUS]] / [[w2v-BERT 2.0]] / 多语种 SSL，看能否拉起 Hard 子集。
2. **Speaker-aware 对齐**：在 frame-wise / pair-wise 之外加一项 speaker prototype 对齐，针对性提 SIM。
3. **流式适配**：history conditioning 已经是 patch-local 设计，理论上配合 chunked LLM 推理就可以做流式 TTS，缺的只是 streaming KV cache 实现和首包延迟评估。
4. **DPO / RLHF 后训练**：1.71% WER 已经接近 GT 2.14%（而 GT 用 Whisper 反解会引入 ASR 错误），主观优化空间仍在 prosody，可以接 [[Speech DPO]]。
5. **更细粒度版本**：60 Hz 配 SFM 对齐 WER 14.71 还是太高，但说不定加大 patch size（4 / 8）+ 更深 LocDiT 能压回去，做出 truly low-bitrate continuous AR TTS。

### 可复现性评估
- [ ] 代码开源 (未发现)
- [ ] 预训练模型 (未发现)
- [x] 训练细节完整 (超参/数据/硬件/损失权重都写明)
- [x] 数据集可获取 ([[Emilia]] 公开，内部 50K 不公开)

---

## 关联笔记

### 基于
- [[DiTAR]]: SemaVoice 的 patch-wise diffusion + LLM backbone 直接借鉴 DiTAR；LocDiT 是 DiTAR 套路的轻量化版本。
- [[σ-VAE]]: 引入随机方差 $\sigma \sim \mathcal{N}(0, C_\sigma)$ 的稳定 latent VAE，原论文 [42]。
- [[VibeVoice]]: 同样的 next-token diffusion 路线，但没有 SFM 对齐。
- [[WavLM]]: 用作 frozen SFM。

### 对比
- [[F5-TTS]]: 同样无显式 phoneme/duration，纯 flow matching 的 NAR；SemaVoice 是 AR 路线。
- [[CosyVoice 2]]: 当前 cascaded D-AR+C-NAR 的代表，SemaVoice 想用纯 C-AR 一体化超越它。
- [[VoxCPM]]: 最直接的对手，同样 100K Emilia 训练，C-AR 路线；SemaVoice-Emilia EN WER 比它好（1.91 vs 2.34）。
- [[IndexTTS 2]]: 1.5B D-AR+C-NAR，SIM 强但 WER 落后于 SemaVoice。
- [[MELLE]]: 早期回归式 C-AR，被 SemaVoice 视为基线 family。

### 方法相关
- [[SFM Alignment]]: 核心创新组件
- [[Pair-wise Structure Alignment]]: 对齐自相似矩阵的全局结构损失
- [[LocDiT]]: 轻量 patch-wise [[DiT]] 头
- [[Patch-wise Diffusion Head]]: 把 next-token 替换为 next-patch diffusion 的范式
- [[Classifier-Free Guidance]]: $w=2.5$ 推理引导
- [[DDPM]]: 扩散头基础
- [[Continuous Autoregressive TTS]]: 论文所属范式

### 硬件/数据相关
- [[Emilia]]: 主训练集
- [[Seed-TTS-eval]]: 评测基准
- [[LibriTTS]]: VAE 重建评测
- [[Qwen2.5-1.5B]]: LLM backbone

---

## 速查卡片

> [!summary] SemaVoice
> - **核心**: 在 σ-VAE 训练里加 [[WavLM]] 双路对齐（frame-wise 余弦 + pair-wise 自相似矩阵），让连续 latent 既保留重建保真度又承载语义结构。
> - **方法**: σ-VAE (24 kHz → 15 Hz, dim=32) + [[Qwen2.5-1.5B]] LLM + patch-wise [[LocDiT]] 扩散头（patch=2，history conditioning）+ CFG ($w=2.5$)。
> - **结果**: Seed-TTS EN WER **1.71%** (SOTA among open systems), ZH CER 1.18, ZH N-MOS 4.07 (>GT)。
> - **训练**: 150K 双语 (100K Emilia + 50K 内部)，VAE 8×A800/280K steps，TTS 8×H200/300K steps。
> - **代码**: 未开源。

---

*笔记创建时间: 2026-05-19*
