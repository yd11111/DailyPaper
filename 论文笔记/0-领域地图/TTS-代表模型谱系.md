---
title: TTS 代表模型谱系
type: 领域地图
domain: TTS
tags: [genealogy, models, tts, timeline]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 代表模型谱系

## 阅读说明

本文按**主技术血缘 + 设计范式亲缘**组织 TTS 代表模型，不是严格的单继承树。

现代 TTS 系统几乎都是混合架构（如 Codec LM + Flow Matching），强行归入单一家族会扭曲真实技术关系。因此：
- 全景表为每个模型标注**所有相关路线**，不做排他归类
- 演化脉络按**时间阶段**叙述，强调"为什么上一代不够用、这一代解决了什么"
- 谱系分支只收录**有明确直接继承关系**的模型链，不把"同路线"等同于"同谱系"

## 全景表

> 路线标注：M = Mel 中介, E = 端到端, C = Codec LM, D = Diffusion/Flow, V = 声码器, L = LLM-native, K = 可控性/指令

| 模型 | 年份 | 路线 | 中间表征 | 生成方式 | 零样本 | 一句话定位 |
|------|------|------|---------|---------|--------|-----------|
| [[WaveNet]] | 2016 | V | — | AR 波形（逐样本点） | — | 首个神经声码器 |
| [[TransformerTTS]] | 2018 | M | mel | AR | — | Transformer 替代 RNN 做声学模型 |
| [[FastSpeech]] | 2019 | M | mel | NAR | — | Length Regulator 实现并行生成 |
| [[FastSpeech2]] | 2020 | M | mel | NAR | — | 显式 duration/pitch/energy 预测 |
| [[VITS]] | 2021 | E | VAE latent | VAE+Flow+GAN | — | 首个高质量端到端 TTS |
| [[NaturalSpeech]] | 2022 | E | VAE latent | VAE+Flow | — | 在 LJSpeech 上达到接近人类的 MOS（作者报告） |
| [[YourTTS]] | 2022 | E | VAE latent | VAE+Flow | ✓ | VITS 多语言 + 零样本扩展 |
| [[AudioLM]] | 2022 | C | semantic+acoustic token | AR | — | 语音 token LM 概念验证（非 TTS 专用） |
| [[Tortoise]] | 2023 | C | mel token | AR+DDPM | ✓ | 社区先驱，质量高但推理慢 |
| [[VALL-E]] | 2023 | C | EnCodec RVQ 8层 | AR(层1)+NAR(层2-8) | ✓ | 开创 Codec LM TTS 范式 |
| [[SPEAR-TTS]] | 2023 | C | 语义→声学 token | 两阶段 AR | ✓ | 语义-声学分阶段，探索 text-only 预训练 |
| [[VALL-E-X]] | 2023 | C | EnCodec RVQ | AR+NAR | ✓ | VALL-E 跨语言扩展 |
| [[NaturalSpeech2]] | 2023 | D | 连续 latent（自训练 VAE） | Diffusion | ✓ | 连续 latent + diffusion，大规模零样本 |
| [[MegaTTS]] | 2023 | M | mel（解耦 content/prosody/timbre） | AR+GAN+Diffusion | ✓ | 三路解耦，mel 路线的精细化巅峰 |
| [[MegaTTS2]] | 2023 | M | mel（解耦） | PLM+Diffusion | ✓ | 音素级韵律 latent + Prosody LM |
| [[HierSpeech++]] | 2023 | E | 层次化 latent | VAE | ✓ | 语义/声学分层 VAE |
| [[BaseTTS]] | 2024 | C | 离散 token | AR | ✓ | 10万h 预训练，观察到韵律涌现 |
| [[NaturalSpeech3]] | 2024 | D | FACodec 4路连续 latent | Flow Matching | ✓ | 分解为 content/prosody/timbre/detail |
| [[SeedTTS]] | 2024 | C+D | 离散 token | AR+Diffusion+RL | ✓ | 自蒸馏 + RL 后训练 |
| [[CosyVoice]] | 2024 | C+D | 语义 token | LLM→Flow Matching | ✓ | LLM 预测语义 token + Flow 生成波形 |
| [[GPT-SoVITS]] | 2024 | C+E | HuBERT token | GPT→VITS | ✓ | GPT 语义预测 + VITS 声学，开源低门槛 |
| [[F5-TTS]] | 2024 | D | — | DiT Flow Matching | ✓ | 无需音素对齐、无需时长模型 |
| [[FireRedTTS]] | 2024 | C | 图像 VQ token | AR | ✓ | 图像 tokenizer 跨模态用于语音 |
| [[XTTS]] | 2024 | C | GPT token | AR+声码器 | ✓ | Coqui 多语言开源（技术混合度高） |
| [[Fish-Speech]] | 2024 | C | 自训练 token | AR | ✓ | 70万h 社区开源 |
| [[CosyVoice2]] | 2024 | C+D | FSQ token | LLM→Flow Matching | ✓ | FSQ 替代 RVQ + chunk-aware 流式 |
| [[MaskGCT]] | 2025 | C | codec latent | Masked Generative（NAR） | ✓ | 非自回归 masked prediction 生成 token |
| [[SemaVoice]] | 2025 | D | 语义 latent | Flow Matching | ✓ | 语义 latent 为主 + shallow diffusion |
| [[IndexTTS2]] | 2025 | C | 自训练 token | 100层 AR + BigVGAN-v2 | ✓ | 极深 LM + 高质量声码器 |
| [[GLM-TTS]] | 2025 | C+D | 自训练 token | Streaming LM→Flow | ✓ | 流式 chunk 级 LM 解码 |
| [[FireRedTTS2]] | 2025 | C | 图像 VQ token | AR | ✓ | FireRedTTS 改进版 |
| [[CosyVoice3]] | 2025 | C+D | FSQ token | LLM→Flow Matching | ✓ | 100万h 数据规模 |
| [[Qwen3-TTS]] | 2025 | L+K | multi-codebook token | LLM 直出 | ✓ | 通用 LLM 直接做 TTS，支持自然语言韵律指令 |
| [[VoxCPM]] | 2025 | L | 连续 mel 帧 | LLM AR 连续预测 | ✓ | 无 tokenizer，LLM 直接预测连续值 |

## 严格谱系分支

以下只收录**有明确直接继承关系**（同团队迭代、论文显式声明"基于 X"）的模型链。不把"同路线但无直接血缘"的模型硬拉在一起。

### 分支 1: Tacotron → FastSpeech 线

```
Tacotron (2017) → Tacotron 2 (2018)
                       ↓ 并行
               TransformerTTS (2018) — 用 Transformer 替代 RNN，平行分支
               FastSpeech (2019) — 非自回归，Length Regulator
                       ↓
               FastSpeech2 (2020) — 显式 variance adaptor
                       ↓
               AdaSpeech 系列 (2021-22) — 自适应微调
```

**解决的核心问题**: 从 AR 的慢+不稳定到 NAR 的快+鲁棒
**现状**: 架构思想（variance adaptor、duration predictor）仍广泛被后续系统借鉴，但纯 mel 生成路线已不是 2024+ 新系统的首选

### 分支 2: VITS 线

```
VITS (2021) — VAE + Normalizing Flow + HiFi-GAN 端到端
    ↓
YourTTS (2022) — 多语言 + 零样本扩展（同团队 Coqui）
    ↓
HierSpeech++ (2023) — 层次化 VAE（非同团队，但显式基于 VITS 架构）
```

**解决的核心问题**: 消除 mel→wav 级联误差
**注意**: [[GPT-SoVITS]] 的后半段（VITS decoder）借鉴了 VITS，但前半段（GPT 语义预测）属于 Codec LM 思路，是跨分支混合体

### 分支 3: NaturalSpeech 线

```
NaturalSpeech (2022) — VAE + Flow，单说话人极致优化
    ↓
NaturalSpeech2 (2023) — 转向连续 latent + diffusion，大规模零样本
    ↓
NaturalSpeech3 (2024) — FACodec 四路 latent + Flow Matching
```

**解决的核心问题**: 从单说话人高质量 → 大规模零样本 → 细粒度解耦
**注意**: NaturalSpeech 1 更接近端到端路线（VAE+Flow），NaturalSpeech 2/3 转向连续 latent + 生成模型路线。虽同团队同系列，但技术范式有明确跃迁

### 分支 4: VALL-E 线

```
VALL-E (2023) — 开创 Codec LM TTS
    ├── VALL-E-X (2023) — 跨语言扩展（同团队）
    └── VALL-E 2 (2024) — 改进采样（同团队）
```

**解决的核心问题**: 证明 LM 可以做零样本 TTS

### 分支 5: CosyVoice 线

```
CosyVoice (2024) — LLM 语义 token + Flow Matching 波形
    ↓
CosyVoice2 (2024) — FSQ 替代 RVQ + 流式
    ↓
CosyVoice3 (2025) — 100万h 数据 scaling
```

**解决的核心问题**: Codec LM + Flow Matching 的工业化落地 → 流式 → 数据规模
**技术决策演进**: RVQ → FSQ（CosyVoice2），说明团队认为量化方式是系统瓶颈之一

### 分支 6: FireRedTTS 线

```
FireRedTTS (2024) — 图像 VQ tokenizer 跨模态
    ↓
FireRedTTS2 (2025) — 改进版
```

### 分支 7: MegaTTS 线

```
MegaTTS (2023) — 内容/韵律/音色三路解耦
    ↓
MegaTTS2 (2023) — 音素级韵律 latent + Prosody LM
```

**解决的核心问题**: mel 路线下如何做细粒度韵律建模

### 分支 8: 声码器线

```
WaveNet (2016) — 自回归，逐样本点，开山之作（太慢不实用）
    ↓ 加速
WaveRNN (2018) — 轻量化 RNN
WaveGlow (2018) — Flow-based 并行
MelGAN (2019) — 首个 GAN 声码器
    ↓ 质量+速度平衡
HiFi-GAN (2020) — 多尺度 GAN（目前多数系统的主流选择）
    ↓ 大规模化
BigVGAN (2023) → BigVGAN-v2 (2024)
```

**现状**: HiFi-GAN/BigVGAN 是当前多数高质量 TTS 系统的主流声码器选择。声码器的独立创新已放缓，更多以组件形式被其他系统直接采用或微调。

## 独立节点（无明确谱系归属）

以下模型在技术上有重要贡献，但与上述分支没有明确的直接继承关系：

| 模型 | 最接近的路线 | 为什么单独列 |
|------|------------|------------|
| [[AudioLM]] | Codec LM 概念源头 | 非 TTS 专用，是语音 token LM 的概念验证 |
| [[SPEAR-TTS]] | Codec LM | Google 独立探索，与 VALL-E 并行而非继承 |
| [[Tortoise]] | Codec LM | 社区先驱，技术路线独特（mel token + DDPM） |
| [[SeedTTS]] | Codec LM + Diffusion | 字节独立系统，借鉴 VALL-E 思路但无直接代码/架构继承 |
| [[F5-TTS]] | Diffusion/Flow | DiT Flow Matching 端到端，独立设计 |
| [[MaskGCT]] | Codec token 生成 | 非自回归 masked prediction，与 AR Codec LM 路线不同 |
| [[SemaVoice]] | Diffusion/Flow | 语义 latent + shallow diffusion，独立设计 |
| [[BaseTTS]] | Codec LM | Amazon 大规模预训练，方法论独立 |
| [[XTTS]] | 混合（GPT + 声码器） | Coqui 工业开源，技术混合度高 |
| [[Fish-Speech]] | Codec LM | 社区开源，独立实现 |
| [[Qwen3-TTS]] | LLM-native | 与 Codec LM 有亲缘但属于新范式（通用 LLM 直出） |
| [[VoxCPM]] | LLM-native / Tokenizer-free | 全新路线，无直接先驱 |

## 演化脉络

### 第一阶段: Mel + 声码器（2016-2021）

**核心思路**: 文本 → mel → 波形，两个独立模型
**解决的核心问题**: 让神经网络生成的语音达到可用质量

- [[WaveNet]]（2016）解决了声码器问题
- Tacotron 2（2018）+ [[TransformerTTS]]（2018）解决了声学模型问题
- [[FastSpeech]]（2019）→ [[FastSpeech2]]（2020）解决了 AR 的速度和鲁棒性问题
- 声码器从 WaveNet → HiFi-GAN（2020），推理速度提升 ~1000x

**留下的问题**: 两阶段级联有误差传播；mel 空间表达力有限；难以做零样本

### 第二阶段: 端到端 + 高质量生成（2021-2023）

**核心思路**: 消除中间表征瓶颈
**解决的核心问题**: 级联误差 + 生成质量天花板

两条并行路线：
- **VAE + Flow 端到端**: [[VITS]]（2021）→ [[YourTTS]]（2022）→ [[NaturalSpeech]]（2022，在作者报告的 LJSpeech 设置下达到接近人类的 MOS）
- **连续 latent + Diffusion**: [[NaturalSpeech2]]（2023）证明在连续 latent 空间做 diffusion 可以实现大规模零样本

**留下的问题**: VAE 容量有限难以 scale；diffusion 推理慢；缺少统一的大规模预训练框架

### 第三阶段: Codec Language Model（2023-至今，当前主流）

**核心思路**: 语音 → 离散 token，复用 LLM 做序列建模
**解决的核心问题**: scaling + 零样本 in-context learning

[[AudioLM]]（2022）验证概念 → [[VALL-E]]（2023）正式开创 TTS 范式 → 此后大多数新系统都采用 Codec LM 或其变体。

Codec LM 内部分化为三个子方向：

| 子方向 | 代表 | 思路 | 现状 |
|--------|------|------|------|
| **纯 AR token** | [[VALL-E]]、[[IndexTTS2]] | LM 预测全部 codec 层 token | 架构简单但序列长 |
| **LM + 连续解码器** | [[CosyVoice]] 系列、[[GLM-TTS]]、[[SeedTTS]] | LM 只预测语义 token，Flow/Diffusion 做波形 | 2024-25 最主流 |
| **非自回归 token 生成** | [[MaskGCT]] | masked prediction 并行生成 | 推理快，质量在追赶 AR |

**留下的问题**: 离散化仍是信息瓶颈；codec 设计缺乏共识；序列长度与音质的 tradeoff

### 第四阶段: LLM-native / Tokenizer-free（2025-萌芽中）

**核心思路**: 不再为 TTS 训练专门模型，让通用 LLM 直接输出语音
**试图解决的核心问题**: TTS 与 LLM 的统一

两条探索路线正在同时推进：
- **LLM 直出离散 token**: [[Qwen3-TTS]] 用 Qwen3 LLM 直接预测 multi-codebook speech token
- **LLM 直出连续值**: [[VoxCPM]] 跳过 tokenizer，LLM 自回归预测连续 mel 帧

**尚未验证**: 通用 LLM 做 TTS 的质量能否真正匹配专用系统——[[Qwen3-TTS]] 的初步结果积极，但尚需更多独立评测确认

## 跨路线技术维度

以下技术不是独立路线，而是可以嫁接到任何路线上的能力维度。

### 解耦表征（Disentanglement）

将语音信号分解为独立因子（内容/韵律/音色/细节），各因子独立建模和控制。

| 模型 | 所属路线 | 解耦方式 | 因子 |
|------|---------|---------|------|
| [[MegaTTS]] | Mel | 三路 encoder | content / prosody / timbre |
| [[MegaTTS2]] | Mel | 音素级 latent + PLM | content / prosody(细粒度) / timbre |
| [[NaturalSpeech3]] | Diffusion/Flow | FACodec 四路 | content / prosody / timbre / detail |
| [[CosyVoice]] | Codec LM + Flow | 隐式分层 | 语义 token ≈ content, Flow ≈ 声学细节 |

### 可控性与指令 TTS（Controllable / Instruction TTS）

从"给什么说什么"到"按指令说"的能力升级。这是 2024-2025 快速增长的维度。

| 控制粒度 | 代表 | 方式 |
|---------|------|------|
| 全局情感 | [[EmotionThinker]] | 情感 embedding 条件 |
| duration/pitch/energy | [[FastSpeech2]] | 显式 variance adaptor |
| 风格参考 | [[MegaTTS]]、[[CosyVoice]] | 参考音频 → speaker/style embedding |
| 自然语言韵律指令 | [[Qwen3-TTS]] | LLM 理解文本语义 → 自动推断韵律 |
| 语音理解+生成统一 | [[StepAudio2.5]]、[[Moshi]] | 统一 Speech LLM |

**为什么重要**: 很多近期系统的核心贡献不是"换了生成器"，而是**控制范式升级**——从显式条件（给定 F0 曲线）到自然语言指令（"用温柔的语气说"）。这个维度在本笔记库中的覆盖尚不充分，后续需要补充 StyleTTS、PromptTTS、InstructTTS 等工作。

### 评测范式演进

TTS 的评测逻辑也在随范式一起变化：

| 阶段 | 时间 | 主流评测方式 | 代表 |
|------|------|------------|------|
| MOS 主导 | ~2020 | 人工 5 分制打分 | Tacotron 2、FastSpeech 论文 |
| 客观代理指标 | 2020-23 | WER + SIM-O + UTMOS | [[VALL-E]]、[[NaturalSpeech2]] |
| 标准化 benchmark | 2024- | Seed-TTS-eval、[[Emilia]]、[[TTSDS2]] | [[CosyVoice2]]、[[IndexTTS2]] |
| 指令跟随评测 | 2025-萌芽 | 可控性/情感一致性/指令遵循率 | [[Qwen3-TTS]]、[[StepAudio2.5]] |

详见 [[TTS-评测体系]]

## 工业系统速查

工业系统通常是多条路线的混合体，不服从单一学术谱系：

| 系统 | 机构 | 技术组合 | 训练数据 | 核心差异点 |
|------|------|---------|---------|-----------|
| [[CosyVoice3]] | 阿里通义 | LLM + FSQ + Flow Matching | 100万h | 数据规模 + FSQ 量化 |
| [[Qwen3-TTS]] | 阿里通义 | Qwen3 LLM 直出 token | 500万h | LLM-native，不额外训练声学模型 |
| [[SeedTTS]] | 字节 | AR LM + Diffusion + RL | 未公开 | 自蒸馏 + RL 后训练 |
| [[GLM-TTS]] | 智谱 AI | Streaming LM + Flow Matching | 10万h（严格筛选） | 流式 chunk 级解码 |
| [[VoxCPM]] | 面壁智能 | LLM 连续 mel 预测 | 180万h | 无 tokenizer |
| [[StepAudio2.5]] | 阶跃星辰 | 统一 Speech LLM | 未公开 | 理解+生成统一 |
| [[IndexTTS2]] | Bilibili | 100层 GPT + BigVGAN-v2 | 5.5万h | 极深 LM |
| [[FireRedTTS]] | 小红书 | 图像 VQ + AR LM | 未公开 | 跨模态 tokenizer |

## 未覆盖 / 待补充

本文档目前存在以下已知缺口：

- **Controllable / Instruction TTS 专题**: StyleTTS、PromptTTS、InstructTTS、ParlerTTS 等需要专题展开
- **Speech Editing 路线**: Voicebox、FluentSpeech、SpeechX 等编辑类系统未纳入
- **非中国团队的 2024-25 工作**: Google（SoundStorm 后续）、Meta（Voicebox 后续）的近期进展覆盖不足
- **商业闭源系统**: ElevenLabs、OpenAI TTS、Azure TTS 等仅有有限公开信息，本文未收录
- **多模态 TTS**: 视觉驱动的 talking head + TTS 联合系统
- **音乐/歌唱合成**: SVS (Singing Voice Synthesis) 作为 TTS 近亲，当前未覆盖

## 如何使用本谱系

1. **定位新论文**: 读到一篇新 TTS 论文，先在全景表中找它最接近哪些模型，判断它在解决哪个阶段留下的问题
2. **理解技术演进**: 顺着谱系分支看"从 A 到 B 改了什么、为什么"
3. **找交叉灵感**: 看不同路线是否有可组合的思路（如 VITS 的端到端 + Codec LM 的 scaling → GPT-SoVITS）
4. **写 related work**: 按谱系分支组织比按时间线更清晰
5. **判断新工作的 novelty**: 如果一篇论文的"创新"其实是另一条分支早已解决的问题，谱系图能帮你快速发现

---

*最后更新: 2026-05-25*
