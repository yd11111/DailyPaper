---
title: "FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level Generative Speech Applications"
method_name: "FireRedTTS"
authors: [Hao-Han Guo, Yao Hu, Kun Liu, Fei-Yu Shen, Xu Tang, Yi-Chen Wu, Feng-Long Xie, Kun Xie, Kai-Tuo Xu]
year: 2024
venue: arXiv
tags: [tts, zero-shot-tts, language-model-tts, flow-matching, industry-report, voice-cloning, emotion-tts, streaming-tts]
image_source: online
arxiv_html: https://arxiv.org/html/2409.03283v2
created: 2026-05-25
---

# 论文笔记：FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level Generative Speech Applications

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | 小红书 FireRed Team |
| 日期 | September 2024 (v2: April 2025) |
| 项目主页 | 无公开主页 |
| 对比基线 | [[CosyVoice]] |
| 链接 | [arXiv](https://arxiv.org/abs/2409.03283) |

---

## 一句话总结

> 小红书提出工业级 TTS 框架 FireRedTTS，完整涵盖数据处理 pipeline、语义感知 tokenizer、AR 语言模型、两阶段波形生成器，并通过指令微调实现情感和副语言行为控制。

---

## 核心贡献

1. **完整数据处理 pipeline**: 从 624k 小时原始音频中锻造出 248k 小时高质量标注 TTS 数据，系统性地分享了工业级数据清洗的最佳实践
2. **语义感知语音 tokenizer (SAST)**: 结合 [[HuBERT]] 语义编码器和 [[ECAPA-TDNN]] 声学编码器，将语音压缩为 40ms 帧率、16384 码本的离散 [[Semantic Token|语义 token]]
3. **两阶段 token-to-waveform 生成**: [[Flow Matching]] decoder 产出高质量 [[Mel-Spectrogram]]，以及基于 [[Mel Codec]] 的可流式 decoder 支持实时推理
4. **指令微调的副语言控制**: 通过 50 小时情感数据集实现 4 类情感控制和 13 种副语言行为（笑声、犹豫、强调等）

---

## 问题背景

### 要解决的问题

随着 LLM 驱动的语音交互需求爆发，TTS 面临**个性化**和**多样化**的新挑战：既要能零样本克隆任意说话人，又要在聊天机器人场景中生成带有情感和副语言行为的拟人语音。

### 现有方法的局限

- [[VALL-E]]、TorToiseTTS 等 LM-based TTS 验证了离散 token 建模的有效性，但缺少完整的**工业级数据处理 pipeline** 分享
- 现有系统在 [[Zero-shot TTS]] 场景下对噪声 prompt 的鲁棒性不足
- 流式推理（streaming）与生成质量之间的 trade-off 缺少实用方案
- 副语言行为（犹豫、笑声、强调等）的细粒度控制在已有工作中很少被系统建模

### 本文的动机

作者认为工业级 TTS 需要一个**端到端的 foundation 框架**，覆盖从原始数据清洗到下游应用的全链路。通过将语音压缩为语义 token 再用 [[Autoregressive Model|AR 语言模型]] 建模，可以复用 LLM 的成熟 scaling 经验；两阶段波形生成器则解决了质量与流式的矛盾。

---

## 方法详解

### 整体架构

FireRedTTS 采用 **三阶段级联** 架构：

- **输入**: 文本 $T$ + 参考音频 prompt $P$（提供音色）
- **Stage 1 — TTS Language Model**: text token → [[Semantic Token|语义 token]]（[[Autoregressive Model|AR]] 生成）
- **Stage 2a — Flow-Matching Decoder**: 语义 token → [[Mel-Spectrogram]]（高质量非流式）
- **Stage 2b — Streamable Decoder**: 语义 token → [[Mel Codec]] token → Mel（可流式备选）
- **Stage 3 — Super-Resolution Vocoder**: 16 kHz Mel → 48 kHz waveform（[[BigVGAN]]-V2）

### 数据处理 Pipeline

数据 pipeline 将大规模原始音频转化为高质量标注 TTS 数据，分 5 个步骤：

1. **语音增强**: 先用 [[Source Separation|音乐源分离]] 模型去背景音乐，再用 DeepFilterNet 做降噪
2. **语音切分**: 基于 [[Mel-Spectrogram]] 的 TDNN [[VAD]] 模型检测语音段，相邻段间距 < 1s 则合并，边界各扩展 0.3s，目标长度 2-20s
3. **说话人聚类**: 用 WeSpeaker 提取说话人 embedding，[[Spectral Clustering|谱聚类]] + K-Means 聚类，cos sim > 0.8 的簇迭代合并；过滤多说话人段和离群样本
4. **转写**: 两遍 [[Transducer]]-based 端到端 [[ASR]] 模型做批量转写
5. **数据过滤**: 三重过滤——[[DNSMOS]] P.835 OVRL > 3.3（语音质量）；roll-off 频率 > 7 kHz（排除伪高采样率）；[[ASR]] 置信度 > 0.8（转写质量）

最终从 **624k 小时** 原始音频中得到 **248k 小时** 标注数据，训练使用 **150k 小时**（110k 中文 + 40k 英文）。

### 核心模块

#### 模块 1: 语义感知语音 Tokenizer (SAST)

**设计动机**: 将语音压缩为承载语言内容的离散 [[Semantic Token|语义 token]]，同时用独立的声学编码器捕捉音色/风格等时不变特征，实现内容与音色的解耦。

**具体实现**:

- **Semantic Encoder**: 预训练 [[HuBERT]] 提取语义 embedding → ResNet 编码器下采样 → [[Vector Quantization|VQ]] 离散化
  - 帧率: **40 ms**
  - 码本大小: **16,384 codewords**
- **Acoustic Encoder**: [[ECAPA-TDNN]] 架构提取全局 utterance-level embedding（音色、风格、声学环境）
  - 使用 **Clip&Shuffle** 预处理：随机截取 25%-75% 的 [[Mel-Spectrogram]]，切成 1s 片段并随机打乱，防止内容泄漏到声学 embedding
- **Decoder**: 全局 embedding 复制后与量化序列相加 → ResNet + 转置卷积上采样 → 同时重建 SSL 特征和 Mel 特征

#### 模块 2: TTS Language Model

**设计动机**: 将 TTS 建模为条件 next-token prediction 任务，复用 LLM 的成熟架构和训练范式。

**具体实现**:

- **文本编码**: [[BPE]]-based text tokenizer（复用 [[Whisper]] tokenizer）
- **说话人条件**: SAST 声学编码器从 prompt 音频提取 utterance-level embedding
- **架构**: **30 层 decoder-only [[Transformer]]**，特征维度 1024，**400M 参数**
- **训练**: 将 text token embedding、speaker embedding、semantic token embedding 拼接为序列，用标准 [[Cross Entropy|交叉熵]] 训练
- **推理**: 给定 text + prompt speaker embedding，[[Autoregressive Model|自回归]] 生成语义 token 序列

#### 模块 3: Flow-Matching Decoder

**设计动机**: 利用 [[Flow Matching]] 从噪声中生成高质量 [[Mel-Spectrogram]]，比 [[DDPM|扩散]] 模型收敛更快、采样更高效。

**具体实现**:

- 语义 token 经 [[Conformer]] 编码并上采样到 Mel 帧长度，作为条件 $\Psi$
- U-Net 估计器接受采样点 $x_t$、时间步 $t$、条件 $\Psi$ → 预测速度场 $v_t^{pred}$
- 每个 self-attention 层后增加 **cross-attention 层**，从参考音频 Mel 中提取音色信息
- 推理时使用 [[Classifier-Free Guidance|CFG]]，$\alpha = 0.7$

#### 模块 4: Streamable Decoder

**设计动机**: [[Flow Matching]] decoder 需要迭代采样不适合流式推理；Streamable Decoder 用 [[Mel Codec]] + multi-stream LM 实现实时生成。

**具体实现**:

- **Mel Codec**: 基于 CNN-GAN 的可流式编码器，将 10ms 帧率的 Mel 压缩为 4 流离散序列（20ms 帧率），每个码本 16,384 codewords
- **Multi-stream LM**: 采用 "lookahead pattern"——语义 token 序列上采样后与声学 token 序列对齐（带 delay $d$），输入为 $s_{i-d} + a_i$
- 语义 token 到达即可开始生成声学 token，实现**同步流式**

#### 模块 5: Super-Resolution Vocoder

- 使用 294 小时高采样率数据训练
- [[BigVGAN]]-V2 架构，上采样因子 480（16 kHz Mel → 48 kHz waveform）

---

## 关键公式

### 公式 1: [[Vector Quantization|SAST 训练损失]]

$$
\mathcal{L}_c = \lambda_{vq} \cdot \mathcal{L}_{vq} + \lambda_s \cdot \mathcal{L}_s + \lambda_a \cdot \mathcal{L}_a
$$

**含义**: SAST tokenizer 的联合训练损失，同时优化量化质量、语义重建和声学重建。

**符号说明**:
- $\mathcal{L}_{vq}$: VQ 损失——量化前后 embedding 的 L2 距离
- $\mathcal{L}_s$: 语义重建损失——SSL 特征与重建 SSL 特征的 L2 距离
- $\mathcal{L}_a$: 声学重建损失——原始 Mel 与重建 Mel 的 L2 距离
- $\lambda_{vq} = 1, \lambda_s = 1000, \lambda_a = 1$

### 公式 2: [[Flow Matching|ODE 变换]]

$$
\frac{d}{dt}\phi_t(x) = v_t(\phi_t(x))
$$

**含义**: Flow Matching 的核心 ODE——定义了从噪声分布到数据分布的连续变换路径。

**符号说明**:
- $\phi_t(x)$: 时间 $t$ 处的流映射
- $v_t(\cdot)$: 时间 $t$ 处的速度场

### 公式 3: [[Optimal Transport|OT 路径]]

$$
\phi_t^{OT}(x_0, x_1) = (1-(1-\sigma)t)x_0 + tx_1
$$

**含义**: 最优传输条件路径，定义了从噪声 $x_0$ 到数据 $x_1$ 的线性插值。

**符号说明**:
- $x_0$: 噪声样本
- $x_1$: 目标数据样本
- $\sigma$: 小常数（接近 0）
- $t \in [0, 1]$: 时间步

### 公式 4: [[Conditional Flow Matching|OT 速度场]]

$$
v_t^{OT} = x_1 - (1-\sigma)x_0
$$

**含义**: 最优传输路径对应的目标速度场，为恒定方向。

**符号说明**:
- $v_t^{OT}$: 时间无关的目标速度场
- $x_0, x_1$: 噪声和数据样本

### 公式 5: [[Conditional Flow Matching|速度场预测]]

$$
v_t^{pred} = NN_\theta(x_t, t, \Psi)
$$

**含义**: 神经网络以当前采样点、时间步和条件信息为输入，预测速度场。

**符号说明**:
- $NN_\theta$: U-Net 参数化的估计网络
- $x_t$: 时间步 $t$ 处的采样点
- $\Psi$: 来自语义 token 的 [[Conformer]] 编码条件

### 公式 6: [[Flow Matching|Flow Matching 训练目标]]

$$
\mathcal{L}_{fm} = \|v_t^{pred} - v_t^{OT}\|^2
$$

**含义**: 最小化预测速度场与 OT 目标速度场之间的 L2 距离。

**符号说明**:
- $v_t^{pred}$: 网络预测的速度场
- $v_t^{OT}$: 最优传输目标速度场

### 公式 7: [[Classifier-Free Guidance|CFG 推理]]

$$
v_t^{cfg} = (1+\alpha) \cdot NN_\theta(x_t, t, \Psi) - \alpha \cdot NN_\theta(x_t, t)
$$

**含义**: 推理时用无条件预测作为负向引导，增强条件生成的一致性。

**符号说明**:
- $\alpha = 0.7$: 引导强度
- $NN_\theta(x_t, t, \Psi)$: 有条件预测
- $NN_\theta(x_t, t)$: 无条件预测（训练时 20% 概率 drop 条件）

---

## 关键图表

### Figure 1: Data Processing Pipeline / 数据处理流水线

![Figure 1: 数据处理 pipeline 总览](https://arxiv.org/html/2409.03283v2/extracted/6353382/image/pipeline_overview.png)

**说明**: FireRedTTS 的五阶段数据处理流水线。从原始音频出发，依次经过语音增强（去音乐+降噪）、语音切分（[[VAD]]）、说话人聚类（WeSpeaker + 谱聚类）、ASR 转写、多维数据过滤（[[DNSMOS]] > 3.3, roll-off > 7kHz, ASR 置信度 > 0.8）。

### Figure 2: Data Statistics / 各阶段数据量

![Figure 2: 各处理步骤后的数据量变化](https://arxiv.org/html/2409.03283v2/x1.png)

**说明**: 展示从 624k 小时原始音频到最终 ~248k 小时标注数据的逐步过滤过程。语音增强和切分保留了大部分数据，质量过滤阶段是主要的数据缩减环节。

### Figure 3: System Overview / 系统总览

![Figure 3: FireRedTTS 基础系统总览](https://arxiv.org/html/2409.03283v2/extracted/6353382/image/fireredtts.png)

**说明**: FireRedTTS 基础系统的四个核心组件：(a) 语义感知语音 tokenizer 将语音编码为离散语义 token；(b) TTS 语言模型将 text token 映射为语义 token；(c) [[Flow Matching]] decoder 将语义 token 转化为高质量 [[Mel-Spectrogram]]；(d) Streamable decoder 基于 [[Mel Codec]] 实现可流式的 token-to-Mel 生成。最终由 [[BigVGAN]] 超分辨率声码器输出 48 kHz 波形。

### Figure 4: Instruction Tuning / 指令微调

![Figure 4: 指令微调实现拟人语音生成](https://arxiv.org/html/2409.03283v2/extracted/6353382/image/spon.png)

**说明**: 展示了情感控制和副语言行为的两种注入方式。情感通过独立 embedding 层注入 prompting 序列；副语言行为分为 **token 插入**（笑声、呼吸等声学事件）和 **embedding 注入**（边笑边说、强调等重叠标签），共支持 13 种行为。

### Figure 5: Emotion Confusion Matrix / 情感混淆矩阵

![Figure 5: 合成情感语音的人工评价混淆矩阵](https://arxiv.org/html/2409.03283v2/extracted/6353382/image/emo_confusion.png)

**说明**: 人工听辨指令微调后的情感语音，大部分样本能正确传达目标情感。少量情感强度较低的样本被误判为 neutral，说明模型在低强度情感上的控制力有待提升。

### Table 1: 副语言行为列表

| 行为 | 文本输入格式 | 标签类型 |
|------|-------------|---------|
| 字重复 | 我 [hic] 我 | token insertion |
| 词重复 | 就是 [rep] 就是 | token insertion |
| 拖长 | 就是 [elong] | token insertion |
| 吸气声 | [sss] | token insertion |
| 弹舌声 | [tsk] | token insertion |
| 呼吸 | [breath] | token insertion |
| 笑声 | [laugh] | token insertion |
| 边笑边说 | 真是笑\^死\^我\^了\^ | embedding injection |
| 强调 | 你真@棒@ | embedding injection |
| 填充停顿 | 嗯P | embedding injection |
| 确认语气 | 啊C | embedding injection |
| 恍然语气 | 哦R | embedding injection |
| 惊讶语气 | 哦S | embedding injection |

**说明**: 13 种副语言行为通过两种机制注入——离散声学事件用特殊 token 标记，与语音重叠的连续行为用 embedding 注入。

### Table 2: 一致性评价 (CoMOS)

| System | CoMOS (↑) |
|--------|-----------|
| GroundTruth | 4.53 |
| CosyVoice | 4.15 |
| **FireRedTTS** | **4.32** |

**说明**: FireRedTTS 在一致性 [[MOS]] 上显著超越 [[CosyVoice]]（4.32 vs 4.15），接近真实音频水平。测试集为 94 对中文 ⟨文本, 音频⟩ 样本，覆盖多种情感和说话风格。

### Table 3: 稳定性评价（句级发音错误率 %）

| System | Overall ZH | Overall EN | Overall MIX | Sub ZH | Sub EN | Sub MIX | Ins&Del ZH | Ins&Del EN | Ins&Del MIX |
|--------|-----------|-----------|------------|--------|--------|---------|-----------|-----------|------------|
| CosyVoice | 5.68 | 12.17 | 29.50 | 3.76 | 6.67 | 25.00 | 1.92 | 5.50 | 4.50 |
| **FireRedTTS** | **2.09** | 12.00 | **8.50** | **1.00** | **0.50** | **4.50** | 1.09 | 11.50 | 4.00 |

**说明**: FireRedTTS 中文整体错误率 2.09%（CosyVoice 5.68%），中英混合 8.50%（CosyVoice 29.50%），大幅领先。但英文 Ins&Del 错误率 11.50% 高于 CosyVoice 的 5.50%，作者归因于英文训练数据多样性不足。

### Table 4: Flow-Matching vs Streamable Decoder (CoMOS)

| System | CoMOS (↑) |
|--------|-----------|
| GroundTruth | 4.52 |
| Flow-matching decoder | 4.48 |
| Streamable decoder | 4.41 |

**说明**: Flow-matching decoder 几乎达到真实音频质量（4.48 vs 4.52），Streamable decoder 略有下降（4.41）但仍是可行的流式替代方案。

### Table 5: 语音克隆结果

| 场景 | 方式 | MOS (↑) | SIM % (↑) |
|------|------|---------|-----------|
| UGC | zero-shot | 4.25 | 73.61 |
| UGC | few-shot 2min | 4.31 | 73.85 |
| PUGC | zero-shot | 3.77 | 68.63 |
| PUGC | **few-shot 1h** | **4.65** | **78.92** |

**说明**: UGC 场景下 zero-shot 和 few-shot 差异不大（MOS 4.25→4.31）；但在 PUGC 专业声优场景下，1 小时数据的 few-shot 微调带来巨大提升（MOS 3.77→4.65, SIM 68.63→78.92%），说明对于高表现力的专业声音，fine-tune 仍是刚需。

### Table 6: Prompt Enhancement 效果

| SNR (dB) | Sim MOS w/o | Sim MOS w/ | SIM% w/o | SIM% w/ |
|----------|-------------|------------|----------|---------|
| 20 | 3.97 | 3.89 | 50.66 | 49.20 |
| 10 | 3.37 | 3.53 | 41.81 | 43.60 |
| 0 | 2.62 | 3.01 | 28.81 | 35.15 |

**说明**: 高信噪比（20dB）下增强反而略损（MOS 3.97→3.89），但低信噪比（0dB）下增强效果显著（SIM 28.81→35.15%）。建议仅对低 SNR prompt 选择性应用增强。

### Table 7: 情感识别准确率

| 模型 | Neutral | Happy | Sad | Angry |
|------|---------|-------|-----|-------|
| Pre-trained | 50% | 45% | 76% | 87% |
| **Fine-tuned** | **97%** | **97%** | **100%** | **98%** |

**说明**: 指令微调后情感控制力大幅提升——neutral 和 happy 从 50%/45% 跃升至 97%，sad 达到 100%。这说明基础模型已有一定情感建模能力，但缺少精确控制力，instruction tuning 是必要的。

### Table 8: 副语言行为偏好测试

| w/o 副语言 | 无差异 | w/ 副语言 |
|-----------|--------|----------|
| 26% | 29% | **45%** |

**说明**: 加入副语言行为后，45% 的听众更偏好有副语言的版本，仅 26% 偏好无副语言版本，确认了副语言行为对拟人语音的重要性。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 内部原始音频 | 624k hours | 大规模未标注 | 数据 pipeline 输入 |
| 清洗后标注数据 | 248k hours | 高质量标注 | 可用训练池 |
| 训练子集 | 150k hours (110k ZH + 40k EN) | 中英双语 | 基础模型训练 |
| 高采样率子集 | 294 hours | 高采样率 | Super-resolution vocoder 训练 |
| 情感/副语言数据 | 50 hours | 富情感标注 | 指令微调 |
| PUGC 微调数据 | 1 hour | 单一专业声优 | Few-shot voice cloning |
| UGC 微调数据 | 2 min | 非录音棚 | Few-shot voice cloning |

### 实现细节

- **TTS Language Model**: 30 层 decoder-only [[Transformer]]，400M 参数，特征维度 1024
- **SAST**: batch size 6400s，300k iterations，$\lambda_{vq}=1, \lambda_s=1000, \lambda_a=1$
- **语义 token**: 40ms 帧率，16,384 codewords
- **Mel Codec**: 4 流，20ms 帧率，每流 16,384 codewords
- **Super-resolution Vocoder**: [[BigVGAN]]-V2，上采样因子 480，16 kHz Mel → 48 kHz waveform
- **CFG**: 训练时 20% 概率 drop 条件，推理 $\alpha = 0.7$
- **Text Tokenizer**: [[BPE]]，复用 [[Whisper]] tokenizer
- **Speaker Similarity**: 使用 [[3D-Speaker]] 预训练说话人验证模型

### 可视化结果

- 情感混淆矩阵（Figure 5）显示微调后模型在 4 类情感上均达到 97%+ 识别率
- 数据量统计图（Figure 2）展示了各处理阶段的数据保留率

---

## 批判性思考

### 优点
1. **完整的工业 pipeline 分享**: 从原始数据到下游应用的全链路经验在学术界少见，对工业实践者有极高参考价值
2. **数据规模与清洗策略**: 624k→248k 小时的清洗比例（~40%）以及 roll-off 频率过滤伪高采样率的技巧值得借鉴
3. **两条解码路径的实用设计**: Flow-Matching（高质量非流式）和 Streamable（流式）两条路径覆盖不同延迟要求的场景
4. **13 种副语言行为的细粒度控制**: 在拟人语音生成上的系统性建模在同期工作中较为少见
5. **Prompt Enhancement 的务实分析**: 诚实地报告了高 SNR 下增强反而有害的结论，提出选择性应用策略

### 局限性
1. **英文和中英混合稳定性不足**: 英文 Ins&Del 错误率 11.50%，远高于 CosyVoice 的 5.50%，训练数据偏向中文
2. **评测缺乏通用基线对比**: 仅与 [[CosyVoice]] 对比，未报告与 [[VALL-E]]、[[F5-TTS]]、[[NaturalSpeech 2]] 等主流基线的数据
3. **未报告推理延迟/[[RTF]]**: 虽然提出了 Streamable Decoder，但未给出首包延迟和 RTF 等关键指标
4. **情感测试集有限**: 仅测试 4 类基本情感，未涵盖更细粒度的情感维度（如 arousal-valence 连续空间）
5. **模型和数据均未开源**: 完全闭源，可复现性低

### 潜在改进方向
1. 增加英文训练数据多样性，改善 code-switch 和英文稳定性
2. 引入更先进的 Mel Codec（如 [[DAC]] 或 [[WavTokenizer]] 的思路）提升 Streamable Decoder 质量
3. 引入 [[Speech DPO]] 或 RLHF 进一步优化主观质量
4. 系统报告推理延迟指标，特别是流式场景

### 可复现性评估
- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（架构参数、超参数基本齐全）
- [ ] 数据集可获取（内部数据）

---

## 关联笔记

### 基于
- [[HuBERT]]: 语义编码器的预训练基础
- [[ECAPA-TDNN]]: 声学编码器架构
- [[VALL-E]]: LM-based TTS 范式的开创
- [[BASE-TTS]]: 工业级 LM TTS 的前置参考
- [[Seed-TTS]]: 同期大规模 TTS 系统

### 对比
- [[CosyVoice]]: 主要对比基线，同样使用语义 token + flow matching，FireRedTTS 在中文上全面领先
- [[FireRedTTS-2]]: 后续改进版本

### 方法相关
- [[Flow Matching]]: 核心生成范式
- [[Conditional Flow Matching]]: Flow-Matching Decoder 的理论基础
- [[Classifier-Free Guidance]]: 推理时的引导策略
- [[Vector Quantization]]: SAST tokenizer 的离散化方法
- [[BigVGAN]]: Super-resolution vocoder
- [[Conformer]]: Flow-Matching Decoder 中的语义 token 编码器
- [[Semantic Token]]: 语音的离散语义表示
- [[Mel Codec]]: Streamable Decoder 的核心组件
- [[BPE]]: 文本 tokenizer

### 硬件/数据相关
- [[DNSMOS]]: 数据过滤的语音质量指标
- [[3D-Speaker]]: 说话人相似度评测模型

---

## 速查卡片

> [!summary] FireRedTTS
> - **核心**: 小红书工业级 TTS 框架，完整覆盖数据清洗→语义 token→波形生成→下游应用
> - **方法**: HuBERT semantic tokenizer + 400M AR LM + Flow Matching / Streamable 双解码 + BigVGAN 超分
> - **结果**: CoMOS 4.32（超 CosyVoice 4.15），中文错误率 2.09%，few-shot PUGC MOS 4.65
> - **代码**: 未开源

---

*笔记创建时间: 2026-05-25*
