---
title: "Raon-OpenTTS: Open Models and Data for Robust Text-to-Speech"
method_name: "Raon-OpenTTS"
authors: [Semin Kim, Seungjun Chung, Taehong Moon, Sangheon Lee, Minyoung Ahn, Keon Lee, Nam Soo Kim, Jaewoong Cho, Ludwig Schmidt, Kangwook Lee, Dongmin Park]
year: 2026
venue: arXiv
tags: [tts, open-data, dit, flow-matching, large-scale-data, robust-tts, benchmark]
zotero_collection: ""
image_source: online
arxiv_html: https://arxiv.org/html/2605.20830v1
created: 2026-05-21
---

# 论文笔记：Raon-OpenTTS: Open Models and Data for Robust Text-to-Speech

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | KRAFTON、Seoul National University、KAIST、Stanford、UW-Madison |
| 日期 | 2026 年 5 月 |
| 项目主页 | https://github.com/krafton-ai/RAON-OpenTTS |
| 对比基线 | [[F5-TTS]]、[[MaskGCT]]、[[CosyVoice 2]]、[[CosyVoice 3]]、[[VoxCPM]]、[[Qwen3-TTS]]、Llasa、Voxtral TTS、[[IndexTTS 2]]、[[Seed-TTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2605.20830) / [Code](https://github.com/krafton-ai/RAON-OpenTTS) |

---

## 一句话总结

> 用 615K 小时全开源英文语音数据 + 510K 小时模型化过滤子集 + 4 场景鲁棒性 benchmark，把 [[F5-TTS]] 同款 [[DiT]] 架构推到能与 [[Qwen3-TTS]]、[[CosyVoice 3]] 等闭源数据 SOTA 掰手腕。

---

## 核心贡献

1. **Raon-OpenTTS-Pool 数据池**: 615K 小时、240M 段英文语音的全开源数据池，重头戏是 335K 小时的 [[Raon-YouTube-Commons]]（CC BY 4.0 自建预处理管线）+ 280K 小时 10 个公开数据集聚合。
2. **Raon-OpenTTS-Core 过滤子集**: 用 [[DNSMOS]] + [[Whisper]] WER + [[Speech Ratio]] 三指标 15 分位组合过滤，得到 510K 小时、194M 段的高质量训练子集，定量验证「Combined 15%」是最优过滤策略。
3. **Raon-OpenTTS-Eval 鲁棒性 benchmark**: 6K 条 prompt-text 对、12 个数据集，覆盖 Clean / Noisy / Wild / Expressive 四种声学条件，弥补 Seed-TTS-eval 只测干净语音的盲区。
4. **Raon-OpenTTS 模型族**: 完全沿用 [[F5-TTS]] 的 [[DiT]] + [[Flow Matching]] 架构（控制变量、隔离数据效应），训 0.3B / 1B 两个尺寸；1B 模型在 Seed-TTS-Eval 上 WER 1.78 / SIM 0.749，全面追平甚至超越用百万小时私有数据训的闭源模型。

---

## 问题背景

### 要解决的问题

TTS 领域的 SOTA（[[CosyVoice 3]]、[[Qwen3-TTS]]、[[Seed-TTS]]）都靠数百万小时私有数据 + 闭权重。即使开源圈 SOTA 如 [[F5-TTS]]、[[MaskGCT]] 也只能用 [[Emilia]]（100K h）这种规模，且**没有同时开放数据、过滤管线、训练代码、checkpoint 的端到端可复现栈**。

### 现有方法的局限

1. **开源数据规模太小**: 主流可用的 [[Emilia]]、[[LibriTTS]]、[[GigaSpeech]] 单独都不够；YouTube-Commons 虽规模大但缺端到端预处理。
2. **过滤策略缺乏定量证据**: 各家都说"我们做了清洗"，但很少有人系统比较「按 DNSMOS 还是按 WER 还是按 Speech Ratio 过滤效果好」、「过滤多少比例最合适」。
3. **TTS 评测过于乐观**: [[Seed-TTS-eval]]、CV3-Hard 主要是干净/朗读音频，无法暴露 in-the-wild 录音或表达性语音场景下的失败模式。

### 本文的动机

如果**只换数据、不动模型架构**，能不能让一个 100% 开源管线的 TTS 追上闭源 SOTA？同时建立一个能区分模型在不同声学条件下鲁棒性的 benchmark。

---

## 方法详解

### 整体管线（三段式：Pool → Core → Model）

Raon-OpenTTS 走 **数据池 → 过滤子集 → 训练 + 评测** 三段：

- **Pool**: 11 个数据源 → 统一格式化 → 615K h 原始数据池
- **Core**: 三指标 15% 分位过滤 → 510K h 高质量训练集
- **Model**: 直接套 [[F5-TTS]] 的 [[DiT]] + [[Flow Matching]]，0.3B / 1B 两档
- **Eval**: 自建 6K 条 4 场景 benchmark + 复用 [[Seed-TTS-eval]] / CV3-Hard

### 模块1: Raon-OpenTTS-Pool 数据池构建

**入池规则**（保证质量下限 + 可复现）:
- 仅取**英文**数据
- 单数据集规模 ≥ 500 小时
- 单段长度 < 30 秒
- 统一存储为 16 kHz、单声道、64 kbps Opus

**11 个数据源**: LibriTTS-R、SPGISpeech、SPGISpeech 2.0、HiFiTTS2、LibriHeavy、GigaSpeech、VoxPopuli、People's Speech、Emilia、Emilia-YODAS、YouTube-Commons。

**Raon-YouTube-Commons 预处理管线**（335K h，占整个 Pool 一半以上）:
1. 音频标准化（16 kHz mono + loudness normalization）
2. [[Source Separation]] / 人声分离 — UVR-MDX
3. [[Speaker Diarization]] — PyAnnote 3.1
4. [[VAD]] / 语音活动检测 — Silero VAD，输出 3–30 s 切片
5. 自动转写 — [[Whisper]]-large-v3

> 这步是论文最大工程贡献：把"原始 YouTube 抓取"打包成可直接喂 TTS 训练的 segments，全过程脚本开源。

### 模块2: Raon-OpenTTS-Core 数据过滤

**三个独立指标**：

- [[DNSMOS]]：感知质量（噪声/失真）
- [[Whisper]] WER：转写难度，反映音文匹配 + 信噪比
- [[Speech Ratio]]（[[VAD]] 输出）：有效语音占比，反映非语音内容比例

**组合过滤策略（Combined）**: 对每段 segment 在三个指标上做 rank，取平均 rank 后再统一砍尾。

**15 分位阈值**（Figure 2）:
- DNSMOS < **2.24**（去掉低音质 15%）
- Speech Ratio < **0.79**（>21% 非语音）
- WER > **0.35**（音文严重不匹配）

经 Table 3 ablation 验证：**Combined 15% 是最优策略**（平均 rank 3.40），优于单指标过滤和更激进的 50% 过滤。

### 模块3: Raon-OpenTTS 模型架构

**完全沿用 [[F5-TTS]]** 的 [[DiT]] + [[Flow Matching]]，作者明说"不修改任何架构以隔离数据效应"：

| 项目 | 0.3B | 1.0B |
|---|---|---|
| Transformer Layers | 22 | 28 |
| Attention Heads | 16 | 22 |
| Attention Embed Dim | 1024 | 1408 |
| Feed-forward Dim | 2048 | 5632 |
| Text Embedding Dim | 512 | 512 |
| Total Parameters | 336M | 1048M |

**音频侧**: 80 通道 log-mel-spectrogram，16 kHz 采样，hop size 256；声码器用 [[HiFi-GAN]]（在 [[LibriTTS]] 上训练）。

**文本侧**: 字符级 tokenizer，词表 5,512。

**推理**: 32 步 NFE 的 ODE 采样。

### 模块4: Raon-OpenTTS-Eval Benchmark 构造

**6,000** 条 prompt-text 对 × 12 个数据集 × 4 个场景：

| 场景 | 段数 | 数据集 |
|---|---|---|
| **Clean** | 2,500 | LibriSpeech-clean、ST American English、CMU-ARCTIC、L2-ARCTIC、VCTK |
| **Noisy** | 1,000 | LibriSpeech-other、TED-LIUM 3 |
| **Wild** | 1,000 | AMI-IHM、AMI-SDM |
| **Expressive** | 1,500 | Expresso、CREMA-D、EmoV-DB |

[[Zero-shot TTS]] 设定：每条 prompt 给一段参考音频 + 一段目标文本。

---

## 关键公式

> 本文为系统论文（data + benchmark），方法侧直接沿用 [[F5-TTS]]，未引入新公式；以下两条是数据过滤管线的核心定义。

### 公式1: [[Speech Ratio]] / 有效语音占比

$$
\mathrm{SR}(x) = \frac{\sum_t \mathbb{1}[\mathrm{VAD}(x_t)=1]}{T}
$$

**含义**: 一段长度为 $T$ 帧的音频 $x$ 中，[[Silero VAD]] 判定为语音的帧占比。论文 SR < 0.79 即认为「超过 21% 是非语音」，过滤掉。

**符号说明**:
- $x_t$: 第 $t$ 帧音频
- $\mathbb{1}[\cdot]$: 指示函数
- $T$: 总帧数

### 公式2: 组合过滤 Rank Score

$$
\mathrm{Score}(s) = \frac{1}{3}\bigl(r_{\mathrm{DNSMOS}}(s) + r_{\mathrm{SR}}(s) + r_{\mathrm{WER}}(s)\bigr)
$$

**含义**: 对每段 segment $s$，把它在三个质量指标上的**排名**（rank）求平均，得到一个总分。按总分排序后砍掉最差的 15% 或 50%。用 rank 而非原值是为了避免不同指标量纲不一致。

**符号说明**:
- $r_{\mathrm{DNSMOS}}(s)$: segment $s$ 在 [[DNSMOS]] 上的全局排名（越低越好）
- $r_{\mathrm{SR}}(s)$: 在 [[Speech Ratio]] 上的排名
- $r_{\mathrm{WER}}(s)$: 在 [[Whisper]] WER 上的排名

### 公式3: 推理 ODE 采样（沿用 [[F5-TTS]]）

$$
x_1 = x_0 + \int_0^1 v_\theta(x_t, t, c)\,dt
$$

**含义**: 从高斯噪声 $x_0$ 出发，沿条件 [[Flow Matching]] 速度场 $v_\theta$ 积分到目标 mel-spectrogram $x_1$；论文用 32 步 NFE 的 ODE solver。

**符号说明**:
- $x_t$: $t$ 时刻的 mel 隐变量
- $v_\theta$: [[DiT]] 参数化的速度场
- $c$: 文本 + 参考音频条件
- NFE: Number of Function Evaluations，推理步数

---

## 关键图表

### Figure 1: Raon-OpenTTS Overview / 整体管线

![Figure 1](https://arxiv.org/html/2605.20830v1/x1.png)

**说明**: 完整流水线示意 — **Raon-OpenTTS-Pool**（335K h 自建 YouTube-Commons + 280K h 10 个公开集 → 615K h）→ 三指标过滤 → **Raon-OpenTTS-Core**（510K h）→ 训练 0.3B / 1B [[DiT]] 模型。右侧 radar chart 展示 Raon-OpenTTS-1B 在 6 个轴（Seed-TTS WER/SIM、CV3-Hard WER/SIM、Raon-Eval CMOS/SMOS）上几乎全面包络 [[CosyVoice 3]]、[[Qwen3-TTS]]、[[VoxCPM]] 等强基线。

### Figure 2: 数据质量分布与过滤阈值

![Figure 2](https://arxiv.org/html/2605.20830v1/x2.png)

**说明**: 三个并排直方图 — DNSMOS、[[Speech Ratio]]、[[Whisper]] WER — 上面叠加 15 分位红色虚线（DNSMOS=**2.24**、SR=**0.79**、WER=**0.35**）。可视化说明三个阈值都落在**长尾**而非分布主峰上，因此过滤掉的是真正的低质数据而非主流样本。论文用这张图为「Combined 15%」的保守过滤策略做合理性辩护。

---

### Table 1: Seed-TTS-Eval 主结果

| Model | Param. | Train Data (h) | Open-Weight | Open-Data | WER↓ | SIM↑ |
|---|---|---|---|---|---|---|
| Human | – | – | – | – | 2.14 | 0.734 |
| [[Seed-TTS]] | – | – | – | – | 2.25 | 0.762 |
| [[CosyVoice 3]] | 1.5B | ~1M | – | – | 2.21 | 0.720 |
| [[IndexTTS 2|Index-TTS 2]] | 1.5B | 55K | ✓ | – | 2.18 | 0.709 |
| Llasa | 8B | 250K | ✓ | – | 3.63 | 0.581 |
| [[VoxCPM]] | 0.5B | 1.8M | ✓ | – | 1.98 | 0.730 |
| [[CosyVoice 2]] | 0.5B | 170K | ✓ | – | 2.61 | 0.659 |
| [[CosyVoice 3]] | 0.5B | ~1M | ✓ | – | 2.50 | 0.698 |
| [[Qwen3-TTS]] | 1.7B | ~5M | ✓ | – | 1.46 | 0.715 |
| Voxtral TTS | 4B | – | ✓ | – | 2.19 | 0.663 |
| [[MaskGCT]] | 0.6B | 100K | ✓ | ✓ | 2.57 | 0.713 |
| [[F5-TTS]] | 0.3B | 100K | ✓ | ✓ | 2.04 | 0.671 |
| **Raon-OpenTTS-0.3B** | 0.3B | 510K | ✓ | ✓ | 1.95 | 0.687 |
| **Raon-OpenTTS-1B** | 1.0B | 510K | ✓ | ✓ | **1.78** | **0.749** |

**说明**: Raon-OpenTTS-1B 在所有 Open-Data 模型中 WER 和 SIM 均第一；与闭源数据的 [[Qwen3-TTS]] 比 WER 略高（1.78 vs 1.46）但 SIM 全场最佳（0.749，超过 [[VoxCPM]] 的 0.730）。

### Table 2: Raon-OpenTTS-Pool 组成

| Dataset | Size(h) | Avg.Dur(s) | Sgmts(M) | License | DNS↑ | WER↓ | SR↑ |
|---|---|---|---|---|---|---|---|
| Raon-YouTube-Commons† | 335K | 8.5 | 141.7 | CC BY 4.0 | 2.74 | 0.30 | 0.90 |
| Emilia-YODAS† | 92K | 9.2 | 36.0 | CC BY-NC 4.0 | 2.82 | 0.19 | 0.90 |
| [[Emilia]]† | 47K | 9.3 | 18.1 | CC BY 4.0 | 3.02 | 0.18 | 0.89 |
| LibriHeavy | 42K | 14.2 | 10.8 | Public Domain | 3.22 | 0.11 | 0.83 |
| [[HiFiTTS2]] | 37K | 10.1 | 13.1 | CC BY 4.0 | 3.20 | 0.11 | 0.84 |
| People's Speech | 28K | 14.2 | 7.0 | CC BY 4.0 | 2.63 | 0.25 | 0.86 |
| [[VoxPopuli]]† | 17K | 27.8 | 2.2 | CC-0 | 2.82 | 0.36 | 0.83 |
| GigaSpeech | 10K | 4.3 | 8.3 | Apache 2.0 | 2.73 | 0.16 | 0.90 |
| SPGISpeech | 5K | 9.2 | 2.0 | Kensho UA | 2.90 | 0.03 | 0.86 |
| SPGISpeech 2.0 | 889 | 14.4 | 0.2 | Kensho UA | 2.72 | 0.08 | 0.90 |
| [[LibriTTS|LibriTTS-R]] | 552 | 5.6 | 0.4 | CC BY 4.0 | 2.96 | 0.06 | 0.91 |
| **Total / Avg.** | **615K** | **9.2** | **239.7** | – | **2.83** | **0.24** | **0.89** |

**说明**: †号标识来自 YouTube 等网络源（噪声偏多）。可以明显看到 LibriHeavy / HiFiTTS2 / LibriTTS-R 这类 audiobook 数据 DNS 最高、WER 最低，而 YouTube 类 DNS 最低、WER 最高 — 论文后续的过滤管线主要打这些 in-the-wild 数据的质量长尾。

### Table 3: 过滤策略 Ablation（核心 ablation）

| Filtering | Seed-TTS WER↓ | Seed-TTS SIM↑ | Seed-TTS DNS↑ | CV3-EN WER↓ | CV3-Hard WER↓ | CV3-Hard SIM↑ | CV3-Hard DNS↑ | Raon-Eval WER↓ | Raon-Eval SIM↑ | Raon-Eval DNS↑ | Rank↓ |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Combined (15%)** | 2.00 | 0.672 | 3.12 | 4.25 | 8.14 | 0.642 | 3.11 | 4.46 | 0.611 | 3.05 | **3.40** |
| DNSMOS (15%) | 1.97 | 0.669 | 3.15 | 4.18 | 7.33 | 0.617 | 3.15 | 4.67 | 0.602 | 3.00 | 3.60 |
| WER (15%) | 1.99 | 0.668 | 3.15 | 4.47 | 8.20 | 0.620 | 3.12 | 3.66 | 0.594 | 3.03 | 4.50 |
| DNSMOS (50%) | 1.98 | 0.671 | 3.14 | 4.90 | 7.20 | 0.622 | 3.13 | 5.34 | 0.588 | 3.02 | 4.70 |
| No filtering | 2.19 | 0.661 | 3.14 | 4.73 | 8.53 | 0.628 | 3.12 | 4.30 | 0.603 | 3.03 | 4.90 |
| Combined (50%) | 2.32 | 0.672 | 3.14 | 5.01 | 7.56 | 0.630 | 3.15 | 4.83 | 0.601 | 3.01 | 4.90 |
| VAD (15%) | 2.22 | 0.665 | 3.13 | 4.81 | 7.86 | 0.621 | 3.11 | 4.24 | 0.604 | 3.04 | 5.20 |
| VAD (50%) | 2.10 | 0.666 | 3.10 | 4.27 | 7.69 | 0.620 | 3.11 | 4.58 | 0.597 | 2.99 | 6.20 |
| WER (50%) | 2.20 | 0.655 | 3.14 | 5.17 | 9.88 | 0.607 | 3.11 | 4.96 | 0.590 | 3.03 | 7.60 |

**说明**: 用 0.3B 变体跑的 ablation。三个结论 —
1. **过滤本身有用**（No filtering 排第 5，Combined 15% 排第 1）；
2. **三指标融合优于单指标**（Combined 15% 优于 DNSMOS/WER/VAD 单一 15%）；
3. **激进过滤反伤性能**（Combined 50% 比 Combined 15% 差很多）。说明 TTS 数据**质量重要但数量更重要**，砍太多会损失多样性。

### Table 4: 各数据集 15% 组合过滤后保留率

| Dataset | Core (M segments) | Retention (%) |
|---|---|---|
| LibriTTS-R | 0.3 | 97.7 |
| HiFiTTS2 | 11.9 | 94.5 |
| LibriHeavy | 10.2 | 94.4 |
| Emilia | 16.8 | 93.2 |
| Emilia-YODAS | 31.6 | 87.8 |
| People's Speech (Clean) | 1.3 | 83.9 |
| Raon-YouTube-Commons | 117.5 | 82.9 |
| VoxPopuli | 1.6 | 71.8 |
| People's Speech (Dirty) | 2.6 | 48.2 |
| **Total** | **193.9** | **84.7** |

**说明**: 高质量 audiobook 类（LibriTTS-R / HiFiTTS2 / LibriHeavy）保留率 94%+，符合直觉；VoxPopuli 因长段慢节奏被砍 28%；People's Speech (Dirty) 直接腰斩。整体保留 84.7%。

### Table 5: CV3-EN / CV3-Hard-EN 结果

| Model | CV3-EN WER↓ | CV3-Hard-EN WER↓ | SIM↑ | DNSMOS↑ |
|---|---|---|---|---|
| [[F5-TTS]] | 8.54 | – | – | – |
| [[MaskGCT]] | 7.73 | 41.09 | 0.624 | 3.48 |
| [[CosyVoice 2]] | 6.27 | 10.28 | 0.710 | 3.95 |
| [[CosyVoice 3]] | 4.96 | 10.77 | 0.740 | **3.98** |
| [[VoxCPM]] | 5.24 | 6.44 | 0.670 | 3.78 |
| [[Qwen3-TTS]] | 4.52 | 7.89 | 0.666 | 3.87 |
| **Raon-OpenTTS-0.3B** | 4.62 | 7.31 | 0.730 | 3.77 |
| **Raon-OpenTTS-1B** | **3.92** | **6.15** | **0.775** | 3.85 |

**说明**: Raon-OpenTTS-1B 在 CV3 系列 WER 和 SIM 双第一。论文特别提到 [[F5-TTS]] 因为**没法处理长输入**，在 CV3-Hard-EN 上大量失败，被排除 — 这是 [[F5-TTS]] 的一个已知工程短板。

### Table 6: Raon-OpenTTS-Eval 主结果（4 场景）

| Model | Clean WER↓ | Clean SIM↑ | Noisy WER↓ | Noisy SIM↑ | Wild WER↓ | Wild SIM↑ | Expr WER↓ | Expr SIM↑ | Overall WER↓ | Overall SIM↑ |
|---|---|---|---|---|---|---|---|---|---|---|
| [[F5-TTS]] | 2.17 | 0.613 | 3.82 | 0.640 | **136.03** | 0.324 | 3.46 | 0.503 | 25.08 | 0.542 |
| [[MaskGCT]] | 3.39 | 0.672 | 5.56 | 0.727 | 28.00 | 0.581 | 6.44 | 0.546 | 8.61 | 0.635 |
| [[CosyVoice 2]] | 2.59 | 0.642 | 4.39 | 0.675 | 49.73 | 0.535 | 3.66 | 0.536 | 11.02 | 0.603 |
| [[CosyVoice 3]] | 2.53 | 0.678 | 3.69 | 0.720 | 8.31 | 0.618 | 5.49 | 0.567 | 4.43 | 0.647 |
| [[VoxCPM]] | 2.24 | 0.686 | 3.42 | 0.738 | 43.83 | 0.553 | 2.66 | 0.565 | 9.48 | 0.642 |
| [[Qwen3-TTS]] | 3.38 | 0.684 | 4.60 | 0.726 | 79.14 | 0.528 | 5.81 | 0.527 | 17.59 | 0.626 |
| **Raon-OpenTTS-0.3B** | 1.57 | 0.645 | 4.03 | 0.700 | 5.83 | 0.571 | 2.53 | 0.570 | 2.93 | 0.623 |
| **Raon-OpenTTS-1B** | **1.44** | **0.718** | 3.51 | **0.769** | **5.61** | **0.656** | 2.77 | **0.633** | **2.81** | **0.695** |

**说明**: 关键看 **Wild 列** — [[F5-TTS]] 直接爆炸到 WER 136%（hallucination 严重）、[[Qwen3-TTS]] 79%、[[VoxCPM]] 44%，而 Raon-OpenTTS-1B 只有 5.61%。说明用 YouTube 等 in-the-wild 数据训练，对 AMI 这类多人远场录音的鲁棒性提升巨大。Overall WER 2.81 是全场唯一低于 3 的模型。

### Table 7: CMOS 自然度主观评测（人评，Raon-1B 为锚）

| Model | Clean | Noisy | Wild | Expressive | Overall |
|---|---|---|---|---|---|
| [[F5-TTS]] | −0.82 | −0.48 | −0.48 | −0.95 | −0.68 |
| [[CosyVoice 2]] | −0.59 | −0.35 | −0.38 | −0.15 | −0.36 |
| [[CosyVoice 3]] | −0.06 | −0.45 | −0.12 | +0.10 | −0.13 |
| [[MaskGCT]] | +0.15 | +0.31 | −0.49 | +0.06 | −0.01 |
| [[Qwen3-TTS]] | +0.08 | −0.46 | −0.38 | +0.25 | −0.13 |
| [[VoxCPM]] | +0.14 | +0.10 | −0.06 | −0.30 | −0.05 |
| Raon-OpenTTS-0.3B | +0.54 | −0.24 | +0.16 | −0.42 | −0.01 |
| **Raon-OpenTTS-1B** | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |

**说明**: 以 Raon-OpenTTS-1B 为锚点（0.00），其余模型为相对偏好（负数代表听众更偏好 1B）。Overall 全部负数，1B 主观自然度全场第一；但在 Expressive 上 [[Qwen3-TTS]] +0.25、[[CosyVoice 3]] +0.10 反超，说明表达性合成仍是该模型的短板。

### Table 8: SMOS 说话人相似度主观评测

| Model | Clean | Noisy | Wild | Expressive | Overall |
|---|---|---|---|---|---|
| [[F5-TTS]] | 3.86 | 3.37 | 3.50 | 3.50 | 3.55 |
| [[CosyVoice 2]] | 3.90 | 3.35 | 3.43 | 3.49 | 3.53 |
| [[CosyVoice 3]] | 3.87 | 3.63 | 3.46 | 3.40 | 3.58 |
| [[MaskGCT]] | 3.85 | 3.30 | 3.77 | 3.43 | 3.58 |
| [[Qwen3-TTS]] | 3.89 | 3.47 | 3.55 | 3.48 | 3.59 |
| [[VoxCPM]] | 3.98 | 3.32 | 3.50 | 3.41 | 3.54 |
| Raon-OpenTTS-0.3B | 4.01 | 3.55 | 3.39 | 3.54 | 3.60 |
| **Raon-OpenTTS-1B** | 3.90 | **3.58** | **3.70** | **3.64** | **3.70** |

**说明**: Raon-OpenTTS-1B 主观说话人相似度 Overall 3.70，超过 [[Qwen3-TTS]] 3.59 / [[MaskGCT]] 3.58。但 Clean 一列 [[VoxCPM]] 3.98 / Raon-0.3B 4.01 反超 1B，说明 1B 主要赢在更难的场景而非干净场景。

### Table 9: 数据来源 Ablation — Emilia vs Pool-Matched-47K

| Data | Clean WER↓ | Clean SIM↑ | Noisy WER↓ | Noisy SIM↑ | Wild WER↓ | Wild SIM↑ | Expr WER↓ | Expr SIM↑ | Overall WER↓ | Overall SIM↑ |
|---|---|---|---|---|---|---|---|---|---|---|
| [[Emilia]] | **1.28** | 0.593 | 4.02 | 0.641 | 7.35 | 0.470 | 3.60 | 0.505 | 3.33 | 0.558 |
| Pool-Matched-47K | 1.53 | **0.631** | **3.90** | **0.687** | **5.97** | **0.549** | **3.22** | **0.546** | **3.09** | **0.605** |

**说明**: 在同等 47K 小时规模下，Raon Pool 的混合采样比纯 [[Emilia]] 几乎全面更好（除 Clean WER 略输 0.25 点）。**这是论文最重要的"数据多样性 vs 单一来源"对照**：证明 Pool 的胜利不只是因为数据多，而是数据**混合**本身就有价值。

### Table 10: YouTube-Commons 数据消融

| Dataset | Clean WER↓ | Clean SIM↑ | Noisy WER↓ | Noisy SIM↑ | Wild WER↓ | Wild SIM↑ | Expr WER↓ | Expr SIM↑ | Overall WER↓ | Overall SIM↑ |
|---|---|---|---|---|---|---|---|---|---|---|
| w/o YC | 2.17 | 0.615 | **4.21** | 0.668 | 7.62 | 0.523 | 3.43 | 0.540 | 3.73 | 0.590 |
| all data | **1.72** | **0.634** | 6.79 | **0.688** | **6.15** | **0.550** | **2.63** | **0.560** | **3.53** | **0.610** |

**说明**: 加入 [[Raon-YouTube-Commons]] 后，Clean / Wild / Expressive 全面提升，但 **Noisy WER 反而从 4.21 恶化到 6.79**。论文承认这个 trade-off — YouTube 数据自身噪声会让模型在已经噪声较低的 LibriSpeech-other / TED-LIUM 3 上反而过度"补偿"。这是一个值得后续研究的方向。

### Table 11: 模型配置（Appendix A）

| Config | 0.3B | 1.0B |
|---|---|---|
| Transformer Layers | 22 | 28 |
| Attention Heads | 16 | 22 |
| Attention Embed Dim | 1024 | 1408 |
| Feed-forward Dim | 2048 | 5632 |
| Text Embedding Dim | 512 | 512 |
| Total Parameters | 336M | 1048M |

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| Raon-OpenTTS-Pool | 615K h / 240M seg | 11 源 + YouTube-Commons | 原始池 |
| Raon-OpenTTS-Core | 510K h / 194M seg | Combined 15% 过滤 | 训练 |
| Raon-OpenTTS-Eval | 6K prompt-text | 4 场景 12 数据集 | 主评测 |
| [[Seed-TTS-eval]] | – | 干净英文 zero-shot | 对外可比性 |
| CV3-EN / CV3-Hard-EN | – | CosyVoice 3 评测集 | 难场景对外可比 |

### 实现细节

- **Backbone**: [[F5-TTS]] 同款 [[DiT]]（控制变量）
- **声码器**: [[HiFi-GAN]]（[[LibriTTS]] 16 kHz）
- **优化器**: AdamW + gradient clipping 1.0
- **0.3B 训练**: 225K steps、batch 35K frames/GPU、LR 7.5e-5、50K warmup、~1K GPU-hours
- **1B 训练**: 550K steps、batch 14K frames/GPU、LR 1e-4、50K warmup、~9K GPU-hours
- **硬件**: NVIDIA B200 GPU
- **推理**: 32 NFE ODE 采样
- **音频**: 16 kHz、80-channel log mel、hop 256；存储 64 kbps Opus
- **文本**: 字符级，词表 5,512

### 人评细节（Appendix B）

- Amazon MTurk（US workers）
- 每条件 30 个 item × 6 个 annotator
- SMOS 5 点制（Bad→Excellent）
- CMOS 7 点制（−3 到 +3）
- 质量控制：剔除全程给相同分数的 annotator

---

## 批判性思考

### 优点

1. **数据 + 管线 + 模型 + benchmark 一条龙开源**：在 TTS 圈这是第一次有人把 615K h 训练数据 + 过滤管线 + 训练代码 + 1B checkpoint + 评测集**同时**放出来；可复现性远超 [[F5-TTS]] / [[MaskGCT]]。
2. **过滤策略有定量证据**：Table 3 系统比较了 9 种过滤方案，明确指出「Combined 15% > 单指标 > 50% 激进过滤」，对后人有直接指导价值。
3. **Wild 场景结果是真正的硬指标**：Table 6 中 Raon-1B 在 AMI 上的 WER 5.61 vs [[F5-TTS]] 的 136% 是**两个数量级**的差距，证明 in-the-wild 数据训练对鲁棒性是质变。
4. **架构控制变量做得干净**：完全沿用 [[F5-TTS]] DiT 而不引入新模块，让"数据效应"这个研究问题的回答相对可信。

### 局限性

1. **仅英文**：作者也承认（Future Work 列在首位），多语种版本是显然的下一步。对中文用户而言短期没有直接可用价值。
2. **架构上零创新**：本质是「[[F5-TTS]] + 更大更好的数据」。如果未来 SOTA 架构换代（[[Flow Matching]] → 新范式），这套数据效应结论可能不再 transfer。
3. **Table 10 的 Noisy 负向作用没解决**：加入 YouTube-Commons 后 Noisy WER 从 4.21 涨到 6.79（+61%），论文承认但没给出修复方案。
4. **缺少 long-form / 流式评测**：CV3-Hard 长输入和流式 TTS 这两个真实使用场景的指标缺位。[[F5-TTS]] 长输入失败的问题作者只是排除掉了，没说自己 1B 能否处理。
5. **CMOS 锚点是自己**：Table 7 把 Raon-OpenTTS-1B 设为 CMOS=0，其余模型为相对值，这种锚点选择天然让自己看起来全是 0 而别人都是负数。如果换 [[CosyVoice 3]] 为锚点结果应该会不一样。
6. **过滤指标本身的偏见**：用 [[DNSMOS]] 过滤会偏好 studio 录音、[[Whisper]] WER 过滤会偏好 Whisper 容易识别的口音 — 这相当于把 Whisper 的偏差注入了训练数据。会不会让模型在 Whisper-bad 的口音/方言上更差？没分析。

### 潜在改进方向

1. **多语种 Pool**：用同样管线处理多语种 YouTube-Commons / VoxPopuli。
2. **过滤替换为修复**：作者自己也提到，比起直接删掉低质段，对其做 denoising / re-transcription 应该能保留更多数据。
3. **来源级别的混合采样策略**：Table 2 中各来源 DNSMOS 跨度从 2.63（People's Speech）到 3.22（LibriHeavy），简单按 segment 数比例采样可能不是最优。
4. **过滤指标的多元化**：用 [[UTMOS]] 替代或补充 [[DNSMOS]]；用多个 ASR 系统的 ensemble 而非单一 Whisper-small 做 WER。
5. **流式版本 + long-form 测试集**：补齐 [[F5-TTS]] 那个长输入硬伤的工程评测。

### 可复现性评估

- [x] 代码开源（GitHub: krafton-ai/RAON-OpenTTS）
- [x] 预训练模型（0.3B + 1B checkpoint）
- [x] 训练细节完整（步数、batch、LR、warmup、硬件都写了）
- [x] 数据集可获取（11 个源 + 自建 YouTube-Commons 管线全开源；CC BY 4.0 主体）

> 这是少见的**四项全 ✓** 的 TTS 论文。

---

## 关联笔记

### 基于

- [[F5-TTS]]: 直接复用其 [[DiT]] + [[Flow Matching]] 架构，唯一的差异就是训练数据
- [[Emilia]]: 之前 100K h 量级的开源 TTS 数据标杆，被 Raon-OpenTTS-Pool 大幅超越（615K h）

### 对比

- [[CosyVoice 3]]: 闭源数据 SOTA，Raon-1B 在 Seed-TTS WER 略输（1.78 vs 1.46 if measured against Qwen3-TTS, 但 vs CosyVoice3 是 1.78 vs 2.21 反超）
- [[Qwen3-TTS]]: 5M 小时私有数据训练的 1.7B 模型；Raon-1B 在 SIM 上反超
- [[VoxCPM]]: 1.8M 小时数据 0.5B 模型；Raon-1B 仅靠 510K 公开数据全面追上
- [[MaskGCT]]: 同为 Open-Data 派别的代表，Raon 在数据规模和评测指标全面碾压

### 方法相关

- [[DiT]] / [[Diffusion Transformer]]: 主干网络
- [[Flow Matching]]: 训练目标
- [[HiFi-GAN]]: 声码器
- [[Whisper]]: 数据标注 + 过滤 + 评测 WER 的核心工具

### 数据集 / Benchmark

- [[Raon-OpenTTS-Pool]]: 本文 615K h 数据池
- [[Raon-OpenTTS-Core]]: 510K h 过滤训练子集
- [[Raon-OpenTTS-Eval]]: 6K prompts、4 场景的鲁棒性 benchmark
- [[Raon-YouTube-Commons]]: 自建 335K h YouTube 预处理子集
- [[Seed-TTS-eval]]、[[LibriTTS]]、[[VoxPopuli]]、[[HiFiTTS2]]、LibriHeavy、People's Speech、SPGISpeech、Emilia-YODAS

### 评测指标

- [[DNSMOS]]: 数据过滤 + 评测
- [[Speech Ratio]]: 数据过滤
- [[MOS]] / [[CMOS]] / [[SMOS]]: 主观评测
- WER / SIM：客观评测

---

## 速查卡片

> [!summary] Raon-OpenTTS (2026)
> - **核心**: 全开源 615K h 数据池 + 510K h 过滤子集 + [[F5-TTS]] 同款 [[DiT]] 架构 = 追平闭源数据 SOTA
> - **方法**: 三指标（[[DNSMOS]] / Whisper-WER / [[Speech Ratio]]）Combined 15% 过滤 + [[Flow Matching]] DiT (0.3B / 1B)
> - **结果**: Seed-TTS WER 1.78 / SIM 0.749，Raon-Eval Overall WER 2.81（唯一 <3 的模型），Wild 场景碾压（5.61 vs F5-TTS 136%）
> - **代码**: https://github.com/krafton-ai/RAON-OpenTTS
> - **可复现**: ★★★★★（数据/过滤/训练/checkpoint 全开放）
> - **痛点**: 仅英文、加 YouTube 数据反伤 Noisy WER（4.21→6.79）、未测长输入和流式

---

*笔记创建时间: 2026-05-21*
