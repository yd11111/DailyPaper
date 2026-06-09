---
title: "Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot Text-to-Speech Synthesis"
method_name: "PALLE"
authors: [Yifan Yang, Shujie Liu, Jinyu Li, Yuxuan Hu, Haibin Wu, Hui Wang, Jianwei Yu, Lingwei Meng, Haiyang Sun, Yanqing Liu, Yan Lu, Kai Yu, Xie Chen]
year: 2025
venue: ACM MM 2025
arxiv_id: "2504.10352"
tags: [tts, codec-lm-tts, zero-shot-tts, pseudo-autoregressive, masked-generative]
zotero_collection:
note_tier: standard

# === 技术决策枚举（核验后只留枚举值；带 [§X] 的 prose 版结论见正文「📋 核验结论」表）===
lm_init_type: cold-start          # 从头训练 177.0M decoder-only transformer
multitask: false                  # 联合多任务失败，采用两个独立 stage
post_training_type: none          # 无 RLHF/DPO
streaming: false                  # 两阶段 masked generative，需全长 T^gen，非流式

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

> **笔记分级**：standard（方法清晰、值得精读）。
>
> **结构**：一、阅读层（核验后口径）/ 二、研究审计层（核验来源 + 完整公式图表 + 可信度 + 批判）/ 三、知识系统层。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Shanghai Jiao Tong University（X-LANCE）+ Microsoft |
| 日期 | April 2025（arXiv v1）；ACM MM 2025 接收 |
| 项目主页 | https://microsoft.com/research/project/vall-e-x/palle |
| 对比基线 | [[VALL-E]] / [[CosyVoice 2]] / [[MaskGCT]] / [[E2 TTS]] / [[F5-TTS]] |
| 链接 | [arXiv](https://arxiv.org/abs/2504.10352) / Code（未见官方开源 repo） |

## 一句话总结

> 提出"伪自回归"(PAR)——在固定时间步上并行生成**动态长度** span，融合 AR 的时序归纳偏置与 NAR 的并行效率；两阶段 PALLE 在 LibriTTS 上以 RTF 0.06 取得领先的 WER/SIM（限小数据 + 低延迟约束）。

---

# 一、阅读层（主文）

## 核心贡献

1. **Pseudo-Autoregressive (PAR) 建模范式**：在 [[Masked Generative Transformer]] 上，每个时间步并行预测所有位置但只保留**最左侧动态长度 span**，既有 [[Autoregressive Model|AR]] 的从左到右时序结构，又有 [[Non-Autoregressive Model|NAR]] 的并行推理；span 长度可变（不像 AR 每步 1 token，也不像 NAR 一次全出）。
2. **两阶段 PALLE 系统**：Stage 1 用 PAR 快速生成完整草稿；Stage 2 用 [[Confidence-based Iterative Decoding|置信度引导的迭代精修]]（NAR）重掩码并重预测低置信 token，提升质量。
3. **效率收益**：受控同骨干对比里 PAR 相对 AR 约 4× 加速、相对 NAR 约 2× 加速，且 WER 更低；两阶段 PALLE 在 LibriTTS 上 RTF 0.06。

## 问题背景

### 要解决的问题
[[Zero-shot TTS|零样本 TTS]]（给一段参考音频即复刻音色）中，离散 [[codec-lm-tts|codec token + 语言模型]]路线存在效率/质量两难：[[Autoregressive Model|AR]] 逐 token 生成质量好但慢；[[Non-Autoregressive Model|NAR]]（如 [[MaskGCT]]）并行快但丢失时序归纳偏置、易在高熵区出现快速韵律跳变。

### 现有方法的局限
作者把两类方法抽象为"时间步 × 每步长度"两个轴：
- **AR**（[[VALL-E]] 系）：固定时间步 + 固定长度（每步 1 token），延迟随序列线性增长。
- **NAR / masked generative**（[[MaskGCT]]）：单步全长、迭代式无序生成提升效率，但依赖置信度调度、在高熵区可能产生快速过渡的韵律。
- 二者都未利用"固定步 + 动态长"这一中间地带。

### 本文的动机
用"固定时间步 + 动态长度 span"统一 AR 与 NAR：保留从左到右的时序顺序（软时序归纳偏置），同时允许每步并行吐出可变数量 token，逼近 O(1) 步数级别的推理延迟。

## 方法详解

### 领域定位

PALLE 属于**离散 [[codec-lm-tts|codec-LM]] 零样本 TTS** 路线，与 [[VALL-E]]（AR）、[[MaskGCT]]（NAR masked generative）、[[E2 TTS]]/[[F5-TTS]]（连续 flow-matching）同处零样本 TTS 竞争位。其 novelty 不在表示或骨干（骨干就是 MaskGIT 式双向 masked transformer、表示用 [[S3Tokenizer]] 语义 token），而在**生成调度范式**：把 AR 的"固定步固定长"与 NAR 的"单步全长"统一为"固定步 + 动态长 span"的 PAR。

### 端到端数据流

PALLE 的完整推理流水线分四段：

1. **编码**：参考语音经 [[S3Tokenizer]]（v2, 25Hz）提取语义 token $\mathbf{c}^{ref}$；目标文本经 2000 类 BPE + 一层 [[ConvNeXt V2]] block 编码为文本特征。二者按 [[E2 TTS]] 式**特征维融合**——文本用 `[PAD]` 补齐、语音用 `[MASK]` 补齐到同长 $T$，各自 embedding 后沿特征维拼接（非时间维 concat），喂入 Transformer。
2. **Stage 1（PAR 草稿生成）**：一个 177M 参数的 decoder-only bidirectional Transformer（12 层 / 16 头 / dim 1024），用 PAR 规则从左到右逐 span 填充 `[MASK]` 位置，约 100 步生成完整语义 token 序列 $\mathbf{c}^{gen}$。
3. **Stage 2（NAR 精修）**：另一个同架构 177M Transformer，以 Stage 1 输出为初始，基于置信度重掩码低质量 token 并重预测，约 7 步完成精修。
4. **解码**：精修后的语义 token 送入 [[CosyVoice 2]] 的 [[Conditional Flow Matching|CFM]] + [[HiFi-GAN]] detokenizer/vocoder 还原波形。

![[_resources/2504.10352/figures/fig2.png]]

> **Figure 2**：PALLE 系统总览。从左到右：文本经 BPE tokenizer 编码，参考语音经 [[S3Tokenizer]] 提取语义 token；二者进入 Stage 1 的 masked generative Transformer 用 PAR 规则生成草稿 token 序列；Stage 2 同架构 Transformer 做置信度引导精修；最终 token 送入 CosyVoice 2 detokenizer 还原波形。两个 stage 架构相同但参数独立（联合训练失败，见附录核验结论）。

![[_resources/2504.10352/figures/fig3.png]]

> **Figure 3**：单 stage 内部架构与两阶段细节。(a) 骨干：输入 embedding + [[ConvNeXt V2]] block + 线性投影 + 卷积位置编码（kernel=7）→ 12 层双向 Transformer → 输出 logits。(b) Stage 1（PAR）：文本与语音特征维融合后，模型预测所有位置但只采纳掩码段最左侧 $k$ 个 token 作为输出。(c) Stage 2（NAR 精修）：对 Stage 1 输出中置信度最低的 token 重掩码并重预测。

### PAR：什么是伪自回归

PALLE 的核心创新是 **Pseudo-Autoregressive (PAR)** 生成范式。要理解它，先看它统一的两个极端：

![[_resources/2504.10352/figures/fig1.png]]

> **Figure 1**：三种范式对照。**AR**（左）：每步只生成 1 个 token，从左到右逐位推进，步数 = 序列长度——质量好但慢。**NAR**（中）：每步预测所有 `[MASK]` 位置，多轮无序补全——快但丢失时序结构，高熵区域容易产生韵律跳变。**PAR**（右）：每步并行预测所有位置但**只采纳最左侧一段**（绿色=保留，浅绿=预测但丢弃），逐步向右推进——既保留从左到右的时序归纳偏置，又每步吐出可变长 span 实现并行加速。

**为什么 PAR work**：AR 的质量优势来自从左到右的时序归纳偏置（每个 token 只依赖已确定的左侧上下文），但代价是每步仅 1 token。PAR 保留了这个偏置——每步只**采纳**最左侧 span，所以被采纳的 token 始终有完整的左侧上下文——同时每步采纳 $k'$ 个 token 而非 1 个，总步数从 $T$ 降到 $T/k'$。NAR 的问题是无序解码（任何位置都可能先被预测），在高熵区域（如句子边界、韵律转换点）容易产生快速过渡；PAR 强制从左到右推进，避免了这个问题。

**具体例子**：假设参考语音有 100 个 token，目标语音需要 200 个新 token（$T^{gen}=200$），每步采纳比例 $r'=0.1$，即每步采纳 $k'=\lfloor 0.1 \times 200 \rfloor = 20$ 个 token。

- **第 0 步**：200 个位置全是 `[MASK]`。模型并行预测所有 200 个位置，但只**保留最左边 20 个**（位置 1-20），其余 180 个预测值丢弃。
- **第 1 步**：位置 1-20 已锁定，位置 21-200 仍是 `[MASK]`。模型再次并行预测剩余 180 个位置，只保留最左边 20 个（位置 21-40）。
- **…重复…**
- **第 9 步**：位置 1-180 已锁定，剩余 20 个位置全部采纳。生成完成。

总共约 10 步生成 200 个 token。对比 AR 需要 200 步（每步 1 个），PAR 快约 20 倍；对比 NAR 一次全出但要多轮无序精修（[[MaskGCT]] 实测约 20 步），PAR 步数相近但每步有时序保障。论文实际用 $r'$ 使得约 100 步收敛（因为 $r'$ 取较小值以换取质量）。

### 训练：模型怎么学 PAR

训练时不做多步迭代，而是**单步监督**：随机采样一个掩码起点 $s$，把 $s$ 之后的位置全部置为 `[MASK]`，模型预测所有掩码位置，但 loss 只计算掩码段**最左侧** $k=\lfloor rT\rfloor$ 个 token（$r=0.1$）。

$$
\arg\max_{\theta}\ p\big([\mathbf{m}\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m})\odot\mathbf{c},\ \mathbf{x};\ \theta\big)
$$

**为什么只监督最左侧 $k$ 个**：如果监督全部掩码位置，模型学到的是 NAR（无序补全）；只监督最左侧 span，模型被迫学习"先预测紧邻已知上下文的位置"——这就是 PAR "从左到右推进"的训练信号。

Stage 1 具体设定：掩码起点 $s$ 从 $\lfloor 0.3T \rfloor$ 到 $T - \lfloor 0.1T \rfloor$ 均匀采样——保证前 30% 作为 prompt（模拟推理时的参考语音），至少最后 10% 用于监督。$k = \lfloor 0.1T \rfloor$。

Stage 2 训练不用 PAR，用标准 NAR：对 prompt 之后的 token 以 $p=0.1$ 独立随机掩码，监督还原全部掩码位置——因为 Stage 2 的任务是精修（无序修补局部错误），不需要从左到右的归纳偏置。

### 推理：两阶段怎么一步步生成

**Stage 1（PAR 生成，约 100 步）**：

1. 估计目标语音长度：$T^{gen} = T^{ref} \times (1 + L^{gen}/L^{ref})$，按参考音频语速线性外推。
2. 构造初始序列：参考语音 token $\mathbf{c}^{ref}$ + $T^{gen}-T^{ref}$ 个 `[MASK]`；文本 token 拼接后用 `[PAD]` 补齐到同长。
3. 迭代：每步模型预测所有位置，只采纳最左侧 $k' = \min(\lfloor r'T^{gen} \rfloor,\ N_{left})$ 个 token（$N_{left}$ 为剩余 `[MASK]` 数），锁定已采纳位置，向右推进。
4. 当所有 `[MASK]` 填完，输出草稿序列 $\mathbf{c}^{draft}$。

**Stage 2（NAR 精修，约 7 步）**：

1. 以 $\mathbf{c}^{draft}$ 为初始，模型预测所有位置的概率分布 $\mathbf{P} \in \mathbb{R}^{T^{gen} \times N}$。
2. 计算置信度 $\mathbf{C} = \log P^{max}$（每个位置取最大概率的对数）。
3. 对置信度最低的 $\gamma$ 分位 token **重掩码**，用模型重新预测这些位置。
4. 已被更新过的 token 置信度永久设为 1（防止反复精修同一位置）。
5. 重复 2-4 约 7 步。

**为什么需要 Stage 2**：PAR 的每步只保留最左侧 span，span 内部的 token 质量不均——靠近 span 左端的 token 有更强的已知上下文（左侧全是已确定的 token），靠近右端的上下文较弱。Stage 2 用置信度定位这些弱点并修补，代价仅增加约 0.01 RTF（0.05 → 0.06）。

## 关键结果

**核心证据**：Tab.3 的**受控 AR/NAR/PAR 对比**是全文最强证据——同 LibriTTS、同 tokenizer/detokenizer/骨干，唯一变量是范式。

| ID | System | Paradigm | Cross WER-W | Cross WER-H | Cross SIM-o | RTF |
|---|---|---|---|---|---|---|
| A | VALL-E (stage-one-only) | [[Autoregressive Model\|AR]] | 4.00 | 4.32 | **0.716** | 0.21 |
| B | MaskGCT (stage-one-only) | [[Non-Autoregressive Model\|NAR]] | 4.52 | 5.18 | 0.703 | 0.10 |
| C1 | PALLE (stage-one-only) | [[Pseudo-Autoregressive\|PAR]] | 2.58 | 3.15 | 0.710 | **0.05** |
| C2 | PALLE (two-stage) | PAR+NAR | **2.23** | **2.83** | **0.716** | 0.06 |

**结论**（同骨干同数据，公平可比）：
- **PAR vs AR**（C1 vs A）：cross WER-W 2.58 vs 4.00（−36%），RTF 0.05 vs 0.21（≈4×）。
- **PAR vs NAR**（C1 vs B）：cross WER-W 2.58 vs 4.52（−43%），RTF 0.05 vs 0.10（≈2×）。
- **两阶段增益**（C2 vs C1）：cross WER-W 2.58→2.23、SIM-o 0.710→0.716，仅多 0.01 RTF。

> 跨系统对比（Tab.1）与主观评测（Tab.2）见附录——跨行训练数据不同（LibriTTS vs Emilia/Librilight），可比性弱于 Tab.3。

---

# 二、研究/审计层（附录）

## 📋 核验结论（技术元数据）

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | from scratch（从头训练 177.0M decoder-only transformer，未用通用 LLM warm-start）| [已 verify §5.1] |
| 训练 loss | 纯 masked-token 交叉熵（MLM），无文本 loss、无 KL；两阶段各自的 MLM 目标 | [已 verify §3 Eq.2 / §4.2 Eq.9 / §4.3 Eq.10] |
| Tokenizer 架构 | text+speech feature-dimension fusion（E2 TTS 式）：text 用 `[PAD]`、speech 用 `[MASK]` 补齐到同长，各自 embedding 后沿特征维拼接 | [已 verify §3 Discussion / §4.2 Eq.6] |
| 多任务 | false——§5.5 报告联合多任务训练失败（cross-sentence WER 升 ~20%、capacity 折半），故采用两个独立 stage | [已 verify §5.5] |
| 训练数据 | LibriTTS ~580h、2306 说话人、纯英文 | [已 verify §5.1] |
| 后训练 | 无（无 RLHF/DPO）| [已 verify 全文未提] |
| Codec 细节 | S3Tokenizer v2（CosyVoice 2）25 Hz 语义 token（非 RVQ 声学 token）；detokenizer = CosyVoice 2 CFM + HiFi-GAN | [已 verify §4.1 / §5.1] |

## 完整公式

### 公式1: [[Pseudo-Autoregressive|PAR 训练掩码]]

$$
\mathbf{m} = [\underbrace{0,0,\ldots,0}_{T-\ell}, \underbrace{1,1,\ldots,1}_{\ell}]
$$

**含义**：把序列尾部连续 $\ell$ 个位置置为掩码。**符号**：$T$ = 下采样后语音 token 长度；$\ell$ = 随机采样掩码 span 长度。

### 公式2: [[Pseudo-Autoregressive|PAR 训练目标]]

$$
\arg\max_{\theta}\ p\big([\mathbf{m}\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m})\odot\mathbf{c},\ \mathbf{x};\ \theta\big)
$$

**含义**：只监督掩码段最左侧 $k=\lfloor rT\rfloor$ 个 token，条件是未掩码部分与文本 $\mathbf{x}$。**符号**：$k=\lfloor rT\rfloor$，$r\in(0,1)$ 固定比例；$\mathbf{c}$ 语音 token；$\odot$ 逐元素乘。

### 公式3: PAR 推理——扩展掩码序列

$$
\mathbf{c}^{ext}_{(1)} = [c^{ref}_1, c^{ref}_2, \ldots, c^{ref}_{T^{ref}}, \underbrace{0,0,\ldots,0}_{T^{gen}}]
$$

**含义**：参考语音 token 后接 $T^{gen}$ 个占位符。

### 公式4: PAR 推理掩码

$$
\mathbf{m}_{(t)} = [\underbrace{0,0,\ldots,0}_{T^{ref}+t\times k'}, \underbrace{1,1,\ldots,1}_{k'}, 0,0,\ldots,0]
$$

**含义**：第 $t$ 步只对从左数第 $t$ 段、长度 $k'$ 的 span 解码。**符号**：$k'=\min(\lfloor r'T^{gen}\rfloor, N_{left})$，$N_{left}$ 为剩余待生成 token 数。

### 公式5: PAR 推理迭代更新

$$
\mathbf{c}^{ext}_{(t+1)} = (1-\mathbf{m}_{(t)})\odot \mathbf{c}^{ext}_{(t)} + \mathbf{m}_{(t)}\odot \arg\max_{\mathbf{c}^{ext}_{(t+1)}} p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ref}, \mathbf{x}^{gen};\theta\big)
$$

**含义**：保留已确定位置、只更新当前 span，逐步向右推进。

### 公式6: [[E2 TTS|特征维融合]]文本补齐

$$
\mathbf{x}^{ext} = [x_0, x_1, \ldots, x_{L-1}, \underbrace{[\text{PAD}],[\text{PAD}],\ldots,[\text{PAD}]}_{T-L}]
$$

**含义**：文本 token 用 `[PAD]` 补齐到与语音同长 $T$。**符号**：$L$ 文本 token 数；$T-L$ 填充数。

### 公式7: Stage 1 训练起点采样

$$
s \sim \mathcal{U}\{\lfloor 0.3T\rfloor,\ \lfloor 0.3T\rfloor+1,\ \ldots,\ T-\lfloor 0.1T\rfloor-1\}
$$

**含义**：随机采样掩码起始位置 $s$，保证前 30% 作 prompt、至少最后 10% 用于监督。

### 公式8: Stage 1 训练掩码

$$
\mathbf{m}' = [\underbrace{0,0,\ldots,0}_{s}, \underbrace{1,1,\ldots,1}_{T-s}]
$$

**含义**：从起点 $s$ 起把后续全部置为掩码。

### 公式9: Stage 1 训练目标

$$
\arg\max_{\theta}\ p\big([\mathbf{m}'\odot\mathbf{c}]_{0:k}\mid (1-\mathbf{m}')\odot\mathbf{c},\ \mathbf{x}^{ext};\theta\big)
$$

**含义**：在 PAR 框架下监督最左 $k=\lfloor 0.1T\rfloor$ span。

### 公式10: Stage 2 训练目标（NAR 精修）

$$
\arg\max_{\theta}\ p\big(\mathbf{m}''\odot\mathbf{c}\mid (1-\mathbf{m}'')\odot\mathbf{c},\ \mathbf{x}^{ext};\theta\big)
$$

**含义**：对前 30% 之后的 token 以 $p=0.1$ 独立随机掩码，监督还原——标准 NAR masked prediction。

### 公式11: [[Duration Predictor|时长线性估计]]

$$
T^{gen} = T^{ref}\times\Big(1 + \frac{L^{gen}}{L^{ref}}\Big)
$$

**含义**：按参考语音/文本长度比线性外推目标语音 token 数。**符号**：$T^{ref}, L^{ref}$ 参考语音/文本 token 数；$L^{gen}$ 目标文本 token 数。

### 公式12: Stage 1 推理——语音 prompt 补齐

$$
\mathbf{c}^{ext} = [c^r_0, \ldots, c^r_{T^{ref}-1}, \underbrace{[\text{MASK}], \ldots, [\text{MASK}]}_{T^{gen}-T^{ref}}]
$$

### 公式13: Stage 1 推理——文本拼接补齐

$$
\mathbf{x}^{ext} = [x^r_0, \ldots, x^r_{L^{ref}-1}, x^g_0, \ldots, x^g_{L^{gen}-1}, \underbrace{[\text{PAD}], \ldots, [\text{PAD}]}_{T^{gen}-(L^{ref}+L^{gen})}]
$$

### 公式14: Stage 1 推理迭代（同 Eq.5）

$$
\mathbf{c}^{ext}_{(t+1)} = (1-\mathbf{m}_{(t)})\odot \mathbf{c}^{ext}_{(t)} + \mathbf{m}_{(t)}\odot \arg\max_{\mathbf{c}^{ext}_{(t+1)}} p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ext};\theta\big)
$$

### 公式15: Stage 2 预测概率矩阵

$$
\mathbf{P}_{(t)} = p\big(\mathbf{c}^{ext}_{(t+1)}\mid \mathbf{c}^{ext}_{(t)}, \mathbf{x}^{ext};\theta\big) \in \mathbb{R}^{T^{gen}\times N}
$$

**含义**：每个位置在 $N$ 个语义 token 类上的预测分布。

### 公式16: [[Confidence-based Iterative Decoding|置信度]]矩阵

$$
\mathbf{C}_{(t)} = \log P^{max}_{(t)}
$$

**含义**：置信度 = 预测分布最大概率的对数（负 min-entropy）；对最低 $\gamma$ 分位 token 重掩码重预测，已更新 token 置信度永久置 1。

## 完整图表

### Figure 3: 两阶段框架细节（已嵌入主文方法段）

> 详见主文「端到端数据流」一节，含完整长 caption。

### Figure 4: 推理步数对客观指标的影响

![[_resources/2504.10352/figures/fig4.png]]

**说明**：cross-sentence 下，Stage 1 约 100 步后 WER-H 平台化；Stage 2 在约 7 步内把 SIM-o 由 ~0.712 提到 ~0.717。

### Figure 5: 总时长倍率对客观指标的影响

![[_resources/2504.10352/figures/fig5.png]]

**说明**：总时长倍率 0.7–1.3×，WER-H 在 1.1× 最低、SIM-o 在 1.2× 最高，0.9–1.3× 整体稳健——Eq.11 线性时长估计不需精确即可工作。

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

**说明**：`*` = 用开源 checkpoint 复测；GT dur = 用真实总时长。同数据集（LibriTTS）下 PALLE cross-sentence WER-W 2.23 优于 [[CosyVoice 2]] 3.00（−26%）且 RTF 0.06 vs 0.45。**可比性注意**：F5-TTS/E2 TTS/MaskGCT 用 Emilia 大规模训练，PALLE 仅 LibriTTS ~580h，跨行不完全同条件。

### Table 2: 主观评测（cross-sentence）

| System | SMOS↑ | CMOS↑ | p-value |
|---|---|---|---|
| Ground Truth | 4.23 ± 0.18 | 0 | - |
| [[E2 TTS]] (32 NFE) | 3.95 ± 0.20 | -0.41 | 0.031 |
| [[F5-TTS]] (32 NFE) | 4.04 ± 0.18 | -0.20 | 0.033 |
| [[MaskGCT]] | 3.98 ± 0.17 | -0.25 | 0.025 |
| PALLE (two-stage) | 4.03 ± 0.17 | -0.15 | 0.036 |
| **PALLE (two-stage, GT dur)** | **4.09 ± 0.19** | **-0.09** | 0.038 |

**说明**：SMOS 带 95% 置信区间 + Wilcoxon 符号秩检验。PALLE (GT dur) SMOS 4.09 / CMOS −0.09 在所列系统中最接近 GT。

### Table 3 完整版: 受控 AR / NAR / PAR 对比（全部 LibriTTS、同 tokenizer/detokenizer/骨干）

| ID | System | Paradigm | Cont WER-W | Cont WER-H | Cont SIM-o | Cross WER-W | Cross WER-H | Cross SIM-o | RTF |
|---|---|---|---|---|---|---|---|---|---|
| A | VALL-E (stage-one-only) | AR | 3.32 | 3.35 | **0.776** | 4.00 | 4.32 | **0.716** | 0.21 |
| B | MaskGCT (stage-one-only) | NAR | - | - | - | 4.52 | 5.18 | 0.703 | 0.10 |
| C1 | PALLE (stage-one-only) | PAR | - | - | - | 2.58 | 3.15 | 0.710 | **0.05** |
| D1 | PALLE (stage-one-only, GT dur) | PAR | 2.41 | 2.70 | 0.775 | 2.64 | 3.14 | 0.711 | **0.05** |
| C2 | PALLE (two-stage) | PAR+NAR | - | - | - | **2.23** | **2.83** | **0.716** | 0.06 |
| D2 | PALLE (two-stage, GT dur) | PAR+NAR | **2.31** | **2.62** | **0.776** | 2.35 | 2.87 | **0.716** | 0.06 |

**说明**：主文「关键结果」已摘出 A/B/C1/C2 核心行；此处为含 GT dur 变体的完整版。

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **高** | Tab.3 受控 AR/NAR/PAR 对比（同骨干/同数据/同 tokenizer）| 唯一变量是范式，公平可比；PAR 有效性的最强证据 |
| **中** | Tab.1 跨系统 WER/SIM/RTF | 跨行训练数据不同（LibriTTS vs Emilia/Librilight），部分 baseline 用开源 ckpt 复测，非同条件 |
| **中** | Tab.2 主观 SMOS/CMOS | 有置信区间 + 显著性检验（较规范），但评审人数未明、绝对差距小（CMOS −0.09 ~ −0.41）|
| **低** | RTF 绝对值横向比 | RTF 受实现/批大小/硬件影响大，跨系统不完全可比（同骨干的 Tab.3 内部可比）|

## 核心 Claim 审查

1. **Paper Claim**：PAR 统一 AR 与 NAR，兼得时序归纳偏置与并行效率。
   **My Assessment**：Tab.3 同骨干对比下 PAR 同时优于 AR/NAR 的 WER 且更快，支撑较强 [已 verify Tab.3]。但"统一"是范式叙述，PAR 本质是"固定步 + 动态 span 的 masked generative 解码调度"，相比 MaskGIT 式置信度无序解码是"强制从左到右"的有序版本——是有意义的设计点但并非全新机制。

2. **Paper Claim**：在 LibriTTS 上以 RTF 0.06 取得领先 WER/SIM。
   **My Assessment**：同数据集对比 [[CosyVoice 2]] 成立（WER-W 2.23 vs 3.00）[已 verify Tab.1]。但与 Emilia 大规模训练的 [[F5-TTS]]/[[E2 TTS]]/[[MaskGCT]] 跨行比训练数据量差异巨大，"领先"需限定在"小数据 + 低延迟"约束下，不宜泛化为绝对最优。

3. **Paper Claim**：联合多任务（单模型同时学两 stage）不可行。
   **My Assessment**：§5.5 报告联合训练使 cross-sentence WER 升 ~20%、有效 capacity 折半 [已 verify §5.5]。诚实的负结果，但也意味着部署需两份 177M 权重（共 ~354M）。

### 优点
1. **Tab.3 受控对比设计干净**：同骨干/同数据/同 tokenizer，单变量为范式，PAR 的 WER + 速度双优结论可信度高。
2. **PAR 范式抽象清晰**：用"时间步 × 每步长度"二维统一 AR/NAR/PAR，Fig.1 直观。
3. **效率收益实在**：RTF 0.05–0.06 远低于 AR 的 0.21、CosyVoice 2 的 0.45。
4. **负结果诚实**：明确报告联合多任务失败，而非掩盖。

### 局限性
1. **仅 LibriTTS ~580h 英文**：未验证大规模、多语种、噪声数据下的可扩展性；与大数据 baseline 跨行比不完全公平 [已 verify §5.1]。
2. **两份独立权重**：两 stage 不能共享参数，部署成本约 2×。
3. **依赖外部 detokenizer**：质量上限受 [[CosyVoice 2]] CFM+vocoder 约束 [已 verify §4.1]。
4. **语义 token 路线天花板**：GT 经 S3Tokenizer v2 重建后 continuation SIM-o 0.793（低于 GT EnCodec 0.823）——说话人相似度天花板被 tokenizer 重建质量约束，continuation 设置下明显低于声学 codec [已 verify Tab.1]。
5. **未见官方开源代码**：截至记录无官方 repo，L2 GitHub verify 不可用，复现依赖论文描述。

### 潜在改进方向
1. 扩到大规模/多语种数据，检验 PAR 在数据充足时是否仍优于 NAR。
2. 探索两 stage 参数共享/蒸馏以降部署成本。
3. 换更高保真的混合 token（semantic+acoustic）提升 SIM-o 上限。

### 可复现性评估
- [ ] 代码开源（未见官方 repo）
- [ ] 预训练模型（未见）
- [x] 训练细节完整（§5.1 给了层数/步数/LR/硬件）
- [x] 数据集可获取（LibriTTS / LibriSpeech 公开）

> **实现细节补全**（§5.1）：ScaledAdam + Eden scheduler；8× V100 32GB；batch 600s/GPU；Stage 1 训 179k steps（LR 0.045），Stage 2 微调 87k steps（LR 0.005）。评测口径：WER-W 用 [[Whisper]] Large-v3、WER-H 用 HuBERT-Large；SIM-o 用 [[WavLM]]-TDNN；RTF 在 A100 80G 测。

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §路线 2（离散 codec-LM TTS）——PALLE 开辟"PAR 生成调度"子分支，介于 AR（[[VALL-E]]）与 NAR masked generative（[[MaskGCT]]）之间
- **核心问题**：[[TTS-核心挑战]] §挑战 3（延迟/RTF）；§挑战 1（零样本克隆 / 说话人相似度）
- **表示层位置**：[[TTS-表示层地图]] §semantic-token（[[S3Tokenizer]] 25Hz 语义 token，非声学 token）
- **相邻工作**：[[VALL-E]]（AR 对照）/ [[MaskGCT]]（NAR 对照）/ [[CosyVoice 2]]（同 tokenizer 同数据 baseline）/ [[E2 TTS]]（特征维融合来源）/ [[F5-TTS]]（flow-matching 对照）

## 🔄 后续重估

- **2026-05-29**：初读（基于 arXiv v3 全文 + PDF 高清核对所有表格/公式/图）。核心价值在 Tab.3 的同骨干受控对比——PAR 相对 AR/NAR 同时降 WER + 提速，证据较硬；但跨数据集的"领先"宣称需限定在小数据 + 低延迟约束。PAR 倾向归为"masked generative 解码调度的有序变体"而非全新建模机制。maturity 标 exploratory（单点工作、未见独立复现/开源）。待回填：[[TTS-技术路线图]] codec-LM 路线下补 PAR 子分支。
- **2026-05-29**：按笔记三层分级结构重写（standard tier 试点）——frontmatter 减负为枚举、核验结论 prose 移入附录、关键图表入论证链、正文改用核验后口径。内容无新增，仅结构 relocate。
- **2026-06-01**：方法段从罗列式重写为叙事式（方法叙事标准试点）——Fig.3 从附录提回主文驱动讲解、加端到端数据流全景、PAR 用具体例子走一遍、训练/推理分开叙述、补 why 因果链。内容无新增，仅叙事重组。

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
> - **结果**: LibriTTS 上 cross-sentence WER-W 2.23、SIM-o 0.716、RTF 0.06；同骨干对比 PAR 较 AR −36% WER/4× 快、较 NAR −43% WER/2× 快（限小数据 + 低延迟约束）
> - **代码**: 未见官方开源 repo（项目页 microsoft.com/research/project/vall-e-x/palle）

---

*笔记创建时间: 2026-05-29*

> 🔍 **对比报告**: [[2026-05-29-VALL-E系列演进调研]]
