---
title: "Conditional Variational Autoencoder with Adversarial Learning for End-to-End Text-to-Speech"
method_name: "VITS"
authors: [Jaehyeon Kim, Jungil Kong, Juhee Son]
year: 2021
venue: ICML
tags: [tts, end-to-end, vae, normalizing-flow, adversarial-training, stochastic-duration]
image_source: online
arxiv_id: "2106.06103"
created: 2026-05-25
---

# 论文笔记：Conditional Variational Autoencoder with Adversarial Learning for End-to-End Text-to-Speech

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Kakao Enterprise (韩国) |
| 日期 | June 2021 |
| 项目主页 | [Demo Page](https://jaywalnut310.github.io/vits-demo/index.html) |
| 对比基线 | [[Tacotron 2]] + [[HiFi-GAN]], [[Glow-TTS]] + [[HiFi-GAN]] |
| 链接 | [arXiv](https://arxiv.org/abs/2106.06103) / [Code](https://github.com/jaywalnut310/vits) |

---

## 一句话总结

> 首个将 [[VAE]]、[[Normalizing Flow]]、[[GAN]] 对抗训练统一到单阶段并行端到端 TTS 的工作，直接从 [[Phoneme]] 序列生成波形，MOS 接近真人。

---

## 核心贡献

1. **端到端并行 TTS**: 首次实现从文本直接并行生成高质量波形，不需要预定义中间表示（如 [[Mel-Spectrogram]]），消除两阶段误差累积
2. **VAE + Flow + GAN 统一框架**: 用 [[VAE]] 连接先验与后验、[[Normalizing Flow]] 增强先验表达力、对抗训练提升波形细节质量
3. **随机时长预测器**: 基于 flow 的 [[Stochastic Duration Predictor]]，学习音素时长的分布而非点估计，生成具有自然韵律变化的语音

---

## 问题背景

### 要解决的问题

传统 TTS 系统是两阶段流水线：先生成中间表示（[[Mel-Spectrogram]] 或语言学特征），再用 [[Vocoder]] 合成波形。这种分离设计导致误差累积、训练流程复杂、无法利用学到的隐表示。

### 现有方法的局限

- **自回归模型**（[[Tacotron 2]]）：质量高但顺序生成，无法充分利用并行硬件
- **非自回归端到端尝试**（[[FastSpeech 2]]s, EATS）：需要额外提取 pitch/energy/duration 等条件信息，合成质量仍不如两阶段系统
- **确定性时长预测器**：只能产生固定韵律，无法反映 text-to-speech 的 one-to-many 本质

### 本文的动机

通过 [[VAE]] 的隐变量将文本编码器和波形解码器连接，用 [[Normalizing Flow]] 弥合先验-后验分布差距，用对抗训练保证波形级细节质量，同时用随机时长预测器捕获韵律多样性。

---

## 方法详解

### 模型架构

VITS 采用 **条件 [[VAE]] + [[Normalizing Flow]] + 对抗训练** 架构：

- **输入**: [[Phoneme]] 序列 $c_\text{text}$（通过 phonemizer 转换的 IPA 音素，夹插 blank token）
- **Posterior Encoder**: 非因果 [[WaveNet]] 残差块，从线性频谱图 $x_\text{lin}$ 编码后验 $q_\phi(z|x_\text{lin})$
- **Prior Encoder**: [[Transformer]] 文本编码器 + [[Normalizing Flow]]，从音素编码先验 $p_\theta(z|c_\text{text}, A)$
- **Decoder**: 本质上就是 [[HiFi-GAN]] V1 生成器，从隐变量 $z$ 直接生成波形 $\hat{y}$
- **Discriminator**: [[HiFi-GAN]] 的 Multi-Period Discriminator
- **Stochastic Duration Predictor**: 基于 Neural Spline Flow 的时长分布预测

### 核心模块

#### 模块 1: Posterior Encoder

**设计动机**: 从高分辨率线性频谱图中提取说话人和韵律信息，编码到隐变量空间

**具体实现**:
- 使用 16 层非因果 [[WaveNet]] 残差块（膨胀卷积 + 门控激活 + 跳跃连接）
- 输入为对数线性频谱图 $x_\text{lin}$（比 [[Mel-Spectrogram]] 分辨率更高）
- 输出通过线性投影得到正态分布的均值 $\mu_\phi$ 和方差 $\sigma_\phi$
- 隐变量维度 192 通道
- 多说话人场景下通过全局条件注入说话人信息

#### 模块 2: Prior Encoder + Normalizing Flow

**设计动机**: 纯文本编码得到的先验分布表达力不够，无法匹配含有丰富声学信息的后验分布

**具体实现**:
- **Text Encoder**: 使用带相对位置编码的 [[Transformer]]，将音素序列 $c_\text{text}$ 编码为 $h_\text{text}$
- **Normalizing Flow**: 4 层 affine coupling layer（每层含 4 个 [[WaveNet]] 残差块），设计为体积保持（Jacobian 行列式 = 1）
- 线性投影层从 $h_\text{text}$ 生成先验分布的 $\mu_\theta, \sigma_\theta$
- Flow 将简单先验分布变换为复杂分布，有效减小 [[KL Divergence]]

#### 模块 3: [[Monotonic Alignment Search]] (MAS)

**设计动机**: 需要在训练中找到文本与隐变量之间的最优单调对齐，不依赖外部对齐器

**具体实现**:
- 沿用 [[Glow-TTS]] 的 MAS 动态规划算法
- 在所有单调非跳过的对齐候选中，搜索使 ELBO 最大化的对齐矩阵 $A$
- 音素时长 $d_i = \sum_j A_{i,j}$
- 推理时不需要 MAS，直接用 [[Stochastic Duration Predictor]] 采样时长

#### 模块 4: Stochastic Duration Predictor

**设计动机**: 确定性时长预测器无法表达 text-to-speech 的 one-to-many 关系（同一文本可以有不同的节奏和时长）

**具体实现**:
- 基于 flow 的生成模型，学习给定文本条件下的时长分布
- 使用 Neural Spline Flow（单调有理二次样条）作为 coupling layer，比仿射变换更具表达力
- 通过变分反量化（variational dequantization）处理离散整数时长
- 通过变分数据增强（variational data augmentation）解决标量维度过低的问题
- 训练时使用 stop gradient 阻止梯度回传到其他模块
- 推理时通过调节噪声标准差控制节奏变化幅度

#### 模块 5: Decoder (HiFi-GAN V1)

**设计动机**: 直接复用已验证的高质量波形生成器

**具体实现**:
- 堆叠转置卷积 + [[Multi-Receptive Field Fusion]] (MRF) 模块
- MRF 将不同感受野大小的残差块输出求和
- 训练时使用 windowed generator training（窗口大小 32），只对 $z$ 的部分序列解码以节省内存
- 输入通道 192，最后一层卷积去掉 bias（混合精度稳定性）

---

## 关键公式

### 公式 1: [[ELBO|变分下界]]

$$
\log p_\theta(x|c) \geq \mathbb{E}_{q_\phi(z|x)}\Big[\log p_\theta(x|z) - \log\frac{q_\phi(z|x)}{p_\theta(z|c)}\Big]
$$

**含义**: 对不可计算的边际对数似然进行变分下界推导，训练目标是最大化此 ELBO

**符号说明**:
- $x$: 观测数据（语音波形）
- $c$: 条件（文本音素 + 对齐）
- $z$: 隐变量
- $q_\phi(z|x)$: 后验编码器参数化的近似后验
- $p_\theta(z|c)$: 先验编码器 + flow 参数化的条件先验
- $p_\theta(x|z)$: 解码器参数化的似然

### 公式 2: [[Reconstruction Loss|重建损失]]

$$
L_\text{recon} = \|x_\text{mel} - \hat{x}_\text{mel}\|_1
$$

**含义**: 用 L1 损失衡量生成波形转换到 mel 域后与真实 mel 的差距（等价于假设 Laplace 分布的最大似然估计）

**符号说明**:
- $x_\text{mel}$: 真实语音的 80 维 [[Mel-Spectrogram]]
- $\hat{x}_\text{mel}$: 解码器输出波形 $\hat{y}$ 经 [[STFT]] + mel 变换得到的频谱

### 公式 3: [[KL Divergence]] 损失

$$
L_\text{kl} = \log q_\phi(z|x_\text{lin}) - \log p_\theta(z|c_\text{text}, A)
$$

$$
z \sim q_\phi(z|x_\text{lin}) = \mathcal{N}(z;\, \mu_\phi(x_\text{lin}),\, \sigma_\phi(x_\text{lin}))
$$

**含义**: 最小化后验分布与先验分布之间的 KL 散度，迫使文本先验能够覆盖语音后验的信息

**符号说明**:
- $x_\text{lin}$: 线性尺度频谱图（比 mel 分辨率更高）
- $A$: 单调对齐矩阵
- $\mu_\phi, \sigma_\phi$: 后验编码器输出的均值和标准差

### 公式 4: [[Normalizing Flow]] 先验

$$
p_\theta(z|c) = \mathcal{N}\!\Big(f_\theta(z);\, \mu_\theta(c),\, \sigma_\theta(c)\Big) \left|\det\frac{\partial f_\theta(z)}{\partial z}\right|
$$

**含义**: 通过可逆变换 $f_\theta$ 将简单正态先验映射到复杂分布，增强先验的表达力

**符号说明**:
- $f_\theta$: normalizing flow 的可逆变换
- $\mu_\theta(c), \sigma_\theta(c)$: 从文本编码得到的基础先验参数
- 行列式项: 变量替换的 Jacobian 校正

### 公式 5: [[Monotonic Alignment Search|MAS 目标]]

$$
A = \operatorname*{arg\,max}_{\hat{A}} \log p_\theta(z|c_\text{text}, \hat{A})
$$

$$
= \operatorname*{arg\,max}_{\hat{A}} \log \mathcal{N}\!\Big(f_\theta(z);\, \mu_\theta(c_\text{text}, \hat{A}),\, \sigma_\theta(c_\text{text}, \hat{A})\Big)
$$

**含义**: 在所有单调不跳过的候选对齐中，动态规划搜索使先验似然最大的对齐矩阵

**符号说明**:
- $\hat{A}$: 候选对齐矩阵（$|c_\text{text}| \times |z|$ 维度）
- 约束：单调（monotonic）且不跳过（non-skipping）

### 公式 6: [[Stochastic Duration Predictor]] 损失

$$
\log p_\theta(d|c_\text{text}) \geq \mathbb{E}_{q_\phi(u,\nu|d,c_\text{text})}\Big[\log\frac{p_\theta(d-u,\nu|c_\text{text})}{q_\phi(u,\nu|d,c_\text{text})}\Big]
$$

**含义**: 时长预测器也是一个 [[VAE]] 结构，通过变分下界学习条件时长分布

**符号说明**:
- $d$: 音素时长（离散整数）
- $u$: 变分反量化噪声（支撑 $[0,1)$），将离散整数连续化
- $\nu$: 变分数据增强引入的随机变量，解决标量维度过低问题
- $L_\text{dur}$: 负变分下界，使用 stop gradient 隔离梯度

### 公式 7: [[GAN|对抗训练]] 损失

$$
L_\text{adv}(D) = \mathbb{E}_{(y,z)}\Big[(D(y)-1)^2 + (D(G(z)))^2\Big]
$$

$$
L_\text{adv}(G) = \mathbb{E}_z\Big[(D(G(z))-1)^2\Big]
$$

**含义**: 最小二乘 GAN 损失，判别器 $D$ 区分真实波形和生成波形

**符号说明**:
- $y$: 真实波形
- $G(z)$: 解码器从隐变量 $z$ 生成的波形
- $D$: Multi-Period Discriminator

### 公式 8: [[Feature Matching Loss]]

$$
L_\text{fm}(G) = \mathbb{E}_{(y,z)}\Big[\sum_{l=1}^{T}\frac{1}{N_l}\|D^l(y) - D^l(G(z))\|_1\Big]
$$

**含义**: 匹配判别器中间层特征，稳定对抗训练

**符号说明**:
- $T$: 判别器总层数
- $D^l$: 第 $l$ 层特征图
- $N_l$: 第 $l$ 层特征数

### 公式 9: 总损失

$$
L_\text{vae} = L_\text{recon} + L_\text{kl} + L_\text{dur} + L_\text{adv}(G) + L_\text{fm}(G)
$$

**含义**: 生成器侧总损失 = 重建 + KL + 时长 + 对抗 + 特征匹配

### 公式 10: [[Voice Conversion|语音转换]]（多说话人模式）

$$
z \sim q_\phi(z|x_\text{lin}, s)
$$

$$
e = f_\theta(z|s)
$$

$$
\hat{y} = G\!\big(f_\theta^{-1}(e|\hat{s})\big|\hat{s}\big)
$$

**含义**: 利用说话人无关的中间表示 $e$ 实现语音转换——编码源说话人 $s$ 的语音到中间空间，再用目标说话人 $\hat{s}$ 解码

**符号说明**:
- $s$: 源说话人 ID
- $\hat{s}$: 目标说话人 ID
- $e$: flow 变换后的说话人无关表示

---

## 关键图表

### Figure 1: System Overview / 系统概览

![Figure 1a: Training procedure](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x1.png)

![Figure 1b: Inference procedure](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x2.png)

**说明**: VITS 的整体架构。(a) 训练过程：Posterior Encoder 从线性频谱图编码隐变量 $z$，Prior Encoder 从文本编码先验分布，[[Normalizing Flow]] 连接两者，[[HiFi-GAN]] Decoder 从 $z$ 生成波形，Discriminator 提供对抗信号。(b) 推理过程：只需 Prior Encoder → Flow → Decoder 路径，[[Stochastic Duration Predictor]] 提供时长。绿色块表示条件先验的组件（flow、线性投影、文本编码器）。

### Figure 2: Speech Duration Variation / 时长多样性

![Figure 2a: LJ Speech duration distribution](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x3.png)

![Figure 2b: VCTK duration distribution](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x4.png)

**说明**: 不同模型合成同一文本的语音时长分布。(a) LJ Speech 数据集上，VITS 的随机时长预测器产生丰富的时长变化，而 [[Glow-TTS]] 因使用确定性时长预测器只能输出单一值。(b) VCTK 多说话人数据集上，不同说话人自然产生不同时长分布。

### Figure 3: Pitch Track Comparison / 音高轨迹对比

![Figure 3a: VITS pitch tracks](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x5.png)

![Figure 3b: Tacotron 2 pitch tracks](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x6.png)

![Figure 3c: Glow-TTS pitch tracks](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x7.png)

![Figure 3d: VITS multi-speaker pitch tracks](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x8.png)

**说明**: 对同一句 "How much variation is there?" 多次合成的 F0 轨迹对比。(a) VITS 产生丰富的 pitch 和节奏变化；(b) [[Tacotron 2]] 通过推理时保持 dropout 实现有限变化；(c) [[Glow-TTS]] 由于确定性时长几乎无变化；(d) VITS 多说话人模式下，不同说话人有截然不同的时长和音高，体现模型捕获的说话人特性。

### Figure 4: MAS Pseudocode / MAS 伪代码

**说明**: [[Monotonic Alignment Search]] 的动态规划伪代码。输入为 $|c_\text{text}| \times |z|$ 的对数似然矩阵，前向填充缓存矩阵 $Q$，然后回溯得到最优单调对齐路径。时间复杂度 $O(|c_\text{text}| \times |z|)$。

### Figure 5: Stochastic Duration Predictor Architecture / 随机时长预测器架构

![Figure 5a: Training procedure](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x9.png)

![Figure 5c: DDSConv residual block](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x11.png)

**说明**: [[Stochastic Duration Predictor]] 的详细架构。(a) 训练过程使用后验编码器和 Neural Spline Flow；(b) 推理时从先验采样；(c) 核心构建模块是膨胀深度可分离卷积 (DDSConv) 残差块。

### Figure 6: Duration Predictor Internal Components

![Figure 6a: Condition encoder](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x12.png)

![Figure 6b: Coupling layer](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x13.png)

**说明**: 随机时长预测器内部组件。(a) 条件编码器由两层 $1\times1$ 卷积 + DDSConv 残差块组成；(b) Coupling layer 使用 Neural Spline Flow（单调有理二次样条），每层产生 29 通道参数控制 10 个有理二次函数。

### Figure 7: Voice Conversion Pitch Tracks / 语音转换音高轨迹

![Figure 7: Voice conversion pitch tracks](https://ar5iv.labs.arxiv.org/html/2106.06103/assets/x14.png)

**说明**: 多说话人 VITS 的 [[Voice Conversion]] 结果。将同一段话转换为不同说话人后，pitch 轨迹呈现相似的走势但不同的音高水平，说明模型成功分离了内容信息和说话人信息。

### Table 1: LJ Speech MOS 评测

| Model | MOS (CI) |
|-------|----------|
| Ground Truth | 4.46 (+-0.06) |
| [[Tacotron 2]] + [[HiFi-GAN]] | 3.77 (+-0.08) |
| [[Tacotron 2]] + [[HiFi-GAN]] (Fine-tuned) | 4.25 (+-0.07) |
| [[Glow-TTS]] + [[HiFi-GAN]] | 4.14 (+-0.07) |
| [[Glow-TTS]] + [[HiFi-GAN]] (Fine-tuned) | 4.32 (+-0.07) |
| VITS (DDP) | 4.39 (+-0.06) |
| **VITS** | **4.43 (+-0.06)** |

**表格说明**: VITS 在 LJ Speech 上 MOS 达到 4.43，几乎持平 Ground Truth 的 4.46，超越所有两阶段系统。DDP 表示确定性时长预测器变体。

### Table 2: 消融实验

| 配置 | MOS (CI) | 说明 |
|------|----------|------|
| Ground Truth | 4.50 (+-0.06) | - |
| Baseline (VITS full) | 4.50 (+-0.06) | 完整模型 |
| w/o [[Normalizing Flow]] | 2.98 (+-0.08) | MOS 暴跌 1.52，Flow 是核心 |
| w/ [[Mel-Spectrogram]] (替代 linear spec) | 4.31 (+-0.08) | 降低 0.19，linear spec 更优 |

**关键发现**: Normalizing Flow 是 VITS 的命脉组件，去掉后质量灾难性下降（4.50 → 2.98）。使用线性频谱图作为后验编码器输入优于 mel 频谱图。

### Table 3: VCTK 多说话人 MOS

| Model | MOS (CI) |
|-------|----------|
| Ground Truth | 4.38 (+-0.07) |
| [[Tacotron 2]] + [[HiFi-GAN]] | 3.14 (+-0.09) |
| [[Tacotron 2]] + [[HiFi-GAN]] (Fine-tuned) | 3.19 (+-0.09) |
| [[Glow-TTS]] + [[HiFi-GAN]] | 3.76 (+-0.07) |
| [[Glow-TTS]] + [[HiFi-GAN]] (Fine-tuned) | 3.82 (+-0.07) |
| **VITS** | **4.38 (+-0.06)** |

**表格说明**: 多说话人场景下 VITS 直接达到 Ground Truth 水平（4.38 vs 4.38），大幅领先两阶段系统。

### Table 4: 合成速度对比

| Model | Speed (kHz) | Real-time 倍率 |
|-------|-------------|----------------|
| [[Glow-TTS]] + [[HiFi-GAN]] | 606.05 | x27.48 |
| VITS | 1480.15 | x67.12 |
| VITS (DDP) | 2005.03 | x90.93 |

**表格说明**: VITS 合成速度是 Glow-TTS + HiFi-GAN 的约 2.4 倍，因为不需要生成中间表示的模块。确定性时长预测器版本 (DDP) 更快。

### Table 5: CMOS 对比（与 Ground Truth）

| Dataset | CMOS |
|---------|------|
| [[LJSpeech]] | -0.106 |
| [[VCTK]] | -0.262 |

**表格说明**: Side-by-side 评测中 VITS 与真实语音仍有微小差距，但差距极小（LJ Speech 仅 -0.106 CMOS）。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LJSpeech]] | 13,100 clips / ~24h | 单说话人英文有声书，22 kHz | 单说话人实验 |
| [[VCTK]] | ~44,000 clips / ~44h | 109 说话人英文，44→22 kHz | 多说话人实验 |

### 实现细节

- **文本前端**: phonemizer 将文本转 IPA [[Phoneme]] 序列，每两个音素之间插入 blank token
- **音频处理**: [[STFT]] FFT=1024, window=1024, hop=256; 80 维 mel 用于重建损失
- **优化器**: AdamW ($\beta_1=0.8$, $\beta_2=0.99$, weight decay $\lambda=0.01$)
- **学习率**: 初始 $2\times10^{-4}$，每 epoch 衰减 $0.999^{1/8}$
- **Batch Size**: 64/GPU
- **训练步数**: 800k steps
- **硬件**: 4x NVIDIA V100 GPU
- **混合精度**: 使用混合精度训练
- **窗口训练**: 解码器只对 $z$ 的窗口片段（window size 32）解码，节省显存

### 关键超参数

- [[Glow-TTS]] 先验标准差: 0.333
- VITS 时长预测器噪声标准差: 0.8
- VITS 先验标准差缩放: 0.667
- [[Tacotron 2]] dropout: 0.5
- Discriminator 周期: [1, 2, 3, 5, 7, 11]

### 可视化结果

- 音高轨迹可视化（Figure 3）清楚展示 VITS 在韵律多样性上的优势
- 语音转换（Figure 7）展示了模型成功解耦内容与说话人信息的能力

---

## 批判性思考

### 优点

1. **里程碑式创新**: 首次让端到端并行 TTS 质量超过两阶段系统，这是 TTS 研究的重大突破
2. **优雅的框架设计**: VAE + Flow + GAN 三个组件各司其职——VAE 做全局结构、Flow 弥合 gap、GAN 保细节，非简单拼凑
3. **随机时长预测器设计精巧**: 用变分反量化和变分数据增强解决离散标量的 flow 建模难题，理论扎实
4. **速度优势显著**: 比两阶段系统快 2.4 倍，因为省去了中间表示生成
5. **开源且可复现**: 代码、demo 页面、清晰的超参数记录

### 局限性

1. **仍需文本预处理**: 依赖 phonemizer 做 text→phoneme 转换，对低资源语言支持有限。作者自己在结论中指出这是主要局限
2. **消融实验不够全面**: 只有 2 个消融条件（去 flow、换 mel），缺少对 stochastic duration predictor 的独立消融
3. **评测指标单一**: 主要靠 MOS，缺少客观指标（[[WER]]、[[SIM-O]] 等）；当时的评测规范尚不成熟
4. **多说话人 CMOS 差距**: VCTK 上 CMOS 为 -0.262，与真人仍有可感知差距
5. **无零样本能力**: 仅支持训练集中的说话人，无法做 in-context TTS

### 潜在改进方向

1. **去掉 phonemizer 依赖**: 后续 F5-TTS 等工作证明可以直接用 character/byte-level 输入
2. **加入零样本 TTS 能力**: 后续 VALL-E、CosyVoice 等通过 prompt 机制实现
3. **更强的时长建模**: 完全去掉显式时长预测器，用 in-context learning 隐式学习（如 F5-TTS 的 mask-and-infill）
4. **更高效的 vocoder**: 可替换为 [[Vocos]]、iSTFT-Net 等更轻量的方案（MB-iSTFT-VITS 已实践）

### 可复现性评估

- [x] 代码开源 (https://github.com/jaywalnut310/vits)
- [x] 训练细节完整（超参数、数据划分、硬件配置齐全）
- [x] 数据集可获取（LJSpeech、VCTK 均为公开数据集）
- [ ] 预训练模型（官方未提供，但社区有大量复现）

---

## 关联笔记

### 基于

- [[Glow-TTS]]: MAS 对齐算法直接来源；先验编码器设计基础
- [[HiFi-GAN]]: 解码器和判别器直接复用 HiFi-GAN V1 架构
- [[VAE]]: 整体框架基于条件 VAE
- [[Normalizing Flow]]: 增强先验分布表达力的核心技术

### 对比

- [[Tacotron 2]]: 自回归两阶段 baseline，VITS 在质量和速度上均大幅领先
- [[Glow-TTS]]: 同团队前作，VITS 在此基础上加入 VAE + GAN + 随机时长
- [[FastSpeech 2]]: NAR 端到端尝试（FastSpeech 2s），仍需额外 pitch/energy 条件

### 后续工作

- [[GPT-SoVITS]]: 将 VITS 作为第二阶段声学解码器
- [[CosyVoice]]: 继承 flow-based 先验思路，发展为零样本 TTS
- [[VITS2]]: 直接后续改进版本

### 方法相关

- [[Stochastic Duration Predictor]]: VITS 提出的核心创新组件
- [[Monotonic Alignment Search]]: 训练时对齐搜索算法
- [[WaveNet]]: 后验编码器和 flow coupling layer 的基础网络
- [[Transformer]]: 文本编码器架构
- [[Feature Matching Loss]]: 稳定对抗训练的辅助损失

### 数据相关

- [[LJSpeech]]: 单说话人英文 TTS 标准评测集
- [[VCTK]]: 多说话人英文 TTS 评测集
- [[MOS]]: 主观自然度评分指标

---

## 速查卡片

> [!summary] VITS: Conditional VAE with Adversarial Learning for End-to-End TTS
> - **核心**: 首个 VAE + Normalizing Flow + GAN 统一的端到端并行 TTS，质量超越两阶段系统
> - **方法**: 后验编码器(WaveNet) + 先验编码器(Transformer+Flow) + HiFi-GAN 解码器 + 随机时长预测器
> - **结果**: LJ Speech MOS 4.43（GT 4.46），VCTK MOS 4.38（GT 4.38），速度 x67 real-time
> - **代码**: https://github.com/jaywalnut310/vits

---

*笔记创建时间: 2026-05-25*
