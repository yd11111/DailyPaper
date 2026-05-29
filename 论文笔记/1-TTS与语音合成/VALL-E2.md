---
title: "VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers"
method_name: "VALL-E 2"
authors: [Sanyuan Chen, Shujie Liu, Long Zhou, Yanqing Liu, Xu Tan, Jinyu Li, Sheng Zhao, Yao Qian, Furu Wei]
year: 2024
venue: arXiv
arxiv_id: "2406.05370"
tags: [tts, zero-shot-tts, codec-lm, ar-nar, robustness]
zotero_collection:

# === 论文核心技术元数据（三层 verify，每条带来源）===
lm_init: "cold-start，AR/NAR Transformer 在 Libriheavy 上从随机初始化训练，沿用 VALL-E 架构；全文未提任何通用 LLM 预训练初始化 [已 verify §4.1.1]"
training_loss: "纯 speech-token 负对数似然，无文本 loss/无 KL/无多任务。AR 对第一码本建模 (Eq.9-11)；NAR 对码本 1-7 建模，训练时每步随机抽一个 j 优化 (Eq.17) [已 verify §3.3, Eq.9-17]"
tokenizer_arch: "text+speech 分离。文本走 BPE；语音走 EnCodec 8 码本 RVQ。文本/码嵌入独立矩阵；层次化 AR(码本0，沿时间轴分组)+NAR(码本1-7) [已 verify §3.2, §4.1.1]"
multitask: false "[已 verify §3.1 单一 TTS codec LM 目标]"
training_data: "Libriheavy 50k 小时，约 7000 说话人，英文有声书（LibriVox/LibriLight 标注版） [已 verify §4.1.1]"
post_training: "无（无 RLHF/DPO 等后训练）[已 verify，全文未提]"
codec_detail: "EnCodec，J=8 量化器（RVQ 8 层），6 kbps，24kHz；解码用 Vocos；AR 分组大小 G∈{1,2,4,8} [已 verify §3.1 (J=8), §4.1.1]"

# === 知识地图联动 ===
domain: TTS
subdomain: zero-shot-tts
routes: [codec-lm-tts]
problems: [zero-shot-cloning, long-form-stability, latency, speaker-similarity, evaluation]
representations: [acoustic-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-代表模型谱系]]"
related_surveys:
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-29

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2406.05370/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2406.05370/paper.html"
figures_dir: "_resources/2406.05370/figures"
github_local:
cached_at: 2026-05-29

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2406.05370v2
created: 2026-05-29
---

# 论文笔记：VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Microsoft Corporation（一作 Sanyuan 实习期间完成，现独立研究者） |
| 日期 | June 2024 |
| 项目主页 | https://aka.ms/valle2 （仅 demo，明确声明不开源、不产品化） |
| 对比基线 | [[VALL-E]]（同 NAR、AR group size=1 的等价复现版） |
| 链接 | [arXiv](https://arxiv.org/abs/2406.05370) / Code: 无（作者声明不发布） |

---

## 一句话总结

> 在 [[VALL-E]] 基础上加 **Repetition Aware Sampling** 治解码不稳定、加 **Grouped Code Modeling** 沿时间轴分组缩短 AR 序列，号称首个在 LibriSpeech/VCTK 零样本 TTS 上达到 human parity 的 codec LM。

---

## 核心贡献

1. **Repetition Aware Sampling (RAS)**: 在 nucleus sampling 基础上，按解码历史中 token 的局部重复率自适应切换 nucleus / random sampling，既稳定解码又规避无限循环，且几乎不增加延迟。
2. **Grouped Code Modeling**: 把第一码本的连续 $G$ 帧打包成一个 AR 帧建模，摆脱 off-the-shelf codec 的固定帧率约束，序列长度缩短为 $1/G$，同时缓解长上下文建模问题。
3. **Human parity 声明**: 在 LibriSpeech test-clean 与 out-of-domain VCTK 上，SIM/WER/DNSMOS 客观指标与 SMOS/CMOS 主观指标均称超过 ground truth，且**仅需句级语音-文本配对数据**训练（不需 force-alignment / 额外参考音频）。

---

## 问题背景

### 要解决的问题
零样本 TTS：给一段陌生说话人的短录音（3 秒级），合成保留其音色、情感、声学环境的任意文本语音。

### 现有方法的局限
[已 verify §1] 作者把 [[VALL-E]] 的痛点归为两条：
1. **稳定性 (Stability)**：VALL-E 推理用 random sampling 导致输出不稳定；若改用小 top-p 的 nucleus sampling 又会陷入无限循环（重复生成）。缓解办法是多次采样后排序，但增加计算成本。
2. **效率 (Efficiency)**：AR 架构被 off-the-shelf codec 的高帧率绑死，无法调整，推理慢。

后续工作的两条路线各有代价 [已 verify §1, §2.1]：
- **引入对齐信息**（ELLA-V、RALL-E）：依赖 forced-alignment 模型，引入对齐误差、复杂化架构、增加数据扩展负担。
- **全 NAR**（SoundStorm、Voicebox、NaturalSpeech 3）：需要帧对齐的文本-语音数据，且预设 duration 限制了生成的搜索空间，牺牲韵律与自然度。

### 本文的动机
保留 VALL-E 的纯 codec LM 范式（不需对齐信息、不需 duration），只在**采样策略**和**序列组织**两处改进，即可同时解决稳定性与效率，并保持数据收集的简单性。

---

## 方法详解

### 领域定位

<!-- R4 -->
VALL-E 2 属于 **离散 codec token + 语言模型（codec-LM-TTS）** 路线，与 [[VALL-E]]、[[VALL-E-X]]、SPEAR-TTS、UniAudio 同类，沿用 VALL-E 的 **AR（粗码本）+ NAR（细码本）层次化** 架构。相对前作的核心差异**不在模型结构而在推理采样与序列分组**：RAS 是纯推理期改动（不改训练），Grouped Code Modeling 是对 AR 第一码本序列沿时间轴的重组。两者都刻意避开了"引入对齐/duration"这条主流提鲁棒路线。

### 模型架构

[已 verify §3.2] VALL-E 2 沿用 VALL-E 的两段式：
- **AR 模型**：以自回归方式生成每帧的**第一码本** code 序列 $\mathbf{c}_{:,0}$，causal attention mask。
- **NAR 模型**：以非自回归方式，基于已生成的前序码本生成**剩余码本（2-8）** 序列，full attention mask。
- **共享设计**：两模型用同一 Transformer 架构，含文本嵌入层、码嵌入层、码预测层；不同码本量化器用**独立**的 code 嵌入；码预测层与码嵌入层**共享参数**（weight tying）。
- **AR 特有**：额外的 **group embedding 层**（把组内 code 嵌入投影成组嵌入）+ **group prediction 层**（一次预测组内全部 code）。
- **NAR 特有**：**code ID embedding 层**，指定当前要预测第几个码本。

### 核心模块

#### 模块1: Grouped Code Modeling（分组 codec 语言建模）

**设计动机** [已 verify §3.1]：摆脱 off-the-shelf codec 的固定帧率约束，让帧率按整数倍降低——既提升推理效率（序列变短），又通过缩短上下文缓解长序列建模问题。

**具体实现** [已 verify §3.1, §3.3.1]：
- 把 codec code 序列 $\mathbf{C}^{T\times J}$（$J=8$）沿**时间轴** $T$ 切成大小为 $G$ 的组，每组 $[\mathbf{c}_0,\dots,\mathbf{c}_{G-1}]$ 当作一个 AR 帧。
- 因句首通常有静音，可从序列开头裁掉几帧让 $T$ 成为 $G$ 的整数倍，不损失语音信息。
- AR 训练时：把第一码本嵌入序列按 $G$ 分组，组内嵌入在隐维度拼接，过 group embedding 矩阵 $\mathbf{W}^g$ 得组嵌入；组间自回归、组内非自回归地预测。
- **注意**：分组是沿**时间维**（连续帧），**不是**跨 RVQ 层堆叠（这是常见误读）。

> **My Assessment**：把"降帧率"从 codec 设计问题转成 LM 侧的序列重组，是个轻量且优雅的解耦。代价是组内并行预测时帧间细粒度依赖被削弱——实验显示 $G$ 增大到 4/8 后 SIM 与 WER 确实退化（见 Table 1/4）。

#### 模块2: Repetition Aware Sampling（RAS，重复感知采样）

**设计动机** [已 verify §3.4.1]：nucleus sampling 稳但小 top-p 会无限循环；random sampling 不循环但不稳。RAS 想兼得两者。

**具体实现**（Algorithm 1）[已 verify §3.4.1]：
- 先用预设 top-p 值 $v$ 做 nucleus sampling 得候选 code $c_{t'}$。
- 计算 $c_{t'}$ 在前序窗口 $K$ 内的重复率 $r$。
- 若 $r > t_r$（重复阈值），改用 random sampling 重采该 code。
- 组内 code 虽以 NAR 方式建模，但为计算重复率与切换采样，仍逐个自回归地预测。
- 几乎不增延迟（采样开销相对模型推理可忽略）。
- 推理超参 [已 verify §4.1.3]：$K=10$，$t_r=0.1$，$v$ 在 $0.0\!-\!0.8$ 间以 0.1 步长扫。

> **My Assessment**：RAS 是纯推理期 trick，不改训练、零额外训练成本，这是它最实用之处。关键效果是**让模型能在极小 top-p（甚至 0）下稳定解码**，从而拿到比 ground truth 还低的 WER（见下文可信度讨论）。

---

## 关键公式

### 公式1: [[Grouped Code Modeling|分组 codec LM 训练目标]]

$$
\mathcal{L} = -\log p(\mathbf{C}^{G}\mid\mathbf{x};\theta) = -\sum_{t=0}^{T/G-1}\log p\big(\mathbf{C}_{t\cdot G:(t+1)\cdot G}\mid \mathbf{C}_{<t\cdot G},\mathbf{x};\theta\big)
$$

**含义**: 在文本条件 $\mathbf{x}$ 下，最大化分组 code 序列的似然；自回归单位从单帧变成"组"。

**符号说明**:
- $\mathbf{x}=[x_0,\dots,x_{L-1}]$: BPE 文本 token 序列
- $\mathbf{C}^{T\times J}$: codec code 矩阵，$J=8$ 个码本
- $G$: 组大小（1/2/4/8）
- $\mathbf{C}_{t\cdot G:(t+1)\cdot G}$: 第 $t$ 个组

### 公式2: [[Autoregressive Model|AR 模型损失]]

$$
\mathcal{L}_{\text{AR}} = -\sum_{t=0}^{T/G-1}\sum_{t'=t\cdot G}^{(t+1)\cdot G-1}\log p\big(c_{t',0}\mid \mathbf{c}_{<t\cdot G,0},\mathbf{x};\theta_{\text{AR}}\big)
$$

**含义**: AR 只建模**第一码本** $\mathbf{c}_{:,0}$；组间自回归（条件只到上一组），组内各帧并行预测。

**符号说明**:
- $c_{t',0}$: 第 $t'$ 帧第 0 个码本的 code
- $\mathbf{c}_{<t\cdot G,0}$: 当前组之前所有帧的第一码本 code

### 公式3: [[Non-Autoregressive Model|NAR 嵌入聚合]]

$$
\mathbf{e}^{c}_{t}=\begin{cases}\sum_{k=0}^{7}\mathbf{W}^{c}\odot c_{t,k}, & t<T'\\[4pt] \sum_{k=0}^{j-1}\mathbf{W}^{c}\odot c_{t,k}, & t\ge T'\end{cases}
$$

**含义**: NAR 训练时显式把序列切成声学条件 $\mathbf{C}_{<T'}$ 与目标 $\mathbf{C}_{\ge T'}$。条件段用全部 8 个码本求和，目标段只用已生成的前 $j$ 个码本求和——这样 NAR 能从 prompt 全 8 码本中抽取更多说话人信息。

**符号说明**:
- $T'$: 随机采样的声学条件长度（训练时取"当前句一半"与 3s–30s 随机值的较大者 [§4.1.1]）
- $\mathbf{W}^{c}$: 共享码嵌入矩阵
- $j$: 当前预测的码本 ID

### 公式4: [[Non-Autoregressive Model|NAR 模型损失]]

$$
\mathcal{L}_{\text{NAR}} = -\sum_{j=1}^{7}\log p\big(\mathbf{c}_{\ge T',j}\mid\mathbf{x},\mathbf{C}_{<T'},\mathbf{C}_{\ge T',<j};\theta_{\text{NAR}}\big)
$$

实际训练为提效，每步**随机抽一个** $j\in[1,7]$ 优化（Eq.17）：

$$
\mathcal{L}_{\text{NAR\_j}} = -\log p\big(\mathbf{c}_{\ge T',j}\mid\mathbf{x},\mathbf{C}_{<T'},\mathbf{C}_{\ge T',<j};\theta_{\text{NAR}}\big)
$$

**含义**: 逐码本预测目标段，每个码本以前序码本为条件；随机抽 $j$ 避免每步遍历全部 7 个码本的开销。

### 公式5: [[Repetition Aware Sampling|重复率]]（Algorithm 1）

$$
r \leftarrow \frac{1}{K}\sum_{k=0}^{K}\mathbb{1}_{c_{t'}=c_{t'-k}}
$$

**含义**: 候选 code 在前 $K$ 个 code 窗口内出现的频率。$r>t_r$ 则触发 random sampling 重采。

**符号说明**:
- $K=10$（窗口大小），$t_r=0.1$（阈值）

### 公式6: 五次采样的 SIM-WER 排序

$$
\hat{\mathbf{y}}_{\text{best}}=\operatorname*{arg\,max}_{\hat{\mathbf{y}}_{i}}\big([\min(\hat{\mathbf{y}}_{i}^{\text{SIM}},0.3),\,1-\hat{\mathbf{y}}_{i}^{\text{WER}}]\big)
$$

**含义**: 五候选里选最优。SIM>0.3 时按 WER 排（字典序），否则按 SIM 排——先保证音色及格再优化可懂度。

---

## 关键图表

> 本论文共 **7 个 Figure、6 个 Table**，全部收录如下。图片用 arXiv HTML 外链（v1，已验证可达）。

### Figure 1: Human Parity 雷达对比

![Figure 1](https://arxiv.org/html/2406.05370v1/x1.png)

**说明** [§1]: 以"相对 ground truth 的差值"($\triangle\text{Score}=\text{Score(Model)}-\text{Score(GT)}$) 展示鲁棒性/自然度/相似度。VALL-E 2 三项均 >0（即超过真人），其余对比工作（VALL-E、ELLA-V 等）均为负。**注意**：作者明确标注这些数字是从各论文报告值算的相对值，不控制架构与训练数据差异——这是跨论文比较，非同设置复现。

### Figure 2: 训练总览（AR + NAR Transformer）

![Figure 2](https://arxiv.org/html/2406.05370v1/x2.png)

**说明** [§3.3]: 上半为 NAR（full attention），下半为 AR（causal attention，生成分组 code）。两模型共享文本/码嵌入设计，AR 多出 group embedding/prediction 层，NAR 多出 code ID embedding 层。

### Figure 3: 推理总览（含 RAS）

![Figure 3](https://arxiv.org/html/2406.05370v1/x3.png)

**说明** [§3.4]: 文本（prompt 转录 + 待合成文本）拼成文本条件；陌生说话人音频转 code 作为 prompt。AR 用 RAS 生成第一码本，NAR 跑 7 次贪心生成码本 2-8，最后过 codec decoder 出波形。

### Figure 4: LibriSpeech 解码稳定性

![Figure 4](https://arxiv.org/html/2406.05370v1/x4.png)

**说明** [§4.2.1]: GS=group size，RAS=repetition aware sampling。RAS 让模型能在极小 top-p（甚至 0）下稳定解码，显著降低错误率——这是拿到低于 GT 的 WER 的关键。

### Figure 5: LibriSpeech 训练数据量消融

![Figure 5](https://arxiv.org/html/2406.05370v1/x5.png)

**说明** [§4.2.3]: 10k 小时已接近 50k 小时性能，多出的 40k 仅带来轻微提升；但 <10k 会明显退化（尤其 ref utterance 设置）。结论限于有声书域。

### Figure 6: VCTK 采样稳定性

![Figure 6](https://arxiv.org/html/2406.05370v1/x6.png)

**说明** [§4.3.1]: 与 LibriSpeech 一致，RAS 在口音多样的 VCTK 上同样能用较小 top-p 生成更鲁棒语音。

### Figure 7: VCTK 训练数据量消融

![Figure 7](https://arxiv.org/html/2406.05370v1/x7.png)

**说明** [§4.3.3]: VCTK（out-of-domain）上训练数据量的影响趋势。

### Table 1: LibriSpeech test-clean 客观评测（节选关键行）

| System | GroupSize | 3s Prefix SIM↑ | 3s Prefix WER↓ | 3s Prefix DNSMOS↑ | Ref Utt SIM↑ | Ref Utt WER↓ | Ref Utt DNSMOS↑ |
|---|---|---|---|---|---|---|---|
| GroundTruth | - | 0.905 | 1.6 | 3.891 | 0.779 | 1.6 | 3.891 |
| ↪ Codec 重建上界 | - | 0.823 | 1.7 | 3.886 | 0.715 | 1.7 | 3.886 |
| **Single Sampling** | | | | | | | |
| VALL-E | 13ms | 0.773 | 2.3 | 3.942 | 0.633 | 3.1 | 3.985 |
| VALL-E 2 | ×1 | 0.782 | **1.6** | 3.947 | 0.643 | **1.5** | 3.987 |
| VALL-E 2 | ×2 | 0.777 | 1.5 | 3.966 | 0.635 | 1.5 | 4.000 |
| VALL-E 2 | ×4 | 0.773 | 1.8 | 3.950 | 0.615 | 2.2 | 3.967 |
| VALL-E 2 | ×8 | 0.766 | 2.5 | 3.937 | 0.566 | 4.2 | 3.875 |
| **Five-Time (Sort SIM&WER)** | | | | | | | |
| VALL-E | 13ms | 0.802 | 1.0 | 3.944 | 0.676 | 0.8 | 3.987 |
| VALL-E 2 | ×1 | 0.807 | 1.0 | 3.943 | 0.687 | 0.7 | 3.994 |
| VALL-E 2 | ×2 | 0.803 | 1.0 | 3.967 | 0.679 | 0.6 | 3.997 |

**说明** [§4.2.1]: 单次采样下 VALL-E 2 的 WER（1.5–1.6）已低于 GT（1.6），且远好于 VALL-E single（2.3/3.1）；这是 RAS 的直接收益。$G{=}2$ 反而 WER/DNSMOS 更优（缓解长上下文）；$G{\ge}4$ 后 SIM/WER 退化。

### Table 2: LibriSpeech 主观评测（40 说话人，ref utterance）

| System | GroupSize | SMOS↑ | CMOS↑ |
|---|---|---|---|
| GroundTruth | - | 4.13 ±0.32 | 0.00 |
| VALL-E | 13ms | 4.45 ±0.28 | -0.268 |
| **VALL-E 2** | ×1 | **4.61 ±0.19** | **+0.033** |
| VALL-E 2 | ×2 | 4.51 ±0.26 | -0.167 |

**说明** [§4.2.2]: VALL-E 2 ×1 的 SMOS（4.61）与 CMOS（+0.033，>0 即优于 GT）双双超过真人，构成 human parity 主张的核心证据。

### Table 3: LibriSpeech 输入消融（group size 1）

| AR Prompt | NAR Text | NAR Prompt | 设置 | Ref Utt SIM↑ | Ref Utt WER↓ |
|---|---|---|---|---|---|
| ✓ | ✓ | ✓ | Single | 0.639 | 1.9 |
| ✗ | ✓ | ✓ | Single | 0.169 | 2.8 |
| ✓ | ✓ | ✦(不显式切分) | Single | 0.530 | 1.9 |
| ✓ | ✓ | ✗ | Single | 0.385 | 1.8 |
| ✓ | ✗ | ✓ | Single | 0.619 | **10.0** |

**关键发现** [§4.2.3]: ① 去掉 AR prompt → SIM 暴跌到 0.169（AR prompt 对音色至关重要，且也降 WER，因约束了一对多搜索空间）；② NAR 不显式切分声学条件（✦）→ SIM 0.530 vs 0.639（显式切分能从 8 码本 prompt 抽更多说话人信息）；③ NAR 去文本 → WER 飙到 10.0（即便 AR 已用文本，NAR 仍需文本保可懂度）。

### Table 4: VCTK 客观评测（节选 single sampling，108 说话人 out-of-domain）

| System | GroupSize | 3s SIM↑ | 3s WER↓ | 5s SIM↑ | 5s WER↓ | 10s SIM↑ | 10s WER↓ |
|---|---|---|---|---|---|---|---|
| GroundTruth | - | 0.623 | 0.3 | 0.679 | 0.3 | 0.709 | 0.3 |
| VALL-E | 13ms | 0.430 | 2.4 | 0.455 | 3.1 | 0.533 | 5.8 |
| VALL-E 2 | ×1 | 0.447 | 0.9 | 0.487 | 1.9 | 0.558 | 3.3 |
| VALL-E 2 | ×2 | 0.426 | 1.5 | 0.481 | 0.9 | 0.557 | 2.3 |

**说明** [§4.3.1]: RAS 在口音多样的 VCTK 上把单次采样 WER 大致砍半。长 prompt（10s）时 grouped code modeling 进一步降 WER（长序列建模收益更明显）。**但客观 SIM 全面低于 GT**（如 3s：0.447 vs 0.623）——客观相似度并未达 parity。

### Table 5: VCTK 主观评测（60 说话人）

| System | GroupSize | 3s SMOS↑ | 3s CMOS↑ | 5s SMOS↑ | 5s CMOS↑ | 10s SMOS↑ | 10s CMOS↑ |
|---|---|---|---|---|---|---|---|
| GroundTruth | - | 4.47 ±0.13 | 0.00 | 4.53 ±0.14 | 0.00 | 4.74 ±0.17 | 0.00 |
| VALL-E | 13ms | 4.32 ±0.16 | +0.028 | 4.05 ±0.20 | +0.144 | 3.50 ±0.49 | +0.094 |
| VALL-E 2 | ×1 | 4.42 ±0.15 | **+0.207** | 4.28 ±0.16 | +0.079 | 3.95 ±0.10 | +0.117 |
| VALL-E 2 | ×2 | **4.47 ±0.13** | +0.163 | 4.14 ±0.17 | +0.217 | 4.26 ±0.42 | +0.109 |

**说明** [§4.3.2]: CMOS 全为正（自然度优于 GT）。但 **SMOS 在 3s 仅 VALL-E 2 ×2 (4.47) 追平 GT (4.47)，×1 (4.42) 仍略低**；5s/10s SMOS 均明显低于 GT。即 VCTK 上"音色相似度"的 parity 主张较弱。

### Table 6: VCTK 输入消融

**说明** [§4.3.3]: 与 Table 3 结论一致——AR/NAR prompt 与 NAR 文本输入均对 SIM/WER 关键。完整数据见原文 Table 6。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[Libriheavy]] | 50k 小时, ~7000 说话人 | 英文有声书（LibriLight 标注版） | 训练 |
| [[LibriSpeech]] test-clean | 4–10s 子集, 2.2h, 40 说话人 | in-domain（同 LibriVox 域，speaker 不重叠） | 测试 |
| [[VCTK]] | 108 说话人, 48kHz | out-of-domain，口音多样 | 测试 |

### 实现细节
[已 verify §4.1.1]
- **文本 tokenizer**: BPE
- **语音 tokenizer**: [[EnCodec]] 6kbps / 24kHz，$J=8$ 码本
- **codec decoder**: [[Vocos]]（替代 EnCodec 原解码器，提质量）
- **架构**: AR/NAR 同 [[VALL-E]] 的 Transformer；评测 4 个 AR 模型（$G$=1/2/4/8）共享同一 NAR；$G{=}1$ 不含 group 层即等价 baseline VALL-E
- **优化器**: AdamW，前 32k step 学习率 warmup 到峰值后线性衰减
- **硬件**: 16× NVIDIA TESLA V100 32GB
- **NAR 训练**: 声学条件长度取"当前句一半"与 3s–30s 随机值的较大者
- **评测指标**: 客观 SIM（[[WavLM]]-TDNN）、WER（NVIDIA Conformer-Transducer XLarge ASR）、DNSMOS；主观 SMOS、CMOS（20 名美式英语母语者众包）

### 结果可信度

<!-- R3 -->

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | LibriSpeech 客观 WER：VALL-E 2 single sampling 优于 VALL-E（1.5 vs 3.1）| 有标准 benchmark、公开 ASR 工具、与等价复现基线公平对比 |
| **中** | "human parity"（超过 GT 的 WER/CMOS/SMOS）| 主观评审仅 20 人、测试集小（LS 40 例 / VCTK 60 例）、未报显著性检验；CMOS 仅 +0.033 在测量噪声内 |
| **中** | "WER 低于 ground truth" | GT 的 WER 用同一 ASR 测得（1.6），合成语音咬字更"标准"反而更易被 ASR 识别，**不等于更自然**；是已知的可懂度悖论 |
| **低** | Figure 1 跨模型雷达对比 | 作者自述为跨论文相对值，不控制架构/数据差异，仅作趋势示意 |
| **低** | VCTK 音色 parity | 客观 SIM 全面低于 GT；主观 SMOS 仅 ×2@3s 追平，其余低于 GT |

---

## 批判性思考

### 核心 Claim 审查

<!-- R1 -->

1. **Paper Claim**: "首个达到 human parity 的零样本 TTS 系统"。
   **My Assessment**: 在 **LibriSpeech（in-domain）** 上证据较强（SMOS 4.61>4.13，CMOS +0.033>0）；但在 **VCTK（out-of-domain）** 上证据明显弱——客观 SIM 全面低于 GT，主观 SMOS 多数设置低于 GT，仅 ×2@3s 追平。作者自己也在 §1 限定"此结论仅基于 LibriSpeech 与 VCTK"。所以"human parity"成立于受限设置，宜读作"在两个朗读语料的部分指标上达到或超过真人"。

2. **Paper Claim**: "WER 甚至低于 ground truth，说明合成语音高度忠实于文本"。
   **My Assessment**: 数字属实 [Tab.1]，但"低于 GT 的 WER"主要反映 RAS 让模型用极小 top-p 生成"咬字更规整、更易被 ASR 识别"的语音，这是 TTS 领域已知的可懂度悖论，**不能等同于更自然或更像人**。CMOS/SMOS 才是自然度证据，而它们的优势幅度很小。

3. **Paper Claim**: "Grouped Code Modeling 既加速又提质"。
   **My Assessment**: $G{=}2$ 确实 WER/DNSMOS 更优且序列减半 [Tab.1]，这点可信；但"提质"有上限——$G{\ge}4$ 后 SIM/WER 持续退化，本质是用细粒度建模换效率。称其"提质"应限定在 $G{=}2$。

### 优点
1. **零训练成本的鲁棒性修复**：RAS 是纯推理 trick，不改训练、不增延迟，却把 single-sampling WER 从 3.1 降到 1.5（LibriSpeech ref utt），工程上极易迁移到任何 codec-LM。
2. **效率/序列长度的优雅解耦**：把"降帧率"从 codec 设计问题转成 LM 侧时间轴分组，$G{=}2$ 几乎白拿 2× 加速且不掉点。
3. **数据需求极简**：只需句级语音-文本配对，不要 forced-alignment、不要 duration、不要同说话人额外参考音频——相比 ELLA-V/RALL-E/NAR 系显著降低数据工程负担。
4. **消融扎实**：Table 3/6 清晰拆出 AR prompt / NAR prompt / NAR 显式切分 / NAR 文本各自的贡献。

### 局限性
1. **human parity 主张过强**：VCTK 客观 SIM 与多数 SMOS 未达 GT；主观评审样本量小、无显著性检验，CMOS 优势在噪声量级。
2. **不开源**：作者明确声明无代码、无 checkpoint、不产品化，复现完全依赖第三方重写，evidence_level 只能 medium。
3. **域窄**：训练/测试都在英文朗读/有声书域；数据量消融结论也自述限于有声书域。无多语言、无对话、无情感表达评测。
4. **codec 仍是 off-the-shelf EnCodec**：codec 重建上界（Table 1 "↪ Codec" 行 SIM 0.715）已是天花板，模型 SIM 不可能超过它——音色相似度被 codec 锁死。
5. **NAR 仍跑 7 次贪心**：码本 2-8 逐个生成，效率收益主要来自 AR 分组，NAR 侧未优化。

### 潜在改进方向
1. 把 RAS 与更强 codec（DAC / Mimi / 单码本 codec）结合，突破 EnCodec 的 SIM 上界。
2. 用语义 token 中介或更大 LM warm-start，验证 codec-LM 是否还能继续提自然度。
3. 引入显著性检验与更大评审规模，使 parity 主张可统计验证。

### 可复现性评估
- [ ] 代码开源（作者明确声明不发布）
- [ ] 预训练模型（无）
- [x] 训练细节较完整（数据/优化器/硬件/超参均给）
- [x] 数据集可获取（Libriheavy / LibriSpeech / VCTK 均公开）

---

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §路线 2（离散 codec token + LM），AR+NAR 层次化子类
- **核心问题**：[[TTS-核心挑战]] §挑战 1（零样本克隆）、§挑战 2/3（长文本稳定性、效率/延迟）、§挑战 6（评估方法论：human parity 主张的可信度边界）
- **表示层位置**：[[TTS-表示层地图]] §acoustic-token（EnCodec RVQ）
- **在 SpeechLM/对话框架内的位置**：[[TTS-SpeechLM-Dialogue关系]] 位置 ②（codec-LM TTS，非统一 SpeechLM——它是从头训的 TTS-only codec LM，无 LLM warm-start [已 verify §4.1.1]）
- **相邻工作**：[[VALL-E]]（直接前作）/ [[VALL-E-X]]（跨语言扩展）/ ELLA-V / RALL-E（对齐增强路线，本文刻意避开）/ [[Voicebox]]（NAR flow 路线对照）

---

## 🔄 后续重估

- **2026-05-29**：初读并三层 verify（L1 论文 §1-§4 + Eq.1-23 + Tab.1-6 已读；L2 GitHub 不可用——作者声明不开源；L3 暂无第三方独立复现）。判断：RAS + Grouped Code Modeling 是两个低成本、可迁移的工程改进，技术上扎实；但"human parity"主张在 in-domain LibriSpeech 成立较好、out-of-domain VCTK 较弱，且无显著性检验 + 不开源，故 evidence_level=medium、maturity=emerging。核心价值在于"纯推理期采样 trick 即可大幅修鲁棒"这一可复用洞察，而非 parity 标题本身。

---

## 关联笔记

### 基于
- [[VALL-E]]: 直接前作，沿用其 AR(粗码本)+NAR(细码本) 架构与 codec-LM 范式
- [[EnCodec]]: 语音 tokenizer（6kbps/24kHz, 8 码本 RVQ）
- [[Vocos]]: 替代 EnCodec 解码器，提升合成质量

### 对比
- ELLA-V / RALL-E: 同为 VALL-E 鲁棒性改进，但走"引入对齐信息"路线（本文刻意避开）
- [[Voicebox]]: NAR flow-matching 路线，需 duration，本文据此论证 NAR 牺牲韵律
- [[VALL-E-X]]: VALL-E 的跨语言扩展，旁系

### 方法相关
- [[Repetition Aware Sampling]]: 核心推理期方法
- [[Grouped Code Modeling]]: 核心序列组织方法
- [[RVQ]]: codec 量化基础
- [[Zero-shot TTS]]: 任务范式

### 硬件/数据相关
- [[Libriheavy]]: 50k 小时训练数据
- [[LibriSpeech]] / [[VCTK]]: 评测集
- [[WavLM]]: SIM 评测的 speaker verification 模型

---

## 速查卡片

> [!summary] VALL-E 2
> - **核心**: VALL-E + RAS(治不稳定) + Grouped Code Modeling(沿时间轴分组提效)，声称首个 human parity 零样本 TTS
> - **方法**: AR(第一码本,分组) + NAR(码本2-8); 纯推理 trick RAS 按重复率切换 nucleus/random sampling
> - **结果**: LibriSpeech single-sampling WER 1.5<GT 1.6, SMOS 4.61>GT 4.13; VCTK 音色 parity 较弱
> - **数据**: Libriheavy 50k h; EnCodec 8码本6kbps + Vocos; 16×V100
> - **代码**: 不开源（作者声明）

---

*笔记创建时间: 2026-05-29*
