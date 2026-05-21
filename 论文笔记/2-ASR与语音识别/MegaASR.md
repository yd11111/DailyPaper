---
title: "Mega-ASR: Towards In-the-wild² Speech Recognition via Scaling Up Real-world Acoustic Simulation"
method_name: "Mega-ASR"
authors: [Zhifei Xie, Kaiyu Pang, Haobin Zhang, Deheng Ye, Xiaobin Hu, Shuicheng Yan, Chunyan Miao]
year: 2026
venue: arXiv
tags: [robust-asr, speech-recognition, acoustic-simulation, reinforcement-learning, asr-foundation-model, in-the-wild-asr]
zotero_collection: ""
image_source: online
arxiv_html: https://arxiv.org/html/2605.19833v1
created: 2026-05-21
---

# 论文笔记：Mega-ASR — 通过扩展真实声学仿真走向"野外²"语音识别

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | NTU（南洋理工）/ NUS（新加坡国立）/ Shanghai AI Lab |
| 日期 | May 2026 (arXiv:2605.19833v1) |
| 项目主页 | https://xzf-thu.github.io/Mega-ASR/ |
| 代码 | https://github.com/xzf-thu/Mega-ASR |
| 数据 | https://huggingface.co/datasets/zhifeixie/Voices-in-the-Wild-2M |
| Bench | https://github.com/xzf-thu/Voices-in-the-Wild-Bench |
| 主要基线 | [[Qwen3-ASR]]、[[Whisper]]、[[Kimi-Audio]]、[[Step-Audio-2]]、[[Qwen2.5-Omni]]、Gemini-3-Flash、GPT-4o |
| 链接 | [arXiv](https://arxiv.org/abs/2605.19833) / [Code](https://github.com/xzf-thu/Mega-ASR) |

---

## 一句话总结

> 用 240 万条覆盖 7 种原子声学效应×54 种复合场景的仿真数据，配合 [[A2S-SFT]] + [[DG-WGPO]] 两阶段训练，把在复杂噪声/远场/混响场景下的 [[WER]] 相对降低 30%+，同时不牺牲干净集成绩。

---

## 核心贡献

1. **Voices-in-the-Wild-2M 数据集**：首个系统覆盖 7 类原子声学效应（噪声 / 远场 / 遮挡 / 回声混响 / 录音劣化 / 电子失真 / 传输丢包）和 54 种**物理可信的复合场景**的大规模仿真数据集，约 240 万条（~11k 小时）；[[Qwen3-ASR]] 在其上平均 [[WER]] 约 35%，证明"难度足够大"。
2. **A2S-SFT（Acoustic-to-Semantic Progressive SFT）**：声学到语义的递进式 SFT，按 [[WER]] 难度分级（<30% → <50% → <70%）+ 模块按序解冻（encoder+aligner → LLM → joint）的 [[Curriculum Learning|课程学习]] 方案。
3. **DG-WGPO（Dual-Granularity WER-Gated Policy Optimization）**：在 [[DAPO]] 基础上提出双粒度奖励——低 [[WER]] 区段用 token 级精修奖励 $R_{fine}$，高 [[WER]] 区段切换到句子级重构奖励 $R_{struc}$，由 [[WER]] 门控融合，专门抑制 [[ASR Hallucination|幻觉]] 和漏字。
4. **Environment-Aware Routing**：用 [[LoRA]] 训一个二分类器在干净/复杂场景间动态路由 robust 与原始权重，干净集与原模型持平、复杂集保持鲁棒收益。

---

## 问题背景

### 要解决的问题

主流 ASR 在干净集（[[LibriSpeech]]、[[AISHELL-1]]）上已经做到 ~1% [[WER]]，但是在真实"野外"条件下退化到 10–30%，极端情况下高达 70%。论文称之为 **acoustic robustness bottleneck**：失去声学锚定（acoustic grounding）后系统会产生**漏识别**（omission）或**幻觉**（hallucination）。

### 现有方法的局限

作者归纳三个 deficiency：

- **D1 — 场景覆盖窄**：[[CHiME-4]]、[[NOIZEUS]]、TED-LIUM 等 benchmark 只覆盖 1–2 种孤立条件，没有多效应复合。
- **D2 — 缺少组合鲁棒性**：现实中"远场+混响+噪声"是同时发生的，模型在单因素鲁棒不代表组合鲁棒。
- **D3 — 训练-测试难度错配**：现有训练数据扰动温和（平均 4–10% WER），与真实场景（>30% WER）数据分布差距大。

### 本文的动机

提出"**ASR-in-the-wild²**"概念：不是"听清楚一个噪声场景"，而是"听清楚多种声学效应组合"。要做到这一点，仿真数据必须**物理可信地复合**多种效应，并且训练目标必须能在**高 WER 区段仍提供有效学习信号**。

---

## 方法详解

### Voices-in-the-Wild-2M 构造流程

四步 pipeline：

1. **原子效应仿真**：基于谱图滤波 / 卷积实现 7 种原子效应；噪声源来自 [[MUSAN]]、DNS Challenge、ESC-50、UrbanSound8K（~42K 条、129 小时）；干净语音源自 [[LibriSpeech]]、[[Common Voice]]、[[WenetSpeech]]、[[AISHELL-1]]。
2. **现实化复合**：从 7 种原子效应中选 2–5 种组合，由 Agent 校验物理合理性，生成 **54 种复合场景**。
3. **可控难度合成**：每个样本带一个严重程度参数 $k\in[0,1]$；试验 Sqrt-Forward / Sqrt-Backward / Gaussian-Mid / Linear 四种采样曲线，最终选 **Linear**。
4. **可学习性过滤**：丢弃 [[WER]] > 70% 的样本（防止训练崩塌）。

成品规模 ~240 万条，[[Qwen3-ASR]] 在其上平均 [[WER]] 18.42%，论文也额外切出一个含 1500 条**真录**+3500 条仿真的双语（EN/ZH）评测集 **Voices-in-the-Wild-Bench**。

### Mega-ASR 训练流程

模型结构沿用 **[[Qwen3-ASR]]-1.7B**（speech encoder + aligner + LLM 三段式 [[Speech LLM]] 架构），训练分两阶段。

#### 阶段 1: A2S-SFT（Acoustic-to-Semantic Progressive SFT）

三个 phase：

| Phase | 训练对象 | 数据筛选 |
|---|---|---|
| 1 | Encoder + Aligner | WER<30% → <50% → <70% 三段难度课程 |
| 2 | LLM | WER<70% 全集 |
| 3 | Encoder + Aligner + LLM 联合 | WER<70% 全集 |

设计动机：先让声学模块"听清楚"，再让 LLM 学"语义恢复"，最后联合微调，避免一开始 LLM 被噪声特征带偏。

#### 阶段 2: DG-WGPO（Dual-Granularity WER-Gated Policy Optimization）

基于 [[DAPO]] 框架。核心观察：**WER<~30% 时错误集中在 token 级（替换/删除单词），WER≥30% 时错误集中在句子级（整段幻觉/漏识）**。所以需要双粒度奖励，并用 WER 门控选择融合权重。

**静态奖励层**：
- $R_{rep}$：n-gram 重复硬门控（出现重复直接判 0，专杀"复读机式"幻觉）
- $R_{wer}$：基础 WER 奖励
- 二者相乘得 $R_{static}$

**动态奖励层**：
- $R_{fine}$：token 级精修，区分 hard / soft 替换（编辑距离相似度 ≥0.5 算 soft）
- $R_{struc}$：句子级重构，结合 LCS 与长度一致性
- $R_{dynamic}$：按当前 [[WER]] 与阈值 $\tau$ 门控，低 WER 偏 $R_{fine}$，高 WER 偏 $R_{struc}$

**最终融合**：$R = (1-\alpha_{dyn}) R_{simple} + \alpha_{dyn} R_{dynamic}$，超参数 $\tau=0.3, \alpha_s=0.4, \alpha_{dyn}=0.6$。

#### 推理时的 Environment-Aware Routing

用一个轻量二分类器（[[LoRA]] 微调在干净 + Voices-in-the-Wild 的混合集上），把输入路由到 Mega-ASR 鲁棒权重或原始 [[Qwen3-ASR]] backbone，保证干净场景与原模型几乎持平。

---

## 关键公式

### 公式1: [[WER]] 静态奖励

$$
R_{wer}(H, R) = 1 - \text{WER}(H, R)
$$

**含义**：基础奖励项，把 [[WER]] 转为 [0,1] 的奖励值。

**符号说明**：
- $H$：模型 hypothesis 输出
- $R$：reference ground truth

### 公式2: [[Anti-Repetition Reward|抗复读硬门控]]

$$
R_{rep}(H) = \begin{cases} 0, & \text{if repeated n-grams above threshold} \\ 1, & \text{otherwise} \end{cases}
$$

**含义**：若 hypothesis 中检测到重复 n-gram 超阈，rollout 直接判 0，专门抑制 RL 训练中常见的复读式 [[ASR Hallucination|幻觉]]。

### 公式3: 静态奖励融合

$$
R_{static} = R_{rep} \cdot R_{wer}
$$

**含义**：抗复读门控与 WER 奖励的乘积。

### 公式4: Token 相似度

$$
\text{sim}(h, r) = 1 - \frac{\text{edit}(h, r)}{\max(|h|, |r|)} \in [0, 1]
$$

**含义**：单 token 替换错误的细分依据——sim ≥ 0.5 算 soft 替换（轻惩），否则算 hard 替换（重惩）；插入/删除一律 hard。

**符号说明**：
- $h, r$：分别是 hypothesis 与 reference 中对齐的 token
- $\text{edit}(\cdot)$：字符级编辑距离

### 公式5: [[Token-Level Refinement Reward|Token 级精修奖励]]

$$
R_{fine} = \frac{n_C}{n_C + n_{hard} + \alpha_s \cdot n_{soft} + \epsilon}
$$

**含义**：以"正确 token 数 / (正确 + hard 错 + 软系数×soft 错)"刻画 token 级精度，专门负责 [[WER]] 较低区段的精修。

**符号说明**：
- $n_C$：正确 token 数
- $n_{hard}, n_{soft}$：hard / soft 错 token 数
- $\alpha_s$：soft 错的折扣系数（论文取 0.4）
- $\epsilon = 10^{-8}$：数值稳定项

### 公式6: [[Sentence-Level Reconstruction Reward|句子级重构奖励]]

$$
R_{struc} = \frac{1}{2}\cdot\frac{\text{LCS}(H, R)}{|R|} + \frac{1}{2}\cdot\max\left(0,\ 1 - \frac{\big||H|-|R|\big|}{|R|}\right)
$$

**含义**：高 [[WER]] 区段下，句子可能已经整体崩坏，token 级信号无意义。换用最长公共子序列 LCS 占比 + 长度一致性的平均，鼓励整体结构对齐。

**符号说明**：
- $\text{LCS}(H, R)$：hypothesis 与 reference 的最长公共子序列长度
- $|H|, |R|$：hypothesis 与 reference 的 token 长度

### 公式7: [[WER-Gated Fusion|WER 门控动态奖励]]

$$
R_{dynamic} = \begin{cases} 0.75\, R_{fine} + 0.25\, R_{struc}, & \text{WER} < \tau \\ 0.25\, R_{fine} + 0.75\, R_{struc}, & \text{WER} \geq \tau \end{cases}
$$

**含义**：按 [[WER]] 当前值与门控阈值 $\tau$ 决定细 / 粗粒度奖励权重。论文取 $\tau=0.3$。

### 公式8: 最终奖励

$$
R = (1-\alpha_{dyn})\, R_{simple} + \alpha_{dyn}\, R_{dynamic}
$$

**含义**：把规则式静态奖励 $R_{simple}$ 与双粒度动态奖励融合，论文取 $\alpha_{dyn}=0.6$。

---

## 关键图表

### Figure 1: Mega-ASR vs Qwen3-ASR 雷达对比

![Figure 1](https://arxiv.org/html/2605.19833v1/x1.png)

**说明**：在选定的 ASR 子集（[[CHiME-4]] real/sim、[[VOiCES]] rm1–rm4、[[NOIZEUS]] 0/5/10/15 dB 等）上 Mega-ASR 几乎全面优于 [[Qwen3-ASR]]-1.7B baseline，体现"鲁棒+不损失干净集"的双重收益。

### Figure 2: Voices-in-the-Wild-2M 场景拓扑

![Figure 2](https://arxiv.org/html/2605.19833v1/x2.png)

**说明**：7 种原子声学效应（noise / far-field / obstructed / echo & reverb / recording / electronic distortion / transmission dropout）展开成 54 种复合场景的可视化，每种场景由 2–5 种原子效应组合，并经 agent 校验物理可信。

### Figure 3: A2S-SFT 准确度曲线 + 难度采样分布

![Figure 3](https://arxiv.org/html/2605.19833v1/ASR_fig/combined_left8_right_sampling_v7.png)

**说明**：左图——A2S-SFT 课程学习下真实样本上准确度随训练步增长；右图——Linear / Sqrt-Forward / Sqrt-Backward / Gaussian-Mid 四种难度采样曲线在 [[NOIZEUS]] 0dB 上的对比，最终 Linear 胜出。

### Figure 4: DG-WGPO 框架

![Figure 4](https://arxiv.org/html/2605.19833v1/x3.png)

**说明**：A2S-SFT 初始化策略，再用门控融合的动态奖励（$R_{fine}$ + $R_{struc}$，按 WER 切换权重）+ 静态规则奖励（$R_{wer} \cdot R_{rep}$）做 [[DAPO]] 风格的策略优化。

### Figure 5: Environment-Aware Routing

![Figure 5](https://arxiv.org/html/2605.19833v1/x4.png)

**说明**：推理时一个轻量分类器判断输入是干净还是复杂场景，分别走原 [[Qwen3-ASR]] 与 Mega-ASR 鲁棒权重，干净集与原模型持平、复杂集保持鲁棒收益。

### Figure 6: Case Study 定性对比

![Figure 6](https://arxiv.org/html/2605.19833v1/x5.png)

**说明**：在 Peak –5.2 dB 的远场+噪声+实体恢复样本上：Qwen3-ASR 直接输出空字符串（WER 100%），Gemini-3-Pro 整段编造（WER 86.1%），Mega-ASR 完整准确（WER 0.0%）。

### Table 1: 数据集覆盖能力对比

| Dataset | Real | Sim | Noise | Far | Barr | E&R | Record | Distort | Drop | Scale | WER |
|---|---|---|---|---|---|---|---|---|---|---|---|
| [[NOIZEUS]] | ✗ | ✔ | ✔ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | 1K | 9.45 |
| [[TED-LIUM]] | ✔ | ✗ | ✗ | ✔ | ✗ | ✗ | ✗ | ✗ | ✗ | 59K | 2.31 |
| [[CHiME-4]] | ✔ | ✔ | ✔ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | 15K | 5.39 |
| [[VOiCES]] | ✔ | ✗ | ✔ | ✔ | ✔ | ✔ | ✔ | ✗ | ✗ | 1M | 8.94 |
| BERSt | ✔ | ✗ | ✗ | ✔ | ✔ | ✗ | ✗ | ✗ | ✗ | 4.5K | 22.41 |
| DAPS | ✔ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✔ | ✗ | 2K | 6.24 |
| **[[Voices-in-the-Wild-2M]]** | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 2M | 18.42 |

**说明**：唯一覆盖全 7 类原子效应的大规模数据集，且 baseline WER 18.42% 反映足够难度。

### Table 2: 噪声/鲁棒 ASR Benchmarks

| Model | CHiME-4 Real | Sim | Avg | VOiCES rm1 | rm2 | rm3 | rm4 | Avg | NOIZEUS 0dB | 5dB | 10dB | 15dB | Avg | **Total Avg** |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Gemini3-Flash | 6.58 | 5.67 | 6.13 | 3.10 | 4.27 | 25.99 | 21.86 | 13.81 | 55.78 | 24.48 | 18.49 | 8.52 | 26.82 | 15.59 |
| Doubao-LLM ASR | 9.95 | 11.62 | 10.79 | 4.86 | 6.99 | 17.23 | 7.85 | 9.23 | 25.78 | 9.51 | 4.96 | 2.87 | 10.78 | 10.27 |
| GPT-4o-trans | 5.36 | 7.57 | 6.47 | 10.97 | 12.56 | 46.68 | 29.38 | 22.65 | 62.40 | 20.56 | 6.15 | 2.64 | 22.94 | 17.35 |
| Voxtral-Mini | 6.01 | 9.04 | 7.53 | 3.50 | 3.51 | 27.54 | 16.45 | 12.75 | 41.06 | 15.80 | 4.85 | 2.94 | 16.16 | 12.15 |
| [[Kimi-Audio]] | 5.66 | 7.46 | 6.56 | 2.10 | 2.23 | 26.95 | 15.13 | 11.60 | 38.33 | 11.36 | 4.34 | 2.27 | 14.08 | 10.74 |
| [[Whisper]]-L-v3 | 5.65 | 8.39 | 7.02 | 2.85 | 2.97 | 25.68 | 15.65 | 11.79 | 34.71 | 12.55 | 3.93 | 2.17 | 13.34 | 10.72 |
| Canary-1B-v2 | 7.19 | 9.73 | 8.46 | 3.14 | 3.00 | 24.88 | 15.56 | 11.65 | 38.53 | 12.76 | 6.56 | 3.77 | 15.41 | 11.84 |
| Parakeet-v3 | 6.61 | 8.82 | 7.72 | 3.23 | 3.27 | 19.77 | 13.84 | 10.03 | 38.95 | 14.67 | 5.99 | 3.15 | 15.69 | 11.15 |
| [[Qwen2.5-Omni]] | 6.62 | 8.13 | 7.37 | 4.15 | 4.03 | 44.76 | 22.53 | 18.87 | 54.91 | 17.72 | 3.20 | 0.88 | 19.18 | 15.14 |
| [[Step-Audio-2]]-mini | 5.35 | 7.06 | 6.20 | 1.81 | 1.98 | 23.25 | 15.19 | 10.56 | 32.02 | 8.94 | 3.72 | 2.27 | 11.74 | 9.50 |
| [[Qwen3-ASR]] | 4.66 | 6.11 | 5.39 | 2.52 | 2.62 | 19.18 | 11.44 | 8.94 | 23.97 | 8.47 | 3.41 | 1.96 | 9.45 | 7.93 |
| **Mega-ASR** | 4.41 | 6.04 | 5.23 | 2.36 | 2.43 | 15.13 | 9.46 | **7.35** | 19.80 | 6.61 | 2.79 | 0.88 | **7.52** | **6.70** |
| **Mega-ASR w/ router** | 4.38 | 5.62 | **5.00** | 2.42 | 2.49 | 15.32 | 9.26 | 7.37 | 19.80 | 6.97 | 3.05 | 1.76 | 7.90 | 6.76 |

**说明**：在三大鲁棒 benchmark 上 Mega-ASR 取得新的 SOTA：总平均 [[WER]] 6.70%（无路由）/ 6.76%（有路由），相对 [[Qwen3-ASR]] 7.93% 提升约 15%；尤其在 VOiCES rm3、NOIZEUS 0dB 这种极端场景上相对降幅达 17–21%。

### Table 3: 标准（干净）ASR Benchmarks

| Model | LibriSp Dev (c/o) | Test | CV zh | en | Fleurs zh | en | AISHELL-1 | WenetSp net | meet | VoxPop en |
|---|---|---|---|---|---|---|---|---|---|---|
| Gemini-3-Flash | 1.7/3.56 | 1.81/4.91 | 13.58 | 8.49 | 7.52 | 4.01 | 2.66 | 14.38 | 17.62 | 7.74 |
| Doubao-LLM | 2.95/4.06 | 2.92/5.32 | 4.60 | 7.12 | 2.92 | 7.22 | 0.98 | 4.46 | 4.90 | 7.14 |
| GPT-4o-trans | 1.52/3.29 | 1.75/4.23 | 12.61 | 7.22 | 2.62 | 2.71 | 3.52 | 15.71 | 31.40 | 7.02 |
| Canary-1B-v2 | 2.07/4.03 | 2.20/3.58 | – | 8.91 | – | 4.48 | – | – | – | 6.20 |
| Parakeet-v3 | 1.91/3.54 | 1.93/3.60 | – | 8.54 | – | 4.88 | – | – | – | 6.11 |
| Voxtral-Mini | 1.89/3.88 | 1.89/4.08 | – | 10.15 | – | 3.84 | – | – | – | 7.08 |
| [[Step-Audio-2]]-mini | 1.21/2.50 | 1.37/2.75 | 4.77 | 7.04 | 2.48 | 3.93 | 0.81 | 5.56 | 5.46 | 7.43 |
| [[Kimi-Audio]]-7B | 1.38/2.56 | 1.34/2.55 | 6.74 | 8.35 | 5.88 | 8.07 | 0.76 | 6.41 | 6.25 | 8.15 |
| [[Whisper]]-L-v3 | 1.74/3.68 | 1.78/3.53 | 15.33 | 16.18 | 7.70 | 4.10 | 5.89 | 12.02 | 17.79 | 9.00 |
| [[Qwen2.5-Omni]]-7B | 2.05/4.19 | 2.37/4.21 | 5.01 | 8.56 | 4.64 | 4.01 | 1.15 | 6.16 | 9.64 | 6.02 |
| [[Qwen3-ASR]]-1.7B | 1.62/3.07 | 1.62/3.40 | 7.42 | 7.57 | 3.93 | 3.19 | 1.52 | 4.99 | 5.80 | 6.25 |
| Ours (Mega-ASR) | 1.62/3.21 | 1.78/3.57 | 5.8 | 8.15 | 5.43 | 3.76 | 1.49 | 5.19 | 6.17 | 7.44 |
| Ours w/ router | 1.64/3.07 | 1.63/3.37 | 7.37 | 7.57 | 3.86 | 3.17 | 1.53 | 4.95 | 5.89 | 6.26 |

**说明**：纯 Mega-ASR 在干净集上有轻微回退（典型 trade-off），加 router 后几乎完全恢复到 [[Qwen3-ASR]] 水平。这是 router 模块的核心价值。

### Table 4: Voices-in-the-Wild-Bench 分场景 WER（Real/Sim）

| Model | Noise R/S | Far R/S | Obst R/S | Echo R/S | Record R/S | Elc.Dis R/S | Trans R/S | Mixed R/S |
|---|---|---|---|---|---|---|---|---|
| Gemini3-Flash | 7.63/10.61 | 5.14/1.90 | 3.73/2.65 | 8.75/14.86 | 8.38/19.85 | 3.15/7.56 | 5.47/7.65 | 7.99/9.62 |
| Seed-ASR | 8.21/8.11 | 3.06/3.19 | 3.10/2.76 | 16.55/18.21 | 18.48/23.33 | 3.89/5.71 | 7.97/7.46 | 6.88/9.29 |
| GPT-4o-trans | 13.19/45.78 | 1.87/2.39 | 1.57/2.77 | 15.62/28.76 | 13.37/22.60 | 3.70/8.43 | 8.76/7.71 | 5.62/11.00 |
| [[Whisper]]-L-v3 | 16.57/18.19 | 3.38/6.85 | 3.06/6.01 | 25.34/39.87 | 18.33/31.81 | 3.74/8.77 | 7.04/8.05 | 8.91/14.79 |
| [[Qwen2.5-Omni]] | 11.92/17.88 | 2.35/2.44 | 2.40/2.08 | 20.01/32.64 | 13.71/30.09 | 2.46/5.96 | 6.34/5.88 | 6.40/10.29 |
| [[Kimi-Audio]] | 35.10/14.59 | 2.71/1.92 | 2.49/1.64 | 24.00/26.58 | 8.73/18.09 | 1.83/2.78 | 4.54/6.33 | 4.44/6.19 |
| [[Qwen3-ASR]] | 7.51/9.52 | 2.23/1.54 | 1.73/1.27 | 10.40/14.61 | 9.57/19.42 | 1.54/3.41 | 4.16/4.19 | 3.30/5.39 |
| **Ours** | 6.33/8.26 | 2.35/1.61 | 1.62/1.23 | 8.62/12.59 | 7.65/14.21 | 1.71/3.72 | 2.59/2.62 | **2.73/4.57** |
| **Ours w/ router** | 6.12/8.09 | 2.33/1.69 | 1.80/1.41 | 8.66/12.22 | 6.91/13.23 | 1.60/3.35 | 2.72/2.88 | 2.63/4.53 |

**说明**：Mega-ASR 在 7 大场景中 Noise、Echo、Record、Trans、Mixed 五大类显著优于 [[Qwen3-ASR]]；最有意思的是 Mixed（混合）场景，相对降幅 ~18%（5.39→4.57），说明对**未在训练时见过的复合**也有泛化。

### Table 5: A2S-SFT + DG-WGPO 消融

| 配置 | Voices Avg | Noizeus Avg |
|---|---|---|
| [[Qwen3-ASR]] baseline | 8.94 | 9.45 |
| + SFT w/o A2S（朴素 SFT） | 8.31 | 8.79 |
| Mega-ASR-Base（A2S-SFT only） | 7.59 | 8.12 |
| + vanilla [[GRPO]]（仅 $R_{wer}$） | 7.73 | 8.11 |
| + vanilla [[DAPO]]（仅 $R_{wer}$） | 7.62 | 7.98 |
| + DG-WGPO w/o $R_{rep}$ | 7.46 | 7.73 |
| + DG-WGPO w/o $R_{fine}$ | 7.45 | 7.71 |
| + DG-WGPO w/o $R_{struc}$ | 7.54 | 7.85 |
| + DG-WGPO w/o gated fusion | 7.41 | 7.68 |
| **Mega-ASR (full)** | **7.35** | **7.64** |

**关键发现**：
- A2S 难度课程比朴素 SFT 多带来 0.7+ WER 降幅；
- 单纯 [[GRPO]] / [[DAPO]] 反而比 SFT 略差（说明只用 WER 当奖励会过拟合"看起来对的"模式）；
- 抗复读 $R_{rep}$、Token 级 $R_{fine}$、句子级 $R_{struc}$、门控融合四个组件各自都能贡献，去掉任意一个都掉点；
- $R_{struc}$ 移除影响最大（7.35→7.54），证明高 WER 区段的句子级信号是关键。

### Table 6: Reward 设计：规则 vs LLM-judge

| Reward | Voices | Noizeus | Voi-R | Avg.Time (rel) |
|---|---|---|---|---|
| LLM-judge | 7.51 | 7.71 | 9.27 | 62.23 |
| Rule-based | 7.53 | 7.64 | 9.38 | 19.57 |

**说明**：规则式（DG-WGPO）与 LLM-as-judge 在 WER 上几乎打平，但**规则式 3.2× 更快**，省去 LLM 调用开销。这是工程上的关键选择。

### Table 7: LLM-as-Judge 语义评估

| Model | Hall.↓ | Miss↓ | Sem.↑ | KeyE.↓ |
|---|---|---|---|---|
| [[Qwen3-ASR]] | 18.7 | 14.2 | 71.3 | 22.5 |
| Mega-ASR-Base | 15.4 | 11.6 | 79.8 | 20.1 |
| Mega-ASR | 11.8 | 5.9 | 86.4 | 19.5 |

**说明**：[[ASR Hallucination|幻觉]]、漏识、关键实体错误（KeyE.）全面降低；漏识 Miss 从 14.2 → 5.9（**降幅 58%**），证明 $R_{struc}$ 确实补上了高 WER 场景下的"整段恢复"能力。

### Table 8: 超参 ($\alpha_{dyn}, \alpha_s$) 敏感性

| Settings | Noise | Far | Nz V.N.R. | V.F. | V.F.R. |
|---|---|---|---|---|---|
| $\alpha_{dyn}=0.4, \alpha_s=0.4$ | 7.7 | 7.6 | 7.8 | 9.5 | – |
| $\alpha_{dyn}=0.4, \alpha_s=0.6$ | 7.8 | 7.6 | 7.9 | 9.4 | – |
| $\alpha_{dyn}=0.6, \alpha_s=0.2$ | 7.8 | 7.5 | 7.6 | 9.3 | – |
| $\alpha_{dyn}=0.6, \alpha_s=0.6$ | 7.5 | 7.5 | 7.4 | 9.3 | – |
| $\alpha_{dyn}=0.8, \alpha_s=0.4$ | 8.1 | 9.1 | 8.0 | 9.9 | – |
| **$\alpha_{dyn}=0.6, \alpha_s=0.4$** | 7.6 | 7.4 | 7.4 | 9.2 | – |

**说明**：$\alpha_{dyn}$ 太大（0.8）会过度依赖动态奖励反而掉点；最佳 sweet spot 在 (0.6, 0.4)。

### Table 9: 门控阈值 $\tau$ 敏感性（NOIZEUS）

| $\tau$ | 0.2 | 0.3 | 0.4 | 0.5 |
|---|---|---|---|---|
| NOIZEUS WER | 7.68 | **7.64** | 7.66 | 7.70 |

**说明**：$\tau=0.3$ 最优，但整体很平坦，说明门控本身鲁棒。

### Table 10: 三大鲁棒 benchmark 平均（Appendix B）

| Model | CHiME-4 | NOIZEUS | VOiCES | Avg |
|---|---|---|---|---|
| [[Qwen3-ASR]]-1.7B | 5.39 | 9.45 | 8.94 | 7.93 |
| Mega-ASR | 5.23 | 7.52 | 7.35 | 6.70 |
| Mega-ASR w/ router | 5.00 | 7.90 | 7.37 | 6.76 |

### Table 11–13: 细分子集结果（Appendix B）

- **Table 11 — CHiME-4 16 子集**（dt05/et05 × bus/caf/ped/str × real/simu）：Mega-ASR w/ router 平均 5.00，代表样例 et05_str_simu 8.64 → 7.19。
- **Table 12 — NOIZEUS 32 子集**（8 种噪声类型 × 4 个 SNR 等级）：典型样例 car_0dB 29.34 → 22.47 (router)；station_0dB 29.34 → 23.71；平均 9.45 → 7.52。极少数子集（如 babble_0dB 24.79 → 27.43）反而掉点，说明 babble 噪声仍是难题。
- **Table 13 — VOiCES**：多房间 rm1–rm4 × babb/musi/none × clo/far 的细分（原文表过长此处略）。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Voices-in-the-Wild-2M]] | 2.4M / ~11k h | 7 原子×54 复合声学场景 | 训练 |
| [[LibriSpeech]] / [[Common Voice]] / [[WenetSpeech]] / [[AISHELL-1]] | 多源 | 干净语音池 | 仿真源 |
| [[MUSAN]] / DNS Challenge / ESC-50 / UrbanSound8K | 42K / 129h | 噪声/环境声池 | 仿真源 |
| [[CHiME-4]] | 16 子集 | 公交/咖啡馆/街道/步行 4 环境 | 鲁棒评测 |
| [[VOiCES]] | rm1–rm4 | 4 个房间 × babb/musi/none × clo/far | 鲁棒评测 |
| [[NOIZEUS]] | 32 子集 | 8 噪声 × 4 SNR | 鲁棒评测 |
| [[LibriSpeech]] dev/test, [[Common Voice]], [[Fleurs]], [[AISHELL-1]], [[WenetSpeech]] net/meet, [[VoxPopuli]] | — | 干净评测 | 防退化评测 |
| [[Voices-in-the-Wild-Bench]] | 5,000 | 3,500 仿真 + 1,500 真录（EN/ZH） | 复合场景评测 |

### 实现细节

- **Backbone**：[[Qwen3-ASR]]-1.7B（speech encoder + aligner + LLM 三段式）
- **SFT 学习率**：encoder/adapter 1e-3、LLM 2e-5、joint 2e-6
- **RL 阶段**：6,000 steps，学习率 1e-6，K=16 rollouts
- **混合奖励**：$0.4 \cdot R_{rule} + 0.6 \cdot R_{dynamic}$
- **超参**：$\tau=0.3$, $\alpha_s=0.4$, $\alpha_{dyn}=0.6$
- **Router**：[[LoRA]] 微调一个二分类器在干净 + Voices-in-the-Wild 混合集上

### 基线（12 系统）

[[Whisper]]-Large-v3 / Canary-1B-v2 / Parakeet-TDT-0.6B-v3 / [[Qwen2.5-Omni]]-7B / [[Step-Audio-2]]-mini / Voxtral-Mini-3B / [[Kimi-Audio]]-7B / Gemini-3-Flash / Seed-ASR / GPT-4o / Step-Audio-2 / [[Qwen3-ASR]]-1.7B。

### 可视化结果（Case Study）

Figure 6 给出最强对比：在远场 Peak –5.2 dB + 噪声 + 实体（财经术语）样本上，[[Qwen3-ASR]] 输出空（漏识 100%），Gemini-3-Pro 整段编造（[[ASR Hallucination|幻觉]] WER 86.1%），Mega-ASR 完整准确（WER 0.0%）。这正对应 LLM-as-judge 评估中漏识 Miss 14.2 → 5.9 的量化结果。

---

## 批判性思考

### 优点

1. **抓对了真问题**：D1/D2/D3 三条 deficiency 是真实存在的痛点，"组合鲁棒性"在 ASR 社区长期被忽视。
2. **数据 + 训练 + 推理三件套配套**：不是单纯堆数据，A2S-SFT 的递进解冻、DG-WGPO 的双粒度门控、Router 的干净集保险，每一环都解决一个具体问题，工程闭环完整。
3. **消融充分**：Table 5/6/7/8/9 把每个组件、每个超参、奖励设计选型（规则 vs LLM-judge）都拆开测了，可信度高。
4. **干净集不退化**：通过 Router 实现了"既要又要"，相比一般鲁棒方法的 trade-off 明显更工程化。

### 局限与风险

1. **arXiv ID `2605.19833` 在标准 YYMM 编号下指向 2026 年 5 月**，目前 [[GRPO]]/[[DAPO]]/Qwen3-ASR 等基线和 Gemini-3-Pro 都是 2025–2026 年的工作，需要交叉验证发表时点的真实性（论文里提到 Step-Audio-2 / Qwen3-ASR 等，这些都是较新的模型，年代基本自洽）。
2. **Babble 噪声仍掉点**（Table 12 station/babble 子集），说明语音对语音的干扰仍是 hard case，论文未深入讨论。
3. **没有真实远场录音对照**：训练用的远场是仿真（卷积 RIR），虽然 Voices-in-the-Wild-Bench 含 1500 真录，但占比小，是否对真实远场（如智能音箱、会议室麦阵）泛化仍需验证。
4. **奖励超参对场景敏感**：Table 8 显示 $\alpha_{dyn}=0.8$ 时 Far 场景从 7.4 跳到 9.1，部署到新场景时可能需要重新搜索。
5. **未报告 [[RTF]] / 首包延迟**：仅在 Reward 设计对比中给了相对耗时，没有给推理侧的 RTF 数据，对实时 ASR 应用方关注度可能不够。
6. **Mega-ASR 名字里的 "Mega" 主要指数据规模 240 万**，模型本体仍是 1.7B，"Mega" 略名实不符，读者需要看清。

### 潜在改进方向

1. **极端 babble**：可考虑结合 speaker diarization 或 target speaker extraction 处理同语音干扰。
2. **真实远场扩展**：把 Voices-in-the-Wild-Bench 真录部分扩到与仿真同量级，对路由器训练也有帮助。
3. **Router 替换为软门控**：当前二分类硬切，可能在边界场景上抖动，可考虑连续权重融合。
4. **跨任务复用 DG-WGPO**：双粒度门控奖励的思想可直接用到 [[Speech LLM]] 的对话生成上（low/high WER 切换 token vs sentence 级反馈）。

### 可复现性评估

- [x] 论文宣称代码开源（GitHub repo 已建立）
- [x] 数据集承诺在 HuggingFace 公开（Voices-in-the-Wild-2M）
- [x] 评测 Bench 单独开源
- [x] 训练超参在 Appendix E 给出完整
- [ ] checkpoint 是否放出（截至论文上线时间未确认）
- [ ] 复合场景的精确配方（54 种组合的具体 RIR/噪声参数）是否完整公开

---

## 关联笔记

### 基于

- [[Qwen3-ASR]]：Mega-ASR 的 backbone
- [[DAPO]]：RL 框架基础，DG-WGPO 在其上扩展
- [[WGPO]]：DG-WGPO 名字源头（WER-Gated Policy Optimization 的早期版本）

### 对比基线

- [[Whisper]]：闭集鲁棒老牌基线
- [[Kimi-Audio]] / [[Step-Audio-2]] / [[Qwen2.5-Omni]]：开源 [[Speech LLM]] / Omni 模型
- Gemini-3-Flash / GPT-4o-trans / Seed-ASR：闭源黑盒对照

### 方法相关

- [[A2S-SFT]]：本文提出的递进式 SFT 方案
- [[DG-WGPO]]：本文提出的双粒度门控强化学习
- [[Curriculum Learning]]：A2S-SFT 的训练范式基础
- [[GRPO]]：消融对照
- [[LoRA]]：Router 微调用

### 数据/评测相关

- [[Voices-in-the-Wild-2M]]：核心训练数据
- [[Voices-in-the-Wild-Bench]]：核心评测集
- [[CHiME-4]] / [[VOiCES]] / [[NOIZEUS]]：三大鲁棒 benchmark
- [[WER]] / [[CER]]：核心评测指标
- [[ASR Hallucination]]：要解决的核心问题之一

---

## 速查卡片

> [!summary] Mega-ASR (arXiv:2605.19833)
> - **核心**：用 7 原子×54 复合声学场景的 2.4M 仿真数据 + A2S-SFT + DG-WGPO RL，把鲁棒 ASR 推到 ASR-in-the-wild²。
> - **方法**：递进难度课程 SFT + DAPO 上扩展的双粒度 WER 门控奖励（token 级 $R_{fine}$ + 句子级 $R_{struc}$）+ LoRA 路由防干净集退化。
> - **结果**：三大鲁棒 benchmark 平均 WER 6.70 vs Qwen3-ASR 7.93（相对 –15%）；漏识 Miss 14.2 → 5.9（–58%）；复合场景 Mixed WER 5.39 → 4.57（–18%）。
> - **代码**：https://github.com/xzf-thu/Mega-ASR
> - **数据**：https://huggingface.co/datasets/zhifeixie/Voices-in-the-Wild-2M

---

*笔记创建时间: 2026-05-21*
