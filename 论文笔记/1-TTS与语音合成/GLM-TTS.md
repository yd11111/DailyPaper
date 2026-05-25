---
title: "GLM-TTS Technical Report"
method_name: "GLM-TTS"
authors: [Jiayan Cui, Zhihan Yang, Naihan Li, Jiankun Tian, Xingyu Ma, Yi Zhang, Guangyu Chen, Runxuan Yang, Zijian Huang, Yuqing Cheng, Yizhi Zhou, Guochen Yu, Xiaotao Gu, Jie Tang]
year: 2024
venue: arXiv
tags: [tts, zero-shot-tts, grpo, reinforcement-learning, speech-tokenizer, vocoder, lora, production-tts]
image_source: online
arxiv_html: https://arxiv.org/html/2512.14291v1
created: 2026-05-25
---

# 论文笔记：GLM-TTS Technical Report

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Zhipu AI; Tsinghua University |
| 日期 | December 2024 |
| 项目主页 | [audio.z.ai](http://audio.z.ai) |
| 对比基线 | [[CosyVoice]] / [[CosyVoice 2]] / [[CosyVoice 3]] / [[F5-TTS]] / [[Seed-TTS]] / [[MaskGCT]] / [[Spark-TTS]] / [[FireRedTTS]] / [[FireRedTTS-2]] / [[VoxCPM]] / [[IndexTTS 2]] |
| 链接 | [arXiv](https://arxiv.org/abs/2512.14291) / [Code](https://github.com/zai-org/GLM-TTS) / [HuggingFace](https://huggingface.co/zai-org/GLM-TTS) |

---

## 一句话总结

> 智谱 AI 的产品级 TTS 系统，仅用 100k 小时数据，通过优化 Whisper-VQ 语音 tokenizer + [[GRPO]] 多奖励强化学习 + [[LoRA]] 音色定制 + Vocos2D 声码器，在 [[Seed-TTS-eval]] 上达到开源 SOTA。

---

## 核心贡献

1. **优化 Speech Tokenizer**: 基于 [[GLM-4-Voice]] 的 Whisper-VQ，将 token 率从 12.5 Hz 提升到 25 Hz，词表从 16k 扩到 32k，新增 Pitch Estimator 模块改善韵律建模
2. **GRPO 多奖励强化学习**: 首次将 [[GRPO]] 应用于 TTS 领域，融合 [[CER]]、SIM、情感、笑声四维奖励，配合动态采样和自适应梯度裁剪
3. **LoRA 音色定制**: 仅微调 ~15% 骨干参数，用约 1 小时单说话人数据即可完成高质量音色克隆，成本降低 80%
4. **Phoneme-in 混合输入**: 文本+音素混合输入机制，解决中文多音字和生僻字发音问题，PER 从 13.23% 降至 5.14%
5. **Vocos2D 声码器**: 用 2D 卷积替换 1D 卷积做子带处理，引入 DiT 风格残差连接和 Discriminator Augmentation，MOS 从 3.58 提升至 4.16

---

## 问题背景

### 要解决的问题
构建一个**产品级** TTS 系统，需要同时满足效率、可控性和高保真度。现有系统在以下方面存在短板：
- 零样本 voice cloning 需要大量数据（[[CosyVoice 3]] 用 1M 小时，[[FireRedTTS-2]] 用 1.1M 小时）和长参考音频
- 中文多音字/生僻字发音错误率高
- TTS 领域缺乏有效的 RL 对齐方法
- 全模型微调的音色定制成本过高

### 现有方法的局限
- **AR 零样本 TTS**（[[VALL-E]]、[[Spark-TTS]]）：依赖 [[EnCodec]]/[[SoundStream]] 等 codec 的离散 token，重建质量受限于 codec
- **Flow Matching / Diffusion NAR**（[[F5-TTS]]）：生成质量好但缺乏自回归的灵活性
- **混合 AR-NAR**（[[CosyVoice]]、[[Seed-TTS]]）：需要海量训练数据

### 本文的动机
继承 [[CosyVoice]] 的两阶段范式（Text→Token AR + Token→Waveform Diffusion），但在 tokenizer、RL 对齐、声码器、发音控制、音色定制五个维度做系统优化，用**仅 100k 小时数据**达到可比甚至超越的效果。

---

## 方法详解

### 模型架构

GLM-TTS 采用 **两阶段生成架构**（类 [[CosyVoice]] 范式）：

- **Stage 1 — Text-to-Token AR**: 自回归语言模型，输入文本（或混合音素+文本），输出 25 Hz 的离散 [[Speech Tokenizer|speech token]] 序列
- **Stage 2 — Token-to-Waveform Diffusion**: 基于 diffusion 的声码器（Vocos2D），将 speech token 转换为 32 kHz 波形
- **总参数**: 1.5B

### 核心模块

#### 模块 1: 数据处理流水线

**设计动机**: 产品级系统需要高质量训练数据，通过多步清洗确保 [[WER]] < 5%。

**具体实现**:
1. **语音标准化 + 粗切分**: 统一为 WAV 格式，用 pyannote [[VAD]] 切分语音段，拼接为 ~10 分钟片段
2. **源分离 + 去噪**: Mel-Band Roformer 分离背景音乐/噪声，再用专有去噪器抑制残余噪声
3. **说话人分离 + 拼接**: pyannote [[Speaker Diarization]] 分离多说话人，振幅归一化后拼接到最长 40 秒
4. **WER 过滤**: 双重 ASR 校验——中文用 [[Paraformer]] + [[SenseVoice]]，英文用 [[Whisper]] + Reverb，仅保留 WER < 5% 的数据
5. **标点优化**: 通过 [[Forced Alignment]] 获取字级时长，阈值 $\mu + 2.6\sigma^2$ 判断是否在间隔处插入标点
6. **特征提取**: 提取 speaker embedding 和 speech token

#### 模块 2: Speech Tokenizer（优化 Whisper-VQ）

**设计动机**: [[GLM-4-Voice]] 的 Whisper-VQ tokenizer 在方言和韵律建模上不足。

**具体实现**:
- **Token 率**: 从 12.5 Hz 加倍到 25 Hz，词表从 16k 扩至 32k
- **Pitch Estimator (PE) 模块**: 新增 F0 约束模块，改善克隆语音的韵律对齐
- **非因果架构**: 移除 block attention 和因果卷积，改用标准卷积（离线推理场景更适合）
- **扩展训练数据**: 加入大规模方言数据和高质量歌声数据

#### 模块 3: Text Tokenizer

**设计动机**: 控制 text token 与 speech token 的长度比在合理范围。

**具体实现**:
- 去除包含两个以上汉字的 token，使文本粒度更细
- 启发式约束 speech-to-text 长度比在 $[2, 20]$ 区间
- 更细的粒度让信息密度归一化，提升 25 Hz token 率下的 AR 生成质量

#### 模块 4: GRPO 多奖励 RL 对齐

**设计动机**: 将 LLM 领域的 [[GRPO]] 引入 TTS，同时优化发音准确率、说话人相似度、情感表现力和笑声自然度。

**具体实现**:
1. **多维正则化奖励**: 四个奖励维度——[[CER]] 奖励、SIM 奖励、情感奖励、笑声奖励。采用"单奖励正则化 → 加权融合 → 整体正则化"的三级处理流程
2. **动态采样**: 当 batch 内奖励分布过于均匀时自动重采样（最多 3 次），避免低质量样本浪费训练步
3. **自适应梯度裁剪**: 借鉴 [[DAPO]] 的 Clip-Higher 技术，$\epsilon_{high} > \epsilon_{low}$，训练早期收紧裁剪防止 reward hacking，后期放松促进探索

#### 模块 5: LoRA 音色定制

**设计动机**: 全模型微调成本高、数据需求大，不适合产品化。

**具体实现**:
- 微调 ~15% 骨干参数（[[LoRA]] 适配器），约 100 epochs
- 仅需约 1 小时单说话人音频
- 0.3%-5% 参数比例探索发现效果有限，≥15% 才有足够泛化能力
- 成本比全模型微调降低约 80%

#### 模块 6: Phoneme-in 混合输入

**设计动机**: 中文多音字和生僻字在纯文本输入下发音错误率高。

**具体实现**:
- **词表构建**: 为多音字和生僻字建立专用音素词表，支持动态维护扩展
- **混合训练**: 标准字符以概率 $p=0.2$ 触发音素替换，替换比例从 $\mathcal{U}(0, 0.5)$ 均匀采样；多音字/生僻字保留原文不替换
- **推理**: 全句先过 G2P → 完整 phoneme_list → 遍历文本，多音字/生僻字替换为对应音素 → 混合输入

#### 模块 7: Vocos2D 声码器

**设计动机**: 原始 [[Vocos]] 的 1D 卷积缺乏频率间通信能力，且 Multi-Period Discriminator 在线性频谱上性能下降。

**具体实现**:
- **2D 卷积**: 初始点卷积 + 可学习 per-frequency embedding 实现频率间通信
- **DiT 风格残差**: ConvNeXt backbone block 中加入来自输入 Mel $X_{in}$ 的 shortcut（通过线性层回归后在 inverted bottleneck 阶段加入）
- **Discriminator 改进**: 移除 Multi-Period Discriminator（在高频率分辨率频谱上反而降低性能），仅保留 Multi-Resolution Discriminator，加入 Discriminator Augmentation（随机 $\pm 6$ dB 响度调整、随机采样偏移、随机相位旋转）
- **歌声数据增强**: 训练数据加入开源歌声数据，扩展 32 kHz 宽带合成的 pitch range 覆盖

---

## 关键公式

### 公式 1: [[Forced Alignment|标点插入阈值]]

$$
\text{threshold} = \mu + 2.6 \cdot \sigma^2
$$

**含义**: 当相邻字符间的发音间隔超过该阈值时，在此处插入或保留标点符号。

**符号说明**:
- $\mu$: 字符发音时长的均值
- $\sigma^2$: 字符发音时长的方差

### 公式 2: [[GRPO|自适应梯度裁剪约束]]

$$
\epsilon_{high} > \epsilon_{low}
$$

**含义**: Clip-Higher 机制中，高概率 token 的裁剪阈值严于低概率 token，鼓励模型生成低概率 token 以增强人声自然感。

**符号说明**:
- $\epsilon_{high}$: 高概率 token 的裁剪阈值（初始值 0.3）
- $\epsilon_{low}$: 低概率 token 的裁剪阈值（初始值 0.2）
- 两者均随训练步数线性增长

### 公式 3: [[Zero-shot TTS|Phoneme 替换概率]]

$$
p = 0.2, \quad r \sim \mathcal{U}(0,\; 0.5)
$$

**含义**: 训练时以概率 $p$ 触发音素替换，触发后替换比例 $r$ 从均匀分布中采样。

**符号说明**:
- $p = 0.2$: 触发概率
- $r$: 实际替换比例，均匀分布于 $[0, 0.5]$

### 公式 4: 笑声奖励函数

$$
R_{\text{laughter}} = \begin{cases} 1, & \text{if ASR transcribes deletion (empty string)} \\ 0, & \text{if ASR transcribes corresponding text} \end{cases}
$$

**含义**: 当文本包含 $\geq 2$ 个连续笑声词时，若 ASR 将该段转录为空（说明模型确实在"笑"而非读文字），则奖励为 1。

**符号说明**:
- $R_{\text{laughter}}$: 笑声奖励值
- $\lambda_{\text{laughter}}$: 笑声奖励权重（实验中测试了 2, 5, 10）

---

## 关键图表

### Figure 1: Overall Architecture / 系统架构

![Figure 1](https://arxiv.org/html/2512.14291v1/x2.png)

**说明**: GLM-TTS 的整体两阶段架构。Stage 1 为 Text-to-Token 自回归模型，将文本（或混合音素+文本）转换为 25 Hz 离散 speech token；Stage 2 为 Token-to-Waveform 扩散模型（Vocos2D），将 speech token 解码为 32 kHz 波形。

### Figure 2: Data Processing Pipeline / 数据处理流水线

![Figure 2](https://arxiv.org/html/2512.14291v1/x3.png)

**说明**: 多步数据处理流程：从原始音频出发，经过标准化、粗切分、源分离去噪、说话人分离拼接、[[WER]] 过滤（中文 [[Paraformer]]+[[SenseVoice]]、英文 [[Whisper]]+Reverb，阈值 < 5%）、标点优化、特征提取。

### Figure 3: GLM-TTS-GRPO Framework / GRPO 强化学习框架

![Figure 3](https://arxiv.org/html/2512.14291v1/x4.png)

**说明**: [[GRPO]] 多奖励强化学习框架。包含四个奖励维度（[[CER]]、SIM、Emotion、Laughter），采用"单奖励正则化 → 加权融合 → 整体正则化"的三级处理，配合动态采样和 Clip-Higher 自适应梯度裁剪。

### Figure 4: Vocos2D Generator Architecture / Vocos2D 生成器架构

![Figure 4](https://arxiv.org/html/2512.14291v1/x5.png)

**说明**: Vocos2D 生成器架构。FC 为全连接层，PwConv 为点卷积，DwConv 为深度卷积。核心改进是将 1D 卷积替换为 2D 卷积实现子带处理，加入来自输入 Mel $X_{in}$ 的 DiT 风格残差 shortcut。

### Figure 5: Vocos vs Vocos2D Loss / 损失函数对比

![Figure 5](https://arxiv.org/html/2512.14291v1/x6.png)

**说明**: 对比 [[Vocos]]（上）和 Vocos2D（下）的损失函数设计。Vocos2D 移除了 Multi-Period Discriminator (MPD)，仅保留 Multi-Resolution Discriminator (MRD)，并在 MRD 前加入 Discriminator Augmentation (DA)——包含随机响度、采样偏移、相位旋转三种数据增强。

### Table 1: Speech Tokenizer ASR 性能（WER/CER %，越低越好）

| Tokenizer | Libri Clean | Libri Other | 四川话 | 胶辽官话 | 台湾华语 | 粤语 | 上海话 |
|---|---|---|---|---|---|---|---|
| GLM4-Voice-tokenizer | 4.90 | 2.10 | 54.11 | 14.04 | 49.09 | 46.81 | 72.06 |
| **GLM-TTS-tokenizer** | **4.51** | **2.12** | **24.40** | **9.11** | **16.92** | **7.27** | **19.15** |

**表格说明**: GLM-TTS tokenizer 在方言识别上大幅优于 [[GLM-4-Voice]] tokenizer，尤其粤语 WER 从 46.81% 降至 7.27%，上海话从 72.06% 降至 19.15%。

### Table 2: Seed-TTS-eval zh 上的 Tokenizer 对比

| Tokenizer | SIM ↑ | CER ↓ |
|---|---|---|
| GLM4-Voice-tokenizer | 75.2 | 1.44 |
| **GLM-TTS-tokenizer** | **76.1** | **1.03** |

**表格说明**: 在 [[Seed-TTS-eval]] 中文测试集上，新 tokenizer 在说话人相似度和字符错误率上均有提升。

### Table 3: Seed-TTS-eval 完整基线对比

| Model | Params | Open Source | test-zh CER ↓ | test-zh SIM ↑ | test-en WER ↓ | test-en SIM ↑ |
|---|---|---|---|---|---|---|
| MegaTTS3 | 0.5B | No | 1.52 | 79.0 | 2.79 | 77.1 |
| [[DiTAR]] | 0.6B | No | 1.02 | 75.3 | 1.69 | 73.5 |
| [[CosyVoice 3]] | 1.5B | No | 1.12 | 78.1 | 2.22 | 72.0 |
| [[Seed-TTS]] | - | No | 1.12 | 79.6 | 2.25 | 76.2 |
| MiniMax-Speech | - | No | 0.83 | 78.3 | 1.65 | 69.2 |
| [[F5-TTS]] | 0.3B | Yes | 1.53 | 76.0 | 2.00 | 67.0 |
| [[MaskGCT]] | - | Yes | 2.27 | 77.4 | 2.62 | 71.7 |
| [[CosyVoice]] | 0.3B | Yes | 3.63 | 72.3 | 4.29 | 60.9 |
| [[CosyVoice 2]] | 0.5B | Yes | 1.38 | 75.7 | 3.09 | 65.9 |
| [[CosyVoice 3]] | 0.5B | Yes | 1.16 | 78.0 | 2.02 | 71.8 |
| [[Spark-TTS]] | 0.5B | Yes | 1.54 | 66.0 | 3.14 | 57.3 |
| [[FireRedTTS]] | 0.5B | Yes | 1.51 | 63.5 | 3.82 | 46.0 |
| [[FireRedTTS-2]] | - | Yes | 1.14 | 73.6 | 1.95 | 66.5 |
| [[Qwen2.5-Omni]] | 7B | Yes | 1.70 | 75.2 | 2.72 | 63.2 |
| [[OpenAudio-s1]] | 0.5B | Yes | 1.18 | 68.5 | 1.94 | 55.0 |
| [[IndexTTS 2]] | 1.5B | Yes | 1.03 | 76.5 | 2.23 | 70.6 |
| [[VibeVoice]] | 1.5B | Yes | 1.16 | 74.4 | 3.04 | 68.9 |
| [[HiggsAudio-v2]] | 3B | Yes | 1.50 | 74.0 | 2.44 | 67.7 |
| [[VoxCPM]] | 0.5B | Yes | 0.93 | 77.2 | 1.85 | 72.9 |
| **GLM-TTS** | **1.5B** | **Yes** | **1.03** | **76.1** | **2.23** | **67.2** |
| **GLM-TTS_RL** | **1.5B** | **Yes** | **0.89** | **76.4** | **1.91** | **68.1** |

**表格说明**: GLM-TTS 在中文 CER 上达到开源 SOTA 水平（1.03%，与 [[IndexTTS 2]] 并列），GRPO RL 后进一步降至 0.89%。英文因训练数据不足（不到中文一半），SIM 偏低（68.1），但 WER 改善明显。在闭源模型中，MiniMax-Speech CER 最优（0.83%），[[Seed-TTS]] SIM 最优（79.6）。

### Table 4: Pretrain-GRPO 消融（Clip-Higher + Dynamic Sampling）

| Model | CER ↓ | SIM ↑ | EMO ↑ |
|---|---|---|---|
| Pretrain-base | 2.05 | 80.0 | 0.525 |
| Pretrain-GRPO | 1.99 | 80.3 | 0.565 |
| Pretrain-GRPO_c (+ Clip-Higher) | 1.93 | 80.4 | 0.660 |
| Pretrain-GRPO_d (+ Dynamic Sampling) | 1.91 | 80.8 | 0.440 |

**关键发现**: Clip-Higher 在三个维度上均正向提升。Dynamic Sampling 改善 CER 和 SIM 但降低 EMO——因为情感奖励分布双峰（接近 0 和 1），重采样对方差过小的判断失效。

### Table 5: SFT-GRPO 超参消融（动态 $\epsilon_h$、$\epsilon_l$、$T$）

| Model | T | $\epsilon_h$ | $\epsilon_l$ | CER ↓ | SIM ↑ | EMO ↑ |
|---|---|---|---|---|---|---|
| SFT-base | - | - | - | 2.13 | 76.1 | 0.695 |
| SFT-GRPO* (frozen) | - | - | - | 2.21 | 76.3 | 0.720 |
| SFT-GRPO | 1.5 | 0.5 | 0.4 | 2.09 | 76.7 | 0.705 |
| SFT-GRPO | 2 | 1 | 0.4 | 2.16 | 75.5 | 0.790 |
| SFT-GRPO | 3 | 1 | 0.4 | 2.21 | 78.1 | 0.885 |

**关键发现**: 更激进的超参（大 $T$、大 $\epsilon_h$）可大幅提升情感表现力（EMO 0.695→0.885），但会牺牲发音准确率和说话人相似度，存在 reward hacking 风险。需根据产品需求权衡。

### Table 6: 笑声奖励权重消融

| Model | $\lambda_{\text{laughter}}$ | CER ↓ | SIM ↑ | EMO ↑ |
|---|---|---|---|---|
| SFT-base | - | 3.11 | 76.3 | 0.44 |
| SFT-GRPO | 2 | 2.86 | 74.6 | 0.64 |
| SFT-GRPO | 5 | 3.18 | 74.8 | 0.66 |
| SFT-GRPO | 10 | 3.06 | 74.8 | 0.72 |

**关键发现**: 增大 $\lambda_{\text{laughter}}$ 提升 EMO 但降低 SIM——笑声音色与说话音色差异大，ASR 也无法正确转录笑声。这体现了多目标优化中不同奖励之间的 trade-off。

### Table 7: Phoneme-in 消融（内部难例数据集）

| Model Settings | Input Modality | PER ↓ |
|---|---|---|
| GLM-TTS (w/o Phoneme-in) | Text Only | 13.23 |
| **GLM-TTS (w/ Phoneme-in)** | **Hybrid (Text + Phoneme)** | **5.14** |

**表格说明**: 在高密度多音字/生僻字的难例数据集上，Phoneme-in 将发音错误率从 13.23% 降至 5.14%，降幅 61%。

### Table 8: Vocos vs Vocos2D 声码器对比

| 指标 | GT | [[Vocos]] | Vocos2D |
|---|---|---|---|
| NISQA ↑ | 3.47 | 3.16 | **3.40** |
| [[UTMOS]] ↑ | 2.11 | 1.87 | **1.91** |
| Ab. Aes.-PQ ↑ | 7.68 | 7.56 | **7.64** |
| [[MOS]] ↑ | 4.77 | 3.58 | **4.16** |

**关键发现**: Vocos2D 在所有指标上优于原始 [[Vocos]]，尤其主观 [[MOS]] 从 3.58 大幅提升至 4.16（+0.58），接近 GT 的 4.77。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| 内部中文数据 | ~100k 小时 | 经多步清洗，WER < 5% | 训练 |
| 内部英文数据 | < 50k 小时 | 不到中文一半 | 训练 |
| 方言数据 | 大规模 | 四川话/胶辽官话/台湾华语/粤语/上海话 | Tokenizer 训练 |
| 歌声数据 | 开源 | 扩展 pitch range | Tokenizer + Vocos2D 训练 |
| [[Seed-TTS-eval]] | test-zh + test-en | 零样本 TTS 标准评测集 | 评测 |
| 内部难例集 | - | 高密度多音字/生僻字 | Phoneme-in 评测 |

### 实现细节

- **Backbone**: 1.5B 参数 LLM（GLM 架构）
- **Speech Token Rate**: 25 Hz
- **Speech Token Vocab**: 32k
- **波形采样率**: 32 kHz
- **GRPO 初始超参**: $\epsilon_h = 0.3$, $\epsilon_l = 0.2$, $T = 1$（线性增长）
- **LoRA 微调**: ~15% 骨干参数，~100 epochs，~1 小时单说话人数据
- **ASR 评测**: 中文用 [[Paraformer]]+[[SenseVoice]]，英文用 [[Whisper]]+Reverb
- **SIM 评测**: [[WavLM]]-large based speaker embedding

### 可视化结果

- Tokenizer 优化后方言 WER 降幅显著（粤语 46.81%→7.27%，上海话 72.06%→19.15%）
- GRPO RL 后 CER 和 SIM 同时改善，说明多奖励融合有效
- Vocos2D 的 MOS 提升 +0.58 是声码器层面的重大改进

---

## 批判性思考

### 优点
1. **数据效率突出**: 仅 100k 小时就达到可比水平，远低于 [[CosyVoice 3]]（1M 小时）和 [[FireRedTTS-2]]（1.1M 小时），说明方法设计的有效性
2. **GRPO 在 TTS 的探索有价值**: 首次系统验证了多维奖励 RL 在 TTS 上的可行性，Clip-Higher 和 Dynamic Sampling 的消融细致
3. **工程完整性强**: 从数据清洗到推理部署，覆盖产品级 TTS 全链路，参考价值高
4. **开源**: 代码和模型均开源，可复现性好

### 局限性
1. **英文能力偏弱**: test-en SIM 仅 68.1，远低于 [[Seed-TTS]]（76.2）和 MegaTTS3（77.1），作者归因于训练数据不足但未给出改善路径
2. **GRPO 公式缺失**: 论文未给出完整的 GRPO 目标函数和多奖励融合公式，只做了描述性说明，难以精确复现
3. **消融不够对称**: Pretrain-GRPO 消融用的内部评测集数据，与 Table 3 的 [[Seed-TTS-eval]] 数字不直接可比
4. **Vocos2D 对比不充分**: 仅与原始 [[Vocos]] 对比，缺乏与 [[HiFi-GAN]] v2、BigVGAN 等强声码器基线的对比
5. **笑声/情感奖励与 CER/SIM 的 trade-off 明显**: Table 5-6 显示激进情感优化会损害发音和相似度，产品场景下如何平衡未充分讨论

### 潜在改进方向
1. 扩充英文训练数据，或探索跨语言迁移学习
2. 公开完整 GRPO for TTS 的数学形式化，方便学术界跟进
3. 多目标 Pareto 优化替代手动调权重
4. 声码器对比加入 BigVGAN 等更强基线

### 可复现性评估
- [x] 代码开源（github.com/zai-org/GLM-TTS）
- [x] 预训练模型（HuggingFace: zai-org/GLM-TTS）
- [ ] 训练细节完整（GRPO 完整公式缺失）
- [ ] 数据集可获取（内部数据，不公开）

---

## 关联笔记

### 基于
- [[GLM-4-Voice]]: Speech tokenizer 基础（Whisper-VQ）
- [[CosyVoice]]: 两阶段范式（Text→Token AR + Token→Wav Diffusion）
- [[GRPO]]: 强化学习对齐方法（源自 DeepSeek-Math）
- [[DAPO]]: Clip-Higher 和 Dynamic Sampling 技术

### 对比
- [[CosyVoice 3]]: 同范式但用 1M 小时数据，GLM-TTS 仅 100k 小时
- [[Seed-TTS]]: 闭源 SOTA，SIM=79.6 vs GLM-TTS 76.4
- [[VoxCPM]]: 开源 CER SOTA（0.93），GLM-TTS_RL 为 0.89
- [[F5-TTS]]: NAR Flow Matching 范式，0.3B 参数
- [[IndexTTS 2]]: 同等 CER（1.03），SIM 稍高（76.5）

### 方法相关
- [[Vocos]]: Vocos2D 的基础声码器
- [[LoRA]]: 音色定制的参数高效微调方法
- [[Speech Tokenizer]]: 离散语音表示
- [[Forced Alignment]]: 标点优化中使用
- [[Speaker Diarization]]: 数据处理中的说话人分离
- [[Whisper]]: 英文数据 WER 过滤 ASR
- [[Paraformer]]: 中文数据 WER 过滤 ASR
- [[SenseVoice]]: 中文数据 WER 过滤的第二路 ASR

### 硬件/数据相关
- [[Seed-TTS-eval]]: 主要评测集
- [[WavLM]]: Speaker embedding 计算

---

## 速查卡片

> [!summary] GLM-TTS Technical Report
> - **核心**: 智谱 AI 产品级 TTS，100k 小时数据，GRPO 多奖励 RL 对齐
> - **方法**: 两阶段 AR+Diffusion，优化 Whisper-VQ tokenizer（25Hz/32k），Vocos2D 声码器，LoRA 音色定制，Phoneme-in 发音控制
> - **结果**: Seed-TTS-eval zh CER=0.89%/SIM=76.4（RL 后），Vocos2D MOS 4.16
> - **代码**: [github.com/zai-org/GLM-TTS](https://github.com/zai-org/GLM-TTS)

---

*笔记创建时间: 2026-05-25*
