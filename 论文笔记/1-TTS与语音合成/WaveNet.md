---
title: "WaveNet: A Generative Model for Raw Audio"
method_name: "WaveNet"
authors: [Aäron van den Oord, Sander Dieleman, Heiga Zen, Karen Simonyan, Oriol Vinyals, Alex Graves, Nal Kalchbrenner, Andrew Senior, Koray Kavukcuoglu]
year: 2016
venue: arXiv
tags: [autoregressive-tts, raw-waveform, dilated-convolution, vocoder, multi-speaker, neural-audio-generation]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/1609.03499
created: 2026-05-25
---

# 论文笔记：WaveNet: A Generative Model for Raw Audio

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Google DeepMind, London, UK |
| 日期 | September 2016 |
| 项目主页 | [DeepMind Blog](https://www.deepmind.com/blog/wavenet-generative-model-raw-audio/) |
| 对比基线 | HMM-driven unit selection concatenative / LSTM-RNN parametric |
| 链接 | [arXiv](https://arxiv.org/abs/1609.03499) |

---

## 一句话总结

> 首个直接在原始波形层面建模的深度自回归生成模型，通过 [[Dilated Causal Convolution|空洞因果卷积]] 实现指数级感受野增长，在 TTS 中取得当时最高 MOS。

---

## 核心贡献

1. **原始波形自回归建模**: 首次将 [[PixelCNN]] 式自回归架构应用于 16 kHz 原始音频波形，每个采样点条件于所有之前的采样点
2. **空洞因果卷积架构**: 提出 [[Dilated Causal Convolution|空洞因果卷积]] 堆叠方案，以指数增长的空洞率获得极大感受野，同时保持计算效率
3. **条件生成机制**: 支持全局条件（说话人身份）和局部条件（语言学特征），单一模型可处理多说话人 TTS
4. **跨任务通用性**: 同一架构在 TTS、音乐生成、语音识别上均展现出强大能力

---

## 问题背景

### 要解决的问题

如何直接在原始音频波形级别生成高质量语音？传统 TTS 依赖参数化声码器或拼接合成，自然度受限。

### 现有方法的局限

1. **拼接合成（Concatenative）**: 依赖大型语音库，拼接痕迹明显，难以灵活切换说话人风格
2. **统计参数合成（Statistical Parametric）**: 使用声码器提取参数（[[Mel-Spectrogram|梅尔频谱]]、F0、非周期性等），存在三大退化因素：声码器质量差产生伪影、生成模型精度不足、过平滑导致闷声
3. **两阶段流程次优**: 先拟合声码器提取参数、再建模参数轨迹，整体训练非最优

### 本文的动机

[[PixelCNN]] / PixelRNN 在图像生成中的成功表明，自回归模型可以有效建模高维数据的联合分布。音频波形虽然时间分辨率更高（每秒 16,000+ 采样点），但同样的因式分解原理应当适用。如果能直接在波形级别端到端学习，就能一举消除声码器伪影、模型精度不足和过平滑三个问题。

---

## 方法详解

### 模型架构

WaveNet 采用 **全自回归（Fully Autoregressive）** 架构，直接在原始波形上操作：

- **输入**: 原始音频波形 $\mathbf{x} = \{x_1, \dots, x_T\}$，经 [[Mu-Law Companding|μ-law 压扩]] 量化为 256 级
- **核心组件**: 多层 [[Dilated Causal Convolution|空洞因果卷积]]，使用 [[Gated Activation Unit|门控激活单元]]
- **连接方式**: [[Residual Connection|残差连接]] + 参数化 [[Skip Connection|跳跃连接]]
- **输出**: 256 维 [[Softmax]] 分类分布，预测下一个采样点值
- **训练**: 最大化对数似然，所有时间步可并行计算

### 核心模块

#### 模块1: 空洞因果卷积（Dilated Causal Convolutions）

**设计动机**: 普通 [[Causal Convolution|因果卷积]] 的感受野增长缓慢（线性于层数），而原始波形需要数千个采样点的感受野才能捕捉有意义的语音结构。

**具体实现**:
- [[Causal Convolution|因果卷积]] 保证 $p(x_{t+1} \mid x_1, \dots, x_t)$ 不依赖未来时间步，通过平移输出实现
- [[Dilated Convolution|空洞卷积]] 在不增加计算量的前提下，按指数增长感受野
- 空洞率按层递增：$1, 2, 4, \dots, 512$，一个 block 的感受野为 1024
- 多个 block 堆叠（如 3 个 block：$1,2,4,\dots,512,1,2,4,\dots,512,1,2,4,\dots,512$）进一步扩大感受野和模型容量
- 可视为标准 $1 \times 1024$ 卷积的非线性、更高效对等物

#### 模块2: 门控激活单元（Gated Activation Units）

**设计动机**: 来自 [[Gated PixelCNN]]，门控非线性在音频建模中显著优于 [[ReLU]]。

**具体实现**:
- 每层同时计算 filter 分支（$\tanh$）和 gate 分支（$\sigma$），逐元素相乘
- filter 分支控制"要表达什么"，gate 分支控制"允许多少通过"
- 初步实验中效果显著优于 ReLU

#### 模块3: 残差与跳跃连接

**设计动机**: 加速收敛、支持训练更深的模型。

**具体实现**:
- 每个残差 block 包含空洞因果卷积 → 门控激活 → 1x1 卷积 → 残差加法
- 每个 block 还输出跳跃连接到最终输出层
- 所有跳跃连接求和后经 ReLU → 1x1 卷积 → ReLU → 1x1 卷积 → Softmax

#### 模块4: 条件机制（Conditioning）

**设计动机**: 控制生成内容的属性（说话人身份、文本内容等）。

**全局条件**: 单一向量 $\mathbf{h}$（如说话人 one-hot）经线性投影后 broadcast 到所有时间步，加入门控激活

**局部条件**: 时变序列 $\mathbf{h}_t$（如语言学特征）经转置卷积上采样到音频帧率后，通过 1x1 卷积加入门控激活

---

## 关键公式

### 公式1: [[Autoregressive Model|自回归分解]]

$$
p(\mathbf{x}) = \prod_{t=1}^{T} p(x_t \mid x_1, \dots, x_{t-1})
$$

**含义**: 波形的联合概率被分解为逐采样点的条件概率之积，这是整个模型的数学基础。

**符号说明**:
- $\mathbf{x} = \{x_1, \dots, x_T\}$: 原始音频波形序列
- $x_t$: 第 $t$ 个采样点
- $T$: 序列总长度（16 kHz 下 1 秒 = 16,000）

### 公式2: [[Mu-Law Companding|μ-law 压扩变换]]

$$
f(x_t) = \text{sign}(x_t) \frac{\ln(1 + \mu |x_t|)}{\ln(1 + \mu)}
$$

**含义**: 将 16-bit 线性 PCM（65,536 级）压缩为 256 级量化，非线性量化在低幅值区域保留更多精度，显著优于线性量化。

**符号说明**:
- $x_t \in (-1, 1)$: 归一化音频采样值
- $\mu = 255$: 压扩系数（ITU-T G.711 标准）
- $\text{sign}(\cdot)$: 符号函数

### 公式3: [[Gated Activation Unit|门控激活函数]]

$$
\mathbf{z} = \tanh(W_{f,k} * \mathbf{x}) \odot \sigma(W_{g,k} * \mathbf{x})
$$

**含义**: 每层的非线性激活，filter 分支和 gate 分支的逐元素乘积，在初始实验中显著优于 ReLU。

**符号说明**:
- $*$: 卷积操作
- $\odot$: 逐元素乘法（Hadamard 积）
- $\sigma(\cdot)$: Sigmoid 函数
- $W_{f,k}$: 第 $k$ 层的 filter 卷积核
- $W_{g,k}$: 第 $k$ 层的 gate 卷积核

### 公式4: [[Conditional Generation|条件生成分解]]

$$
p(\mathbf{x} \mid \mathbf{h}) = \prod_{t=1}^{T} p(x_t \mid x_1, \dots, x_{t-1}, \mathbf{h})
$$

**含义**: 给定额外条件输入 $\mathbf{h}$ 时的条件自回归分布，使模型可以生成具有特定属性的音频。

**符号说明**:
- $\mathbf{h}$: 条件输入（全局：说话人嵌入；局部：语言学特征序列）

### 公式5: 全局条件下的门控激活

$$
\mathbf{z} = \tanh(W_{f,k} * \mathbf{x} + V_{f,k}^T \mathbf{h}) \odot \sigma(W_{g,k} * \mathbf{x} + V_{g,k}^T \mathbf{h})
$$

**含义**: 全局条件（如说话人身份向量）通过线性投影加入 filter 和 gate 两个分支，$V^T \mathbf{h}$ 在时间维度上 broadcast。

**符号说明**:
- $V_{f,k}, V_{g,k}$: 可学习的线性投影矩阵
- $V_{f,k}^T \mathbf{h}$: 投影后广播到所有时间步

### 公式6: 局部条件下的门控激活

$$
\mathbf{z} = \tanh(W_{f,k} * \mathbf{x} + V_{f,k} * \mathbf{y}) \odot \sigma(W_{g,k} * \mathbf{x} + V_{g,k} * \mathbf{y})
$$

**含义**: 局部条件（如语言学特征 $\mathbf{h}_t$）先经转置卷积上采样为 $\mathbf{y} = f(\mathbf{h})$，再通过 1x1 卷积加入。

**符号说明**:
- $\mathbf{y} = f(\mathbf{h})$: 经转置卷积上采样后的局部条件信号
- $V_{f,k} * \mathbf{y}$: 1x1 卷积

### 公式7: [[Statistical Parametric Speech Synthesis|统计参数合成训练目标]]（Appendix）

$$
\hat{\Lambda} = \arg\max_{\Lambda} p(\mathbf{o} \mid \mathbf{l}, \Lambda)
$$

**含义**: 传统统计参数 TTS 的训练目标——最大化给定语言学特征 $\mathbf{l}$ 下声码器参数 $\mathbf{o}$ 的似然。

**符号说明**:
- $\Lambda$: 模型参数
- $\mathbf{o} = \{\mathbf{o}_1, \dots, \mathbf{o}_N\}$: 声码器参数序列
- $\mathbf{l}$: 语言学特征

### 公式8: 统计参数合成推理

$$
\hat{\mathbf{o}} = \arg\max_{\mathbf{o}} p(\mathbf{o} \mid \mathbf{l}, \hat{\Lambda})
$$

**含义**: 给定训练好的模型参数 $\hat{\Lambda}$，推理时选择最大后验估计的声码器参数。

### 公式9: [[Linear Predictive Coding|线性预测模型]]（Appendix）

$$
x_t = \sum_{p=1}^{P} a_p x_{t-p} + \epsilon_t
$$

**含义**: 传统线性预测分析假设语音为线性自回归高斯过程，WaveNet 可视为其**非线性**的推广。

**符号说明**:
- $a_p$: 第 $p$ 阶线性预测系数
- $P$: 预测阶数
- $\epsilon_t \sim \mathcal{N}(0, G^2)$: 建模误差（高斯假设）

---

## 关键图表

### Figure 1: A Second of Generated Speech / 生成语音波形示例

![Figure 1](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/Figures/one_second.png)

**说明**: 展示 WaveNet 生成的一秒语音波形。模型直接在 16 kHz 原始波形级别生成，每秒产生 16,000 个采样点，波形呈现出自然语音的包络和细节。

### Figure 2: Causal Convolutions / 因果卷积堆叠

![Figure 2](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x1.png)

**说明**: 标准 [[Causal Convolution|因果卷积]] 的可视化。每层只能看到过去的输入，感受野随层数线性增长。图中 5 层卷积（filter size=2）的感受野仅为 5，需要大量层才能覆盖有意义的音频上下文。

### Figure 3: Dilated Causal Convolutions / 空洞因果卷积堆叠

![Figure 3](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x2.png)

**说明**: [[Dilated Causal Convolution|空洞因果卷积]] 的核心创新。空洞率 $d = 1, 2, 4, 8$ 按指数增长，使感受野指数级扩大。4 层即可覆盖 16 个时间步，相比标准因果卷积效率大幅提升。这是 WaveNet 能够在原始波形上建模长程依赖的关键。

### Figure 4: Residual Block and Architecture Overview / 残差块与整体架构

![Figure 4](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x3.png)

**说明**: WaveNet 的完整架构图。左侧展示单个残差 block 的内部结构：空洞因果卷积 → [[Gated Activation Unit|门控激活]]（tanh ⊙ σ）→ 1x1 卷积 → 残差加法 + 跳跃连接输出。右侧展示整体网络：多个残差 block 堆叠，所有 [[Skip Connection|跳跃连接]] 汇聚后经两层 1x1 卷积和 ReLU 输出 [[Softmax]] 分布。

### Figure 5: Subjective Preference Scores / 主观偏好评分

![Figure 5a](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x4.png)

![Figure 5b](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x5.png)

![Figure 5c](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x6.png)

**说明**: TTS 主观偏好评测结果（英语和中文）。三组对比：(上) 两个基线互比——LSTM 参数合成 vs HMM 拼接合成；(中) 两个 WaveNet 互比——仅语言学特征条件 vs 语言学特征+F0 条件；(下) 最佳基线 vs 最佳 WaveNet。WaveNet (L+F) 在两种语言中均以压倒性优势胜出。

### Figure 6: Statistical Parametric Speech Synthesis / 统计参数语音合成流程

![Figure 6](https://ar5iv.labs.arxiv.org/html/1609.03499/assets/x7.png)

**说明**: 传统统计参数 TTS 的流程框图（Appendix A 背景知识）。文本经 NLP 前端提取语言学特征，再由生成模型预测声码器参数（频谱、F0、非周期性），最后由声码器重建波形。WaveNet 跳过了中间的声码器参数化步骤，直接从条件特征生成原始波形。

### Table 1: MOS 评测结果

| 系统 | 北美英语 | 普通话 |
|------|---------|--------|
| LSTM-RNN parametric | 3.67 ± 0.098 | 3.79 ± 0.084 |
| HMM-driven concatenative | 3.86 ± 0.137 | 3.47 ± 0.108 |
| **WaveNet (L+F)** | **4.21 ± 0.081** | **4.08 ± 0.085** |
| Natural (8-bit μ-law) | 4.46 ± 0.067 | 4.25 ± 0.082 |
| Natural (16-bit linear PCM) | 4.55 ± 0.075 | 4.21 ± 0.071 |

**表格说明**: WaveNet (L+F) 在两种语言上均取得当时最高 [[MOS]]。与自然语音的差距从英语 0.69 缩小到 0.34（**缩小 51%**），中文从 0.42 缩小到 0.13（**缩小 69%**）。这是首次 TTS 系统 MOS 突破 4.0。

### Table 2: 完整配对比较结果（北美英语）

| System A | System B | A (%) | B (%) | No pref (%) | p value |
|----------|----------|-------|-------|-------------|---------|
| LSTM | Concat | 23.3 | 63.6 | 13.1 | ≪10⁻⁹ |
| LSTM | WaveNet (L) | 18.7 | 69.3 | 12.0 | ≪10⁻⁹ |
| LSTM | WaveNet (L+F) | 7.6 | 82.0 | 10.4 | ≪10⁻⁹ |
| Concat | WaveNet (L) | 32.4 | 41.2 | 26.4 | 0.003 |
| Concat | WaveNet (L+F) | 20.1 | 49.3 | 30.6 | ≪10⁻⁹ |
| WaveNet (L) | WaveNet (L+F) | 17.8 | 37.9 | 44.3 | ≪10⁻⁹ |

### Table 2 (续): 完整配对比较结果（普通话）

| System A | System B | A (%) | B (%) | No pref (%) | p value |
|----------|----------|-------|-------|-------------|---------|
| LSTM | Concat | 50.6 | 15.6 | 33.8 | ≪10⁻⁹ |
| LSTM | WaveNet (L) | 25.0 | 23.3 | 51.8 | 0.476 |
| LSTM | WaveNet (L+F) | 12.5 | 29.3 | 58.2 | ≪10⁻⁹ |
| Concat | WaveNet (L) | 17.6 | 43.1 | 39.3 | ≪10⁻⁹ |
| Concat | WaveNet (L+F) | 7.6 | 55.9 | 36.5 | ≪10⁻⁹ |
| WaveNet (L) | WaveNet (L+F) | 10.0 | 25.5 | 64.5 | ≪10⁻⁹ |

**表格说明**: 配对偏好测试全面验证了 WaveNet 的优势。WaveNet (L+F) vs LSTM 在英语中获得 82.0% 偏好率，在中文中获得 29.3% vs 12.5%（显著胜出）。注意中文 WaveNet (L) vs LSTM 的 p=0.476 说明仅用语言学特征的 WaveNet 在中文韵律上不够好，需要外部 F0 模型。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| CSTR VCTK | 44 hours, 109 speakers | 英文多说话人 | 多说话人语音生成 |
| 内部英文 TTS | 24.6 hours, 1 speaker | 北美英语专业女性录音 | TTS 评测 |
| 内部中文 TTS | 34.8 hours, 1 speaker | 普通话专业女性录音 | TTS 评测 |
| MagnaTagATune | ~200 hours | 音乐，188 种标签 | 条件音乐生成 |
| YouTube Piano | ~60 hours | 钢琴独奏 | 无条件音乐生成 |
| TIMIT | 标准集 | 英文语音识别 | 音素识别 |

### 实现细节

- **采样率**: 16 kHz（TTS / 多说话人）
- **量化**: 8-bit [[Mu-Law Companding|μ-law]] 编码 → 256 级 softmax
- **感受野**: ~240 ms（TTS 实验）；多说话人约 300 ms
- **空洞率**: $1, 2, 4, \dots, 512$，多个 block 堆叠
- **条件特征**: 音素、音节、词、短语、句子级语言学特征，每 5 ms 一帧，通过强制对齐关联
- **外部模型**: LSTM-RNN 音素时长预测 + 自回归 CNN log F0 预测（MSE 训练）
- **评测**: 盲测众包，100 句测试句，每对/每句 8 名被试，排除约 40% 不戴耳机的评分
- **音素识别**: 空洞卷积后接 mean-pooling（160× 下采样至 10 ms 帧）→ 非因果卷积，双损失（下一采样点预测 + 帧分类）

### 关键实验结果

**TTS**: MOS 4.21（英语）/ 4.08（中文），首次突破 4.0 大关，与自然语音的差距缩小 51%~69%。

**多说话人生成**: 109 个说话人的单一模型生成了不存在但听起来像人类语言的语音，能逼真模仿呼吸声、口腔运动、录音环境。添加更多说话人反而提升了验证集性能，暗示说话人间存在内部表征共享。感受野约 300 ms（2-3 个音素），长程一致性不足。

**音乐生成**: 扩大感受野对音乐质量至关重要。即使达到多秒感受野，仍缺乏长程结构一致性（风格/乐器/音量秒级变化），但短期内常产生和谐悦耳的音乐片段。条件模型可通过标签控制风格/乐器。

**语音识别**: TIMIT 上 18.8 PER，是当时直接从原始波形训练的最佳结果。

---

## 批判性思考

### 优点

1. **范式开创**: 首次证明自回归模型可以直接在原始波形级别生成高自然度语音，彻底绕过声码器参数化的三大退化因素
2. **架构优雅**: [[Dilated Causal Convolution|空洞因果卷积]] 的设计兼顾了大感受野和计算效率，后来被广泛借鉴到 [[TCN]]、[[ByteNet]] 等架构
3. **通用性强**: 同一架构在 TTS、音乐、语音识别三个不同任务上均有竞争力，展示了原始波形建模的通用框架潜力
4. **多说话人能力**: 通过简单的全局条件实现单模型多说话人，且说话人间存在表征共享

### 局限性

1. **推理极慢**: 自回归逐采样点生成，16 kHz 下生成 1 秒需要 16,000 步前向传播，实际 [[RTF]] 远大于 1，无法实时。后续 Parallel WaveNet (2017) 和 WaveGlow (2018) 才解决此问题
2. **感受野有限**: 240-300 ms 的感受野仅覆盖 2-3 个音素，长程韵律和结构一致性不足，需依赖外部 F0 模型补偿
3. **量化损失**: 8-bit μ-law（256 级）相比 16-bit 线性 PCM 有质量损失，MOS 上自然语音的 8-bit 版也比 16-bit 版低
4. **内部数据评测**: TTS 实验使用 Google 内部数据集，不可复现

### 潜在改进方向

1. **并行化生成**: 知识蒸馏/Flow-based 方法实现并行生成（后续 Parallel WaveNet 实现）
2. **端到端 TTS**: 去除外部时长/F0 预测模型，直接从文本到波形（后续 [[Tacotron 2]] + WaveNet 部分实现）
3. **连续分布输出**: 用混合逻辑分布替代 256 级 softmax（后续 WaveRNN/SampleRNN 探索）
4. **更大感受野 / 注意力**: 引入 self-attention 提升长程依赖建模（后续 Transformer TTS 实现）

### 可复现性评估

- [ ] 代码开源（Google 未开源原始实现，但社区有多个复现）
- [ ] 预训练模型（未公开）
- [x] 训练细节完整（Appendix B 详细描述了语言学特征、评测设置）
- [ ] 数据集可获取（VCTK/TIMIT 公开，TTS 数据集为内部数据）

---

## 关联笔记

### 基于
- [[PixelCNN]]: 自回归图像生成架构，WaveNet 的直接灵感来源
- [[PixelRNN]]: 同系列图像生成模型

### 对比
- [[Tacotron 2]]: 后续将 WaveNet 作为声码器组件使用
- [[HiFi-GAN]]: GAN-based 声码器，实时推理，后来替代 WaveNet 成为主流声码器

### 方法相关
- [[Dilated Causal Convolution]]: 核心架构创新
- [[Mu-Law Companding]]: 音频量化方法
- [[Gated Activation Unit]]: 门控非线性激活
- [[Residual Connection]]: 残差连接
- [[Skip Connection]]: 跳跃连接
- [[Softmax]]: 输出分布
- [[Causal Convolution]]: 因果卷积基础

### 硬件/数据相关
- [[VCTK]]: 多说话人英文语料库
- [[TIMIT]]: 语音识别标准评测集

---

## 速查卡片

> [!summary] WaveNet: A Generative Model for Raw Audio
> - **核心**: 首个直接在原始波形级别自回归生成音频的深度模型
> - **方法**: 空洞因果卷积堆叠（指数感受野） + 门控激活 + 残差/跳跃连接 + μ-law 256 级 softmax
> - **结果**: TTS MOS 4.21（英语）/4.08（中文），与自然语音差距缩小 51%~69%；TIMIT 18.8 PER
> - **代码**: 社区复现（r9y9/wavenet_vocoder 等），官方未开源

---

*笔记创建时间: 2026-05-25*
