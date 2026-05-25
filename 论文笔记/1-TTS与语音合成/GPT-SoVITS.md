---
title: "GPT-SoVITS: Few-Shot Voice Cloning and Text-to-Speech WebUI"
method_name: "GPT-SoVITS"
authors: [RVC-Boss, 花儿不哭 (Community)]
year: 2024
venue: GitHub (Open-Source Project)
tags: [zero-shot-tts, few-shot-tts, voice-cloning, autoregressive, vits, semantic-token, cross-lingual, open-source]
image_source: none
created: 2026-05-25
---

# 技术笔记：GPT-SoVITS

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | 开源社区（RVC-Boss 主导） |
| 日期 | 2024 年 1 月首发，持续迭代至 2025+ |
| 项目主页 | [GitHub](https://github.com/RVC-Boss/GPT-SoVITS) |
| 对比基线 | [[CosyVoice 2]] / [[VALL-E]] / [[F5-TTS]] |
| 链接 | [GitHub](https://github.com/RVC-Boss/GPT-SoVITS) / Stars: 57.8k+ |

---

## 一句话总结

> 中文社区最流行的开源零样本/少样本 TTS 系统，结合 GPT 自回归语义 token 预测与 [[VITS]] 声学解码器，仅需 5 秒音频即可克隆音色。

---

## 核心贡献

1. **两阶段架构（GPT + SoVITS）**: 将 TTS 拆分为 GPT 自回归预测 [[Semantic Token]] 和 [[VITS]] 基声码器两个阶段，兼顾质量和可控性
2. **极低门槛少样本语音克隆**: 零样本仅需 5 秒参考音频，少样本仅需约 1 分钟数据即可微调出高相似度克隆
3. **完整工具链 WebUI**: 集成语音分离（UVR5）、自动分割、[[ASR]] 标注、训练、推理的一站式 WebUI，极大降低使用门槛
4. **多语种跨语言支持**: 支持中文、英语、日语、韩语、粤语，且可跨语言合成
5. **持续迭代多版本**: V1→V2→V3→V4→V2Pro/V2ProPlus，逐步提升音色相似度、稳定性和音质

---

## 问题背景

### 要解决的问题

如何以极低的数据门槛（5 秒～1 分钟）实现高质量的语音克隆 TTS，并提供开箱即用的完整工具链。

### 现有方法的局限

- [[VALL-E]] 等学术方案需要大规模预训练数据，且未开源完整可用版本
- [[VITS]] 原版需要较多训练数据才能达到好的质量
- 现有开源方案缺少从数据预处理到推理的一体化工具链
- 跨语言 TTS 在中文社区的支持不够成熟

### 本文的动机

将 GPT 风格的 [[Autoregressive Model|自回归语言建模]] 引入 [[Semantic Token]] 预测，让模型具备强大的上下文学习能力（in-context learning），再通过 [[VITS]] 解码器将语义 token 转化为自然波形。两阶段解耦设计让每个模块都能独立优化。

---

## 方法详解

### 整体架构

GPT-SoVITS 采用**两阶段级联**架构：

- **第一阶段 — GPT（Text2Semantic）**: 输入 [[Phoneme]] 序列 + [[BERT]] 文本特征 + 参考音频的语义 token 作为 prompt，[[Autoregressive Model|自回归]]预测目标语义 token 序列
- **第二阶段 — SoVITS（Semantic2Waveform）**: 输入第一阶段预测的语义 token + 文本 + 参考音频 Mel，通过 [[VITS]] 架构生成最终波形

### 第一阶段：GPT — Text2SemanticDecoder

#### 输入表示

1. **Phoneme Embedding**: [[Phoneme]] ID 通过 TokenEmbedding（512 维）+ Sinusoidal Positional Encoding（learnable alpha）
2. **BERT Feature**: 使用 Chinese-RoBERTa-WWM-Ext-Large 提取的 1024 维上下文特征，通过线性层投影到 512 维，与 phoneme embedding 相加
3. **Semantic Token Embedding**: 目标语义 token（词表大小 1025 = 1024 码本 + 1 EOS）独立的 TokenEmbedding + Positional Encoding

#### Transformer 配置

| 超参数 | 值 |
|--------|-----|
| Hidden Dim / Embedding Dim | 512 |
| Attention Heads | 8 |
| Transformer Layers | 12 |
| FFN Dim | 2048 (4x) |
| Dropout | 0.1（Transformer 层内） |
| Vocab Size | 1025 |
| Phoneme Vocab Size | 512 |

#### 注意力掩码设计

文本和音频 token 拼接为一个序列，使用**非对称因果掩码**：

- **Text-to-Text**: 全可见（双向注意力），文本 token 之间互相可见
- **Text-to-Audio**: 全遮蔽，文本不能看到音频 token
- **Audio-to-Text**: 全可见，音频 token 可以看到所有文本
- **Audio-to-Audio**: 标准因果掩码（上三角遮蔽），保证自回归性

这种设计让模型在生成语义 token 时能完整利用文本信息（类似 prefix LM），同时保持音频部分的自回归属性。

#### 训练损失

$$
\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{CE}} + \mathcal{L}_{\text{DPO}}
$$

**含义**: 总损失由交叉熵损失和 [[DPO]] 对齐损失两部分组成

**符号说明**:
- $\mathcal{L}_{\text{CE}}$: 使用 `reduction="sum"` 的 [[Cross Entropy]] 损失，duration 越长梯度更新越多
- $\mathcal{L}_{\text{DPO}}$: [[DPO|Direct Preference Optimization]] 损失，reference-free 模式，$\beta = 0.2$

训练使用 [[ScaledAdam]] 优化器（lr=0.01, betas=(0.9, 0.95), clipping_scale=2.0），配合 WarmupCosine 学习率调度。

#### 推理过程

1. 文本端：Phoneme embedding + BERT 特征 → Positional Encoding → Transformer 前缀
2. 参考音频的语义 token 作为 prompt（最多 150 个或序列 50%），初始化 KV Cache
3. 自回归逐 token 生成，使用 [[Top-k Sampling]] + Top-p + Temperature + [[Repetition Penalty]]
4. 前 10 个 token 强制不停止（EOS logit 被 mask）
5. 停止条件：采样到 EOS / 超过 `early_stop_num` / 达到 1500 步上限
6. 支持流式推理：按 `chunk_length` 分块输出

### 第二阶段：SoVITS — SynthesizerTrn

基于 [[VITS]] 架构的条件 VAE + [[Normalizing Flow]] + [[HiFi-GAN]] 解码器。

#### 核心模块

| 模块 | 类名 | 功能 |
|------|------|------|
| **Prior Encoder** | `TextEncoder` (`enc_p`) | 融合量化 SSL + 文本，通过 [[MRTE]] 注入音色，输出先验分布均值/方差 |
| **Posterior Encoder** | `PosteriorEncoder` (`enc_q`) | WaveNet 风格，从频谱图编码后验分布 $z$（重参数化采样） |
| **[[Normalizing Flow]]** | `ResidualCouplingBlock` (`flow`) | 4 层残差耦合层 + flip，在先验和后验之间变换 |
| **Decoder/Vocoder** | `Generator` (`dec`) | [[HiFi-GAN]] 架构，从隐变量 $z$ 生成波形 |
| **Quantizer** | `ResidualVectorQuantizer` | 将 SSL 特征离散化为语义码（1 层 [[RVQ]]，1024 码本） |
| **Reference Encoder** | `MelStyleEncoder` (`ref_enc`) | 从参考音频 Mel 提取全局说话人/风格向量 |
| **[[Multi-Period Discriminator|判别器]]** | `MultiPeriodDiscriminator` | Scale + Period（2, 3, 5, 7, 11）判别器 |

#### MRTE（Multi-Reference Timbre Encoder）

MRTE 是 GPT-SoVITS 的关键创新模块，通过 [[Cross-Attention]] 融合音色和内容：

- **Query**: SSL 编码（内容信息）
- **Key/Value**: 文本编码（语言信息）
- 通过 4 头 [[Cross-Attention]] 进行融合
- 残差连接：`CrossAttention(ssl, text) + ssl + speaker_embedding`
- 最终通过 1D Conv 投影到输出维度（192）

MRTE 内置消融模式：mode 1 去除 cross-attention（仅音色），mode 2 去除 SSL（仅文本），用于分析各组件贡献。

#### 训练流程

$$
\mathcal{L}_{\text{VITS}} = \mathcal{L}_{\text{recon}} + \mathcal{L}_{\text{KL}} + \mathcal{L}_{\text{adv}} + \mathcal{L}_{\text{fm}}
$$

**含义**: VITS 标准训练损失

**符号说明**:
- $\mathcal{L}_{\text{recon}}$: [[Mel Loss|Mel 频谱重建损失]]
- $\mathcal{L}_{\text{KL}}$: [[KL Divergence|KL 散度]]，后验编码 $z$ 经 flow 变换到 $z_p$ 后与先验分布对齐
- $\mathcal{L}_{\text{adv}}$: [[Adversarial Loss|对抗损失]]，[[Multi-Period Discriminator]] 提供
- $\mathcal{L}_{\text{fm}}$: [[Feature Matching Loss|特征匹配损失]]

#### 推理流程

1. 量化 SSL token + 文本 → TextEncoder → 先验分布 $(\mu, \sigma)$
2. 采样 $z_p \sim \mathcal{N}(\mu, \sigma)$（可调 noise scale）
3. $z_p$ 经 reverse flow → $z$
4. $z$ 经 [[HiFi-GAN]] Generator → 波形

### 预训练模型与文本前端

#### SSL 特征提取

使用 [[ContentVec]] / Chinese Speech Pretrain（腾讯 GameMate）提取 768 维 SSL 特征，经 [[RVQ]]（1 层，1024 码本）量化为 [[Semantic Token]]。

#### 文本前端

| 语言 | 方案 |
|------|------|
| 中文 | [[G2P|G2PW]] + pypinyin-g2pW + PaddleSpeech 文本正规化 |
| 英文 | Faster [[Whisper]] Large V3 |
| 日文 | Faster [[Whisper]] Large V3 |
| 韩文/粤语 | V2+ 新增支持 |
| 多语种切分 | split-lang 库 |

#### 说话人验证

V2Pro+ 版本引入 ERes2NetV2（w24s4ep4）[[Speaker Encoder|说话人编码器]]，提取 20480 维向量投影到 `gin_channels`。

---

## 关键公式

### 公式1: [[Cross Entropy|自回归语义 token 预测损失]]

$$
\mathcal{L}_{\text{CE}} = \sum_{t=1}^{T} -\log p_\theta(s_t \mid s_{<t}, \mathbf{x}_{\text{text}}, \mathbf{s}_{\text{prompt}})
$$

**含义**: GPT 阶段的核心训练目标，在给定文本和参考音频语义 prompt 的条件下自回归预测语义 token

**符号说明**:
- $s_t$: 第 $t$ 步的目标语义 token（词表大小 1024 + EOS）
- $s_{<t}$: 前 $t-1$ 步已预测的语义 token
- $\mathbf{x}_{\text{text}}$: phoneme + BERT 特征的文本表示
- $\mathbf{s}_{\text{prompt}}$: 参考音频的语义 token 前缀

### 公式2: [[DPO|Direct Preference Optimization 损失]]

$$
\mathcal{L}_{\text{DPO}} = -\mathbb{E}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y_w)}{\pi_\theta(y_l)}\right)\right]
$$

**含义**: 对齐阶段损失（reference-free 模式），通过偏好对提升生成质量

**符号说明**:
- $\beta = 0.2$: KL 约束系数
- $y_w, y_l$: 优选和劣选的语义 token 序列
- $\pi_\theta$: 当前策略模型
- $\sigma$: Sigmoid 函数

### 公式3: [[KL Divergence|VITS 变分推断]]

$$
\mathcal{L}_{\text{KL}} = D_{\text{KL}}\left(q_\phi(z \mid \mathbf{x}_{\text{spec}}) \;\|\; p_\theta(z \mid \mathbf{x}_{\text{ssl}}, \mathbf{x}_{\text{text}}, \mathbf{g})\right)
$$

**含义**: 后验编码器与先验编码器之间的 KL 散度，通过 [[Normalizing Flow]] 桥接

**符号说明**:
- $q_\phi(z \mid \mathbf{x}_{\text{spec}})$: 后验分布（WaveNet 编码频谱图）
- $p_\theta(z \mid \mathbf{x}_{\text{ssl}}, \mathbf{x}_{\text{text}}, \mathbf{g})$: 先验分布（TextEncoder 编码 SSL + 文本 + 说话人）
- $\mathbf{g}$: MelStyleEncoder 提取的全局说话人/风格向量

### 公式4: [[Cross-Attention|MRTE 跨注意力融合]]

$$
\text{MRTE}(\mathbf{h}_{\text{ssl}}, \mathbf{h}_{\text{text}}, \mathbf{g}) = \text{Conv}_{1\times1}\left(\text{MHA}(\mathbf{h}_{\text{ssl}}, \mathbf{h}_{\text{text}}, \mathbf{h}_{\text{text}}) + \mathbf{h}_{\text{ssl}} + \mathbf{g}\right)
$$

**含义**: 以 SSL 编码为 Query、文本编码为 Key/Value 做 4 头交叉注意力，加上残差和说话人嵌入

**符号说明**:
- $\mathbf{h}_{\text{ssl}}$: SSL 内容编码（经 1D Conv 投影到 hidden_size）
- $\mathbf{h}_{\text{text}}$: 文本编码（经 1D Conv 投影到 hidden_size）
- $\mathbf{g}$: 全局说话人嵌入向量
- MHA: Multi-Head Attention（4 头）

---

## 版本演进

### Table 1: 各版本特性对比

| 版本 | 预训练数据 | 输出采样率 | 声码器 | 音色特点 | 关键改进 |
|------|-----------|-----------|--------|---------|---------|
| V1 | ~2,000h | 32kHz | [[HiFi-GAN]] | 倾向训练集整体音色 | 首版：零样本 + 少样本 + 中英日 |
| V2 | ~5,000h | 32kHz | [[HiFi-GAN]] | 倾向训练集整体音色 | 扩展预训练 + 韩语/粤语 + 优化文本前端 |
| V3 | - | 24kHz | [[BigVGAN]] v2 | **倾向参考音频** | 更高音色相似度 + GPT 更稳定 + 更富表达力 |
| V4 | - | **48kHz** | 新 Vocoder | 倾向参考音频 | 修复 V3 金属感 + 原生 48kHz + 防闷声 |
| V2Pro | - | 32kHz | [[HiFi-GAN]] | 兼顾两者 | V4 品质 + V2 速度/显存 + ERes2NetV2 |
| V2ProPlus | - | 32kHz | [[HiFi-GAN]] | 兼顾两者 | V2Pro 进一步优化 |

**关键行为差异**: V1/V2/V2Pro 合成音色更偏向训练集整体，对低质量训练数据容忍度高；V3/V4 音色更偏向参考音频，需要更高质量的训练数据。

### Table 2: 推理速度（RTF — V2 ProPlus）

| 硬件 | RTF | 备注 |
|------|-----|------|
| RTX 4060 Ti | 0.028 | - |
| RTX 4090 | **0.014** | ~1400 字（4 分钟语音）仅需 3.36 秒 |
| Apple M4 CPU | 0.526 | - |

---

## 实验

### 数据集

| 数据集 | 规模 | 用途 |
|--------|------|------|
| 内部预训练集 V1 | ~2,000h | V1 预训练 |
| 内部预训练集 V2 | ~5,000h | V2 预训练 |
| 目标扩展 | ~10,000h | TODO 中已标记完成 |

### 实现细节

- **优化器**: [[ScaledAdam]]（lr=0.01, betas=(0.9, 0.95), clipping_scale=2.0）
- **学习率调度**: WarmupCosine
- **梯度累积**: 每 4 步
- **精度**: 支持 fp16（`is_half` 可配置）
- **框架**: PyTorch 2.x + PyTorch Lightning
- **硬件**: CUDA 12.4-12.8 / Apple Silicon / CPU

### 外部评测数据

根据 [[CosyVoice 2]] 论文中的对比（[[LibriSpeech]] test-clean）：

| 模型 | WER (%) ↓ | SIM-O ↑ |
|------|-----------|---------|
| [[CosyVoice 2]] | **2.57** | **0.764** |
| [[F5-TTS]] | 4.85 | 0.623 |
| GPT-SoVITS | 5.13 | 0.405 |

GPT-SoVITS 在学术评测上与 [[CosyVoice 2]] 等有明显差距，但在中文社区实际使用体验和易用性上表现优异。

---

## 批判性思考

### 优点

1. **极低使用门槛**: WebUI 集成全流程（数据处理→训练→推理），非技术用户也能上手
2. **社区生态最活跃**: GitHub 57.8k+ stars，中文语音克隆事实标准
3. **版本迭代快速**: 从 V1 到 V4/V2ProPlus，持续改进音质、稳定性和速度
4. **两阶段解耦合理**: GPT 负责语义建模（可利用 LM 的上下文能力），[[VITS]] 负责声学转换，各自可独立优化
5. **工程实现完善**: KV Cache、批量推理、流式输出、动态 batch 剔除等优化齐全
6. **跨语言能力**: 中英日韩粤五语种 + 跨语言合成

### 局限性

1. **无正式论文**: 缺少系统性的实验对比和消融研究，技术细节散落在代码中
2. **学术评测弱**: 在 [[LibriSpeech]] 上 WER 5.13%、SIM-O 0.405，远低于 [[CosyVoice 2]]（2.57% / 0.764）
3. **音色相似度受限**: V1/V2 音色偏向训练集整体而非参考音频；V3/V4 改善但引入金属感等新问题
4. **GPT 稳定性**: 自回归生成存在重复/遗漏问题（V3 有改善但未彻底解决）
5. **训练数据未公开**: 预训练数据集不公开，影响可复现性
6. **单一说话人嵌入**: 依赖全局 speaker embedding 而非更细粒度的风格控制

### 潜在改进方向

1. 引入 [[Conditional Flow Matching]] 替代 [[Normalizing Flow]]（参考 [[F5-TTS]] / [[CosyVoice 2]]）
2. 使用更强的 [[Semantic Token]] 提取方案（如 supervised semantic token from [[CosyVoice 2]]）
3. 增加 [[Duration Predictor]] 做显式时长控制
4. 引入 MOS/UTMOS 做 [[Speech DPO]] 训练

### 可复现性评估

- [x] 代码开源（MIT License）
- [x] 预训练模型（所有版本均提供）
- [ ] 训练细节完整（缺少正式文档，需看代码）
- [ ] 数据集可获取（预训练数据未公开）

---

## 关联笔记

### 基于

- [[VITS]]: 第二阶段声学解码器的基础架构
- [[VALL-E]]: 自回归语义 token 预测的思路来源
- [[SoundStorm]]: 影响了 AR token 建模设计
- [[ContentVec]]: SSL 特征提取 backbone

### 对比

- [[CosyVoice 2]]: 阿里通义 TTS，在学术评测上显著优于 GPT-SoVITS
- [[F5-TTS]]: 纯 Flow Matching 无 phoneme 方案，WER/SIM-O 均优于 GPT-SoVITS
- [[Fish-Speech]]: 同为开源社区 AR token TTS
- [[Spark-TTS]]: 单一 LLM-based TTS 方案

### 方法相关

- [[HiFi-GAN]]: V1/V2 使用的声码器
- [[BigVGAN]]: V3+ 使用的声码器
- [[RVQ]]: 量化 SSL 特征为语义码
- [[BERT]]: 提供文本上下文特征
- [[Cross-Attention]]: MRTE 的核心融合机制
- [[Normalizing Flow]]: VITS 架构中先验-后验变换
- [[DPO]]: GPT 阶段的对齐训练

### 硬件/数据相关

- [[LibriSpeech]]: 外部评测数据集

---

## 速查卡片

> [!summary] GPT-SoVITS
> - **核心**: GPT 自回归语义 token 预测 + VITS 声学解码的两阶段零样本/少样本 TTS
> - **方法**: Phoneme + BERT → GPT Transformer(12L/512d) → Semantic Token → VITS(MRTE + Flow + HiFi-GAN) → Waveform
> - **结果**: 中文社区最流行的开源 TTS（57.8k+ stars），RTF 0.014（4090），学术评测 WER 5.13% / SIM-O 0.405
> - **代码**: [GitHub](https://github.com/RVC-Boss/GPT-SoVITS)

---

*笔记创建时间: 2026-05-25*
