---
title: "FastSpeech 2: Fast and High-Quality End-to-End Text to Speech"
method_name: "FastSpeech2"
authors: [Yi Ren, Chenxu Hu, Xu Tan, Tao Qin, Sheng Zhao, Zhou Zhao, Tie-Yan Liu]
year: 2021
venue: ICLR 2021
tags: [tts, non-autoregressive, duration-predictor, pitch-prediction, energy-prediction, variance-adaptor, parallel-synthesis]
image_source: online
arxiv_html: https://ar5iv.labs.arxiv.org/html/2006.04558
created: 2026-05-25
---

# 论文笔记：FastSpeech 2: Fast and High-Quality End-to-End Text to Speech

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Zhejiang University, Microsoft Research Asia, Microsoft Azure Speech |
| 日期 | June 2020 (ICLR 2021) |
| 项目主页 | [speechresearch/fastspeech](https://speechresearch.github.io/fastspeech2/) |
| 对比基线 | [[Tacotron 2]], [[FastSpeech]], Transformer TTS |
| 链接 | [arXiv](https://arxiv.org/abs/2006.04558) / [Code](https://github.com/microsoft/NeuralSpeech) |

---

## 一句话总结

> 去掉 FastSpeech 的 teacher-student 蒸馏，直接用 ground-truth 训练 + 引入 pitch/energy/duration 三重 variance 信息，实现更快训练和更高质量的并行 TTS。

---

## 核心贡献

1. **去除 teacher-student 蒸馏**: 直接用 ground-truth [[Mel-Spectrogram]] 训练，训练速度比 [[FastSpeech]] 快 3 倍
2. **Variance Adaptor**: 引入 [[Duration Predictor]]、pitch predictor、energy predictor 三路 variance 信息，显式建模语音的一对多映射问题
3. **FastSpeech 2s**: 首次实现从 [[Phoneme]] 序列完全并行生成 [[Waveform]] 的端到端 TTS

---

## 问题背景

### 要解决的问题

TTS 本质是一个 **一对多映射** (one-to-many mapping) 问题——同一段文本可以对应多种不同 pitch、duration、energy、韵律的语音。[[Autoregressive]] 模型（如 [[Tacotron 2]]）推理慢、存在 word skipping/repeating 的鲁棒性问题；[[FastSpeech]] 虽然解决了并行生成，但引入了新的问题。

### 现有方法的局限

[[FastSpeech]] 的三个问题：
1. **teacher-student 蒸馏管线复杂**: 需要先训练 AR teacher，再用 teacher 的输出做 target，训练耗时
2. **teacher 输出的 Mel 信息损失**: 蒸馏后的 target mel 比 ground-truth 更简化，丢失了细节
3. **从 teacher attention 提取的 duration 不够准确**: attention 本身就可能不准

### 本文的动机

直接用 ground-truth mel 训练可以获得更完整的信息，同时用 [[Montreal Forced Alignment|MFA]] 替代 teacher attention 提取更精确的 duration，再通过额外的 pitch/energy variance 信息显式缓解一对多映射问题。

---

## 方法详解

### 模型架构

FastSpeech 2 采用 **Encoder-Variance Adaptor-Decoder** 架构：
- **输入**: [[Phoneme]] 序列（vocabulary size = 76）
- **Encoder**: 4 层 [[Feed-Forward Transformer|FFT]] block，将 phoneme embedding 转为 phoneme hidden sequence
- **Variance Adaptor**: 包含 [[Duration Predictor]] + pitch predictor + energy predictor + [[Length Regulator]]
- **Decoder**: 4 层 FFT block，并行生成 80 维 [[Mel-Spectrogram]]
- **Vocoder**: 预训练 [[Parallel WaveGAN]] 合成波形
- **总参数**: 27M

### 核心模块

#### 模块1: Feed-Forward Transformer (FFT) Block

**设计动机**: 利用 [[Self-Attention]] + 1D [[Convolution]] 捕获序列级和局部模式

**具体实现**:
- 每个 FFT block 包含 multi-head self-attention（2 heads）+ 1D convolution（kernel=9, filter=1024）
- Encoder 和 Decoder 各堆叠 4 层
- hidden dim = 256, dropout = 0.1

#### 模块2: Variance Adaptor

**设计动机**: 显式提供 duration/pitch/energy 三类 variance 信息，缓解一对多映射导致的过平滑问题

**具体实现**:

1. **Duration Predictor**:
   - 2 层 1D Conv (kernel=3, filter=256) + ReLU + [[Layer Normalization|LayerNorm]] + Dropout(0.5) + Linear
   - 训练 target: [[Montreal Forced Alignment|MFA]] 提取的 ground-truth duration
   - Loss: MSE（log domain）
   - 推理时用预测值通过 [[Length Regulator]] 扩展 phoneme hidden → frame-level hidden

2. **Pitch Predictor**:
   - 相同的 predictor 架构
   - 对 F0 序列做 [[CWT|连续小波变换 (CWT)]] 分解为 pitch spectrogram（10 个尺度），预测 pitch spectrogram
   - F0 量化为 256 个对数等分 bin → pitch embedding $\mathbf{p}$ 加到 hidden sequence
   - 推理时: 预测 pitch spectrogram → iCWT 恢复 F0 contour

3. **Energy Predictor**:
   - 相同的 predictor 架构
   - Energy = 每帧 STFT 幅度的 L2-norm
   - 量化为 256 个均匀 bin → energy embedding $\mathbf{e}$
   - 不做 CWT 变换（energy 变化不如 pitch 剧烈）

训练时使用 ground-truth duration/pitch/energy 作为条件输入；推理时使用各 predictor 的预测值。

#### 模块3: FastSpeech 2s — Waveform Decoder

**设计动机**: 完全跳过 Mel 阶段，直接从文本并行生成 [[Waveform]]

**具体实现**:
- 基于 [[WaveNet]] 的非因果卷积 + gated activation
- 30 个 dilated 1D Conv block (kernel=3)
- Transposed 1D Conv 上采样（filter=64）
- 输入: 截取的 hidden sequence（对应 20480 个 waveform sample）
- 判别器: 与 [[Parallel WaveGAN]] 相同，10 层 non-causal dilated 1D Conv + Leaky ReLU
- Loss: multi-resolution STFT loss + LSGAN adversarial loss
- 保留 mel decoder 辅助文本特征学习（训练时用完整文本，waveform decoder 只用截取片段）

---

## 关键公式

### 公式1: [[Duration Predictor|Duration Loss]]

$$
\mathcal{L}_{dur} = \text{MSE}(\hat{d}_i, \log d_i)
$$

**含义**: Duration predictor 在 log domain 上优化 MSE 损失，$d_i$ 是 MFA 提供的 ground-truth phoneme duration

**符号说明**:
- $\hat{d}_i$: 预测的 log duration
- $d_i$: 第 $i$ 个 phoneme 的 ground-truth duration（帧数）

### 公式2: [[CWT|连续小波变换 (CWT)]]

$$
W(\tau, t) = \tau^{-1/2} \int_{-\infty}^{+\infty} F_0(x) \psi\!\left(\frac{x - t}{\tau}\right) dx
$$

**含义**: 将 F0 contour 分解为多尺度的 pitch spectrogram，使 pitch predictor 能在频域建模 pitch 变化

**符号说明**:
- $W(\tau, t)$: 尺度 $\tau$、时间 $t$ 处的小波系数
- $F_0(x)$: 原始基频序列
- $\psi(\cdot)$: Mexican hat 母小波函数
- $\tau$: 尺度参数

### 公式3: [[CWT|逆连续小波变换 (iCWT)]]

$$
F_0(t) = \int_{-\infty}^{+\infty} \int_{0}^{+\infty} W(\tau, t) \cdot \tau^{-5/2} \cdot \psi\!\left(\frac{x - t}{\tau}\right) dx \, d\tau
$$

**含义**: 推理阶段从预测的 pitch spectrogram 恢复 F0 contour

### 公式4: Pitch Spectrogram 离散分解

$$
W_i(t) = W(2^{i+1}\tau_0,\, t) \cdot (i + 2.5)^{-5/2}, \quad i = 1, \ldots, 10
$$

**含义**: 将 CWT 离散化为 10 个尺度的 pitch spectrogram，$\tau_0 = 5\text{ms}$

**符号说明**:
- $W_i(t)$: 第 $i$ 个尺度的小波系数
- $\tau_0$: 基础尺度参数（5ms）

### 公式5: F0 重建

$$
\hat{F}_0(t) = \sum_{i=1}^{10} \hat{W}_i(t) \cdot (i + 2.5)^{-5/2}
$$

**含义**: 从预测的 10 个尺度小波系数重建 F0 contour

### 公式6: Pitch 预处理流程

$$
F_0 \xrightarrow{\text{线性插值}} F_0' \xrightarrow{\log} \log F_0' \xrightarrow{\text{z-score}} \tilde{F}_0 \xrightarrow{\text{CWT}} W_i(t)
$$

**含义**: F0 预处理四步——对 unvoiced 帧线性插值 → log 变换 → 每句 zero-mean unit-variance 归一化（保存 mean/var 用于推理恢复） → CWT 分解

### 公式7: 总损失（FastSpeech 2）

$$
\mathcal{L} = \mathcal{L}_{mel} + \mathcal{L}_{dur} + \mathcal{L}_{pitch} + \mathcal{L}_{energy}
$$

**含义**: FastSpeech 2 的总训练损失为 mel 重建 MAE + duration MSE + pitch MSE + energy MSE

**符号说明**:
- $\mathcal{L}_{mel}$: mel-spectrogram 重建损失（MAE）
- $\mathcal{L}_{dur}$: duration 预测损失（MSE, log domain）
- $\mathcal{L}_{pitch}$: pitch spectrogram 预测损失（MSE）
- $\mathcal{L}_{energy}$: energy 预测损失（MSE）

### 公式8: 总损失（FastSpeech 2s）

$$
\mathcal{L}_{2s} = \mathcal{L}_{mel} + \mathcal{L}_{dur} + \mathcal{L}_{pitch} + \mathcal{L}_{energy} + \mathcal{L}_{stft} + \mathcal{L}_{adv}
$$

**含义**: FastSpeech 2s 额外加上 multi-resolution STFT loss 和 LSGAN adversarial loss

**符号说明**:
- $\mathcal{L}_{stft}$: multi-resolution STFT 重建损失
- $\mathcal{L}_{adv}$: 对抗训练损失（LSGAN）

---

## 关键图表

### Figure 1(a): Overall Architecture / 系统概览

![Figure 1(a)](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x1.png)

**说明**: FastSpeech 2/2s 整体架构。Encoder 将 phoneme 序列编码为 hidden sequence，经 Variance Adaptor 添加 duration/pitch/energy 信息后，Mel Decoder 并行生成 mel-spectrogram；FastSpeech 2s 额外包含 Waveform Decoder 直接生成波形。

### Figure 1(b): Variance Adaptor / 方差适配器

![Figure 1(b)](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x2.png)

**说明**: Variance Adaptor 内部结构。LR 为 [[Length Regulator]]，先经 [[Duration Predictor]] 确定每个 phoneme 的帧数并扩展序列，然后依次加上 pitch embedding 和 energy embedding。

### Figure 1(c): Duration/Pitch/Energy Predictor / 预测器结构

![Figure 1(c)](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x3.png)

**说明**: 三个 predictor 共享相同架构：2 层 1D Conv + ReLU + [[Layer Normalization|LayerNorm]] + Dropout + Linear。LN 为 Layer Normalization。

### Figure 1(d): Waveform Decoder / 波形解码器

![Figure 1(d)](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x4.png)

**说明**: FastSpeech 2s 的 Waveform Decoder，基于 [[WaveNet]] 非因果卷积架构，包含 gated activation 和 skip connection。

### Figure 2: Pitch Predictor Details / Pitch 预测器细节

![Figure 2](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x5.png)

**说明**: Pitch predictor 详细结构。输入 hidden sequence 经 [[CWT]] 分解预测 pitch spectrogram（10 个尺度），同时从全局平均 hidden state 预测 mean/variance 用于 iCWT 恢复。

### Figure 3: Pitch Contour Comparison / Pitch 轮廓对比

![Figure 3(a) Ground-truth](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x6.png)

![Figure 3(b) FastSpeech](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x7.png)

![Figure 3(c) FastSpeech 2](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x8.png)

![Figure 3(d) FastSpeech 2s](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x9.png)

**说明**: 同一句话 "The worst, which perhaps was the English, was a terrible falling-off from the work of the earlier presses" 的 pitch contour 对比。FastSpeech 的 pitch 变化过于平滑（缺乏 variance 信息），FastSpeech 2/2s 的 pitch 更接近 ground-truth，变化更丰富。

### Figure 4: Pitch Control / F0 缩放控制

![Figure 4(a) F0 × 1.0](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x10.png)

![Figure 4(b) F0 × 0.75](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x11.png)

![Figure 4(c) F0 × 1.50](https://ar5iv.labs.arxiv.org/html/2006.04558/assets/x12.png)

**说明**: FastSpeech 2 通过缩放 F0 实现 pitch 可控合成。输入文本 "They discarded this for a more completely Roman and far less beautiful letter"，分别展示原始(1.0×)、降调(0.75×)、升调(1.50×) 的 mel-spectrogram，证明 pitch predictor 的可控性。

### Table 1a: Audio Quality — MOS 评测

| Method | MOS |
|--------|-----|
| GT | 4.30 ± 0.07 |
| GT (Mel + PWG) | 3.92 ± 0.08 |
| Tacotron 2 (Mel + PWG) | 3.70 ± 0.08 |
| Transformer TTS (Mel + PWG) | 3.72 ± 0.07 |
| FastSpeech (Mel + PWG) | 3.68 ± 0.09 |
| **FastSpeech 2 (Mel + PWG)** | **3.83 ± 0.08** |
| FastSpeech 2s | 3.71 ± 0.09 |

**说明**: FastSpeech 2 MOS 达 3.83，超越所有 AR 基线（Tacotron 2: 3.70, Transformer TTS: 3.72），也大幅超越 FastSpeech (3.68)。20 名英语母语者评测。

### Table 1b: CMOS 对比

| Method | CMOS |
|--------|------|
| FastSpeech 2 | 0.000 |
| FastSpeech | -0.885 |
| Transformer TTS | -0.235 |

**说明**: FastSpeech 2 相对 FastSpeech 有 +0.885 的 CMOS 优势，相对 Transformer TTS 有 +0.235 优势。

### Table 2: Training and Inference Speed / 训练与推理速度

| Method | Training Time (h) | Inference RTF | Inference Speedup |
|--------|-------------------|---------------|-------------------|
| Transformer TTS | 38.64 | 9.32×10⁻¹ | / |
| FastSpeech | 53.12 | 1.92×10⁻² | 48.5× |
| FastSpeech 2 | **17.02** | 1.95×10⁻² | 47.8× |
| FastSpeech 2s | 92.18 | **1.80×10⁻²** | **51.8×** |

**说明**: FastSpeech 2 训练仅需 17.02 小时，比 FastSpeech 快 3.12 倍（去除了 teacher 训练）。推理 RTF 约 0.02，比 Transformer TTS 快 ~48 倍。

### Table 3: Pitch Distribution Analysis / Pitch 分布分析

| Method | σ | γ (Skewness) | K (Kurtosis) | DTW |
|--------|---|---|---|-----|
| GT | 54.4 | 0.836 | 0.977 | / |
| Tacotron 2 | 44.1 | 1.28 | 1.311 | 26.32 |
| Transformer TTS | 40.8 | 0.703 | 1.419 | 24.40 |
| FastSpeech | 50.8 | 0.724 | -0.041 | 24.89 |
| **FastSpeech 2** | **54.1** | **0.881** | **0.996** | **24.39** |
| FastSpeech 2 - CWT | 42.3 | 0.771 | 1.115 | 25.13 |
| FastSpeech 2s | 53.9 | 0.872 | 0.998 | 24.37 |

**说明**: FastSpeech 2 的 pitch 标准差 (σ=54.1)、偏度 (γ=0.881)、峰度 (K=0.996) 都最接近 ground-truth，证明 CWT pitch predictor 能还原真实的 pitch 分布。去掉 CWT 后 σ 从 54.1 骤降到 42.3。

### Table 4: Energy MAE

| Method | FastSpeech | FastSpeech 2 | FastSpeech 2s |
|--------|-----------|-------------|--------------|
| MAE | 0.142 | **0.131** | 0.133 |

**说明**: 显式建模 energy 后 MAE 从 0.142 降至 0.131。

### Table 5a: Duration Accuracy / Duration 精度

| Method | Δ (ms) |
|--------|--------|
| Duration from teacher model | 19.68 |
| Duration from MFA | **12.47** |

**说明**: MFA 提取的 duration 误差比 teacher attention 低 36.6%。

### Table 5b: Duration Source CMOS

| Setting | CMOS |
|---------|------|
| FastSpeech + Duration from teacher | 0 |
| FastSpeech + Duration from MFA | **+0.195** |

**说明**: 仅替换 duration 来源就带来 +0.195 CMOS 提升。

### Table 6a: FastSpeech 2 Ablation Study / 消融实验

| Setting | CMOS |
|---------|------|
| FastSpeech 2 | 0 |
| FastSpeech 2 − energy | −0.040 |
| FastSpeech 2 − pitch | −0.245 |
| FastSpeech 2 − pitch − energy | −0.370 |

**关键发现**: Pitch 的贡献 (−0.245) 远大于 energy (−0.040)，但两者同时去掉的损失 (−0.370) 大于各自之和，说明存在协同效应。

### Table 6b: FastSpeech 2s Ablation Study / FastSpeech 2s 消融实验

| Setting | CMOS |
|---------|------|
| FastSpeech 2s | 0 |
| FastSpeech 2s − energy | −0.160 |
| FastSpeech 2s − pitch | −1.130 |
| FastSpeech 2s − pitch − energy | −1.355 |

**关键发现**: Variance 信息对 FastSpeech 2s 的影响远大于 FastSpeech 2（pitch: −1.130 vs −0.245），因为 waveform 的方差远大于 mel-spectrogram，更需要显式 variance 信息。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LJSpeech]] | 13,100 clips (~24h) | 单说话人英文有声书 | 训练/验证/测试 |

Split: 12,228 训练 / 349 验证 (LJ003) / 523 测试 (LJ001, LJ002)

### 实现细节

- **Backbone**: [[Feed-Forward Transformer]] (FFT) block
- **优化器**: Adam ($\beta_1$=0.9, $\beta_2$=0.98, $\epsilon$=10⁻⁹)
- **学习率**: Vaswani et al. (2017) schedule（warmup + inverse sqrt decay）
- **Batch Size**: 48 (FS2) / 12 (FS2s, 6 per GPU)
- **训练步数**: 160k (FS2) / 600k (FS2s)
- **硬件**: 1 NVIDIA V100 (FS2) / 2 NVIDIA V100 (FS2s)
- **Mel 参数**: 80 维, frame size=1024, hop=256, sample rate=22050
- **Vocoder**: 预训练 [[Parallel WaveGAN]]
- **文本前端**: g2p (phoneme vocabulary = 76)
- **Alignment**: [[Montreal Forced Alignment|MFA]]
- **Loss**: MAE (mel) + MSE (duration, pitch, energy)

### 可视化结果

- Pitch contour 对比 (Figure 3) 直观显示 FastSpeech 2 的 pitch 变化更自然、更接近 ground-truth
- F0 缩放控制 (Figure 4) 演示了精细的 pitch 可控性
- FastSpeech 的 pitch 过于平滑是因为 teacher distillation 损失了 variance 信息

---

## 批判性思考

### 优点
1. **方法简洁有效**: 去掉 teacher-student 蒸馏，训练管线大幅简化，训练速度提升 3 倍
2. **显式 variance 建模思想深远**: pitch/energy/duration 三路显式条件化，奠定了 NAR TTS variance 建模的范式
3. **CWT pitch 建模**: 相比直接预测 F0（如 FastPitch），CWT 分解能更好地捕捉多尺度 pitch 变化
4. **FastSpeech 2s 的探索意义**: 首次验证了全并行 text-to-waveform 的可行性

### 局限性
1. **单说话人评测**: 仅在 [[LJSpeech]] (24h 单说话人) 上实验，未验证多说话人 / zero-shot 场景的效果
2. **依赖外部工具链**: 需要 [[Montreal Forced Alignment|MFA]] 做 forced alignment、需要 F0 提取工具（pyworld），不是真正的 end-to-end
3. **Vocoder 瓶颈未解**: FastSpeech 2 (非 2s) 仍需 vocoder，MOS 上限受 vocoder 质量制约（GT Mel+PWG 只有 3.92）
4. **FastSpeech 2s 质量不如 FS2**: MOS 3.71 vs 3.83，waveform 直接生成的质量仍有差距
5. **Pitch/Energy 量化粒度固定**: 固定 256 bin 量化，可能不是最优

### 潜在改进方向
1. 用 [[Flow Matching]] / diffusion 替换 mel decoder（NaturalSpeech 2/3 的路线）
2. 去除 forced alignment 依赖，用学习到的 alignment（如 [[VITS]] 的 MAS）
3. 引入 speaker embedding 扩展到多说话人 / zero-shot
4. 用 [[HiFi-GAN]] / [[BigVGAN]] 等更强 vocoder 提升上限

### 可复现性评估
- [x] 代码开源 (Microsoft NeuralSpeech)
- [x] 预训练模型（社区实现广泛）
- [x] 训练细节完整
- [x] 数据集可获取 (LJSpeech 公开)

---

## 关联笔记

### 基于
- [[FastSpeech]]: 前作，teacher-student 蒸馏管线
- [[Transformer TTS]]: AR baseline，FastSpeech 系列的起点

### 对比
- [[Tacotron 2]]: AR TTS baseline，MOS 3.70
- [[FastSpeech]]: 前作，MOS 3.68，训练需 53h
- [[VITS]]: 后续工作，用 [[Monotonic Alignment Search|MAS]] 替代外部 alignment，端到端 flow-based

### 方法相关
- [[Duration Predictor]]: 核心组件，预测 phoneme 帧数
- [[Forced Alignment]]: 获取 ground-truth duration 的工具
- [[Montreal Forced Alignment]]: 具体使用的 alignment 工具
- [[Length Regulator]]: 将 phoneme sequence 扩展到 frame-level
- [[CWT]]: 用于 pitch spectrogram 分解
- [[Feed-Forward Transformer]]: Encoder/Decoder 的基础 block
- [[Parallel WaveGAN]]: 使用的 vocoder
- [[WaveNet]]: FastSpeech 2s waveform decoder 的基础

### 硬件/数据相关
- [[LJSpeech]]: 训练和评测数据集
- [[MOS]]: 主观评测指标

---

## 速查卡片

> [!summary] FastSpeech 2: Fast and High-Quality End-to-End Text to Speech
> - **核心**: 去除 teacher-student 蒸馏 + 引入 pitch/energy/duration variance adaptor 的 NAR TTS
> - **方法**: FFT Encoder → Variance Adaptor (duration + CWT pitch + energy) → FFT Mel Decoder
> - **结果**: MOS 3.83 (超越 Tacotron 2/Transformer TTS)，训练速度 3× faster，推理 ~48× faster than AR
> - **代码**: [Microsoft/NeuralSpeech](https://github.com/microsoft/NeuralSpeech)

---

*笔记创建时间: 2026-05-25*
