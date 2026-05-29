---
title: TTS 技术路线图
type: 技术路线
domain: TTS
tags: [tech-route, tts, architecture]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 技术路线图

TTS 的技术演进可以按**中间表征**划分为四条主干路线，外加一条正在浮现的新路线。每条路线有其核心假设、代价和上限。

> ⚠️ **未 verify 警告**（2026-05-26 添加）：本文档 2026-05-26 修订时新增的内容（路线 2 五维 taxonomy / 路线 4 扩散三阶段 / Instruction TTS 五策略 / 对话系统中的 TTS 等）含具体技术对照 cell，部分基于综述/笔记摘要 + ML 直觉推断，**未按新工作流 §11 三层 verify（论文原文 §X / GitHub 源码）**。**结构层（5 条路线 + 决策树 + 融合趋势）保留**，**具体 cell 的"X 路线代表是 Y / 用了哪种 codec / 哪种 loss"等技术断言需要逐 cell 重审**。详见 [[方法论复盘-2026-05-26-知识地图建设]]，verify 任务追踪在 [[待回填地图]]。
>
> **2026-05-26 已 verify 修订**：[[Qwen3-TTS]] 相关 cell（line 71 模型表 + line 251 路线融合）已基于 [[Qwen3-TTS]] frontmatter `lm_init` verified 元数据软化"warm-start"表述为"paper claim warm-start，literal weight warm-start 未独立 verify"。CosyVoice3 / StepAudio2.5 类似 cell 仍未 dogfood verify。

---

## 路线 1: 声学特征中介（Mel/Linear Spectrogram）

### 范式

```
Text → 文本编码器 → 声学模型(预测mel) → 声码器(mel→wav)
```

### 核心假设

mel-spectrogram 是一种足够好的中间表征——它保留了说话人音色、韵律、内容的大部分信息，同时维度远低于波形。

### 代表工作

| 模型 | 年份 | 声学模型 | 声码器 | 关键创新 |
|------|------|---------|--------|---------|
| [[TransformerTTS]] | 2018 | Transformer + autoregressive | Griffin-Lim / WaveRNN | 首次将 Transformer 用于 TTS |
| [[FastSpeech]] | 2019 | Transformer + 非自回归 | 任意 | Length Regulator 实现并行生成 |
| [[FastSpeech2]] | 2020 | Transformer + 显式 variance adaptor | HiFi-GAN | 直接预测 duration/pitch/energy |
| [[MegaTTS]] | 2023 | Transformer + GAN + prosody diffusion | HiFi-GAN | 解耦内容/韵律/音色为独立编码 |
| [[MegaTTS2]] | 2023 | Transformer + prosody LM | HiFi-GAN | 音素级韵律 latent + PLM |

### 优势与局限

- **优势**: 架构成熟、训练稳定、推理快（非自回归）、可解释性好（mel 可视化）
- **局限**: mel 到 wav 的声码器引入不可逆误差；mel 维度固定难以适配不同采样率；难做真正的端到端优化
- **现状**: 纯 mel 路线已不是 2024+ 的主流选择，但 mel 仍作为 tokenizer-free 路线的连续目标（见路线 5）

## 路线 2: Codec Token 路线（离散 token 序列）

### 范式

```
Text → LLM/AR模型 → 离散speech token序列 → Codec解码器(token→wav)
```

### 核心假设

语音可以被 [[RVQ]]（Residual Vector Quantization）压缩为离散 token 序列，从而复用 LLM 的序列建模能力。第一层 token 捕获语义/内容，深层 token 捕获声学细节。

### 代表工作

| 模型 | 年份 | 语言模型 | Codec | 关键创新 |
|------|------|---------|-------|---------|
| [[VALL-E]] | 2023 | AR(第1层) + NAR(2-8层) | [[EnCodec]] | 开创 Codec LM 范式 |
| [[SPEAR-TTS]] | 2023 | 语义LM → 声学LM | 自定义 | 两阶段：先预测语义 token 再预测声学 token |
| [[CosyVoice]] | 2024 | LLM(语义token) + Flow Matching | 自训练 | 将 LLM 与 Flow Matching 解码结合 |
| [[CosyVoice2]] | 2024 | LLM(FSQ token) + Flow Matching | [[FSQ]]-based | [[Finite Scalar Quantization]] 替代 RVQ |
| [[CosyVoice3]] | 2025 | LLM + Flow Matching | 同 v2 | 100万h 数据，工业极致 |
| [[IndexTTS2]] | 2025 | 100层 GPT + BigVGAN-v2 | 自训练 | 极深 LM + 高质量声码器 |
| [[GPT-SoVITS]] | 2024 | GPT(语义) + VITS(声学) | [[HuBERT]] token | 语义 token + VITS 联合 |
| [[SeedTTS]] | 2024 | AR LM + Diffusion | 自训练 | 自蒸馏 + 强化学习后训练 |
| [[GLM-TTS]] | 2024 | Streaming LM + Flow | 自训练 | 流式 chunk 级解码 |
| [[Qwen3-TTS]] | 2026 | 基于 Qwen3 LM family（架构 Qwen3-style；literal weight warm-start 未 verify）| multi-codebook | paper claim LLM-native，不额外训练独立声学模型；详见 [[Qwen3-TTS]] frontmatter `lm_init` |
| [[FireRedTTS]] | 2024 | AR LM | 图像 tokenizer 跨模态 | 用图像 VQ 做语音 tokenization |

### Token 设计演进

#### 时间线视角（单轴速览）

```
EnCodec RVQ 8层    →    RVQ 简化(1-2层语义+NAR补全)    →    FSQ 连续量化
(2022, 信息分散)        (2023-24, VALL-E/SPEAR-TTS)        (2024, CosyVoice2)
                                                            ↓
                                                    Tokenizer-free
                                                    (2025, VoxCPM)
```

#### 五维 Taxonomy 视角（Mousavi 2025 #7 提供）

Mousavi 2025 明确反对传统"语义 vs 声学"二分（认为 insufficient），提出五维分类。该五维比时间线视角更能解释不同 codec 的设计权衡：

| 维度 | 选项 | 代表 |
|---|---|---|
| **架构** | CNN / CNN+RNN / Transformer / CNN+T | SoundStream(CNN) / EnCodec(CNN+RNN) / Mimi(CNN+T) |
| **量化** | K-means / RVQ / SVQ / GVQ / FSQ / MSRVQ / CSRVQ / PQ | EnCodec(RVQ) / WavTokenizer(SVQ) / CosyVoice2(FSQ) |
| **训练** | 分离 vs 联合；目标 Recon/VQ/GAN/Feat/Diff/MP | SoundStream(GAN+Recon) / DAC(+mel loss) |
| **流式** | 因果卷积 / 因果注意力 / 算法延迟 / 帧率 | Mimi(因果 T, 12.5fps) vs EnCodec(75fps) / DAC(50fps) |
| **领域** | 单域 Speech/Music/Audio vs 多域 | SpeechTokenizer(speech-only) vs EnCodec(多域) |

**辅助维度**：解耦（FACodec 分 content/prosody/timbre/detail）、语义蒸馏（SpeechTokenizer/Mimi 首层 SSL 蒸馏）。

详细分类与每个 codec 的对应见 [[TTS-表示层地图]] §2。

### 优势与局限

- **优势**: 复用 LLM scaling law；天然支持 in-context learning（零样本）；token 序列可做 text-speech 联合训练
- **局限**: RVQ 多层 token 带来序列长度爆炸（1s 语音 ≈ 600-1200 token）；离散化丢失细节；codec 质量是系统上限；对 codec 选择敏感

### Codec 重建上界 → TTS 质量天花板（Mousavi 2025 #7 形式化）

**关键洞察**：**codec 的重建上界 = TTS LM 输出还原后的音质天花板**。即使 LM 预测完美，decoder 无法恢复的细节就是不可逆损失。

这给出了 codec 选型的硬约束：
- 不能用纯语义 token 做最高保真度 TTS
- codec 重建质量（PESQ / STOI / ViSQOL）应作为 TTS LM scaling 之前的必要前置检验
- 工业实践中"换 codec"比"换 LM" 经常带来更大 TTS 质量提升（CosyVoice v1 → v2 的 RVQ → FSQ 跃迁即为例证）

详见 [[TTS-表示层地图]] §4.2 信息保真度。

## 路线 3: 端到端路线（Text → Waveform）

### 范式

```
Text → 单一模型 → Waveform（无中间表征/显式中间表征在模型内部）
```

### 核心假设

消除级联系统的信息损失，让梯度从波形直接流回文本编码器。

### 代表工作

| 模型 | 年份 | 架构 | 关键创新 |
|------|------|------|---------|
| [[VITS]] | 2021 | VAE + Normalizing Flow + HiFi-GAN | 首个真正端到端的高质量 TTS |
| [[YourTTS]] | 2022 | VITS 多语言扩展 | 跨语言零样本 |
| [[NaturalSpeech]] | 2022 | VAE + Flow + 大规模预训练 | 首次在 LJSpeech 上达到人类水平 MOS |
| [[F5-TTS]] | 2024 | DiT + Flow Matching | 无需音素对齐、无需时长模型 |
| [[VoxCPM]] | 2025 | LLM 直接预测连续 mel → BigVGAN | 无 tokenizer 的端到端 |

### 优势与局限

- **优势**: 训练目标统一、无级联误差、架构简洁
- **局限**: 训练不稳定（波形空间高维）；调参难度大；大规模数据下训练成本高

## 路线 4: 连续 Latent 路线（Diffusion/Flow on Latent Space）

### 范式

```
Text → 条件编码 → Diffusion/Flow Matching → 连续latent → 解码器 → Waveform
```

### 核心假设

在连续 latent 空间做生成比离散 token 空间更自然——语音的韵律、音色本就是连续变化的，强制离散化会丢失信息。

### 扩散在 TTS 的三阶段引入（Zhang 2023 #4 分类）

Zhang 2023 提供扩散介入 TTS 系统的**三阶段分类**——按目标表示空间区分：

| 引入阶段 | 目标空间 | 扩散在做什么 | 代表 |
|---|---|---|---|
| **声学模型阶段** | mel 或 latent | 生成 mel/latent，再交给独立声码器 | Grad-TTS / Diff-TTS / ProDiff / DiffGAN-TTS / NaturalSpeech 2 / EmoDiff |
| **声码器阶段** | 波形 | 从 mel 直接生成波形 | WaveGrad / DiffWave / BDDM / PriorGrad / SpecGrad / WaveFit |
| **端到端阶段** | 波形 | 文本直接到波形，无显式中介 | WaveGrad 2 / CRASH / FastDiff / Itôn |

**我的归纳**：引入越深，表示空间问题越被回避（端到端阶段几乎不需要 codec）；但训练越难、推理越慢。

**注意 Zhang 2023 时间局限**：未覆盖 Flow Matching、NaturalSpeech 3、DiffSinger 等。但**三阶段分类框架本身仍有效**——可以把后续工作套进来：F5-TTS 属端到端，CosyVoice 系列的 Flow Matching 部分属声学模型阶段。

### 代表工作

| 模型 | 年份 | 生成模型 | Latent 来源 | 关键创新 |
|------|------|---------|------------|---------|
| Diff-TTS | 2021 | Diffusion (DDPM) | mel | 首个将 DDPM 应用于 mel 生成的声学模型，用 DDIM 加速采样 |
| Grad-TTS | 2021 | Diffusion (SDE) | mel | 基于 SDE 公式化，U-Net 解码器，提供端到端可能性 |
| WaveGrad | 2020 | Diffusion (声码器) | 波形 | 开创性扩散声码器，结合 score matching，6 步生成高质量波形 |
| DiffWave | 2021 | Diffusion (声码器) | 波形 | 首个展示扩散在波形生成多任务通用性的模型 |
| ProDiff | 2022 | Diffusion | mel | generator-based 参数化直接估计 clean data，通过知识蒸馏减半步数 |
| DiffGAN-TTS | 2022 | Diffusion+GAN | mel | 用预训练 GAN 建模去噪分布，1 步即可高质量生成 |
| BDDM | 2022 | Diffusion (声码器) | 波形 | 额外 schedule network 预测采样噪声计划，7 步达人声级 |
| [[NaturalSpeech2]] | 2023 | Diffusion | 自训练 VAE latent | 连续 latent + diffusion，首个大规模零样本 |
| [[NaturalSpeech3]] | 2024 | Flow Matching | FACodec 分解 latent | 分解为 content/prosody/timbre/detail 四路 latent |
| [[SemaVoice]] | 2026 | Flow Matching | 语义 latent | SFM 对齐 + patch-wise diffusion head |
| [[MaskGCT]] | 2024 | Masked Generative | Codec latent | 非自回归 masked prediction |

### 优势与局限

- **优势**: 避免离散化信息瓶颈；生成质量理论上限更高；Flow Matching 训练比 Diffusion 稳定
- **局限**: 推理速度受 diffusion steps 限制（需 ODE solver）；latent 空间设计需要精心调参

## 路线 5（快速增长中）: Tokenizer-free / 连续 AR

### 范式

```
Text → LLM/AR模型 → 连续mel帧或连续latent序列 → 声码器 → Waveform
```

### 核心假设

语音的连续特性不应被强制离散化。跳过 codec tokenization 这个"人为瓶颈"，直接在连续空间做自回归建模，同时保留 LM 的 scaling 优势。

### 代表工作

| 模型 | 年份 | 连续目标 | 生成方式 | 关键创新 |
|------|------|---------|---------|---------|
| [[MELLE]] | 2024 | 连续 mel | AR + latent sampling module | 首个无 VQ 的 AR TTS |
| [[LatentLM]] | 2024 | 连续 latent | AR + diffusion loss | 用 diffusion loss 替代 cross-entropy |
| [[VoxCPM]] | 2025 | 连续 mel 帧 | LLM AR 连续预测 | 无 tokenizer，180 万小时数据 |
| [[FELLE]] | 2025 | 连续 mel | AR + token-wise Flow Matching | 在 MELLE 基础上加逐 token Flow Matching 精修 |
| [[CLEAR]] | 2025 | 连续 latent | AR 连续预测 | 专注低延迟，港中大/华为 |
| [[SemaVoice]] | 2026 | 语义 latent | 连续 AR + patch-wise diffusion | SFM 对齐 + diffusion head |

### 优势与局限

- **优势**: 避开 codec 设计这个"开放战场"；理论信息保真度更高；连续空间天然适配语音的渐变特性
- **局限**: 训练稳定性不如离散 token（连续值回归的方差更大）；scaling 能力是否匹配 Codec LM 路线尚无定论；缺乏公认的连续表征评测标准
- **现状**: 从 2024 年的单点探索（MELLE）发展为 2026 年的活跃方向（5+ 篇工作），但尚未出现百万小时级以上的成功案例（VoxCPM 180 万 h 是目前最大）

## 路线选择决策树

```
你的场景需要什么？
│
├─ 需要零样本说话人克隆？
│  ├─ 有 >10万h 数据 → 路线2 (Codec LM): CosyVoice/SeedTTS/IndexTTS2
│  ├─ 数据有限 (<1万h) → 路线4 (Latent Diffusion): NaturalSpeech2
│  └─ 社区/开源优先 → 路线3 (E2E): F5-TTS
│
├─ 需要流式/低延迟？
│  ├─ 首包 <200ms → 路线2 (Streaming LM): CosyVoice2/GLM-TTS
│  └─ 可接受 500ms+ → 路线4 (Diffusion): 需多步推理
│
├─ 需要多语言？
│  └─ 路线2/3: YourTTS / CosyVoice3 / Qwen3-TTS
│
└─ 需要最高质量（不考虑延迟）？
   └─ 路线4: NaturalSpeech3 / SemaVoice
```

## 路线融合趋势

2024-2026 的趋势是**路线融合**而非路线替代：

- **Codec LM + Flow Matching**（路线 2+4）: [[CosyVoice]] 系列——LM 预测语义 token，Flow Matching 生成波形
- **Codec LM + Diffusion**（路线 2+4）: [[SeedTTS]]——AR 预测 token + diffusion 精修
- **LM + 连续目标**（路线 2+5）: [[VoxCPM]]——LM 架构但预测连续值
- **连续 AR + Flow**（路线 5+4）: [[FELLE]]——连续 AR 主体 + token-wise Flow Matching 精修；[[SemaVoice]]——连续 AR + patch-wise diffusion head
- **E2E + Flow**（路线 3+4）: [[F5-TTS]]——DiT 端到端但用 Flow Matching 训练
- **LLM-native + 指令控制**: [[Qwen3-TTS]]——基于 Qwen3 LM family 做 TTS + 自然语言韵律理解（paper claim warm-start，但 GitHub 开源代码显示 talker 为 custom Qwen3-style transformer，hidden_size=1024 与 Qwen3 LLM 不匹配，literal weight warm-start 未独立 verify [§3.1 + GitHub: configuration_qwen3_tts.py]，详见 [[Qwen3-TTS]] frontmatter）；[[FlexiVoice]]——NL 指令控制风格

纯单一路线的系统越来越少，融合是主旋律。2026 年新增的趋势是 **LLM-native**（TTS 成为 LLM 的模态能力）和 **自然语言指令可控**（不再需要参考音频或显式条件）。

## 控制范式：Instruction TTS 渐进路径（Xie 2024 #2）

5 条架构路线讲"模型怎么搭"；这一节讲"用户怎么控制"——可控性维度的 taxonomy。

Xie 2024 把可控 TTS 控制策略整理为**五阶段渐进路径**：

| 阶段 | 控制策略 | 输入形式 | 代表 | 年份 |
|---|---|---|---|---|
| 1 | **Style Tagging** | 离散标签 / 连续信号 / 潜变量修改 | 早期 GST/Tacotron 系 | 2018-2020 |
| 2 | **Reference Speech Prompt** | 少量参考音频做零样本克隆 | [[VALL-E]] / [[CosyVoice]] / [[MegaTTS]] | 2023-2024 |
| 3 | **Natural Language Descriptions** | 自然语言描述属性 | PromptTTS / [[TextRolSpeech]] | 2023-2024 |
| 4 | **Instruction-Guided Synthesis** | 统一指令格式驱动 | InstructTTS / [[FlexiVoice]] / VoxInstruct / [[Qwen3-TTS]] | 2024-2026 |
| 5 | **Instruction-Guided Editing** | 指令驱动语音编辑 | Voicebox (editing) / SpeechX | 2023-2024 |

**可控维度六类**（Xie 2024 §2）：韵律 / 音色 / 情感 / 风格 / 语言 / 环境。所有五策略都可在六维中部分或全部组合。

**架构与控制策略的交叉**：

| 架构 \ 策略 | Tagging | Reference | NL Desc | Instruction | Editing |
|---|---|---|---|---|---|
| Mel 路线（路线 1） | ✓ | △ | — | — | — |
| Codec LM（路线 2） | ✓ | ✓ | ✓ | ✓ | △ |
| 端到端 Flow（路线 3） | △ | ✓ | — | — | — |
| 连续 Latent（路线 4） | △ | ✓ | ✓ | △ | ✓ |
| Tokenizer-free（路线 5） | — | ✓ | △ | △ | — |
| LLM-native | — | ✓ | ✓✓ | ✓✓ | △ |

（✓✓ 强势 / ✓ 支持 / △ 部分 / — 罕见）

**关键判断**：Codec LM 和 LLM-native 路线最容易支持完整五策略——因为 LM 序列建模天然适合「文本 + 指令 + 音频 token」的混合输入。Mel 路线和端到端 Flow 在策略 3-5 上较弱。

**Xie 2024 自述局限**：未探讨属性间耦合效应（如改 pitch 影响情感）——这是 instruction TTS 的核心未解难点。

## 对话系统中的 TTS（Ji 2024 #8）

5 条路线讲"模型架构"；本节讲"TTS 作为对话系统组件时的额外约束"——这是一个**跨路线的应用层维度**。

### 四维重定义

| 维度 | 含义 | 代表实现 |
|---|---|---|
| **低延迟流式** | 不能等全文本输入完才合成；首包 < 300ms | Freeze-Omni: NAR prefill + AR generate；[[CosyVoice2]] chunk-aware；[[GLM-TTS]] streaming |
| **可被打断** | 全双工架构下 TTS 生成随时停止并切换 | Moshi 实时监测用户流；OmniFlatten 推理无文本 |
| **状态保持** | 多轮中维持音色 / 风格一致 | Moshi 的 depth Transformer 在 token 级保持声学连贯 |
| **从文本驱动到隐状态驱动** | TTS 不再接受文本，而是 LLM 的 hidden states | LLaMA-Omni / IntrinsicVoice / PSLM |

### 全双工的底层硬约束

- **因果性硬约束**：非因果 SSL tokenizer（HuBERT / Wav2vec）需"因果蒸馏"或 chunk 化（Mimi 把 WavLM 蒸馏成因果模型）
- **流式 codec 必须**：12.5 fps 级低帧率 + 因果架构（Mimi 是当前新基准）
- **数据极度稀缺**：高质量双轨/全双工对话数据，dGSLM 仅用 Fisher 语料

### Ji 2024 的关键警示

- **端到端 ≠ 真端到端**：多数所谓端到端模型训练阶段仍重度依赖文本对齐，**推理时才省略文本**，是"训练时级联、推理时端到端"
- **全双工评估缺失**：缺乏系统化 benchmark 度量 interrupt latency / overlap handling 质量

### 与其他路线 / 章节的关系

- 在 [[TTS-SpeechLM-Dialogue关系]] 有完整 11 子类分类（级联 5 + 端到端 6）和全双工底座定义
- 在 [[TTS-核心挑战]] §挑战 8 作为独立挑战展开
- 与 §对话系统中的 codec 选型 相关——非因果 SSL 不能用，必须因果 codec

---

*最后更新: 2026-05-26（基于 11 篇综述综合，新增 Mousavi 五维 / Zhang 三阶段 / Xie 五策略 / Ji 四维重定义）*
