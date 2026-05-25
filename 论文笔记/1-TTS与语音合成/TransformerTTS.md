---
title: "Neural Speech Synthesis with Transformer Network"
method_name: "TransformerTTS"
authors: [Naihan Li, Shujie Liu, Yanqing Liu, Sheng Zhao, Ming Liu, Ming Zhou]
year: 2018
venue: AAAI 2019
tags: [tts, transformer, attention, parallel-training, mel-spectrogram, end-to-end, prosody]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/1809.08895
created: 2026-05-25
---

# 论文笔记：Neural Speech Synthesis with Transformer Network

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | University of Electronic Science and Technology of China, Microsoft Research Asia, Microsoft STC Asia, CETC Big Data Research Institute |
| 日期 | September 2018 (AAAI 2019) |
| 项目主页 | 无 |
| 对比基线 | [[Tacotron 2]] |
| 链接 | [arXiv](https://arxiv.org/abs/1809.08895) |

---

## 一句话总结

> 用 [[Transformer]] 的 [[Self-Attention|多头自注意力]] 替换 [[Tacotron 2]] 中的 RNN，实现 4.25 倍训练加速并达到接近人类录音的合成质量（[[MOS]] 4.39 vs 4.44）。

---

## 核心贡献

1. **Transformer 架构引入 TTS**: 首次将 [[Transformer]] 的多头注意力机制完整适配到端到端 TTS 系统，替代 [[Tacotron 2]] 中的 RNN 结构
2. **Scaled Positional Encoding**: 提出可训练缩放因子 $\alpha$ 的位置编码方案，解决文本与 [[Mel-Spectrogram]] 两个域尺度不匹配的问题
3. **Pre-net 输出重中心化**: 在 encoder/decoder pre-net 后加线性投影层，使输出中心化到原点附近，与位置编码的值域 $[-1,1]$ 匹配

---

## 问题背景

### 要解决的问题

端到端神经 TTS 模型（如 [[Tacotron 2]]）使用 RNN（LSTM/GRU）作为 encoder 和 decoder 的核心模块，存在两大问题：
1. **训练效率低**：RNN 要求按序列顺序逐步计算隐藏状态，无法并行化
2. **长程依赖建模困难**：长句子中的全局语义信息经过多步递归处理后会产生偏差，影响韵律自然度

### 现有方法的局限

- [[Tacotron 2]] 使用双向 LSTM encoder + 2 层 LSTM decoder + location-sensitive attention
- RNN 的序列依赖使得第 $t$ 步必须等第 $t-1$ 步完成才能计算，严重限制了 GPU 并行利用率
- 韵律（prosody）依赖句子级语义，RNN 传播路径长度为 $O(T)$，远距离信息衰减严重

### 本文的动机

[[Transformer]] 在 NMT 中已证明可以完全抛弃 RNN，通过 [[Self-Attention]] 将任意两个时间步之间的路径缩短到 $O(1)$，既实现并行训练又改善长程依赖建模。作者认为这两个优势同样适用于 TTS 场景。

---

## 方法详解

### 模型架构

TransformerTTS 采用 **encoder-decoder + 后处理** 架构：

- **输入**: [[Phoneme|音素]] 序列（经 [[G2P|文本转音素]] 转换）
- **Encoder Pre-net**: 3 层 CNN + 线性投影（重中心化）
- **Encoder**: [[Transformer]] encoder（多层 [[Self-Attention]] + FFN）
- **Decoder Pre-net**: 2 层 FC (256 units) + ReLU + 线性投影
- **Decoder**: [[Transformer]] decoder（masked self-attention + cross-attention + FFN）
- **输出**: [[Mel-Spectrogram]] 帧 + stop token
- **Post-net**: 5 层 CNN 残差网络精修 Mel
- **Vocoder**: [[WaveNet]] 生成最终波形

### 核心模块

#### 模块 1: Scaled Positional Encoding

**设计动机**: [[Transformer]] 缺乏序列位置信息，需要注入 [[Positional Encoding|位置编码]]。但在 NMT 中源和目标都是文本嵌入（尺度相近），而 TTS 中源端是 [[Phoneme]] 嵌入、目标端是 [[Mel-Spectrogram]]，两者经 pre-net 后尺度差异大。固定位置编码会过度约束 pre-net 的学习。

**具体实现**:
- 保留标准三角函数位置编码 $PE(pos, 2i)$ 和 $PE(pos, 2i+1)$
- 引入可训练标量 $\alpha$ 作为缩放因子
- 最终输入 = pre-net 输出 + $\alpha \cdot PE$
- encoder 和 decoder 各自独立学习 $\alpha$，实验发现 decoder 的 $\alpha$ 收敛值更小（因为 mel 子空间更紧凑）

#### 模块 2: Pre-net 输出重中心化

**设计动机**: Encoder pre-net 最后一层 ReLU 输出在 $[0, +\infty)$，而位置编码在 $[-1, 1]$。将以 0 为中心的位置信息加到非负值上会导致波动不以原点为中心，影响注意力对齐的学习。

**具体实现**:
- Encoder pre-net: 3 层 CNN + BatchNorm + ReLU + dropout 后，额外加一个线性投影层（无 bias 限制），将输出重中心化
- Decoder pre-net: 2 层 FC (256 units) + ReLU 后，同样加线性投影
- 消融实验显示该操作贡献 MOS 提升 0.04（4.32→4.36）

#### 模块 3: Stop Token 正样本加权

**设计动机**: 每个序列只有最后 1 帧是 stop 正样本，其余数百帧都是负样本，极端类别不平衡。

**具体实现**:
- 在 binary cross entropy 中对正样本施加权重 5.0~8.0
- 保证模型能学到正确的停止时机

---

## 关键公式

### 公式 1: [[Encoder-Decoder Transformer|Seq2Seq 编码]]

$$
h_t = \text{encoder}(h_{t-1}, x_t)
$$

**含义**: Encoder 将输入序列逐步编码为隐藏状态（RNN 范式，本文替换为 Transformer 并行编码）

**符号说明**:
- $h_t$: 第 $t$ 步的 encoder 隐藏状态
- $x_t$: 第 $t$ 步的输入

### 公式 2: [[Cross-Attention|Decoder 解码]]

$$
s_t = \text{decoder}(s_{t-1}, y_{t-1}, c_t)
$$

**含义**: Decoder 结合前一步状态、前一步输出和上下文向量生成当前步状态

**符号说明**:
- $s_t$: 第 $t$ 步的 decoder 隐藏状态
- $y_{t-1}$: 前一步的输出（训练时为 ground truth，推理时为模型预测）
- $c_t$: 注意力上下文向量

### 公式 3: [[Attention Alignment|注意力上下文]]

$$
c_t = \text{attention}(s_{t-1}, \mathbf{h})
$$

**含义**: 用 decoder 前一步状态查询所有 encoder 隐藏状态，加权求和得到上下文向量

**符号说明**:
- $\mathbf{h}$: encoder 隐藏状态序列
- $c_t$: 第 $t$ 步的上下文向量

### 公式 4: [[Autoregressive Model|自回归生成概率]]

$$
p(y_1, \ldots, y_{T'} | x_1, \ldots, x_T) = \prod_{t=1}^{T'} p(y_t | \mathbf{y}_{<t}, \mathbf{x})
$$

**含义**: 输出序列的联合概率分解为逐步条件概率的乘积

**符号说明**:
- $T$: 输入序列长度
- $T'$: 输出序列长度（$T \neq T'$）
- $\mathbf{y}_{<t}$: 已生成的所有输出

### 公式 5: [[Mel-Spectrogram|输出投影]]

$$
p(y_t | \mathbf{y}_{<t}, \mathbf{x}) = f(s_t)
$$

**含义**: TTS 中不需要 softmax，decoder 隐藏状态通过线性投影直接输出 mel 帧

**符号说明**:
- $f(\cdot)$: 全连接线性层

### 公式 6-7: [[Positional Encoding|三角函数位置编码]]

$$
PE(pos, 2i) = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

$$
PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)
$$

**含义**: 用正弦/余弦函数为每个位置生成唯一的位置向量，使模型获得序列顺序信息

**符号说明**:
- $pos$: 时间步索引
- $2i, 2i+1$: 向量维度通道索引
- $d_{model}$: 模型隐藏维度

### 公式 8: [[Positional Encoding|Scaled Positional Encoding]]

$$
x_i = \text{prenet}(\text{phoneme}_i) + \alpha \cdot PE(i)
$$

**含义**: 本文核心创新——引入可训练缩放因子 $\alpha$ 调节位置编码幅度，适配 TTS 中文本域与声学域的尺度差异

**符号说明**:
- $\alpha$: 可训练标量，encoder 和 decoder 各自独立学习
- $\text{prenet}(\cdot)$: 预处理网络输出
- $PE(i)$: 第 $i$ 个位置的位置编码

---

## 关键图表

### Figure 1: Tacotron2 System Architecture / Tacotron2 系统架构

![Figure 1](https://ar5iv.labs.arxiv.org/html/1809.08895/assets/Tacotron2.jpg)

**说明**: [[Tacotron 2]] 的基线架构。输入文本经 character/phoneme embedding → 3 层 CNN → 双向 LSTM encoder；decoder 端使用 2 层 FC pre-net + 2 层 LSTM + location-sensitive attention；输出 [[Mel-Spectrogram]] 经 5 层 CNN post-net 精修。本文的 TransformerTTS 正是将其中所有 RNN 模块替换为 [[Transformer]] 组件。

### Figure 2: Transformer System Architecture / Transformer 系统架构

![Figure 2](https://ar5iv.labs.arxiv.org/html/1809.08895/assets/Transformer.jpg)

**说明**: 原始 [[Transformer]] 架构（Vaswani et al. 2017）。Encoder 由多层 [[Self-Attention]] + [[Feed-Forward Network|FFN]] 组成；Decoder 在此基础上增加 masked self-attention 层和 encoder-decoder [[Cross-Attention]]。所有子层使用残差连接和 [[Layer Normalization]]。

### Figure 3: TransformerTTS System Architecture / 本文模型架构

![Figure 3](https://ar5iv.labs.arxiv.org/html/1809.08895/assets/Transtron.jpg)

**说明**: TransformerTTS 的完整架构。左侧 encoder 部分：[[Phoneme]] embedding → 3 层 CNN encoder pre-net（含重中心化线性投影）→ 加 scaled [[Positional Encoding|PE]] → $N$ 层 Transformer encoder blocks。右侧 decoder 部分：前一帧 mel → 2 层 FC decoder pre-net → 加 scaled PE → $N$ 层 Transformer decoder blocks（含 masked self-attention + cross-attention）→ mel 线性投影 + stop token 预测 → 5 层 CNN post-net 残差精修。

### Figure 4: Mel Spectrogram Comparison / Mel 频谱图对比

![Figure 4](https://ar5iv.labs.arxiv.org/html/1809.08895/assets/mel_comp.jpg)

**说明**: 3 层 TransformerTTS、6 层 TransformerTTS 与 [[Tacotron 2]] 的 mel 频谱图对比。红色矩形标注处显示：6 层模型在高频区域的细节重建明显优于 [[Tacotron 2]] 和 3 层模型，后两者在高频区域出现纹理模糊。这验证了更多 Transformer 层可以逐步精修 mel 细节的观点。

### Figure 5: PE Scale of Encoder and Decoder / 位置编码缩放因子

![Figure 5](https://ar5iv.labs.arxiv.org/html/1809.08895/assets/scale.png)

**说明**: 训练过程中 encoder 和 decoder 的位置编码缩放因子 $\alpha$ 的变化。Decoder 的 $\alpha$ 收敛值小于 encoder，证实了 [[Mel-Spectrogram]] 经 pre-net 映射后的子空间更紧凑，需要更小的位置编码幅度。这一现象支持了使用可训练缩放因子而非固定位置编码的设计决策。

### Table 1: MOS and CMOS Comparison / 主观评测对比

| System | [[MOS]] | [[CMOS]] |
|--------|---------|----------|
| [[Tacotron 2]] | 4.39 ± 0.05 | 0 |
| **TransformerTTS** | **4.39 ± 0.05** | **+0.048** |
| Ground Truth | 4.44 ± 0.05 | - |

**说明**: TransformerTTS 在 MOS 上与 [[Tacotron 2]] 持平（均为 4.39），但 CMOS 对比测试中获得 +0.048 的偏好优势。与人类录音（4.44）仅差 0.05 分。每条测试音频由 ≥20 名母语测试者评分。

### Table 2: Re-centering Ablation / 重中心化消融

| Re-center | [[MOS]] |
|-----------|---------|
| No | 4.32 ± 0.05 |
| **Yes** | **4.36 ± 0.05** |
| Ground Truth | 4.43 ± 0.05 |

**说明**: Pre-net 输出重中心化贡献了 0.04 的 MOS 提升，证实了将 ReLU 非负输出映射到以原点为中心的分布对注意力对齐学习的重要性。

### Table 3: Positional Encoding Comparison / 位置编码方法对比

| PE Type | [[MOS]] |
|---------|---------|
| Original (fixed) | 4.37 ± 0.05 |
| **Scaled (trainable α)** | **4.40 ± 0.05** |
| Ground Truth | 4.41 ± 0.04 |

**说明**: 可训练缩放因子 $\alpha$ 比固定位置编码提升 0.03 MOS。值得注意的是 scaled PE 的 MOS (4.40) 已非常接近录音 (4.41)。

### Table 4: Layer Number Ablation / 层数消融

| Layer Number | [[MOS]] |
|--------------|---------|
| 3-layer | 4.33 ± 0.06 |
| **6-layer** | **4.41 ± 0.05** |
| Ground Truth | 4.44 ± 0.05 |

**说明**: 6 层模型比 3 层提升 0.08 MOS。作者通过"类泰勒展开"视角解释：残差连接下低层贡献最大（可解释的对角线注意力），高层逐步精修残差细节。

### Table 5: Head Number Ablation / 注意力头数消融

| Head Number | [[MOS]] |
|-------------|---------|
| 4-head | 4.39 ± 0.05 |
| **8-head** | **4.44 ± 0.05** |
| Ground Truth | 4.47 ± 0.05 |

**说明**: 8 个注意力头比 4 个提升 0.05 MOS，8-head 的 4.44 已与录音质量（4.47）非常接近。多头注意力允许从多个子空间建模帧间关系。

### Table 6: Training Time per Step / 不同配置训练速度

| Configuration | 3-layer (s/step) | 6-layer (s/step) |
|---------------|-------------------|-------------------|
| 4-head | — | 0.44 |
| 8-head | 0.29 | 0.50 |

**说明**: 3 层 8-head 模型每步仅需 0.29 秒，6 层 8-head 需 0.50 秒。相比 [[Tacotron 2]] 的 ~1.7 秒/步，最快配置实现约 5.9 倍加速。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 内部美式英语数据集 | 25 小时, 17,584 对 | 单一女性专业播音员 | 训练 + 测试 |
| 测试集 | 38 条 | 不同长度，与训练集无重叠 | 主观评测 |

### 实现细节

- **Backbone**: [[Transformer]] encoder-decoder（6 层, 8 头为最佳配置）
- **Phoneme embedding**: 512 维
- **Encoder Pre-net**: 3 层 CNN, 每层 512 通道, BatchNorm + ReLU + dropout
- **Decoder Pre-net**: 2 层 FC, 256 units, ReLU
- **Post-net**: 5 层 CNN, 残差连接
- **Batch Size**: 动态 batch，约 16 samples/GPU（总 mel 帧数固定）
- **硬件**: 4× Nvidia Tesla P100 GPU
- **Vocoder**: [[WaveNet]] (2 层 QRNN + 20 层 dilated conv, 256 channels)
- **采样率**: 16,000 Hz, 帧率 80 fps
- **训练时间**: ~3 天（对比 [[Tacotron 2]] ~4.5 天）
- **每步耗时**: ~0.4s（对比 [[Tacotron 2]] ~1.7s，加速 4.25 倍）

### 关键发现

- **Batch size 至关重要**: 单 GPU 训练（小 batch）极不稳定，合成音频"如同梦呓、无法理解"，出现 missing phonemes 和 weird prosody。多 GPU 大 batch 后问题消失。
- **注意力模式观察**: 无论 3 层还是 6 层，仅前 2 层的某些头展示可解释的对角线注意力对齐，后续层的注意力图"杂乱无序"。但更多层仍然降低 loss 并提升质量。
- **Location-sensitive 多头注意力尝试**: 将 [[Tacotron 2]] 的 location-sensitive attention 引入多头注意力，训练时间翻倍且容易 OOM，最终放弃。

---

## 批判性思考

### 优点
1. **开创性工作**: 首次将 [[Transformer]] 完整引入 TTS，为后续 [[FastSpeech]]、[[FastSpeech 2]] 等非自回归方法奠定基础
2. **训练加速显著**: 4.25 倍训练加速是实打实的工程收益，参数量虽然 ~2 倍于 [[Tacotron 2]] 但并行化补偿了这一开销
3. **消融实验扎实**: 对 scaled PE、re-centering、层数、头数逐一消融，每个组件的贡献都有量化数据支撑
4. **对 TTS 特有问题的针对性适配**: 不是简单套用 NMT Transformer，而是深入分析了文本域与声学域的尺度差异并给出解决方案

### 局限性
1. **推理仍然是自回归的**: 训练可并行但推理仍需逐帧生成，未解决推理速度瓶颈（作者在 conclusion 中明确承认）
2. **数据集过小且不公开**: 仅 25 小时单说话人内部数据集，无法验证泛化性和多说话人场景
3. **评测规模有限**: 仅 38 条测试音频，无客观指标（[[WER]]、[[PESQ]] 等），缺少与更多基线的对比
4. **Batch size 依赖**: 单 GPU 训练失败暴露了 Transformer TTS 对大 batch 的依赖，限制了资源有限场景的使用
5. **Vocoder 耦合**: 使用非开源的内部 [[WaveNet]] vocoder，合成质量的贡献难以与声学模型解耦

### 潜在改进方向
1. **非自回归解码**: 作者已提及，后续 [[FastSpeech]] 正是沿此方向发展
2. **公开数据集验证**: 在 [[LJSpeech]]、[[LibriTTS]] 等公开数据集上验证
3. **替换为更快的 vocoder**: 如 [[HiFi-GAN]]（2020 年后出现）
4. **探索多说话人和零样本设定**: 结合 speaker embedding

### 可复现性评估
- [ ] 代码开源
- [ ] 预训练模型
- [x] 训练细节完整（架构参数、学习率未详述）
- [ ] 数据集可获取

---

## 关联笔记

### 基于
- [[Tacotron 2]]: TransformerTTS 的直接基线，保留了其 pre-net/post-net 设计
- [[Transformer]]: 核心架构来源，提供多头注意力和并行训练能力

### 对比
- [[Tacotron 2]]: 唯一直接对比基线，MOS 持平但 CMOS +0.048

### 后续发展
- [[FastSpeech]]: 直接受本文启发，在 Transformer encoder 基础上实现非自回归 TTS
- [[FastSpeech 2]]: 进一步改进 duration/pitch/energy 预测

### 方法相关
- [[Self-Attention]]: encoder/decoder 核心组件
- [[Cross-Attention]]: encoder-decoder 注意力
- [[Positional Encoding]]: 本文提出 scaled PE 变体
- [[Mel-Spectrogram]]: 中间声学表示
- [[WaveNet]]: 波形生成 vocoder
- [[Phoneme]]: 输入表示
- [[Autoregressive Model]]: 生成范式

---

## 速查卡片

> [!summary] Neural Speech Synthesis with Transformer Network
> - **核心**: 用 Transformer 多头注意力替换 Tacotron2 中的 RNN，并行训练加速 4.25 倍
> - **方法**: Scaled Positional Encoding + Pre-net 重中心化 + 6 层 8 头 Transformer
> - **结果**: MOS 4.39（人类 4.44），CMOS 比 Tacotron2 高 0.048
> - **代码**: 未开源（社区有第三方实现）

---

*笔记创建时间: 2026-05-25*
