---
title: "YourTTS: Towards Zero-Shot Multi-Speaker TTS and Zero-Shot Voice Conversion for everyone"
method_name: "YourTTS"
authors: [Edresson Casanova, Julian Weber, Christopher Shulby, Arnaldo Candido Junior, Eren Gölge, Moacir Antonelli Ponti]
year: 2022
venue: ICML 2022
tags: [zero-shot-tts, multi-speaker-tts, voice-conversion, multilingual-tts, vits, speaker-adaptation]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2112.02418
created: 2026-05-25
---

# 论文笔记：YourTTS: Towards Zero-Shot Multi-Speaker TTS and Zero-Shot Voice Conversion for everyone

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Universidade de São Paulo (ICMC-USP), Coqui AI, NVIDIA |
| 日期 | December 2021 (ICML 2022) |
| 项目主页 | https://edresson.github.io/YourTTS/ |
| 对比基线 | [[Attentron]], [[SC-GlowTTS]], [[AutoVC]], [[NoiseVC]] |
| 链接 | [arXiv](https://arxiv.org/abs/2112.02418) / [Code](https://github.com/coqui-ai/TTS) / [Checkpoints](https://github.com/Edresson/YourTTS) |

---

## 一句话总结

> 在 [[VITS]] 基础上加入外部 [[Speaker Encoder]] 全局条件和多语言训练，首次实现跨语言零样本多说话人 TTS 与语音转换，且不到 1 分钟微调即可适配新说话人。

---

## 核心贡献

1. **零样本多说话人 TTS SOTA**: 在 [[VCTK]] 上取得英文零样本多说话人 TTS 最佳结果，SECS 甚至超过 Ground Truth
2. **首个多语言零样本 TTS**: 第一个将多语言训练引入零样本多说话人 TTS，仅用单说话人数据即可实现目标语言的零样本能力
3. **极少数据说话人适配**: 不到 1 分钟语音微调即可在不同录制条件/口音的说话人上达到接近 GT 的相似度

---

## 问题背景

### 要解决的问题
传统 TTS 系统只能用训练时见过的说话人声音合成；零样本多说话人 TTS (ZS-TTS) 目标是：**仅凭几秒参考音频，即可用新说话人的声音合成任意文本**，无需对该说话人做专门训练。

### 现有方法的局限
1. 已有 ZS-TTS 方法（[[Attentron]], [[SC-GlowTTS]]）在 seen vs. unseen speaker 之间仍有明显质量差距
2. ZS-TTS 训练需要**大量多说话人数据**，低资源语言很难获取
3. 当测试说话人的声音/录制条件与训练数据差异大时，质量显著退化
4. 此前没有多语言 ZS-TTS 的工作——跨语言零样本合成尚未被探索

### 本文的动机
[[VITS]] 作为端到端 TTS 已展现出色的合成质量；作者认为在 VITS 架构上引入外部 [[Speaker Encoder]] 条件化 + 多语言联合训练，可以同时解决零样本和跨语言两大挑战。通过渐进式多语言训练策略，即使某语言只有单说话人数据，也能获得该语言的零样本能力。

---

## 方法详解

### 模型架构

YourTTS 采用 **[[VITS]] + 外部 Speaker Embedding 全局条件** 架构：

- **输入**: 原始文本字符 + 参考音频的 speaker embedding + 语言 embedding
- **Text Encoder**: 10 层 [[Transformer]] blocks，196 hidden channels（比 VITS 默认更大），输入为 raw text（非 phoneme）
- **Flow-based Decoder**: 4 层 [[Affine Coupling Layer]]，每层含 4 个 [[WaveNet]] 残差块
- **Posterior Encoder**: 16 层 non-causal WaveNet 残差块，输入线性频谱图 → 预测 latent $z$
- **Vocoder**: [[HiFi-GAN]] V1 + VITS 判别器修改，通过 [[VAE]] 连接实现端到端训练
- **Duration Predictor**: [[VITS]] 的 [[Stochastic Duration Predictor]]，支持多样化语音节奏
- **Speaker Encoder**: 外部预训练的 H/ASP 模型（在 [[VoxCeleb]] 2 上训练）

### 核心模块

#### 模块1: 多语言文本输入

**设计动机**: 许多语言缺少高质量开源 [[Grapheme-to-Phoneme]] 转换器

**具体实现**:
- 直接使用**原始文本**作为输入，不经过 phoneme 转换
- 为多语言训练，将 **4 维可训练语言 embedding** 拼接到每个字符 embedding 上
- 这使得模型能直接接受任何书写系统的文本

#### 模块2: Speaker Conditioning（说话人条件化）

**设计动机**: 利用外部 [[Speaker Encoder]] 提取的高质量说话人表示进行全局条件化

**具体实现**:
- Flow-based decoder 的所有 affine coupling layer、posterior encoder、vocoder 均以 speaker embedding 为条件
- 在 WaveNet 残差块中采用**全局条件化**（global conditioning）
- Speaker embedding 与 text encoder 输出求和后送入 duration predictor（通过线性投影层对齐维度）
- Speaker embedding 与 decoder 输出求和后送入 vocoder

#### 模块3: Speaker Consistency Loss (SCL)

**设计动机**: 进一步提升零样本时的说话人相似度

**具体实现**:
- 用预训练 speaker encoder 分别从生成音频和 GT 音频提取 embedding
- 最大化二者的 [[余弦相似度]]

> **重要勘误（Appendix A）**: 由于实现 bug，SCL 的梯度实际**未被传播**。"+SCL" 实验等价于额外训练更多步但无 SCL。Bug 已在 Coqui TTS v0.12.0+ 修复。

#### 模块4: 零样本语音转换 (Zero-Shot VC)

**设计动机**: VITS 的 posterior encoder 不接收 speaker 信息，因此其预测的 latent 分布是 speaker-independent 的

**具体实现**:
- 使用 Posterior Encoder 从源语音提取 speaker-independent latent
- 用 Flow-based decoder + HiFi-GAN Generator 在目标说话人 embedding 条件下重建波形
- 无需额外训练，TTS 模型天然支持 VC

### 训练策略

**渐进式多语言训练**:
1. 先在 [[LJSpeech]]（单说话人英文）上预训练 1M 步
2. Exp 1: 迁移到 [[VCTK]]（109 说话人英文）继续 200K 步
3. Exp 2: 加入葡萄牙语（单说话人）继续 ~140K 步
4. Exp 3: 再加入法语继续 ~140K 步
5. Exp 4: 在 Exp 3 基础上加入 [[LibriTTS]] train-clean-100/360（1151 说话人）
6. 每个实验最后用 SCL 微调 50K 步（$\alpha = 9$）

**多语言平衡**: 使用加权随机采样确保语言平衡的 batch

---

## 关键公式

### 公式1: [[余弦相似度|Speaker Consistency Loss (SCL)]]

$$
L_{SCL} = \frac{-\alpha}{n} \sum_{i=1}^{n} \cos\_sim(\phi(g_i), \phi(h_i))
$$

**含义**: 通过最大化生成音频与 GT 音频的说话人 embedding 余弦相似度，鼓励模型保持说话人特征一致性。

**符号说明**:
- $\phi(\cdot)$: 预训练 speaker encoder 的输出 embedding
- $\cos\_sim$: 余弦相似度函数
- $\alpha$: 正实数，控制 SCL 影响强度（实验中 $\alpha = 9$）
- $n$: batch size
- $g_i$: 第 $i$ 个 ground truth 音频
- $h_i$: 第 $i$ 个生成音频

> **注意**: 由于实现 bug，此损失在论文实验中实际未生效（见勘误）。

---

## 关键图表

### Figure 1(a): Training Procedure / 训练流程

![Figure 1(a): YourTTS Training](https://ar5iv.labs.arxiv.org/html/2112.02418/assets/x1.png)

**说明**: YourTTS 训练流程。Posterior Encoder 接收线性频谱图和 speaker embedding，预测 latent $z$。$z$ 与 speaker embedding 一起送入 [[HiFi-GAN]] vocoder generator 生成波形（训练时从 $z$ 随机采样部分序列以提高效率）。Flow-based decoder 将 $z$ 和 speaker embedding 映射到先验分布 $P_{Z_p}$。[[Monotonic Alignment Search]] (MAS) 对齐 $P_{Z_p}$ 与 text encoder 输出。Stochastic duration predictor 接收 speaker embedding、language embedding 和 MAS 导出的 duration。红色连接表示不传播梯度；`(++)` 表示拼接；虚线连接为可选。

### Figure 1(b): Inference Procedure / 推理流程

![Figure 1(b): YourTTS Inference](https://ar5iv.labs.arxiv.org/html/2112.02418/assets/x2.png)

**说明**: YourTTS 推理流程。不使用 MAS。$P_{Z_p}$ 分布由 text encoder 直接预测。Duration 通过 stochastic duration predictor 的逆变换从随机噪声采样并取整。Latent $z_p$ 从 $P_{Z_p}$ 采样，经 flow-based decoder 逆变换为 $z$，最后由 vocoder generator 合成波形。

### Table 1: 零样本多说话人 TTS 主实验结果

**SECS, MOS 和 Sim-MOS（95% 置信区间）**

| 实验 | VCTK SECS | VCTK MOS | VCTK Sim-MOS | LibriTTS SECS | LibriTTS MOS | LibriTTS Sim-MOS | MLS-PT SECS | MLS-PT MOS | MLS-PT Sim-MOS |
|------|-----------|----------|--------------|---------------|--------------|------------------|-------------|------------|----------------|
| Ground Truth | 0.824 | 4.26±0.04 | 4.19±0.06 | 0.931 | 4.22±0.05 | 4.22±0.06 | 0.902 | 4.61±0.05 | 4.41±0.05 |
| Attentron ZS | 0.731 | 3.86±0.05 | 3.30±0.06 | – | – | – | – | – | – |
| SC-GlowTTS | 0.804 | 3.78±0.07 | 3.99±0.07 | – | – | – | – | – | – |
| Exp 1 (VCTK) | 0.864 | 4.21±0.04 | 4.16±0.05 | 0.754 | 4.25±0.05 | 3.98±0.07 | – | – | – |
| Exp 1 + SCL | 0.861 | 4.20±0.05 | 4.13±0.06 | 0.765 | 4.21±0.04 | 4.05±0.07 | – | – | – |
| Exp 2 (+ PT) | 0.857 | 4.24±0.04 | 4.15±0.06 | 0.762 | 4.22±0.05 | 4.01±0.07 | 0.740 | 3.96±0.08 | 3.02±0.10 |
| Exp 2 + SCL | 0.864 | 4.19±0.05 | 4.17±0.06 | 0.773 | 4.23±0.05 | 4.01±0.07 | 0.745 | 4.09±0.07 | 2.98±0.10 |
| Exp 3 (+ FR) | 0.851 | 4.21±0.04 | 4.10±0.06 | 0.761 | 4.21±0.04 | 4.01±0.05 | 0.761 | 4.01±0.08 | 3.19±0.10 |
| Exp 3 + SCL | 0.855 | 4.22±0.05 | 4.06±0.06 | 0.778 | 4.17±0.05 | 3.98±0.07 | 0.766 | 4.11±0.07 | 3.17±0.10 |
| **Exp 4 + SCL** | 0.843 | 4.23±0.05 | 4.10±0.06 | **0.856** | 4.18±0.05 | 4.07±0.07 | **0.798** | 3.97±0.08 | 3.07±0.10 |

**表格说明**:
- **VCTK**: 所有 YourTTS 实验的 SECS 均超过 GT（0.824），表明模型在 VCTK 上的说话人相似度已达 SOTA。GT 的 SECS 反而较低，因 VCTK 含明显呼吸声干扰 speaker encoder
- **LibriTTS**: Exp 4（~1200 说话人）SECS 最高（0.856），说话人数量对泛化至关重要
- **MLS-PT**: 仅用 1 个男性葡萄牙语说话人训练，仍能产生女性语音，Sim-MOS ~3.19 接近 [[Attentron]] 用 ~100 说话人训练的水平

### Table 2: 零样本语音转换结果

**MOS 和 Sim-MOS（按性别转换方向）**

| 参考/目标语言 | M→M MOS | M→M Sim | M→F MOS | M→F Sim | F→F MOS | F→F Sim | F→M MOS | F→M Sim | All MOS | All Sim |
|---------------|---------|---------|---------|---------|---------|---------|---------|---------|---------|---------|
| en→en | 4.22±0.10 | 4.15±0.12 | 4.14±0.09 | 4.11±0.12 | 4.16±0.12 | 3.96±0.15 | 4.26±0.09 | 4.05±0.11 | 4.20±0.05 | 4.07±0.06 |
| pt→pt | 3.84±0.18 | 3.80±0.15 | 3.46±0.10 | 3.12±0.17 | 3.66±0.20 | 3.35±0.19 | 3.67±0.16 | 3.54±0.16 | 3.64±0.09 | 3.43±0.09 |
| en→pt | 4.17±0.09 | 3.68±0.10 | 4.24±0.08 | 3.54±0.11 | 4.14±0.09 | 3.58±0.12 | 4.12±0.10 | 3.58±0.11 | 4.17±0.04 | 3.59±0.05 |
| pt→en | 3.62±0.16 | 3.80±0.10 | 2.95±0.20 | 3.67±0.11 | 3.51±0.18 | 3.63±0.11 | 3.47±0.18 | 3.57±0.11 | 3.40±0.09 | 3.67±0.05 |

**表格说明**:
- **en→en**: MOS 4.20, Sim-MOS 4.07，大幅超过 [[AutoVC]]（MOS ~3.54, Sim ~1.91）和 [[NoiseVC]]（MOS ~3.38, Sim ~3.05）
- **pt→pt**: 女性说话人 Sim-MOS（3.35）明显低于男性（3.80），因训练中无葡萄牙语女性说话人
- **跨语言**: en→pt 质量（4.17）甚至超过 pt→pt（3.64），说明英文训练数据质量更高

### Table 3: 说话人适配结果（不到 1 分钟微调）

**SECS, MOS 和 Sim-MOS**

| 语言 | 性别 | 数据量 (样本数) | 模式 | SECS | MOS | Sim-MOS |
|------|------|-----------------|------|------|-----|---------|
| EN | M | 61s (15) | GT | 0.875 | 4.17±0.09 | 4.08±0.13 |
| | | | ZS | 0.851 | 4.11±0.07 | 4.04±0.09 |
| | | | **FT** | **0.880** | **4.17±0.07** | **4.08±0.09** |
| EN | F | 44s (11) | GT | 0.894 | 4.25±0.11 | 4.17±0.13 |
| | | | ZS | 0.814 | 4.12±0.08 | 4.11±0.08 |
| | | | **FT** | **0.896** | **4.10±0.08** | **4.17±0.08** |
| PT | M | 31s (7) | GT | 0.880 | 4.76±0.12 | 4.31±0.14 |
| | | | ZS | 0.817 | 4.03±0.11 | 3.35±0.12 |
| | | | **FT** | **0.915** | 3.74±0.12 | **4.19±0.07** |
| PT | F | 20s (5) | GT | 0.873 | 4.62±0.19 | 4.65±0.14 |
| | | | ZS | 0.743 | 3.59±0.13 | 2.77±0.15 |
| | | | **FT** | **0.930** | 3.48±0.13 | **4.43±0.06** |

**表格说明**:
- GT = Ground Truth, ZS = Zero-Shot, FT = Fine-Tuned (1500 步)
- **英文**: 零样本已经很好，微调后 SECS 和 Sim-MOS 达到甚至超过 GT
- **葡萄牙语女性**: 仅 20 秒 5 个样本微调，Sim-MOS 从 2.77 飙升到 4.43（接近 GT 的 4.65），提升巨大
- **质量-相似度权衡**: 数据 < 45 秒时，微调提升相似度但可能牺牲自然度（PT-M MOS 从 4.03→3.74）

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[VCTK]] | 44h, 109 说话人 | 英文多口音, 48KHz | 训练 + 测试 (11 说话人) |
| [[LibriTTS]] train-clean-100/360 | ~460h, 1151 说话人 | 英文有声书 | Exp 4 训练 + 测试 (10 说话人) |
| TTS-Portuguese | ~10h, 1 说话人 | 巴西葡萄牙语, 有环境噪声 | 训练 |
| M-AILABS French | 175h, 5 说话人 | 法文, LibriVox | 训练 |
| [[MLS]] Portuguese | — | 葡萄牙语有声书 | 测试 (10 说话人) |
| [[Common Voice]] | — | 多语种众包 | 说话人适配测试 (4 说话人) |

### 实现细节

- **Speaker Encoder**: H/ASP model, Prototypical Angular + Softmax loss, [[VoxCeleb]] 2 训练
- **评估 Speaker Encoder**: Resemblyzer（与文献可比）
- **优化器**: [[AdamW]] ($\beta_1 = 0.8, \beta_2 = 0.99$, weight decay = 0.01, 初始 LR = 0.0002, 指数衰减 $\gamma = 0.999875$)
- **Batch Size**: 64
- **硬件**: NVIDIA Tesla V100 32GB
- **预处理**: 下采样至 16KHz, VAD (Webrtcvad), 响度归一化至 -27dB (ffmpeg-normalize), 静音移除
- **葡萄牙语去噪**: [[FullSubNet]] 语音增强
- **MOS 评估**: Defined.ai 众包, 276 英文 / 90 葡萄牙语评估者

### 关键实验发现

1. **VCTK SECS 超过 GT**: 所有实验的 SECS 均高于 GT 的 0.824，这是因为 VCTK 录音含明显呼吸声干扰 speaker encoder
2. **加法语改善葡萄牙语**: Exp 2→Exp 3 后葡萄牙语质量和相似度均提升——高质量法语数据降低了低质量葡语数据在平衡 batch 中的比例
3. **SCL 实际未生效**: 由于代码 bug，所有 +SCL 实验等价于额外训练步数
4. **数据量-自然度关系**: ~1 分钟数据可同时保持自然度和提升相似度；< 45 秒时自然度下降

---

## 批判性思考

### 优点
1. **工程完整性高**: 代码开源于 Coqui TTS、checkpoint 公开、有 demo 页面、评测设置详尽（多语种/多性别/众包 MOS）
2. **渐进式训练策略实用**: 每次只学一种语言、从预训练模型迁移的策略，对低资源语言非常友好
3. **VC 作为副产品**: 利用 VITS 架构天然的 speaker-content 解耦，TTS 模型免费获得 VC 能力
4. **诚实的勘误**: Appendix A 主动披露 SCL 的实现 bug，学术态度端正

### 局限性
1. **SCL 失效意味着 speaker similarity 的改善来源不明**: +SCL 实验实际只是多训了步数，论文的 SCL 贡献点需打折扣
2. **葡萄牙语性别偏差严重**: 仅 1 个男性说话人训练，女性合成质量显著差——这在实际部署中是硬伤
3. **Duration Predictor 不稳定**: 论文承认对某些说话人/句子会产生不自然的 duration
4. **评测局限**: SECS 用 Resemblyzer（非 WavLM-TDNN 等更现代 encoder），且 SECS 与主观 Sim-MOS 在葡萄牙语上不一致
5. **无 WER 评测**: 未报告 ASR 反解的可懂度指标，无法判断 raw text 输入是否导致更多发音错误
6. **16KHz 采样率**: 对 2022 年标准尚可接受，但限制了实际合成质量上限

### 潜在改进方向
1. 修复 SCL 后验证其真实效果（已在 Coqui TTS v0.12.0+ 修复）
2. 增加目标语言的多说话人数据或用数据增强缓解性别偏差
3. 引入 phoneme 输入分支或 [[Grapheme-to-Phoneme]] 减少发音错误
4. 升级到更高采样率（24KHz/44.1KHz）和更现代的 vocoder（[[BigVGAN]], [[Vocos]]）

### 可复现性评估
- [x] 代码开源（Coqui TTS）
- [x] 预训练模型公开
- [x] 训练细节完整
- [x] 数据集可获取（VCTK, LibriTTS, M-AILABS 均公开）

---

## 关联笔记

### 基于
- [[VITS]]: 核心架构基础，YourTTS 在其上增加 speaker conditioning 和多语言支持
- [[HiFi-GAN]]: vocoder 组件
- [[SC-GlowTTS]]: 同一作者的前序工作，首个 flow-based ZS-TTS

### 对比
- [[Attentron]]: 细粒度 attention-based ZS-TTS baseline
- [[AutoVC]]: 自编码器方式的零样本 VC baseline
- [[NoiseVC]]: 噪声注入的零样本 VC baseline

### 后续发展
- [[XTTS]]: Coqui 的后续工作，YourTTS 的直接继承者
- [[VALL-E]]: 同期出现的另一条 ZS-TTS 路线（离散 token 语言建模，而非 VITS-based）

### 方法相关
- [[Speaker Encoder]]: 外部说话人编码器（H/ASP model）
- [[Monotonic Alignment Search]]: VITS/Glow-TTS 的对齐算法
- [[Stochastic Duration Predictor]]: VITS 的随机时长预测器
- [[VAE]]: 连接 posterior encoder 与 vocoder 的变分框架
- [[Affine Coupling Layer]]: Flow-based decoder 的核心组件

### 数据相关
- [[VCTK]]: 主要英文训练/测试集
- [[LibriTTS]]: 扩展英文说话人数
- [[VoxCeleb]]: Speaker Encoder 训练数据

---

## 速查卡片

> [!summary] YourTTS (ICML 2022)
> - **核心**: VITS + 外部 speaker encoder + 多语言训练 → 零样本多说话人 TTS & VC
> - **方法**: raw text 输入, H/ASP speaker embedding 全局条件化, 渐进式多语言迁移训练
> - **结果**: VCTK ZS-TTS SECS 0.864 (超 GT), MOS 4.21; 不到 1 分钟微调 Sim-MOS 达 GT 水平
> - **代码**: https://github.com/coqui-ai/TTS

---

*笔记创建时间: 2026-05-25*
