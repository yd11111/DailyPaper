---
title: "VibeVoice Technical Report"
method_name: "VibeVoice"
authors: [Zhiliang Peng, Jianwei Yu, Wenhui Wang, Yaoyao Chang, Yutao Sun, Li Dong, Yi Zhu, Weijiang Xu, Hangbo Bao, Zehua Wang, Shaohan Huang, Yan Xia, Furu Wei]
year: 2025
venue: arXiv
tags: [tts, long-form-tts, multi-speaker, next-token-diffusion, speech-tokenizer, llm-tts]
zotero_collection: 1-TTS与语音合成
image_source: online
arxiv_html: https://arxiv.org/html/2508.19205v1
created: 2026-05-19
---

# 论文笔记：VibeVoice Technical Report

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft Research |
| 日期 | Aug 2025 |
| 项目主页 | <https://microsoft.github.io/VibeVoice> |
| 代码 | <https://github.com/microsoft/VibeVoice> |
| HuggingFace | <https://huggingface.co/collections/microsoft/vibevoice-68a2ef24a875c44be47b034f> |
| Demo | <https://aka.ms/VibeVoice-Demo> |
| 对比基线 | [[CosyVoice]] · [[F5-TTS]] · [[Seed-TTS]] · [[Spark-TTS]] · [[MaskGCT]] · [[FireRedTTS]] · NotebookLM · Mooncast · Higgs Audio V2 · Sesame CSM · Nari Dia · ElevenLabs v3 · Gemini 2.5 Pro TTS |
| 链接 | [arXiv](https://arxiv.org/abs/2508.19205) / [HTML](https://arxiv.org/html/2508.19205v1) |

---

## 一句话总结

> 用 [[Next-Token Diffusion]] + 7.5 Hz 超低码率连续 [[VAE|σ-VAE]] 语音 tokenizer，把 LLM 上下文塞进 64K 就能在单条 prompt 内合成 90 分钟、4 个说话人的播客级长对话。

---

## 核心贡献

1. **超低帧率连续语音 tokenizer**: 提出基于 σ-VAE 的 [[Acoustic Tokenizer]]，从 24 kHz 输入做 3200× 下采样，得到 7.5 Hz 单维连续 latent；相比 [[EnCodec]] 同等保真度下码率压缩 80×（300 → 7.5 token/s 量级），在 [[LibriTTS]] test-clean 上 PESQ 3.07、UTMOS 4.18 超过 [[DAC]]、[[SpeechTokenizer]]、[[WavTokenizer]]。
2. **Next-Token Diffusion 框架直接搬到 TTS**: 用 [[Qwen2.5]] (1.5B / 7B) 当主干，在每个 token 的 hidden state 上挂一个 4 层 [[Diffusion Head]] 预测连续 acoustic VAE 特征，省掉 RVQ + 多码本展开的复杂结构，端到端 AR 建模长序列。
3. **64K 上下文 = 90 分钟、4 说话人**: 课程学习把 LLM 上下文从 4K 扩到 64K；语音 token : 文本 token ≈ 2 : 1，64K 足以装下 90 min 多角色播客；在 1 小时长对话主观评测上 Realism / Richness / Preference 均超过 ElevenLabs v3 alpha 与 Gemini 2.5 Pro TTS。

---

## 问题背景

### 要解决的问题

如何在**单次推理**中稳定生成**长篇（数十分钟～90 分钟）、多说话人（≥4）、内容感知**的对话语音（播客 / 多人有声书），同时保持自然 turn-taking 与音色一致性。

### 现有方法的局限

- 拼接式合成（每句单独合成再拼）破坏 turn-taking 与韵律连贯。
- 已有长对话方案要么**闭源**（NotebookLM、Gemini 2.5 Pro TTS），要么**生成长度 / 稳定性受限**：CoVoMix2、Sesame CSM、Mooncast、MOSS-TTSD 都难以稳定突破数分钟尺度。
- 现有离散 [[Audio Codec|codec]]（[[EnCodec]] 75-300 token/s、[[DAC]] 100 token/s）token 率太高，AR 建模 90 分钟语音上下文动辄数十万 token，硬上 LLM 几乎不可行。
- 单码本低码率方案（[[WavTokenizer]] 40 / 75 Hz）虽然降到 ~75 token/s，但仍是离散 RVQ 路线，且重建保真度在 test-other 上掉得比较厉害。

### 本文的动机

- **降码率是长音频的命门**：把 token 率从 75-300 Hz 压到 7.5 Hz，10-40× 缩短序列；语音 token : 文本 token ≈ 2 : 1，使 LLM 一次能"读懂并生成"完整长对话剧本。
- **continuous + diffusion 比离散 + RVQ 更省事**：避开 RVQ 多码本展开 / coarse-to-fine 多 stage 的工程负担，沿用 [[Next-Token Diffusion]]（来自 LatentLM）做"AR 输出 hidden、扩散头解码连续 VAE"的统一形态。
- **架构极简化**：voice prompt 与 text script **直接拼成单一序列**喂 LLM，不再额外搞 cross-attention 或独立 speaker encoder，统一交给预训练 LLM 去理解角色与上下文。

---

## 方法详解

### 模型架构

VibeVoice 采用 **decoder-only LLM + token-level [[Diffusion Head]]** 架构：

- **输入序列**: $X = [\texttt{Speaker}_1{:}\,\bm{z}_1, \dots, \texttt{Speaker}_N{:}\,\bm{z}_N] + [\texttt{Speaker}_1{:}\,T_1, \dots, \texttt{Speaker}_N{:}\,T_N]$
  - 前段：每个说话人的 voice prompt 经过 [[Acoustic Tokenizer]] 得到 acoustic latent $\bm{z}_k$
  - 后段：每段台词的文本脚本 $T_k$（用 BPE tokenizer，[[Qwen2.5]] 自带）
  - 用 `Speaker_k` 角色标识符交错；同一段生成语音同时被 acoustic + semantic tokenizer 编码成 hybrid 表示供 AR 建模
- **Backbone**: [[Qwen2.5]]（1.5B 或 7B），保持原参数化，curriculum 把上下文从 4 096 扩到 65 536
- **Token 化双轨**: 
  - [[Acoustic Tokenizer]]（连续 σ-VAE，340M encoder + 340M decoder，7.5 Hz / 单 latent）— 负责声学还原
  - [[Semantic Tokenizer]]（与 Acoustic encoder 镜像但去掉 VAE，[[ASR]] proxy 训练）— 负责内容对齐
- **生成头**: 4 层轻量 [[Diffusion Head]]，输入是 LLM 当前 token 的 hidden $\bm{h}_i$，输出是去噪后的 acoustic latent $\bm{z}_{a,i}$，用 [[Classifier-Free Guidance|CFG]] 引导，[[DPM-Solver|DPM-Solver++]] 跑 10 步采样
- **解码**: 预测出的连续 acoustic latent 序列 → 冻结的 acoustic decoder → 24 kHz 波形
- **训练参数**: 仅 LLM + diffusion head 可学；两个 tokenizer 全程冻结

### 核心模块

#### 模块 1：Acoustic Tokenizer（σ-VAE 连续语音编解码）

**设计动机**: 把高码率 [[Audio Codec|离散 codec]] 换成**单维连续 latent**，让"语音 token : 文本 token ≈ 2 : 1"，同时绕开离散 [[RVQ]] 的多码本展开。

**具体实现**:
- 24 kHz 输入波形经 7 stage 的修改版 Transformer block：把 self-attention 换成 **1D depth-wise causal 卷积**，保证 streaming + 因果性
- 6 次下采样合计 3200×（→ 7.5 token/s）；encoder ≈ 340M、decoder 镜像 340M
- 采用 [[VAE|σ-VAE]] 设计（来自 LatentLM）：编码器只学 $\mu$，方差 $\sigma$ 来自固定先验 $\mathcal{N}(0, C_\sigma)$，避免 AR 建模时常见的 variance collapse
- 训练目标沿用 [[DAC]]（含其判别器与 multi-scale loss）

#### 模块 2：Semantic Tokenizer（ASR 代理任务对齐内容）

**设计动机**: 长篇合成需要 LLM 真正理解"说什么"，单靠声学特征容易跑偏；用 [[ASR]] 监督训出来的特征天然与文本语义对齐。

**具体实现**:
- 架构镜像 acoustic encoder 但**去掉 VAE 头**（确定性输出）
- 训练时其输出接几层 Transformer decoder 预测对应文本 transcript；预训练完成后解码器丢弃，只保留 encoder
- 与 acoustic 特征拼在一起作为 hybrid speech 表示送进 LLM

#### 模块 3：Token-Level Diffusion Head

**设计动机**: 直接 AR 预测高维连续 acoustic latent 难度大、易飘；改成"AR 出 hidden + 扩散去噪"的两段式（沿用 [[LatentLM]] / [[MAR]]），训练监督更稳定。

**具体实现**:
- LLM hidden $\bm{h}_i$ 当条件，4 层网络 ([[DDPM]] 范式) 预测加在 clean acoustic VAE 特征 $\bm{z}_{a,i}$ 上的噪声
- 推理时从高斯噪声起步，[[Classifier-Free Guidance|CFG]] 系数 1.3，[[DPM-Solver|DPM-Solver++]] 10 步去噪
- 输出 clean $\bm{z}_{a,i}$，再走 acoustic decoder 还原成波形

---

## 关键公式

### 公式 1：[[VAE|σ-VAE]] 重参数化

$$
\bm{z} = \mu + \sigma \odot \bm{\epsilon}, \quad \text{where}~ \bm{\epsilon} \sim \mathcal{N}(0, 1),~ \sigma \sim \mathcal{N}(0, C_\sigma)
$$

**含义**: 与传统 [[VAE]] 让 $\sigma$ 可学不同，σ-VAE 把 $\sigma$ 固定为预定义先验分布，从而保证 latent 有充足方差，缓解后续 AR 建模时 latent 坍缩到均值的问题——这是 LatentLM 系列的关键技巧。

**符号说明**:
- $\mu$: encoder $\phi$ 输出的均值（学习参数）
- $\sigma$: 固定先验采样的方差（$C_\sigma$ 是预定义常数）
- $\bm{\epsilon}$: 标准高斯噪声
- $\bm{z}$: 最终输出的 acoustic latent

### 公式 2：[[Next-Token Diffusion]] 输入序列构造

$$
\begin{aligned}
X = & [\texttt{Speaker}_1{:}\,\bm{z}_1, \texttt{Speaker}_2{:}\,\bm{z}_2, \ldots, \texttt{Speaker}_N{:}\,\bm{z}_N] \\
  & + [\texttt{Speaker}_1{:}\,T_1, \texttt{Speaker}_2{:}\,T_2, \ldots, \texttt{Speaker}_N{:}\,T_N]
\end{aligned}
$$

**含义**: 单一拼接序列即模型输入——前半段是各说话人的 voice prompt acoustic latent（决定音色），后半段是按角色排列的对话脚本；不再做 cross-attention，全部交给 LLM 上下文学习。

**符号说明**:
- $\bm{z}_k$: 第 $k$ 个说话人 voice prompt 的 acoustic latent 序列
- $T_k$: 第 $k$ 段台词的 BPE 文本 token
- $\texttt{Speaker}_k$: 角色标识 token

### 公式 3：扩散头训练目标（与 [[DDPM]] 一致）

$$
\mathcal{L}_{\mathrm{diff}} = \mathbb{E}_{t,\bm{\epsilon}} \big\| \bm{\epsilon} - \epsilon_\theta(\bm{z}_{a,i}^{(t)},\, t,\, \bm{h}_i) \big\|^2
$$

**含义**: 让扩散头在条件 $\bm{h}_i$（LLM 当前 token 的 hidden）下预测加在 clean acoustic latent $\bm{z}_{a,i}$ 上的高斯噪声；本质是 token-level 的条件 [[DDPM]]。

**符号说明**:
- $\bm{z}_{a,i}^{(t)}$: 在 timestep $t$ 处加噪后的 acoustic latent
- $\bm{h}_i$: LLM 在第 $i$ 个 token 位置的 hidden state，作为条件
- $\epsilon_\theta$: 扩散头网络（4 层）
- $\bm{\epsilon}$: 真实加入的噪声

> 注：论文正文用文字描述损失（"predicting the noise added to the clean acoustic VAE features"），未列显式公式，本笔记按 [[DDPM]] 标准形式补全。

### 公式 4：推理时 [[Classifier-Free Guidance|CFG]] 引导（标准形式）

$$
\hat{\bm{\epsilon}}_\theta = (1+w)\,\epsilon_\theta(\bm{z}_{a,i}^{(t)}, t, \bm{h}_i) - w\,\epsilon_\theta(\bm{z}_{a,i}^{(t)}, t, \varnothing)
$$

**含义**: 把条件预测与无条件预测做线性外推，强化条件对生成的影响；论文中 guidance scale $w = 1.3$，迭代步数 10 步。

**符号说明**:
- $w$: guidance scale，论文取 $1.3$
- $\varnothing$: 无条件（drop 掉 hidden）
- $\hat{\bm{\epsilon}}_\theta$: 用于 [[DPM-Solver|DPM-Solver++]] 反向积分的引导噪声估计

---

## 关键图表

### Figure 1: Subjective preference vs. duration / 主观偏好与生成时长

![Figure 1](https://arxiv.org/html/2508.19205v1/x5.png)

**说明**: VibeVoice 能稳定合成 5000 + 秒（≈ 90 分钟）单段音频；横轴是生成时长，纵轴是主观评分（preference / realism / richness）。它在长尾长度下仍维持高主观分，**领先所有开源 / 闭源对手**——开源端的 Sesame CSM、Higgs Audio V2、Mooncast、Nari Dia 在长度增加后大多数无法持续，闭源 Gemini 2.5 Pro TTS 与 ElevenLabs v3 alpha 也被 VibeVoice-7B 反超。

### Figure 2: VibeVoice inference architecture / 整体推理架构

![Figure 2](https://arxiv.org/html/2508.19205v1/x6.png)

**说明**: 沿用 [[LatentLM]] 的 [[Next-Token Diffusion]] 思路。左边：voice prompts (经 acoustic tokenizer + semantic tokenizer) 与文本脚本拼接成 hybrid context。中间：[[Qwen2.5]] LLM 处理整段 context，输出每个 token 的 hidden state $\bm{h}_i$。右边：hidden 作为条件喂给 token 级 [[Diffusion Head]] (D)，去噪得到 acoustic VAE latent；最后由 acoustic decoder (A) 还原 24 kHz 波形。整个流水线**没有显式 duration predictor、没有 phoneme、没有 forced alignment**——纯 LLM-style sequence 建模。

### Table 1: 长对话主观 + 客观评测（≈ 1 小时测试集）

| Model | Realism | Richness | Preference | Average | WER (Whisper) ↓ | WER (Nemo) ↓ | SIM ↑ |
|---|---|---|---|---|---|---|---|
| Nari Labs Dia | – | – | – | – | 11.96 | 10.79 | 0.541 |
| Mooncast | – | – | – | – | 2.81 | 3.29 | 0.562 |
| SesameAI CSM | 2.89 ± 1.15 | 3.03 ± 1.11 | 2.75 ± 1.08 | 2.89 ± 1.12 | 2.66 | 3.05 | 0.685 |
| Higgs Audio V2 | 2.95 ± 1.13 | 3.19 ± 1.06 | 2.83 ± 1.16 | 2.99 ± 1.13 | 5.94 | 5.97 | 0.543 |
| ElevenLabs v3 alpha | 3.34 ± 1.11 | 3.48 ± 1.05 | 3.38 ± 1.12 | 3.40 ± 1.09 | 2.39 | 2.47 | 0.623 |
| Gemini 2.5 Pro preview TTS | 3.55 ± 1.20 | 3.78 ± 1.11 | 3.65 ± 1.15 | 3.66 ± 1.16 | 1.73 | 2.43 | – |
| **VibeVoice-1.5B** | 3.59 ± 0.95 | 3.59 ± 1.01 | 3.44 ± 0.92 | 3.54 ± 0.96 | **1.11** | **1.82** | 0.548 |
| **VibeVoice-7B** | **3.71 ± 0.98** | **3.81 ± 0.87** | **3.75 ± 0.94** | **3.76 ± 0.93** | 1.29 | 1.95 | **0.692** |

**说明**: 测试集为 8 段、合计约 1 小时长对话；24 名标注员、每人听 ~6 小时。
- **主观三项**（Realism / Richness / Preference）VibeVoice-7B 全部第一，超过 Gemini 2.5 Pro TTS 与 ElevenLabs v3 alpha。
- **WER**：1.5B 版本最优（Whisper 1.11 / Nemo 1.82），意味着在 1 小时尺度仍极稳定地复述脚本——这是 long-form TTS 上最难保证的指标。
- **SIM**：VibeVoice-7B 0.692 第一；1.5B 偏低 (0.548) 是因为它 SIM 折中换了 WER。
- 注：Nari Dia 与 Mooncast 没有提供主观分；Gemini 不支持 voice prompt 故 SIM 缺失。

### Table 2: SEED 短句基准（test-zh / test-en）

| Model | Frame Rate (Hz) | test-zh CER (%) ↓ | test-zh SIM ↑ | test-en WER (%) ↓ | test-en SIM ↑ |
|---|---|---|---|---|---|
| [[MaskGCT]] | 50 | 2.27 | 0.774 | 2.62 | 0.714 |
| [[Seed-TTS]] | – | 1.12 | 0.796 | 2.25 | 0.762 |
| [[FireRedTTS]] | 25 | 1.51 | 0.635 | 3.82 | 0.460 |
| [[CosyVoice|CosyVoice 2]] | 25 | 1.45 | 0.748 | 2.57 | 0.652 |
| [[Spark-TTS|Spark TTS]] | 50 | 1.20 | 0.672 | 1.98 | 0.584 |
| **VibeVoice-1.5B** | **7.5** | 1.16 | 0.744 | 3.04 | 0.689 |

**说明**: VibeVoice 主要训练在长篇语料上，仍能在短句基准上保持竞争力——尤其考虑到它的 token rate **比 CosyVoice 2 / FireRedTTS 低 3-7×、比 MaskGCT / Spark-TTS 低 6-7×**，每秒解码所需 step 数大幅减少。test-zh CER 1.16 优于 MaskGCT、CosyVoice 2、FireRedTTS，仅次于 Seed-TTS / Spark-TTS；test-en SIM 0.689 在开源对手里仅次于 Seed-TTS 与 MaskGCT。

### Table 3: Acoustic Tokenizer 重建质量（[[LibriTTS]] test-clean / test-other）

| Tokenizer | $N_q$ | Token Rate (Hz) | clean PESQ | clean STOI | clean UTMOS | other PESQ | other STOI | other UTMOS |
|---|---|---|---|---|---|---|---|---|
| Ground-Truth | – | – | – | – | 4.056 | – | – | 3.483 |
| [[EnCodec]] | 8 | 600 | 2.72 | **0.939** | 3.04 | 2.682 | **0.924** | 2.657 |
| [[DAC]] | 4 | 400 | 2.738 | 0.928 | 3.433 | 2.595 | 0.908 | 2.945 |
| [[EnCodec]] | 4 | 300 | 2.052 | 0.901 | 2.307 | 2.052 | 0.884 | 2.088 |
| [[SpeechTokenizer]] | 4 | 300 | 1.931 | 0.878 | 3.563 | 1.737 | 0.837 | 3.018 |
| [[DAC]] | 1 | 100 | 1.246 | 0.771 | 1.494 | 1.245 | 0.751 | 1.499 |
| [[WavTokenizer]] | 1 | 75 | 2.373 | 0.914 | 4.049 | 2.261 | 0.891 | 3.431 |
| [[WavTokenizer]] | 1 | 40 | 1.703 | 0.862 | 3.602 | 1.662 | 0.834 | 3.055 |
| **Ours (Acoustic)** | **1** | **7.5** | **3.068** | 0.828 | **4.181** | **2.848** | 0.823 | **3.724** |

**说明**: 这是论文最有冲击力的一张表——
- **Token rate 比对手低 1-2 个数量级**（7.5 vs. EnCodec 8 层 600 Hz、DAC 100 Hz、WavTokenizer 75 Hz），即"压缩 80×"的具体含义（300 → 7.5 是 40×；600 → 7.5 是 80×，对应摘要里"80 times"的 EnCodec 8 层基线）。
- **PESQ / UTMOS 全场最高**（test-other UTMOS 3.724 已逼近 GT 3.483）；STOI 略输（0.828 vs EnCodec 0.939），但 STOI 本就更敏感时间细节，连续 latent 在该项与 RVQ 离散 token 比有天然劣势。
- 关键含义：**只用 1 个 VAE 维（$N_q=1$ 连续）就能在 7.5 Hz 实现"几乎接近 GT 的感知质量"**，这才是把 90 分钟语音放进 64K 上下文的物理基础。

---

## 实验

### 数据集

| 数据集 | 用途 |
|---|---|
| 长对话训练数据（未公开细节） | LLM + diffusion head 训练；课程学习 4K → 64K 上下文 |
| [[LibriTTS]] test-clean / test-other | acoustic tokenizer 重建评测（Table 3） |
| SEED test-zh（约 2000 句，CommonVoice 子集） | 短句中文 CER + SIM（Table 2） |
| SEED test-en（约 1000 句，CommonVoice 子集） | 短句英文 WER + SIM（Table 2） |
| 自建长对话测试集（8 段，约 1 小时） | 主观 MOS + 客观 WER/SIM（Table 1） |

### 实现细节

- **LLM Backbone**: [[Qwen2.5]] 1.5B / 7B（参数全可学）
- **Diffusion Head**: 4 层（论文未给逐层维度，沿用 [[MAR]] 风格）
- **Tokenizer**: encoder/decoder 各 ~340M，全程冻结
- **上下文 curriculum**: 4 096 → 65 536 tokens
- **采样器**: [[DPM-Solver|DPM-Solver++]]，10 步；CFG scale = 1.3
- **支持语言**: 仅英文 + 中文
- **采样率**: 输入/输出均为 24 kHz
- **客观评测工具**: WER 用 [[Whisper|Whisper-large-v3]]（en）+ NeMo ASR / [[Paraformer]]（zh）；SIM 用 [[WavLM|WavLM-large]] 提 speaker embedding

### 关键发现

- **Tokenizer 越省，长序列越省**：7.5 Hz 让 90 分钟语音 ≈ 4 万 token，加上文本后正好落在 64K 上下文里——这是端到端 long-form TTS 落地的核心条件。
- **LLM 越大，主观越好，WER 不变**：1.5B → 7B 在 SIM、Realism、Richness 都涨；WER 反而 1.5B 更低（1.11 vs 1.29）——说明 7B 在"复述准确"已饱和、增益花在了表达力 / 音色还原。
- **课程学习有效**：4K → 64K 渐进扩展上下文是稳定训练长篇语料的必要 trick，避免直接用 64K 训不收敛。

---

## 批判性思考

### 优点

1. **第一个实证 90 分钟、4 说话人单条推理可行的开源框架**，对播客 / 多人有声书是直接可用的"长 TTS 基线"。
2. **架构极简**：把"voice prompt + 文本脚本"压成一条 LLM 序列，没有 [[Duration Predictor]]、没有 phoneme 前端、没有 forced alignment、没有 RVQ；工程友好度极高，与现成 Qwen2.5 训练栈完全兼容。
3. **Tokenizer 是真正的关键贡献**：7.5 Hz / 单维连续 latent 同时刷掉 PESQ + UTMOS SOTA，这是后续别家做长 TTS 都会直接复用的能力。
4. **CFG + 10 step DPM-Solver++ 的小代价换来质量**：每个 token 只多 10 次扩散迭代，相对每秒只要 7.5 token，整体推理预算并不大。

### 局限性

1. **没有报推理延迟、RTF、首包延迟**：long-form TTS 在工程上更怕"首包慢"或"无法流式"；论文标榜 streaming-friendly（causal conv 设计）但缺数据支持。
2. **不显式建模 overlapping speech**：人类播客有大量"嗯哼"/抢话/打断，论文 Limitations 自己承认不处理；和 [[Moshi]] 等全双工模型相比缺关键能力。
3. **只支持中英**，且**不处理背景音 / 音乐 / 音效**——离 NotebookLM / 真实播客制作还有距离。
4. **训练数据完全未公开**：长对话数据来源、清洗、标注（角色 boundary、turn-taking）只字未提，可复现性受限。
5. **STOI 略输给离散 codec**：连续 latent 在精细时间细节上仍有差距；后续若要做歌唱 / 高动态音频可能需要补码本。
6. **σ-VAE 的 $C_\sigma$ 选择没消融**：这个超参对 AR 稳定性是关键，论文一句带过。
7. **主观评测样本量小**：8 段对话、24 名标注员，与 Gemini 2.5 Pro TTS 等的差距 ±0.05 在 95% CI 上未必显著（论文也没做显著性检验）。

### 潜在改进方向

1. 加 [[Speech DPO]] / 偏好对齐：进一步推主观 preference。
2. 显式 turn-taking / overlap 控制：把 [[Moshi]] 的双流思想搬到 hybrid context。
3. 用 [[Flow Matching]] 替代 DDPM diffusion head：可能再把 10 步压到 1-4 步。
4. 试试 streaming decoding 的实际首包延迟，公开 benchmark。
5. 把 acoustic tokenizer 开放使用（HuggingFace 上已 release），验证它当 [[Audio Codec]] 通用底座的能力。

### 可复现性评估

- [x] 代码开源：<https://github.com/microsoft/VibeVoice>
- [x] 预训练模型：HuggingFace 1.5B / 7B
- [ ] 训练细节完整：缺 batch / lr / steps / 数据规模
- [ ] 数据集可获取：长对话训练集未公开
- [x] tokenizer / diffusion head 推理可独立运行（github 仓库提供）

---

## 关联笔记

### 基于（核心借鉴）

- [[LatentLM]] / [[Next-Token Diffusion]]: 整套 "AR LLM 输出 hidden + 扩散头解码连续 latent" 范式来自 LatentLM，是这篇文章最重要的母方法
- [[MAR]]（Autoregressive Image Generation without Vector Quantization）: 4 层 token-level diffusion head 的来源
- [[Qwen2.5]]: LLM backbone
- [[DDPM]] + [[DPM-Solver|DPM-Solver++]]: 扩散头训练 / 推理基础
- [[DAC]]: acoustic tokenizer 的判别器与重建 loss 设计
- [[VAE|σ-VAE]]: 防止 AR 时 latent 方差坍缩

### 对比（同期长对话 / 多说话人 TTS）

- [[Mooncast]]: 同样针对播客生成；WER 接近但缺主观分
- Sesame CSM / Higgs Audio V2 / Nari Dia / MOSS-TTSD: 开源对话 TTS，长度与稳定性均不及 VibeVoice
- ElevenLabs v3 alpha / Gemini 2.5 Pro preview TTS / NotebookLM: 闭源对手
- [[CoVoMix2]]: 全 NAR Flow Matching 多说话人对话生成

### 对比（短句零样本 TTS 强基线）

- [[CosyVoice|CosyVoice 2]] / [[F5-TTS]] / [[Seed-TTS]] / [[Spark-TTS]] / [[MaskGCT]] / [[FireRedTTS]] / [[VALL-E]]

### Tokenizer 对比

- [[EnCodec]] / [[DAC]] / [[SpeechTokenizer]] / [[WavTokenizer]]

### 评测工具

- [[Whisper]]（WER） · [[Paraformer]]（CER） · [[WavLM]]（SIM） · [[UTMOS]] · [[PESQ]] · [[STOI]]

### 数据相关

- [[LibriTTS]] · CommonVoice / SEED test-zh / SEED test-en

---

## 速查卡片

> [!summary] VibeVoice Technical Report
> - **核心**: [[Next-Token Diffusion]] + 7.5 Hz 连续 σ-VAE tokenizer，让 LLM 64K 上下文一次合成 90 分钟、4 说话人对话
> - **方法**: [[Qwen2.5]] (1.5B/7B) AR 出 hidden → 4 层 diffusion head 去噪 → 单维连续 acoustic latent → decoder → 24 kHz
> - **结果**: 1 小时长对话主观 + WER 全部领先 ElevenLabs v3 / Gemini 2.5 Pro TTS；tokenizer 在 [[LibriTTS]] PESQ/UTMOS SOTA（test-other UTMOS 3.72，逼近 GT 3.48）
> - **代码**: <https://github.com/microsoft/VibeVoice> · HF: microsoft/VibeVoice 1.5B/7B
> - **跳过条件**: 不支持 overlap / 背景音 / 中英外语种；未报 RTF / 首包延迟

---

*笔记创建时间: 2026-05-19*
