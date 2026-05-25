---
title: "CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models"
method_name: "CosyVoice2"
authors: [Zhihao Du, Yuxuan Wang, Qian Chen, Xian Shi, Xiang Lv, Tianyu Zhao, Zhifu Gao, Yexin Yang, Changfeng Gao, Hui Wang, Fan Yu, Huadai Liu, Zhengyan Sheng, Yue Gu, Chong Deng, Wen Wang, Shiliang Zhang, Zhijie Yan, Jingren Zhou]
year: 2024
venue: arXiv
tags: [tts, streaming-tts, zero-shot-tts, flow-matching, speech-tokenizer, llm-backbone]
image_source: online
arxiv_html: https://arxiv.org/html/2412.10117v3
created: 2026-05-25
---

# 论文笔记：CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | SpeechLab@Tongyi, Alibaba Group |
| 日期 | December 2024 |
| 项目主页 | https://funaudiollm.github.io/cosyvoice2 |
| 对比基线 | [[CosyVoice]], [[F5-TTS]], [[MaskGCT]], [[SeedTTS]], [[FireRedTTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2412.10117) / [Code](https://github.com/FunAudioLLM/CosyVoice) / [ModelScope](https://www.modelscope.cn/models/iic/CosyVoice2-0.5B) |

---

## 一句话总结

> 在 [[CosyVoice]] 基础上，用 [[FSQ]] 替换 VQ 实现 100% 码本利用率、以预训练 [[Qwen2.5]] 为 LM backbone 并统一流式/非流式架构，配合 chunk-aware causal [[Flow Matching]] 实现 150ms 首包延迟且几乎无损的流式 TTS。

---

## 核心贡献

1. **[[FSQ]] 替换 VQ**: 将语音 tokenizer 中的向量量化替换为有限标量量化，码本利用率从 23% 提升至 100%，ASR 错误率下降 40%+
2. **简化的统一 LM 架构**: 移除 text encoder 和 speaker embedding，直接使用预训练 Qwen2.5-0.5B 作为 backbone，同一模型同时支持流式和非流式推理
3. **Chunk-aware causal [[Flow Matching]]**: 设计四种注意力掩码实现统一的流式/非流式 flow matching，流式模式质量几乎无损
4. **增强的指令式生成**: 支持情感、方言、角色扮演、细粒度控制（笑声/呼吸/重读），与 zero-shot 能力统一在一个模型中

---

## 问题背景

### 要解决的问题
现有零样本 TTS 模型大多以非流式（offline）模式运行，导致首包延迟高，无法满足 GPT-4o 等语音对话场景的实时性需求。流式合成在 LM-based TTS 中有探索，但 diffusion/flow matching 类和混合架构缺乏成熟的流式方案。

### 现有方法的局限
- **CosyVoice v1**: 非流式架构延迟高；VQ 码本利用率仅 23%；text encoder + speaker embedding 增加架构复杂度且引起信息泄漏（speaker embedding 中包含语言/副语言信息，影响韵律和跨语言能力）
- **纯 codec LM 方案**（[[VALL-E]] 等）: 依赖 codec vocoder 重建质量受限
- **纯 diffusion/flow 方案**（[[F5-TTS]] 等）: 非自回归方式难以原生支持流式
- **通用问题**: 流式和非流式通常需要两套模型分别维护

### 本文的动机
通过 "progressive semantic decoding"（渐进式语义解码）的设计哲学——分离语义和声学信息、独立建模——同时实现流式低延迟和非流式高质量，在一个统一模型中解决。

---

## 方法详解

### 模型架构

CosyVoice 2 采用**三阶段渐进式语义解码**架构：
- **输入**: 原始文本（BPE tokenization，无需 [[G2P]]）+ 参考语音 prompt
- **阶段 1 — Supervised Speech Tokenizer**: 基于 [[SenseVoice]]-Large 的 ASR encoder + [[FSQ]]，将语音编码为 25 Hz 的离散语义 token
- **阶段 2 — Unified Text-Speech LM**: 以 [[Qwen2.5]]-0.5B 为 backbone 的自回归 LM，预测语义 token 序列
- **阶段 3 — Chunk-aware Causal [[Flow Matching]]**: 条件化于 speaker embedding + 语义 token + masked Mel，生成 50 Hz Mel 频谱
- **Vocoder**: Mel -> 24kHz Waveform

### 核心模块

#### 模块 1: Supervised Speech Tokenizer with [[FSQ]]

**设计动机**: VQ 的码本利用率低（仅 23%），导致语义 token 表达能力不足。[[FSQ]] 将每个维度独立量化到有限整数集合 $[-K, K]$，天然实现 100% 利用率。

**具体实现**:
- 基于 [[SenseVoice]]-Large 的前 6 层 Transformer（带 [[RoPE]]）作为 Encoder_1，产出中间表示 $H$
- [[FSQ]] 模块对 $H$ 做低秩投影 + bounded round 量化
- 量化后经 Encoder_2 和 ASR Decoder 联合训练，确保 token 保留充分语义信息
- 使用 [[Straight-Through Estimator]] 进行梯度近似
- **token 率**: 25 Hz（每秒 25 个 speech token）
- **码本大小**: $(2K+1)^D = 6561$（$K=4, D=4$ 时）

#### 模块 2: Unified Text-Speech LM

**设计动机**: 利用预训练 LLM 的文本理解能力，简化架构的同时提升文本-语音对齐质量。

**相比 CosyVoice v1 的关键简化**:
- **移除 Speaker Embedding**: 实验发现 speaker embedding 包含语言/副语言信息，造成信息泄漏，损害韵律和跨语言合成能力。说话人信息改由 flow matching 阶段通过 [[CAM++]] 提取的 embedding 来建模
- **移除 Text Encoder**: Qwen2.5-0.5B 的 BPE tokenizer + 预训练 Transformer 已足够完成文本理解和文本-语音对齐

**序列构造**:
- **非流式**: `[S] text_tokens [T] speech_tokens [E]`
- **流式**: 文本和语音 token 按 $N{:}M = 5{:}15$ 比例交错排列。模型预测到 "filling token" 时，拼接 $N$ 个文本 token；文本耗尽后接 `[T]` 和剩余语音 token

#### 模块 3: Chunk-aware Causal [[Flow Matching]]

**设计动机**: 让同一个 flow matching 模型在训练时同时学习不同粒度的因果约束，推理时按延迟需求选择注意力掩码。

**具体实现**:
- 声学特征: 50 Hz Mel 频谱（24 kHz 采样率）
- 语义 token 2x 上采样匹配 Mel 帧率
- Look-ahead 1-D 卷积层（右填充，pad 大小 $P$，kernel 大小 $P{+}1$）提供有限的未来上下文
- 使用 causal convolutional Transformer UNet 学习 ODE

**四种注意力掩码**（训练时均匀随机采样）:

| 掩码类型 | 可见范围 | 适用场景 |
|----------|----------|----------|
| Non-causal | 全部帧 | 离线非流式，最高质量 |
| Full-causal | 仅过去帧 | 极低延迟 |
| Chunk-M | 过去 + M 帧未来 | 第一个 chunk，低延迟 |
| Chunk-2M | 过去 + 2M 帧未来 | 后续 chunk，更好质量 |

更大上下文的掩码隐式充当对更小上下文掩码的教师（self-distillation 效果）。

#### 模块 4: Instructed Generation

**设计动机**: 将情感/方言/角色/细粒度控制与 zero-shot 能力统一在同一模型中。

**具体实现**:
- 收集 1500 小时指令式训练数据
- 自然语言指令前缀 + `<|endofprompt|>` 分隔符 + 合成文本
- 支持内联标记: `[laughter]`、`[breath]`、`<laughter>XXX</laughter>`、`<strong>XXX</strong>`
- 覆盖: 8 种情感、语速控制、6 种中文方言、14+ 种角色扮演

#### 模块 5: Multi-Speaker Fine-tuning (mSFT)

- 多说话人同时微调，避免灾难性遗忘
- Speaker prompt 格式: `"Speaker A<|endofprompt|>"`
- 学习率 1e-5
- 最少约 400 条录音即可获得良好效果

#### 模块 6: Reinforcement Learning

**设计动机**: 进一步提升 SFT 模型的发音准确性和说话人相似度。

**两种奖励信号**:
- **DPO**: 基于说话人相似度（SS）构建偏好对
- **可微分 ASR 奖励**: 通过 Gumbel softmax 采样使离散 token 可微分，用冻结的 ASR 后端计算文本重建概率作为奖励

---

## 关键公式

### 公式 1: [[FSQ|有限标量量化]]

$$
\bar{H} = \text{ROUND}(\text{Proj}_{down}(H))
$$

$$
\hat{H} = \text{Proj}_{up}(\bar{H})
$$

**含义**: 将 encoder 的中间表示 $H$ 先投影到低秩空间，每维独立四舍五入到整数，再投影回原空间。

**符号说明**:
- $H \in \mathbb{R}^{L \times d}$: Encoder_1 输出的中间表示
- $\text{Proj}_{down}$: 降维投影，$d \to D$（$D{=}4$）
- $\text{ROUND}(\cdot)$: 逐维四舍五入到 $[-K, K]$ 的整数（$K{=}4$）
- $\text{Proj}_{up}$: 升维投影，$D \to d$

### 公式 2: [[FSQ|Speech Token Index 计算]]

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j} (2K+1)^j
$$

**含义**: 将 $D$ 维量化整数向量映射为一个唯一的码本索引，实现 $(2K{+}1)^D = 6561$ 大小的码本。

**符号说明**:
- $\mu_i$: 第 $i$ 帧的 speech token 索引
- $\bar{h}_{i,j}$: 第 $i$ 帧第 $j$ 维的量化值
- $D$: 低秩维度（4）
- $K$: 每维量化范围的半径（4）

### 公式 3-6: [[Flow Matching|Conditional Flow Matching (CFM)]]

$$
\omega_t(\phi^{OT}_t(X_0, X_1)|X_1) = X_1 - X_0
$$

$$
\phi^{OT}_t(X_0, X_1) = (1-t)X_0 + tX_1
$$

$$
X_0 \sim p_0(X) = \mathcal{N}(0, I)
$$

$$
X_1 \sim q(X)
$$

**含义**: 定义从高斯噪声 $X_0$ 到真实 Mel 频谱 $X_1$ 的最优传输路径。$t \in [0,1]$ 参数化插值轨迹。

**符号说明**:
- $X_0$: 采样自标准高斯的噪声
- $X_1$: 目标 Mel 频谱
- $\phi^{OT}_t$: 最优传输流的时间 $t$ 处的插值
- $\omega_t$: 目标速度场（常量方向 $X_1 - X_0$）

### 公式 7: [[Flow Matching|Flow Matching UNet]]

$$
\nu_t(\phi^{OT}_t(X_0,X_1)|\theta) = \text{UNet}_\theta(\phi^{OT}_t(X_0,X_1), t; \mathbf{v}, \{\mu\}_{1:L}, \tilde{X}_1)
$$

**含义**: 用 causal convolutional Transformer UNet 参数化速度场，条件化于说话人 embedding $\mathbf{v}$、语义 token 序列 $\{\mu\}$ 和 masked Mel 特征 $\tilde{X}_1$。

**符号说明**:
- $\theta$: UNet 参数
- $\mathbf{v}$: [[CAM++]] (3D-Speaker) 提取的说话人 embedding
- $\{\mu\}_{1:L}$: LM 预测的语义 token 序列
- $\tilde{X}_1$: 随机 mask 70%-100% 末尾帧后的 Mel 频谱

### 公式 8: [[Flow Matching|训练损失]]

$$
\mathcal{L} = \mathbb{E}_{t, X_0, X_1} \| \nu_t(\phi^{OT}_t(X_0,X_1)|\theta) - (X_1 - X_0) \|_1
$$

**含义**: 最小化 UNet 预测速度与真实速度之间的 L1 距离。$t \sim \mathcal{U}(0,1)$。

### 公式 9: [[Flow Matching|Cosine 时间调度]]

$$
t := 1 - \cos\left(\frac{1}{2} t \pi\right)
$$

**含义**: 推理时将均匀时间步映射到余弦调度，在 $t$ 接近 0 和 1 时放慢采样步，提升生成质量。

### 公式 10: [[Classifier-Free Guidance|CFG]]

$$
\tilde{\nu}_t = (1+\beta)\cdot\nu_t(\cdot|\theta;\Psi) - \beta\cdot\nu_t(\cdot|\theta)
$$

**含义**: 用引导强度 $\beta{=}0.7$ 放大条件信号 $\Psi$ 的影响，提升生成的忠实度。

**符号说明**:
- $\beta = 0.7$: CFG 引导强度
- $\Psi$: 条件信号（speaker embedding + 语义 token + masked Mel）
- NFE = 10: 推理步数

### 公式 11: [[Streaming TTS|TTS 首包延迟]]

$$
L_{TTS} = M \cdot d_{lm} + M \cdot d_{fm} + M \cdot d_{voc}
$$

**含义**: 首包延迟由 LM、flow matching、vocoder 三部分的逐 token 计算时间乘以 chunk 大小 $M$ 决定。

**符号说明**:
- $M = 15$: 每个 chunk 的 speech token 数
- $d_{lm}, d_{fm}, d_{voc}$: LM / flow matching / vocoder 的单 token 推理时间

### 公式 12: [[Streaming TTS|语音对话端到端延迟]]

$$
L_{Chat} \leq N \cdot d_{llm} + L_{TTS}
$$

**含义**: 语音对话场景的端到端延迟上界，包括 LLM 生成 $N$ 个文本 token 的时间加上 TTS 首包延迟。

### 公式 13: [[DPO|DPO 损失]]

$$
L_{DPO}(\pi_\theta;\pi_{ref}) = -\log\sigma\left(\beta\log\frac{\pi_\theta(\mu^w|y)}{\pi_{ref}(\mu^w|y)} - \beta\log\frac{\pi_\theta(\mu^l|y)}{\pi_{ref}(\mu^l|y)}\right)
$$

**含义**: 基于说话人相似度构建偏好对 $(\mu^w, \mu^l)$，训练 LM 偏好生成与目标说话人更相似的 token 序列。

**符号说明**:
- $\mu^w$: 偏好样本（speaker similarity 更高）
- $\mu^l$: 非偏好样本
- $\pi_\theta$: 当前策略
- $\pi_{ref}$: 参考策略（SFT 模型）

### 公式 14-15: [[Differentiable ASR Reward|可微分 ASR 奖励]]

$$
\hat{H} = \text{Proj}_{up}(\text{GumbelSoftmax}(\text{logits}))
$$

$$
L_{ASR} = -\log P(Y|\hat{H};\theta_{ASR})
$$

**含义**: 通过 Gumbel softmax 使离散 token 采样可微分，用冻结的 ASR 后端重新预测输入文本，最大化文本重建概率。比 DPO 的泛化能力更强。

---

## 关键图表

### Figure 1: Overview / CosyVoice 2 系统概览

![Figure 1: CosyVoice 2 系统概览](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/x1.png)

**说明**: CosyVoice 2 的三阶段架构。(a) 基于 [[SenseVoice]]-Large 的有监督语音 tokenizer，[[FSQ]] 模块插入在 Encoder_1 和 Encoder_2 之间，虚线模块仅训练时使用；(b) 统一的 text-speech LM 支持流式和非流式序列构造；(c) Causal flow matching 模型，以 speaker embedding $\mathbf{v}$、语义 token $\mu$、masked Mel $\tilde{X}$ 和中间状态 $X_t$ 为条件。

### Figure 2: Unified Text-Speech LM / 统一语言模型

![Figure 2: 统一 Text-Speech LM 的流式与非流式序列构造](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/x2.png)

**说明**: 上半部分为流式模式，文本和语音 token 按 $N{:}M = 5{:}15$ 交错排列，模型预测 "filling token" 时拼接下一批文本 token；下半部分为非流式模式，所有文本 token 在前、所有语音 token 在后。两种模式共享同一个 LM。

### Figure 3: Chunk-aware Flow Matching / 块感知流匹配

![Figure 3: 统一的 chunk-aware flow matching 模型](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/x3.png)

**说明**: 展示四种注意力掩码（non-causal / full-causal / chunk-M / chunk-2M）如何在同一模型中共存。训练时均匀采样，推理时按延迟需求选择。Look-ahead 卷积层提供有限的未来信息。

### Figure 4: t-SNE Visualization / FSQ 特征可视化

![Figure 4: VoxCeleb1 上三个说话人的语音表示 t-SNE 可视化](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/img/fsq_vis.png)

**说明**: (a) 量化前的语音表示呈现明显的说话人聚类；(b) 量化后说话人分布几乎不可区分，证明 [[FSQ]] 有效解耦了说话人信息；(c) 码本利用率可视化，所有 token 均被充分使用。

### Figure 5: SID Training Convergence / 说话人识别训练收敛曲线

![Figure 5: 量化前后 token 的 SID 训练收敛曲线](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/img/training_development_values.png)

**说明**: 使用量化后的 token 训练说话人识别模型无法收敛，进一步证实 tokenizer 成功将说话人信息与语义内容分离。

### Figure 6: SFT Results / 多说话人微调结果

![Figure 6: CosyVoice 2 SFT 模型在 SEED 评测下的结果](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/x4.png)

**说明**: 展示多个目标说话人的 CER/WER 和 SS 结果。约 400 条录音即可实现良好的合成效果，大多数说话人继承了 zero-shot 模型的上下文理解能力。

### Table 1: 指令类型与示例

| 类别 | 具体内容 |
|------|----------|
| 情感 | Happy, Sad, Surprised, Angry, Fearful, Disgusted, Calm, Serious |
| 语速 | Fast, Very Fast, Slow, Very Slow |
| 方言 | 粤语, 四川话, 上海话, 郑州话, 长沙话, 天津话 |
| 角色扮演 | 神秘、凶猛、好奇、优雅、孤独、机器人、佩奇等 14+ 种 |
| 声音特效 | `[laughter]`, `[breath]` |
| 细粒度标记 | `<laughter>XXX</laughter>`, `<strong>XXX</strong>` |

**说明**: CosyVoice 2 支持的完整指令式生成类型覆盖情感、语速、方言、角色和细粒度声音控制。

### Table 2: Tokenizer 训练数据

| 语言 | 时长（小时） |
|------|-------------|
| 中文 | 110,884 |
| 英文 | 99,918 |
| **总计** | **~200,000** |

**说明**: Tokenizer 仅在中英数据上训练，但对日语和韩语展示了零样本泛化能力。

### Table 3: CosyVoice 2 训练数据

| 语言 | 时长（小时） |
|------|-------------|
| 中文 | 130,000 |
| 英文 | 30,000 |
| 日文 | 4,600 |
| 韩文 | 2,200 |
| **总计** | **~166,800** |

**说明**: TTS 模型训练数据以中文为主。文本标签由 [[Paraformer]]（中文）和 [[SenseVoice]]（其他语言）生成伪标签，内部强制对齐模型用于质量过滤。

### Table 4: VQ vs. FSQ 对比

| 方法 | 码本大小 | 利用率 | C.V. EN ASR Error | C.V. CN ASR Error | Fluers EN | Fluers CN |
|------|---------|--------|-------------------|-------------------|-----------|-----------|
| VQ | 4,096 | 963 (23%) | 18.26% | 11.56% | 7.65% | 5.03% |
| **FSQ** | **6,561** | **6,561 (100%)** | **10.67%** | **7.29%** | **6.58%** | **4.43%** |

**说明**: [[FSQ]] 在码本利用率和 ASR 错误率上全面大幅领先 VQ。100% 利用率意味着每个 token 都承载独特的语义信息，不存在 dead codes。

### Table 5: LibriSpeech test-clean 基线对比

| Model | WER (%) | NMOS | SS |
|-------|---------|------|-----|
| Human | 2.66 | 3.84 | 0.697 |
| ChatTTS | 6.84 | 3.89 | - |
| [[GPT-SoVITS]] | 5.13 | 3.93 | 0.405 |
| OpenVoice | 3.47 | 3.87 | 0.299 |
| ParlerTTS | 3.16 | 3.86 | - |
| EmotiVoice | 3.14 | 3.93 | - |
| [[CosyVoice]] | 2.89 | 3.93 | 0.743 |
| **CosyVoice 2** | **2.47** | **3.96** | **0.745** |
| **CosyVoice 2-S** (streaming) | **2.45** | **3.90** | **0.751** |

**说明**: CosyVoice 2 在 WER、NMOS、SS 三项指标上均超越 Human recording，流式模式的 WER 甚至略优于非流式（2.45 vs 2.47）。

### Table 6: SEED Test Set 结果

| Model | test-zh CER% | test-zh SS | test-en WER% | test-en SS | test-hard WER% | test-hard SS |
|-------|-------------|-----------|-------------|-----------|---------------|-------------|
| Human | 1.26 | 0.755 (0.775) | 2.14 | 0.734 (0.742) | - | - |
| Vocoder Resyn. | 1.27 | 0.720 | 2.17 | 0.700 | - | - |
| [[SeedTTS]]† | 1.12 | 0.796 | 2.25 | 0.762 | 7.59 | 0.776 |
| [[FireRedTTS]] | 1.51 | 0.635 (0.653) | 3.82 | 0.460 (0.526) | 17.45 | 0.621 (0.639) |
| [[MaskGCT]] | 2.27 | 0.774 (0.752) | 2.62 | 0.714 (0.730) | 10.27 | 0.748 (0.720) |
| E2 TTS† (32 NFE) | 1.97 | 0.730 | 2.19 | 0.710 | - | - |
| [[F5-TTS]] (32 NFE) | 1.56 | 0.741 (0.794) | 1.83 | 0.647 (0.742) | 8.67 | 0.713 (0.762) |
| [[CosyVoice]] | 3.63 | 0.723 (0.775) | 4.29 | 0.609 (0.699) | 11.75 | 0.709 (0.755) |
| **CosyVoice 2** | **1.45** | **0.748 (0.806)** | 2.57 | 0.652 (0.736) | **6.83** | 0.724 (0.776) |
| **CosyVoice 2-S** | 1.45 | 0.753 (0.812) | 2.38 | 0.654 (0.743) | 8.08 | 0.732 (0.785) |

**说明**: 括号外为 WavLM-based SS，括号内为 ERes2Net SS。† 为闭源模型。CosyVoice 2 在 test-zh 上超越所有开源模型，仅次于商用 [[SeedTTS]]；在 test-hard（offline）上取得全场 SOTA。流式模式在典型场景几乎无损。

### Table 7: 逐步消融实验

| Model | test-zh CER% | test-zh SS | test-en WER% | test-en SS | test-hard WER% | test-hard SS |
|-------|-------------|-----------|-------------|-----------|---------------|-------------|
| [[CosyVoice]] | 3.63 | 0.775 | 4.29 | 0.699 | 11.75 | 0.755 |
| + LLM init. | 2.96 | 0.808 | 4.57 | 0.730 | 9.94 | 0.789 |
| + Drop Spk Emb. | 2.56 | 0.804 | 3.81 | 0.740 | 9.66 | 0.778 |
| + FSQ (= CosyVoice 2) | 1.45 | 0.806 | 2.57 | 0.736 | 6.83 | 0.776 |
| + Pitch Loss | 1.19 | 0.802 | 2.40 | 0.728 | 6.29 | 0.769 |

**关键发现**:
- **LLM 初始化**: test-zh CER 相对下降 18.46%，test-hard 下降 15.40%
- **移除 Speaker Embedding**: CER 显著下降而 SS 基本维持，证实内容由 LM 建模、说话人信息由 flow matching 建模的分离策略有效
- **FSQ**: CER 大幅下降（2.56→1.45），SS 不变，源于 100% 码本利用率带来的更强语义表达

### Table 8: 流式模块影响

| Model | LM | FM | test-zh CER% | SS | test-en WER% | SS | test-hard CER% | SS |
|-------|-----|-----|-------------|-----|-------------|-----|---------------|-----|
| M1 | Offline | Offline | 1.45 | 0.806 | 2.57 | 0.736 | 6.83 | 0.776 |
| M2 | Offline | Stream. | 1.46 | 0.811 | 2.60 | 0.743 | 7.12 | 0.788 |
| M3 | Stream. | Offline | 1.38 | 0.806 | 2.51 | 0.737 | 7.88 | 0.773 |
| M4 | Stream. | Stream. | 1.45 | 0.812 | 2.38 | 0.743 | 8.08 | 0.785 |

**关键发现**: 流式 LM 对典型场景影响极小，主要影响 hard cases（因上下文受限）。流式 flow matching 反而略提升 SS（初始 chunk 中 prompt 占比更高）。

### Table 9: 日语和韩语结果

| Model | test-ja CER% | test-ja SS | test-ja NMOS | test-ko CER% | test-ko SS | test-ko NMOS |
|-------|-------------|-----------|-------------|-------------|-----------|-------------|
| CosyVoice 2 | 18.79 | 0.630 | 3.42 | 7.98 | 0.707 | 3.73 |
| CosyVoice 2-S | 21.41 | 0.629 | 3.35 | 9.06 | 0.714 | 3.60 |

**说明**: 日语性能受限于与中文的字符集重叠（汉字在日语上下文中被读为中文发音）。韩语因无字符重叠表现显著更好。

### Table 10: 指令式生成评测

| Model | CER (%) | SS | NMOS | MOS-I |
|-------|---------|-----|------|-------|
| CosyVoice-Instruct | 1.72 | 0.797 | 3.94 | 3.09 |
| **CosyVoice 2** | **1.52** | **0.804** | 3.94 | **4.06** |
| CosyVoice 2 w/o Instruction | 0.97 | 0.817 | 4.02 | 2.28 |

**说明**: 290 样本中文测试集（29 种指令 x 10 文本 x 5 说话人 prompt）。MOS-I 由 10 名中文母语评分员评定。去掉指令后 MOS-I 从 4.06 降至 2.28，说明指令可控性不会隐式地从文本内容中涌现。

### Table 11: 强化学习结果（Speaker E）

| Model | In-house WER% | NMOS | SS | SEED zh% | SEED en% | SEED hard% |
|-------|--------------|------|-----|---------|---------|-----------|
| Ground Truth | 6.00 | 3.87 | 0.697 | 1.26 | 2.14 | - |
| CosyVoice 2 | 5.34 | 3.91 | 0.721 | 1.45 | 2.57 | 6.83 |
| CosyVoice 2-SFT | 7.15 | 3.96 | 0.795 | 1.50 | 4.26 | 7.90 |
| + $L_{ASR}$ | 6.79 | 3.96 | 0.795 | 1.29 | 3.53 | 7.30 |
| + $L_{DPO}$ | 6.83 | 3.96 | 0.792 | 1.43 | 4.02 | 8.31 |
| + $L_{ASR}$ + $L_{DPO}$ | 6.64 | 3.97 | 0.796 | **1.25** | **3.17** | **6.66** |

**关键发现**: 单独 DPO 反而恶化 hard cases（重复词被当作 rejected sample）。可微分 ASR 奖励泛化能力更强。两者结合效果最佳。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 内部 ASR 数据集 + 开源数据集 | ~200K 小时 | 中英双语 | Tokenizer 训练 |
| 内部 TTS 数据集 | ~167K 小时 | 中/英/日/韩四语种 | TTS 模型训练 |
| 指令式数据 | 1,500 小时 | 情感/方言/角色/细粒度控制 | Instruct 训练 |
| [[LibriSpeech]] test-clean | 标准测试集 | 英文有声书 | 基线对比 |
| Seed-TTS-eval (test-zh/en/hard) | ~3,400 样本 | CommonVoice + 困难样本 | 核心评测 |
| test-ja / test-ko | 各 1,000 样本 | CommonVoice | 跨语言评测 |

### 实现细节

- **LM Backbone**: [[Qwen2.5]]-0.5B 预训练参数初始化
- **Speech Token Rate**: 25 Hz
- **Mel 帧率**: 50 Hz（24 kHz 采样率）
- **FSQ 码本大小**: 6,561（$D{=}4, K{=}4$，利用率 100%）
- **流式 N:M 比例**: 5:15
- **Flow Matching**: CFG $\beta{=}0.7$，NFE=10，cosine 时间调度
- **Mel Masking**: 训练时随机 mask 70%-100% 末尾帧
- **Speaker Embedding**: [[CAM++]] (3D-Speaker)
- **SFT 学习率**: 1e-5
- **ASR 评测**: Whisper-large V3（英文）/ [[Paraformer]]（中文）
- **SS 评测**: WavLM-finetuned SV model + ERes2Net

### 可视化结果

项目主页提供了丰富的 demo 音频:
- **Zero-shot in-context**: 中/英/日/韩四种语言各 3-6 组 prompt + 生成对
- **跨语言合成**: 4 个说话人 x 4 种语言 = 16 个音频，展示跨语言音色保持
- **混合语言**: 6 组 v1.0 vs v2.0 对比，展示 code-switch 改善
- **困难样本**: 中英绕口令、罕见字（煢煢孑立、龍行龘龘）、极端重复文本
- **指令式生成**: 角色扮演（12 种）、方言（6 种）、细粒度标记（6 组）、情感/语速（18 组）

---

## 批判性思考

### 优点
1. **工程价值极高**: 统一 streaming/non-streaming 的设计避免了维护两套模型的开发和部署成本，150ms 首包延迟具备商用级实时性
2. **FSQ 设计简洁有效**: 100% 码本利用率是对传统 VQ dead code 问题的根本解决方案，CER 下降 40%+ 的改进非常显著
3. **消融实验充分**: 逐步消融（Table 7）清晰展示每个改进的独立贡献；流式模块消融（Table 8）定量分析了流式降质的来源
4. **开源且可复现**: 代码、模型、评测基准均公开，社区生态已建立
5. **多场景覆盖**: 从 zero-shot ICL 到 instruct generation 到 mSFT 到 RL，形成完整的工业级方案

### 局限性
1. **日语合成质量差**: CER 18.79% 远高于中文（1.45%）和韩文（7.98%），字符集重叠问题未解决，限制了作为通用多语言 TTS 的定位
2. **依赖大规模内部数据**: ~167K 小时训练数据大部分为内部工业数据，社区无法在同等条件下复现
3. **不支持歌唱合成**: 论文明确指出 singing 是局限
4. **指令不能控制音色**: 无法通过文本指令改变声音特征（如 "用低沉的男声"），音色完全依赖 prompt audio
5. **流式模式在困难样本上仍有降质**: test-hard WER 从 6.83% 升至 8.08%（+18.3%），对含大量重复/绕口令的文本仍需改进

### 潜在改进方向
1. 解决 CJK 字符集重叠的日语问题（可能需要 language-aware tokenization 或显式语言标记）
2. 探索更大的 LM backbone（目前仅 0.5B）对质量的提升潜力
3. 将 chunk-aware flow matching 思路推广到其他 NAR 生成任务（如 NAR TTS、audio editing）
4. 扩展歌唱合成和更丰富的音色控制指令

### 可复现性评估
- [x] 代码开源（GitHub + ModelScope + HuggingFace）
- [x] 预训练模型（CosyVoice2-0.5B 公开）
- [ ] 训练细节完整（学习率等有描述，但大规模训练超参数细节不足）
- [ ] 数据集可获取（核心训练数据为内部数据）

---

## 关联笔记

### 基于
- [[CosyVoice]]: 直接前作，CosyVoice 2 在其基础上改进 tokenizer、LM、flow matching 三个模块
- [[Qwen2.5]]: 0.5B 预训练 LLM 用作 backbone
- [[SenseVoice]]: Large 模型的 encoder 作为 speech tokenizer 基础

### 对比
- [[SeedTTS]]: 商用闭源 SOTA，CosyVoice 2 在 test-zh 上接近其性能
- [[F5-TTS]]: 纯 flow matching 方案的代表，CosyVoice 2 在 test-zh CER 上更优（1.45 vs 1.56）
- [[MaskGCT]]: mask-and-predict 范式的代表，CosyVoice 2 在 CER 上显著领先
- [[FireRedTTS]]: 小红书方案，CosyVoice 2 全面领先

### 方法相关
- [[FSQ]]: 核心创新，有限标量量化替代 VQ
- [[Flow Matching]]: 声学建模的核心方法，chunk-aware causal 设计是本文关键贡献
- [[Classifier-Free Guidance]]: flow matching 推理时使用 CFG ($\beta{=}0.7$)
- [[DPO]]: 强化学习阶段使用的偏好优化
- [[CAM++]]: 3D-Speaker 说话人 embedding 提取器
- [[RoPE]]: Encoder_1 使用旋转位置编码
- [[Straight-Through Estimator]]: FSQ 训练时的梯度近似方法

### 硬件/数据相关
- [[LibriSpeech]]: test-clean 评测
- [[Paraformer]]: 中文 ASR 评测工具
- [[UTMOS]]: 自动 MOS 评测（NMOS）

---

## 速查卡片

> [!summary] CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models
> - **核心**: 统一流式/非流式 TTS，FSQ tokenizer + Qwen2.5 backbone + chunk-aware causal flow matching
> - **方法**: FSQ 100% 码本利用率 + 预训练 LLM 简化架构 + 四种注意力掩码统一训练
> - **结果**: LibriSpeech WER 2.47%（超人类）、SEED test-zh CER 1.45%、首包延迟 150ms、流式几乎无损
> - **代码**: https://github.com/FunAudioLLM/CosyVoice

---

*笔记创建时间: 2026-05-25*
