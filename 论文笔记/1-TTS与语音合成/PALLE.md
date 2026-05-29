---
title: "Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot Text-to-Speech Synthesis"
method_name: "PALLE"
authors: [Yifan Yang, Shujie Liu, Jinyu Li, Yuxuan Hu, Haibin Wu, Hui Wang, Jianwei Yu, Lingwei Meng, Haiyang Sun, Yanqing Liu, Yan Lu, Kai Yu, Xie Chen]
year: 2025
venue: ACM MM 2025
arxiv_id: "2504.10352"
tags: [tts, codec-lm-tts, zero-shot-tts, pseudo-autoregressive, masked-generative]
zotero_collection:

# === 论文核心技术元数据（三层 verify，每条标 [§X] / [GitHub] 来源）===
lm_init: "from scratch（从头训练 177.0M decoder-only transformer，未用通用 LLM warm-start）[已 verify §5.1 Model Configurations]"
training_loss: "纯 masked-token 交叉熵（MLM），无文本 loss、无 KL；两阶段各自的 MLM 目标 [已 verify §3 Eq.2 / §4.2 Eq.9 / §4.3 Eq.10]"
tokenizer_arch: "text+speech feature-dimension fusion（E2 TTS 式）：text 用 [PAD] 补齐、speech 用 [MASK] 补齐到同长，各自 embedding 后沿特征维拼接 [已 verify §3 Discussion / §4.2 Eq.6]"
multitask: false "[已 verify §5.5：联合多任务训练失败，cross-sentence WER 升 ~20%、capacity 折半，故采用两个独立 stage]"
training_data: "LibriTTS ~580h、2306 说话人、纯英文 [已 verify §5.1 Training Dataset]"
post_training: "无（无 RLHF/DPO）[已 verify 全文未提后训练]"
codec_detail: "S3Tokenizer v2（CosyVoice 2）25 Hz 语义 token（非 RVQ 声学 token）；detokenizer = CosyVoice 2 CFM + HiFi-GAN [已 verify §4.1 / §5.1]"

# === 知识地图联动 ===
domain: TTS
routes: [codec-lm-tts, "其他: pseudo-autoregressive (PAR)"]
problems: [zero-shot-cloning, latency, speaker-similarity]
representations: [semantic-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-表示层地图]]"
related_surveys:
  - "[[TTS-11篇综述综合-2026-05]]"
evidence_level: medium
maturity: exploratory
last_repositioned: 2026-05-29

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2504.10352/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2504.10352/paper.html"
figures_dir: "_resources/2504.10352/figures/"
github_local: ""
cached_at: 2026-05-29

# === 通用元数据 ===
image_source: local
arxiv_html: https://arxiv.org/html/2504.10352v3
created: 2026-05-29
---

# 论文笔记：Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot TTS

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Shanghai Jiao Tong University（X-LANCE）+ Microsoft |
| 日期 | April 2025（arXiv v1）；ACM MM 2025 接收 |
| 项目主页 | https://microsoft.com/research/project/vall-e-x/palle |
| 对比基线 | [[VALL-E]] / [[CosyVoice 2]] / [[MaskGCT]] / [[E2 TTS]] / [[F5-TTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2504.10352) / Code（未见官方开源 repo） |

---

## 一句话总结

> 提出"伪自回归"(PAR) 建模——在固定时间步上并行生成**动态长度**的 span，融合 AR 的时序归纳偏置与 NAR 的并行效率；据作者称两阶段 PALLE 在 LibriTTS 上以 RTF 0.06 取得零样本 TTS 的领先 WER/SIM。

---

## 核心贡献

1. **Pseudo-Autoregressive (PAR) 建模范式**：在 [[Masked Generative Transformer]] 上，让每个时间步并行预测所有位置但只保留**最左侧动态长度 span**，从而既有 [[Autoregressive Model|AR]] 的从左到右时序结构，又有 [[Non-Autoregressive Model|NAR]] 的并行推理；span 长度可变（不像 AR 每步 1 token，也不像 NAR 一次全出）[已 verify §3]。
2. **两阶段 PALLE 系统**：Stage 1 用 PAR 快速生成完整草稿；Stage 2 用 [[Confidence-based Iterative Decoding|置信度引导的迭代精修]]（NAR）重掩码并重预测低置信 token，提升质量 [已 verify §4.2/§4.3]。
3. **效率收益**：在受控同骨干对比里，PAR 相对 AR 约 4× 加速、相对 NAR 约 2× 加速，且达到更低 WER；两阶段 PALLE 在 LibriTTS 上 RTF 0.06 [已 verify §5.4 / Tab.3]。

---

## 问题背景

### 要解决的问题
[[Zero-shot TTS|零样本 TTS]]（给一段参考音频即复刻音色）中，离散 [[codec-lm-tts|codec token + 语言模型]]路线存在效率/质量两难：[[Autoregressive Model|AR]] 逐 token 生成质量好但慢；[[Non-Autoregressive Model|NAR]]（如 [[MaskGCT]]）并行快但丢失时序归纳偏置、易出现高熵区的快速韵律跳变。

### 现有方法的局限
- **AR**（[[VALL-E]] 系）：固定时间步 + 固定长度（每步 1 token），延迟随序列线性增长 [已 verify §2/§3]。
- **NAR / masked generative**（[[MaskGCT]]）：通过迭代式无序生成提升效率，但依赖置信度调度、在高熵区可能产生快速过渡的韵律 [已 verify §2，作者归纳]。
- 作者把两者抽象为"时间步 × 每步长度"两个轴：AR=固定步/固定长，NAR=单步/全长，二者都未利用"固定步 + 动态长"这一中间地带 [已 verify §3 + Fig.1]。

### 本文的动机
用"固定时间步 + 动态长度 span"统一 AR 与 NAR：保留从左到右的时序顺序（软时序归纳偏置），同时允许每步并行吐出可变数量 token，逼近 O(1) 步数级别的推理延迟 [已 verify §3]。

---

## 方法详解

### 领域定位

<!-- R4 -->
PALLE 属于**离散 [[codec-lm-tts|codec-LM]] 零样本 TTS** 路线，与 [[VALL-E]]（AR）、[[MaskGCT]]（NAR masked generative）、[[E2 TTS]]/[[F5-TTS]]（连续 flow-matching）同处零样本 TTS 竞争位。其核心 novelty 不在表示或骨干（骨干就是 MaskGIT 式双向 masked transformer、表示用 [[S3Tokenizer]] 语义 token），而在**生成调度范式**：把 AR 的"固定步固定长"与 NAR 的"单步全长"统一为"固定步 + 动态长 span"的 PAR [已 verify §3]。

### 模型架构

PALLE 两个 stage 共享同一骨干 [已 verify §4.1/§5.1]：
- **输入**：参考语音 → [[S3Tokenizer]] 得语义 token $\mathbf{c}^{ref}$；目标文本 → 2000 类 BPE token，经一层 [[ConvNeXt V2]] block 编码。
- **模态融合**：[[E2 TTS]] 式**特征维融合**——文本序列用 `[PAD]` 补齐、语音序列用 `[MASK]` 补齐到同长 $T$，各自 embedding 后**沿特征维拼接**（非时间维 concat）[已 verify §3 Discussion / §4.2 Eq.6]。
- **Backbone**：decoder-only Transformer，12 层 / 16 头 / 模型维 1024 / FFN 4096；卷积位置编码 kernel=7；每 stage 177.0M 参数 [已 verify §5.1]。
- **输出**：语义 token $\mathbf{c}^{gen}$ → [[CosyVoice 2]] 的 [[Conditional Flow Matching|CFM]] + [[HiFi-GAN]] detokenizer/vocoder 合成波形 [已 verify §4.1]。

### 核心模块

#### 模块1: PAR 语言建模（通用范式）

**设计动机**：用"固定时间步 + 动态长度 span"取代 AR 的"每步 1 token"和 NAR 的"单步全长"，在保留从左到右时序顺序的同时并行解码 [已 verify §3]。

**具体实现**：
- 训练时随机采样掩码 span（Eq.1），模型只被监督预测掩码段**最左侧** $k=\lfloor rT\rfloor$ 个 token（Eq.2），$r$ 为固定比例 [已 verify §3]。
- 推理时每步并行预测所有位置，但只**采纳最左侧** $k'=\min(\lfloor r'T^{gen}\rfloor, N_{left})$ 个 token，其余丢弃；逐步向右推进直到生成完（Eq.3–5）[已 verify §3]。
- **PAR 是范式层面的定义**，不绑定模态融合方式：既可像 [[MaskGCT]] 用时间维拼接，也可像 [[E2 TTS]] 用特征维融合 [已 verify §3 Discussion]。

#### 模块2: PALLE Stage 1（PAR 生成）

**具体实现**：把 $\mathbf{c}^{ref}$ 用 `[MASK]` 补齐到目标长度 $T^{gen}$（Eq.12），文本拼接补 `[PAD]`（Eq.13）；按 Eq.14（即 Eq.5）迭代更新，每步保留最左侧 span，论文实测 100 步即收敛 [已 verify §4.3 / §5.3]。训练时保留前 30% 语音作 prompt，监督最左 $k=\lfloor 0.1T\rfloor$ span（Eq.7–9）[已 verify §4.2]。

#### 模块3: PALLE Stage 2（置信度引导 NAR 精修）

**设计动机**：Stage 1 草稿仍有局部错误，用 NAR 迭代精修提质 [已 verify §4.3]。

**具体实现**：以 Stage 1 输出为初始 $\mathbf{c}^{ext}_{(0)}$，每步计算置信度矩阵 $\mathbf{C}_{(t)}=\log P^{max}_{(t)}$（负 min-entropy，Eq.15–16），对置信度最低的 $\gamma$ 分位 token 重掩码并重预测；已更新 token 置信度永久置 1 防重复精修；论文用约 7 步 [已 verify §4.3 / §5.3]。训练时对 30% 之后的 token 以 $p=0.1$ 独立掩码（Eq.10）[已 verify §4.2]。

---

## 关键公式

### 公式1: [[Pseudo-Autoregressive|PAR 训练掩码]]

$$
\mathbf{m} = [\underbrace{0,0,\ldots,0}_{T-\ell}, \underbrace{1,1,\ldots,1}_{\ell}]
$$

**含义**：构造一个把序列尾部连续 $\ell$ 个位置置为掩码的二值掩码（$\ell$ 为掩码 span 长度）。

**符号说明**：
- $T$: 下采样后语音 token 序列长度
- $\ell$: 随机采样的掩码 span 长度

### 公式2: [[Pseudo-Autoregressive|PAR 训练目标]]

$$
\arg\max_{\theta}\ p\big([\mathbf{m}\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m})\odot\mathbf{c},\ \mathbf{x};\ \theta\big)
$$

**含义**：只监督掩码段**最左侧** $k=\lfloor rT\rfloor$ 个 token，条件是未掩码部分与文本 $\mathbf{x}$。这是 PAR "固定步 + 动态长"的训练核心。

**符号说明**：
- $k=\lfloor rT\rfloor$: 监督的最左 span 长度，$r\in(0,1)$ 固定比例
- $\mathbf{c}$: 语音 token；$\odot$: 逐元素乘

### 公式3: PAR 推理——扩展掩码序列

$$
\mathbf{c}^{ext}_{(1)} = [c^{ref}_1, c^{ref}_2, \ldots, c^{ref}_{T^{ref}}, \underbrace{0,0,\ldots,0}_{T^{gen}}]
$$

**含义**：参考语音 token 后接 $T^{gen}$ 个占位符，作为待生成位置。

### 公式4: PAR 推理掩码

$$
\mathbf{m}_{(t)} = [\underbrace{0,0,\ldots,0}_{T^{ref}+t\times k'}, \underbrace{1,1,\ldots,1}_{k'}, 0,0,\ldots,0]
$$

**含义**：第 $t$ 步只对从左数第 $t$ 段、长度 $k'$ 的 span 解码。

**符号说明**：
- $k'=\min(\lfloor r'T^{gen}\rfloor, N_{left})$: 每步保留的最左 span 长度，$N_{left}$ 为剩余待生成 token 数

### 公式5: PAR 推理迭代更新

$$
\mathbf{c}^{ext}_{(t+1)} = (1-\mathbf{m}_{(t)})\odot \mathbf{c}^{ext}_{(t)} + \mathbf{m}_{(t)}\odot \arg\max_{\mathbf{c}^{ext}_{(t+1)}} p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ref}, \mathbf{x}^{gen};\theta\big)
$$

**含义**：保留已确定位置、只更新当前 span，逐步向右推进。

### 公式6: [[E2 TTS|特征维融合]]文本补齐

$$
\mathbf{x}^{ext} = [x_0, x_1, \ldots, x_{L-1}, \underbrace{[\text{PAD}],[\text{PAD}],\ldots,[\text{PAD}]}_{T-L}]
$$

**含义**：文本 token 用 `[PAD]` 补齐到与语音 token 同长 $T$，以便沿特征维与语音 embedding 拼接。

**符号说明**：
- $L$: 文本 token 数；$T-L$: 填充的 `[PAD]` 数

### 公式7: Stage 1 训练起点采样

$$
s \sim \mathcal{U}\{\lfloor 0.3T\rfloor,\ \lfloor 0.3T\rfloor+1,\ \ldots,\ T-\lfloor 0.1T\rfloor-1\}
$$

**含义**：随机采样掩码起始位置 $s$，保证前 30% 语音作 prompt、且至少最后 10% 可用于训练监督。

### 公式8: Stage 1 训练掩码

$$
\mathbf{m}' = [\underbrace{0,0,\ldots,0}_{s}, \underbrace{1,1,\ldots,1}_{T-s}]
$$

**含义**：从起点 $s$ 起把后续全部置为掩码（再据此监督最左 $k=\lfloor 0.1T\rfloor$ span）。

### 公式9: Stage 1 训练目标

$$
\arg\max_{\theta}\ p\big([\mathbf{m}'\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m}')\odot\mathbf{c},\ \mathbf{x}^{ext};\theta\big)
$$

**含义**：在 PAR 框架下，监督最左 $k=\lfloor 0.1T\rfloor$ span，条件为未掩码语音与补齐文本。

### 公式10: Stage 2 训练目标（NAR 精修）

$$
\arg\max_{\theta}\ p\big(\mathbf{m}''\odot\mathbf{c}\mid (1-\mathbf{m}'')\odot\mathbf{c},\ \mathbf{x}^{ext};\theta\big)
$$

**含义**：对前 30% 之后的 token 以 $p=0.1$ 独立随机掩码（掩码 $\mathbf{m}''\in\{0\}^{\lfloor 0.3T\rfloor}\oplus\{0,1\}^{T-\lfloor 0.3T\rfloor}$），监督还原被掩码 token——这是标准 NAR masked prediction。

### 公式11: [[Duration Predictor|时长线性估计]]

$$
T^{gen} = T^{ref}\times\Big(1 + \frac{L^{gen}}{L^{ref}}\Big)
$$

**含义**：按参考语音/文本长度比线性外推目标语音 token 数（也可预先指定）。

**符号说明**：
- $T^{ref}, L^{ref}$: 参考语音 token 数 / 参考文本 token 数
- $L^{gen}$: 目标文本 token 数

### 公式12: Stage 1 推理——语音 prompt 补齐

$$
\mathbf{c}^{ext} = [c^r_0, \ldots, c^r_{T^{ref}-1}, \underbrace{[\text{MASK}], \ldots, [\text{MASK}]}_{T^{gen}-T^{ref}}]
$$

**含义**：参考语音 token 后接 `[MASK]` 占位补到 $T^{gen}$。

### 公式13: Stage 1 推理——文本拼接补齐

$$
\mathbf{x}^{ext} = [x^r_0, \ldots, x^r_{L^{ref}-1}, x^g_0, \ldots, x^g_{L^{gen}-1}, \underbrace{[\text{PAD}], \ldots, [\text{PAD}]}_{T^{gen}-(L^{ref}+L^{gen})}]
$$

**含义**：参考文本 + 目标文本拼接后用 `[PAD]` 补到 $T^{gen}$。

### 公式14: Stage 1 推理迭代（同 Eq.5）

$$
\mathbf{c}^{ext}_{(t+1)} = (1-\mathbf{m}_{(t)})\odot \mathbf{c}^{ext}_{(t)} + \mathbf{m}_{(t)}\odot \arg\max_{\mathbf{c}^{ext}_{(t+1)}} p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ext};\theta\big)
$$

**含义**：推理阶段的 PAR 迭代更新规则（条件改为 $\mathbf{x}^{ext}$）。

### 公式15: Stage 2 预测概率矩阵

$$
\mathbf{P}_{(t)} = p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ext};\theta\big) \in \mathbb{R}^{T^{gen}\times N}
$$

**含义**：每个位置在 $N$ 个语音 token 类上的预测分布。

### 公式16: [[Confidence-based Iterative Decoding|置信度]]矩阵

$$
\mathbf{C}_{(t)} = \log P^{max}_{(t)}
$$

**含义**：置信度定义为预测分布最大概率的对数（即负 min-entropy）；对最低 $\gamma$ 分位的 token 重掩码重预测，已更新 token 置信度永久置 1 防重复精修。

---

## 关键图表

### Figure 1: AR / NAR / PAR 生成过程对比

![[_resources/2504.10352/figures/fig1.png]]

**说明**：三种范式在"时间步 × 每步长度"上的差异。[[Autoregressive Model|AR]]（左）固定步、每步 1 token，逐位生成直至 `[EOS]`；[[Non-Autoregressive Model|NAR]]（中）单步把全部 `[MASK]` 并行预测、多轮无序补全；[[Pseudo-Autoregressive|PAR]]（右）固定步但每步并行吐出**动态长度** span（绿色=已采纳 retained token，浅绿=预测但未采纳 predicted token）[已 verify Fig.1]。

### Figure 2: PALLE 总览

![[_resources/2504.10352/figures/fig2.png]]

**说明**：四大组件——文本 tokenizer（→ 子词 token）、语音 tokenizer（[[S3Tokenizer]]，→ 离散语音 token）、两个共享架构的 masked generative transformer（PALLE Stage One 由文本生成语音 token，Stage Two 精修），最后语音 detokenizer 还原波形 [已 verify Fig.2]。

### Figure 3: 两阶段框架细节

![[_resources/2504.10352/figures/fig3.png]]

**说明**：(a) 单 stage 内部结构——`[Embedding]` + [[ConvNeXt V2]] block + 线性 + 卷积位置编码，送入 Bidirectional Transformer；(b) Stage One（PAR）：文本 `[PAD]` 补齐 + 语音 `[MASK]`，特征维融合后只输出最左 span $[\mathbf{m}'\odot\mathbf{c}]_{0:k}$；(c) Stage Two（NAR 精修）：对 $\mathbf{m}''\odot\mathbf{c}$ 重预测 [已 verify Fig.3]。

### Figure 4: 推理步数对客观指标的影响

![[_resources/2504.10352/figures/fig4.png]]

**说明**：cross-sentence 任务下，(a) Stage 1 步数 vs WER-H、(b) Stage 1 步数 vs SIM-o、(c) Stage 2 步数 vs WER-H、(d) Stage 2 步数 vs SIM-o。Stage 1 约 100 步后 WER-H 平台化；Stage 2 在约 7 步内把 SIM-o 由 ~0.712 提到 ~0.717 [已 verify Fig.4 / §5.3]。

### Figure 5: 总时长倍率对客观指标的影响

![[_resources/2504.10352/figures/fig5.png]]

**说明**：横轴为总时长倍率（0.7–1.3×）。WER-H 在 1.1× 最低、SIM-o 在 1.2× 最高，0.9–1.3× 区间整体稳健，说明 Eq.11 的线性时长估计不需要精确即可工作 [已 verify Fig.5 / §5.5]。

### Table 1: 客观性能对比（continuation + cross-sentence）

| System | Speech Tokenizer | Train Data | Cont WER-W | Cont WER-H | Cont SIM-o | Cross WER-W | Cross WER-H | Cross SIM-o | RTF |
|---|---|---|---|---|---|---|---|---|---|
| Ground Truth | - | - | 2.14 | 2.15 | 0.905 | 2.14 | 2.15 | 0.779 | - |
| GT (EnCodec) | - | - | - | 2.33 | 0.823 | - | 2.33 | 0.715 | - |
| GT (S3Tokenizer v2 25Hz) | - | - | 2.69 | 3.16 | 0.793 | 2.94 | 3.51 | 0.743 | - |
| VALL-E [65] | EnCodec | Librilight | - | 3.80 | 0.773 | - | 5.90 | 0.633 | 0.73 |
| VALL-E [10] | EnCodec | Libriheavy | - | 3.00 | 0.770 | - | 3.80 | 0.630 | 0.73 |
| [[E2 TTS]] (32 NFE)* | - | Emilia (EN+ZH) | - | - | - | 2.55 | 2.92 | **0.756** | 0.68 |
| [[F5-TTS]] (32 NFE)* | - | Emilia (EN+ZH) | - | - | - | 2.25 | **2.77** | 0.705 | 0.15 |
| [[MaskGCT]]* | Semantic+Acoustic Codec | Emilia (EN+ZH) | - | - | - | 3.69 | 4.22 | 0.756 | 0.65 |
| VALL-E | EnCodec | LibriTTS | 3.78 | 4.47 | 0.730 | 7.36 | 8.64 | 0.531 | 0.73 |
| [[CosyVoice]] [60] | S3Tokenizer v1 25Hz | LibriTTS | - | - | - | 3.47 | - | - | 0.45 |
| [[CosyVoice 2]] [60] | S3Tokenizer v2 25Hz | LibriTTS | - | - | - | 3.00 | - | - | 0.45 |
| **PALLE (two-stage)** | S3Tokenizer v2 25Hz | LibriTTS | - | - | - | **2.23** | 2.83 | 0.716 | **0.06** |
| **PALLE (two-stage, GT dur)** | S3Tokenizer v2 25Hz | LibriTTS | **2.31** | **2.62** | **0.776** | 2.35 | 2.87 | 0.716 | **0.06** |

**说明**：`*` 表示用开源 checkpoint 复测；GT dur = 用真实总时长。PALLE 在 small-scale（LibriTTS）训练下，cross-sentence WER-W 2.23 优于同数据集的 [[CosyVoice 2]] 3.00（−26%）且 RTF 0.06 vs 0.45（约 7.5×）；与大规模训练的 [[F5-TTS]]（WER-W 2.25, RTF 0.15）相比 RTF 更低 2.5× 且 SIM-o 0.716 > 0.705 [已 verify Tab.1]。**注意可比性**：F5-TTS/E2 TTS/MaskGCT 在 Emilia 大规模数据训练，PALLE 仅 LibriTTS ~580h，跨行不完全同条件。

### Table 2: 主观评测（cross-sentence）

| System | SMOS↑ | CMOS↑ | p-value |
|---|---|---|---|
| Ground Truth | 4.23 ± 0.18 | 0 | - |
| [[E2 TTS]] (32 NFE) | 3.95 ± 0.20 | -0.41 | 0.031 |
| [[F5-TTS]] (32 NFE) | 4.04 ± 0.18 | -0.20 | 0.033 |
| [[MaskGCT]] | 3.98 ± 0.17 | -0.25 | 0.025 |
| PALLE (two-stage) | 4.03 ± 0.17 | -0.15 | 0.036 |
| **PALLE (two-stage, GT dur)** | **4.09 ± 0.19** | **-0.09** | 0.038 |

**说明**：SMOS 带 95% 置信区间 + Wilcoxon 符号秩检验 p 值。PALLE (GT dur) SMOS 4.09 / CMOS −0.09 在所列系统中最接近 GT [已 verify Tab.2]。

### Table 3: 受控 AR / NAR / PAR 对比（全部 LibriTTS、同 tokenizer/detokenizer/骨干）

| ID | System | Paradigm | Cont WER-W | Cont WER-H | Cont SIM-o | Cross WER-W | Cross WER-H | Cross SIM-o | RTF |
|---|---|---|---|---|---|---|---|---|---|
| A | VALL-E (stage-one-only) | [[Autoregressive Model\|AR]] | 3.32 | 3.35 | **0.776** | 4.00 | 4.32 | **0.716** | 0.21 |
| B | MaskGCT (stage-one-only) | [[Non-Autoregressive Model\|NAR]] | - | - | - | 4.52 | 5.18 | 0.703 | 0.10 |
| C1 | PALLE (stage-one-only) | [[Pseudo-Autoregressive\|PAR]] | - | - | - | 2.58 | 3.15 | 0.710 | **0.05** |
| D1 | PALLE (stage-one-only, GT dur) | PAR | 2.41 | 2.70 | 0.775 | 2.64 | 3.14 | 0.711 | **0.05** |
| C2 | PALLE (two-stage) | PAR+NAR | - | - | - | **2.23** | **2.83** | **0.716** | 0.06 |
| D2 | PALLE (two-stage, GT dur) | PAR+NAR | **2.31** | **2.62** | **0.776** | 2.35 | 2.87 | **0.716** | 0.06 |

**关键发现**（同骨干、同数据的公平对比，是全文最强证据）：
- **PAR vs AR**（C1 vs A）：cross WER-W 2.58 vs 4.00（−36%），RTF 0.05 vs 0.21（≈4×）[已 verify Tab.3]。
- **PAR vs NAR**（C1 vs B）：cross WER-W 2.58 vs 4.52（−43%），RTF 0.05 vs 0.10（≈2×）[已 verify Tab.3]。
- **两阶段增益**（C2 vs C1）：cross WER-W 2.58→2.23、SIM-o 0.710→0.716，仅多 0.01 RTF [已 verify Tab.3]。

---

## 实验

### 数据集

| 数据集 | 规模 | 特点 | 用途 |
|--------|------|------|------|
| [[LibriTTS]] | ~580h, 2306 spk, EN | 多说话人英文 TTS | 训练 |
| LibriSpeech test-clean | 1234 样本, 40 spk, 4–10s | 标准零样本评测 | 测试 |

**任务**：Continuation（续写）+ Cross-Sentence（跨句克隆）两种零样本设置 [已 verify §5.1]。

### 实现细节

- **Backbone**：decoder-only Transformer，12 层 / 16 头 / dim 1024 / FFN 4096；一层 [[ConvNeXt V2]] block（dim 1024 + FFN 2048）；卷积位置编码 kernel 7；每 stage 177.0M 参数 [已 verify §5.1]。
- **Tokenizer**：文本 2000 类 BPE；语音 [[S3Tokenizer]] v2 25 Hz；detokenizer = [[CosyVoice 2]] CFM + [[HiFi-GAN]] [已 verify §5.1]。
- **优化**：ScaledAdam + Eden scheduler；8× V100 32GB；batch 600s/GPU。Stage 1 训 179k steps（LR 0.045），Stage 2 微调 87k steps（LR 0.005）[已 verify §5.1]。
- **评测口径**：WER-W 用 [[Whisper]] Large-v3、WER-H 用 HuBERT-Large；SIM-o 用 [[WavLM]]-TDNN；RTF 在 A100 80G 测 [已 verify §5.1]。

### 结果可信度

<!-- R3 -->

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | Tab.3 受控 AR/NAR/PAR 对比（同骨干/同数据/同 tokenizer） | 唯一变量是范式，公平可比；这是 PAR 有效性的最强证据 |
| **中** | Tab.1 跨系统 WER/SIM/RTF | 跨行训练数据不同（LibriTTS vs Emilia/Librilight），且部分 baseline 用开源 ckpt 复测，非同条件 |
| **中** | Tab.2 主观 SMOS/CMOS | 有置信区间 + 显著性检验（较规范），但评审人数未明、绝对差距小（CMOS −0.09 ~ −0.41） |
| **低** | RTF 绝对值横向比 | RTF 受实现/批大小/硬件影响大，跨系统不完全可比（但同骨干的 Tab.3 内部可比） |

---

## 批判性思考

### 核心 Claim 审查

<!-- R1 -->

1. **Paper Claim**：PAR 统一 AR 与 NAR，兼得时序归纳偏置与并行效率。
   **My Assessment**：Tab.3 同骨干对比下 PAR 同时优于 AR/NAR 的 WER 且更快，支撑较强 [已 verify Tab.3]。但"统一"是范式叙述，PAR 本质是"固定步 + 动态 span 的 masked generative 解码调度"，与 MaskGIT 式置信度无序解码相比是"强制从左到右"的有序版本——是有意义的设计点但并非全新机制。

2. **Paper Claim**：在 LibriTTS 上以 RTF 0.06 取得领先 WER/SIM。
   **My Assessment**：在**同数据集**（LibriTTS）对比 [[CosyVoice 2]] 成立（WER-W 2.23 vs 3.00）[已 verify Tab.1]。但与 Emilia 大规模训练的 [[F5-TTS]]/[[E2 TTS]]/[[MaskGCT]] 跨行比，训练数据量差异巨大，"领先"需限定在"小数据 + 低延迟"约束下，不宜泛化为绝对最优。

3. **Paper Claim**：联合多任务（单模型同时学两 stage）不可行。
   **My Assessment**：§5.5 报告联合训练使 cross-sentence WER 升 ~20%、有效 capacity 折半，故采用两个独立 stage [已 verify §5.5]。这是诚实的负结果，但也意味着部署需两份 177M 权重（共 ~354M）。

### 优点
1. **Tab.3 的受控对比设计干净**：同骨干/同数据/同 tokenizer，单变量为范式，PAR 的 WER + 速度双优结论可信度高。
2. **PAR 范式抽象清晰**：用"时间步 × 每步长度"二维把 AR/NAR/PAR 统一，Fig.1 直观。
3. **效率收益实在**：RTF 0.05–0.06 远低于 AR 的 0.21、CosyVoice 2 的 0.45。
4. **负结果诚实**：明确报告联合多任务失败，而非掩盖。

### 局限性
1. **仅 LibriTTS ~580h 英文**：未验证大规模、多语种、噪声数据下的可扩展性；与大数据 baseline 跨行比不完全公平 [已 verify §5.1]。
2. **两份独立权重**：两 stage 不能共享参数，部署成本约 2×。
3. **依赖外部 detokenizer**：质量上限受 [[CosyVoice 2]] CFM+vocoder 约束，端到端非自给 [已 verify §4.1]。
4. **语义 token 路线天花板**：用 [[S3Tokenizer]] 语义 token，其重建上限受限——GT 经 S3Tokenizer v2 重建后 continuation SIM-o 0.793（低于 GT EnCodec 0.823）、cross-sentence SIM-o 0.743（高于 GT EnCodec 0.715），即说话人相似度天花板被 tokenizer 重建质量约束，且在 continuation 设置下明显低于声学 codec [已 verify Tab.1]。
5. **未见官方开源代码**：截至记录无官方 repo，L2 GitHub verify 不可用，复现依赖论文描述。

### 潜在改进方向
1. 扩到大规模/多语种数据，检验 PAR 在数据充足时是否仍优于 NAR。
2. 探索两 stage 参数共享/蒸馏以降部署成本。
3. 换更高保真的混合 token（如 mixed semantic+acoustic）提升 SIM-o 上限。

### 可复现性评估
- [ ] 代码开源（未见官方 repo）
- [ ] 预训练模型（未见）
- [x] 训练细节完整（§5.1 给了层数/步数/LR/硬件）
- [x] 数据集可获取（LibriTTS / LibriSpeech 公开）

---

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §路线 2（离散 codec-LM TTS）——PALLE 在该路线内开辟"PAR 生成调度"子分支，介于 AR（[[VALL-E]]）与 NAR masked generative（[[MaskGCT]]）之间
- **核心问题**：[[TTS-核心挑战]] §挑战 3（延迟/RTF）；§挑战 1（零样本克隆 / 说话人相似度）
- **表示层位置**：[[TTS-表示层地图]] §semantic-token（[[S3Tokenizer]] 25Hz 语义 token，非声学 token）
- **相邻工作**：[[VALL-E]]（AR 对照）/ [[MaskGCT]]（NAR 对照）/ [[CosyVoice 2]]（同 tokenizer 同数据 baseline）/ [[E2 TTS]]（特征维融合来源）/ [[F5-TTS]]（flow-matching 对照）

---

## 🔄 后续重估

- **2026-05-29**：初读（基于 arXiv v3 全文 + PDF 高清核对所有表格/公式/图）。判断核心价值在 Tab.3 的同骨干受控对比——PAR 相对 AR/NAR 同时降 WER + 提速，证据较硬；但跨数据集的"领先"宣称需限定在小数据 + 低延迟约束。PAR 我倾向归为"masked generative 解码调度的有序变体"而非全新建模机制。maturity 标 exploratory（单点工作、未见独立复现/开源）。待回填：[[TTS-技术路线图]] codec-LM 路线下补 PAR 子分支。

---

## 关联笔记

### 基于
- [[MaskGCT]]: PAR 骨干即 MaskGIT 式双向 masked generative transformer；PAR 是其"强制从左到右"的有序变体
- [[E2 TTS]]: 特征维模态融合（text [PAD] + speech [MASK] 同长拼接）直接借鉴
- [[CosyVoice 2]]: 语音 tokenizer（[[S3Tokenizer]] v2 25Hz）+ detokenizer（CFM+HiFi-GAN）

### 对比
- [[VALL-E]]: AR 范式对照——PALLE 证明 PAR 在同骨干下 WER 更低且约 4× 快
- [[F5-TTS]] / [[E2 TTS]]: 连续 flow-matching 路线对照
- [[MaskGCT]]: NAR masked generative 对照——PALLE 证明 PAR 在同骨干下 WER 更低且约 2× 快

### 方法相关
- [[Pseudo-Autoregressive]]: 核心建模范式
- [[Masked Generative Transformer]]: 骨干
- [[Confidence-based Iterative Decoding]]: Stage 2 精修机制
- [[ConvNeXt V2]]: 文本编码组件
- [[Conditional Flow Matching]] / [[HiFi-GAN]]: detokenizer/vocoder

### 硬件/数据相关
- [[LibriTTS]]: 训练数据
- [[Whisper]] / [[WavLM]]: 评测口径（WER-W / SIM-o）

---

## 速查卡片

> [!summary] PALLE (ACM MM 2025)
> - **核心**: 伪自回归 PAR——固定时间步上并行生成动态长度 span，统一 AR + NAR
> - **方法**: 两阶段（Stage 1 PAR 生成 + Stage 2 置信度引导 NAR 精修），MaskGIT 式双向骨干 + S3Tokenizer 语义 token + CosyVoice 2 detokenizer
> - **结果**: LibriTTS 上 cross-sentence WER-W 2.23、SIM-o 0.716、RTF 0.06；同骨干对比 PAR 较 AR −36% WER/4× 快、较 NAR −43% WER/2× 快
> - **代码**: 未见官方开源 repo（项目页 microsoft.com/research/project/vall-e-x/palle）

---

*笔记创建时间: 2026-05-29*
