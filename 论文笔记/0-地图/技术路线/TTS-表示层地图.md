---
title: TTS 表示层地图
type: 技术路线
domain: TTS
tags: [representation, codec, tokenizer, tts]
created: 2026-05-26
last_updated: 2026-05-26
based_on: [Mousavi-2025-arXiv:2506.10274, Zhang-2023-arXiv:2303.13336, XuTan-2021-arXiv:2106.15561, Cui-2024-arXiv:2410.03751]
---

# TTS 表示层地图

> ⚠️ **未 verify 警告**（2026-05-26 添加）：本笔记 2026-05-26 新建，大部分内容来自 11 篇综述（特别是 Mousavi 2025）的章节摘录 + 部分基于已有论文笔记的归纳。**结构层（五维 taxonomy / 三阶段引入 / 决策树）保留**，**具体 cell（"X codec 用 RVQ Y 层"、"Z 模型选 mixed-token"等）的技术细节需要逐 cell 按 §11 三层 verify**——尤其是涉及具体 codec 内部细节（层数 / 码本大小 / 帧率）和模型选择的，论文常常不写清楚必须查 GitHub 源码。详见 [[方法论复盘-2026-05-26-知识地图建设]]。

---

## 0. 为什么需要这张地图

所有 TTS 模型本质上都在做同一件事：**在不同表示空间之间做映射**。`Text → ? → Waveform` 中间的「?」是什么，决定了模型的几乎所有设计权衡——LM scaling 友好性、信息保真度、流式可行性、训练稳定性。

[[TTS-技术路线图]] 把模型按"中间表征"切成 5 条路线，但**没有把表示层本身作为独立研究对象展开**。而 2023+ 的 TTS 差异化竞争点已经从"模型架构"大幅迁移到"表示选择" —— 同样的 LM backbone 用不同 codec 训练，效果可以差出一个量级。

本笔记从表示层视角重组 TTS 知识。

## 1. 表示空间全景

```
TTS 表示空间
├── 连续表示
│   ├── Mel / Linear Spectrogram         (经典中介，~80 维 / 帧，10ms hop)
│   ├── 自训练 VAE Latent                (NaturalSpeech 2/3 路线，紧凑可控)
│   └── 连续 mel/帧 直接预测              (Tokenizer-free 路线：MELLE/FELLE/VoxCPM)
└── 离散表示
    ├── 语义 Token                        (HuBERT/Wav2vec/WavLM/USM，~25-50 fps)
    ├── 声学 Token                        (EnCodec/SoundStream/DAC，多层 RVQ)
    └── 混合 Token                        (SpeechTokenizer/Mimi/X-Codec，语义+声学融合)
```

**Survey claim — Mousavi 2025 (#7)** 明确反对传统"语义 vs 声学"二分："Acoustic tokenizers can capture semantic information... while semantic tokenizers have been effectively used in generative tasks"，故提出五维分类（见 §2）。

## 2. Codec/Tokenizer 五维 Taxonomy（来源：Mousavi 2025 §2-3）

这是表示层最完整的分类框架，**取代了过去单一的"层数 / 比特率"二维比较**。

### 维度 1：编码器-解码器架构

| 架构 | 代表 | 说明 |
|---|---|---|
| CNN | SoundStream / DAC | 经典选择，因果版可流式 |
| CNN + RNN | EnCodec / Language-Codec / FACodec | 兼顾时频域 |
| Transformer | （较少） | 全注意力 |
| CNN + Transformer | WavTokenizer / Mimi / SemantiCodec | 2024+ 主流 |

### 维度 2：量化方法

| 量化方式 | 全称 | 特点 | 代表 |
|---|---|---|---|
| K-means | — | 简单，无端到端梯度 | GSLM 早期 |
| **RVQ** | Residual Vector Quantization | 多层残差累积，主流但序列长 | SoundStream / EnCodec / DAC |
| SVQ | Single Vector Quantization | 单码本大词表 | WavTokenizer |
| GVQ | Grouped VQ | 分组并行 | — |
| **FSQ** | Finite Scalar Quantization | 标量量化，码本利用率高 | CosyVoice2/3 |
| MSRVQ | Multi-Scale RVQ | 不同层不同帧率 | SNAC |
| CSRVQ | Cross-Scale RVQ | — | — |
| PQ | Product Quantization | 子空间分解 | — |

**我的归纳**：2024-2026 的真实战场在 RVQ vs FSQ vs SVQ 三选一 —— RVQ 是默认基线，FSQ 是 CosyVoice 系列推动的工业选择，SVQ 是简化 LM 训练的实验方向。

### 维度 3：训练范式

| 训练范式 | 说明 | 代表 |
|---|---|---|
| 分离式（Post-Training） | codec 独立训练，再喂给 TTS | 多数早期工作 |
| 联合式（End-to-End） | codec 与下游 TTS 联合训练 | 较少，VAE 端到端如 VITS |
| **目标函数组合** | Recon / VQ / GAN / Feat / Diff / MP | SoundStream 用 GAN+Recon，DAC 加 mel loss |

### 维度 4：流式能力

| 维度 | 选项 |
|---|---|
| 因果卷积 | 是 / 否（非因果如 SSL tokenizer 不能流式） |
| 因果注意力 | 是 / 否 |
| 算法延迟 | chunk 大小决定首包延迟 |
| 计算复杂度 | 帧率 × 模型参数 |

**关键数据 — Mousavi 2025**：Mimi 实现因果 Transformer + **12.5 fps** 帧率，是当前流式 codec 的新基准。EnCodec 默认 75 fps，DAC 默认 50 fps。

### 维度 5：目标领域

| 领域 | 代表 |
|---|---|
| 单域 - Speech | SpeechTokenizer / Mimi / FACodec / Language-Codec |
| 单域 - Music | — |
| 单域 - Audio | — |
| **多域** | EnCodec / DAC / SoundStream / WavTokenizer / X-Codec / SemantiCodec / SQ-Codec |

### 辅助维度（Mousavi 2025 §2.3 单列）

- **解耦（Disentanglement）**：FACodec 显式分解 content / prosody / timbre / detail
- **语义蒸馏（Semantic Distillation）**：SpeechTokenizer / Mimi 首层用 SSL 模型蒸馏
- **有监督语义标记化**：少数工作用 ASR 监督

## 3. 扩散模型对表示空间的影响（来源：Zhang 2023 #4）

Zhang 2023 (#4) 提供扩散在 TTS 中的**三阶段引入分类**，正是从"目标表示空间是什么"切入：

| 引入阶段 | 目标空间 | 扩散在做什么 | 代表 |
|---|---|---|---|
| **声学模型阶段** | mel 或 latent | 生成 mel/latent，再交给独立声码器 | Grad-TTS / Diff-TTS / ProDiff / DiffGAN-TTS / NaturalSpeech 2 / EmoDiff |
| **声码器阶段** | 波形 | 从 mel 直接生成波形 | WaveGrad / DiffWave / BDDM / PriorGrad / SpecGrad |
| **端到端阶段** | 波形 | 文本直接到波形，无显式中介 | WaveGrad 2 / CRASH / FastDiff / Itôn |

**我的归纳**：这三个阶段对应了**扩散过程"侵入"TTS 系统的不同深度**。引入越深，表示空间问题越被回避（端到端阶段几乎不需要 codec）；但训练越难、推理越慢。

**Zhang 2023 局限**（2023-03 截止）：未覆盖 Flow Matching（Voicebox / Matcha-TTS / F5-TTS / CosyVoice 系列）、NaturalSpeech 2/3 完整版、DiffSinger 等。但**三阶段分类框架本身仍有效**，可以把后续 Flow Matching 模型套进来——例如 F5-TTS 属于"端到端阶段"，CosyVoice 系列的 Flow Matching 部分属于"声学模型阶段"（在 LM 输出的 token 上做）。

## 4. 表示空间的选择决定了什么

### 4.1 LM scaling 友好性

| 表示 | 是否友好 | 原因 |
|---|---|---|
| 离散 token（语义/声学/混合） | ✓ 高 | 直接复用 LLM cross-entropy，无修改 |
| 连续 mel/latent | △ 中 | 需要 latent sampling module（MELLE）或 token-wise flow matching（FELLE）或 diffusion loss |
| 波形 | ✗ 低 | 维度过高（16kHz 即 16000/s），不可行 |

**Cui 2024 (#3) 的范式**：把 TTS 重新定义为 SpeechLM 输出端的 "Token-to-Speech Vocoder" —— 这个范式只在**离散 token**表示下成立。Tokenizer-free 路线（路线 5）是对此范式的反叛。

### 4.2 信息保真度（上限）

```
波形（原始）
   > 高保真 mel（128+ bins, 24/48kHz）
       > 标准 mel（80 bins, 22.05kHz）
           > 多层 RVQ codec（8 层）
               > FSQ codec（4-6 层）
                   > 混合 token（语义+声学）
                       > 语义 token（HuBERT 单层）
```

**Mousavi 2025 关键洞察**：**codec 重建上界 = TTS LM 输出还原后的音质天花板**。即使 LM 预测完美，decoder 无法恢复的细节就是不可逆损失。这给出了 codec 选型的硬约束 —— 不能用语义 token 做最高保真度 TTS。

### 4.3 流式可行性

**因果性是硬约束**：
- ✗ HuBERT / Wav2vec / WavLM 等 SSL tokenizer 默认非因果，**不能直接做流式 TTS**
- ✗ EnCodec 默认非因果（但有因果版本变种）
- ✓ Mimi（12.5 fps 因果 Transformer）
- ✓ CosyVoice2 的 FSQ codec 改造为 chunk-aware causal

**Cui 2024 / Ji 2024 提醒**：全双工对话系统 → 必须流式 → SSL-based tokenizer 需要"因果蒸馏"或"chunk 化"改造（如 Mimi 把 WavLM 蒸馏成因果模型）。

### 4.4 训练稳定性

| 表示 | 训练稳定性 | 主要风险 |
|---|---|---|
| 离散 token | 高 | 量化损失需要 EMA / commitment loss 调参 |
| 连续 mel/latent AR | 中 | 连续值回归方差大，易发散 |
| 连续 latent + diffusion | 中高 | diffusion 训练相对稳定，但步数多 |
| 波形直接预测 | 低 | 高维空间，几乎不可行 |

## 5. 跨表示的桥接技术

### 5.1 语义-声学混合 token

| 模型 | 做法 |
|---|---|
| **SpeechTokenizer** | 首层 RVQ 经 HuBERT 蒸馏（语义），后层捕获声学细节 |
| **Mimi** | 流式因果版本，12.5 fps，语义蒸馏 |
| **X-Codec** | 双编码器融合 SSL 语义特征与声学特征 |
| **SemantiCodec** | CNN+T 编码器配扩散解码器，语义蒸馏 |
| **FACodec** | 解耦内容 / 韵律 / 音色 / 声学细节为独立子空间（4 路 latent） |

**为什么混合是趋势**：LM 适合预测语义（首层），声学细节交给 decoder（后层 / vocoder）。这就是 [[CosyVoice]] 系列"LLM + Flow Matching"的本质——LLM 预测语义 token，Flow Matching 补齐声学。

### 5.2 Tokenizer-free 路线（绕过表示层问题）

| 模型 | 做法 |
|---|---|
| [[MELLE]] | AR + latent sampling module 预测连续 mel |
| [[FELLE]] | MELLE + token-wise coarse-to-fine flow matching 精修 |
| [[VoxCPM]] | LLM 直接 AR 预测连续 mel 帧 |
| [[CLEAR]] | 连续 latent AR，专注低延迟 |
| [[SemaVoice]] | SFM 对齐 + patch-wise diffusion head |

**核心理由**：codec 设计本身是开放战场（Mousavi 2025 的五维就有几十种组合），既然没共识，不如绕过。

**Mousavi 2025 警示**：Tokenizer-free 路线在 §4 的 benchmark 中**未被覆盖**（benchmark 默认对象是 tokenizer），其相对优势尚无系统化对比数据。

### 5.3 跨域表示的可迁移性

| 方向 | 现状 |
|---|---|
| Speech codec → Music | EnCodec / DAC / WavTokenizer 多域设计，可直接用 |
| Speech codec → Audio | 同上 |
| Music codec → Speech | 较少尝试 |

**Mousavi 2025**：跨域 codec 在 speech 任务上通常**略弱于 speech-only codec**，但差距在缩小。

## 6. 评估表示质量的方法

### 6.1 重建质量（codec 本身）

| 指标 | 测什么 | 范围 |
|---|---|---|
| PESQ | 主观感知质量预测 | 1-4.5 |
| STOI | 短时客观可懂度 | 0-1 |
| ViSQOL | 视觉相似度感知 | 1-5 |
| Mel-distance | mel 频谱距离 | 越低越好 |
| SI-SDR | 尺度不变信噪比 | dB |

### 6.2 下游 LM 任务（codec 对 TTS 的影响）

**统一 benchmark**（Mousavi 2025 提供）：
- **Codec-SUPERB** / **VERSA** — 重建质量跨域评估
- **DASB** — 下游判别 + 生成任务（含 TTS / ASR / 增强 / 源分离）
- **SALMon** — 声学语言建模质量
- **Zero-resource** — 零资源语音理解

### 6.3 对 TTS 的特定影响

**Mousavi 2025 §3.3.2 设置 TTS 评估通道**：固定 LM 架构，换不同 tokenizer，观察 WER / SIM / MOS。结论是**语义蒸馏型 tokenizer（SpeechTokenizer / Mimi / X-Codec）通常更优** —— 首层带语义信息让 LM 更易学习。

**未解决**：LM 架构是单一配置（不覆盖 Flow Matching 解码、多说话人场景），故"哪种 codec 最适合 TTS"仍无共识。

## 7. 表示选型决策树

```
你的场景是什么？
│
├─ 工业大规模 + 兼顾 LLM-native?
│  → 多 codebook FSQ + 流式（CosyVoice2/3 路线）
│  → 或 LLM 直出 multi-codebook（Qwen3-TTS）
│
├─ 学术研究 zero-shot + 重建质量?
│  → 语义蒸馏混合 token（SpeechTokenizer / Mimi / X-Codec）
│
├─ 实时对话系统（首包 < 300ms）?
│  → 因果架构 + 低帧率（Mimi 12.5fps）
│  → 避开非因果 SSL tokenizer
│
├─ 想绕过 codec 设计问题?
│  → Tokenizer-free 路线（MELLE/FELLE/VoxCPM/CLEAR/SemaVoice）
│  → 接受连续 AR 的训练稳定性挑战
│
├─ 跨域统一（speech + music + audio）?
│  → 多域 codec（EnCodec/DAC/WavTokenizer/X-Codec）
│  → 接受 speech 任务上略弱的损失
│
└─ 最高质量（不考虑延迟）?
   → 高保真 mel + Flow Matching 解码（NaturalSpeech 3 / F5-TTS）
   → 或连续 AR + patch-wise diffusion head（SemaVoice）
```

## 8. 表示层 vs 模型架构：哪个更决定 TTS 质量？

**我的归纳**（基于 11 篇综述综合）：

- **2020-2022**：模型架构主导（FastSpeech vs Tacotron 的 NAR vs AR 之争）
- **2023-2024**：表示层崛起（VALL-E 的成功一半归功于 EnCodec；CosyVoice 路线 RVQ→FSQ 的 v1→v2 跃迁；NaturalSpeech 3 的 FACodec 解耦）
- **2025-2026**：两者并重，但**表示层成为可识别的独立设计维度** —— ICLR 2026 涌现 FlexiCodec（动态帧率）、StableToken（噪声鲁棒）、ScalingSpeechTokenizers（diffusion autoencoder 替代 VQ）等专门探索

**判断依据**：
- 工业系统（CosyVoice / Qwen3-TTS / VoxCPM）的版本迭代中，**表示层变更**比 LM 架构变更更频繁
- Tokenizer-free 路线的出现本身就是"表示层定义了上限"的反向证明
- Mousavi 2025 之所以能成为重要综述，说明社区已经把表示层独立成研究对象

## 9. 与其他笔记的关系

- [[TTS-技术路线图]] — 模型架构视角的 5 条路线（本笔记是表示层视角的补充）
- [[TTS-核心挑战]] §挑战 5 Codec/Token 设计 — 把表示层作为 7 大挑战之一
- [[TTS-评测体系]] — 评测指标（PESQ/STOI 等可作为表示层质量代理）
- [[TTS-11篇综述综合-2026-05]] — 本笔记的来源依据 §3.4

## 10. 主要来源

| 综述 | 贡献到本笔记的哪节 |
|---|---|
| [[ControllableTTS-Survey]] (Xie 2024, arXiv:2412.06602) | §1 表示空间分类的部分语义 |
| Mousavi 2025 (arXiv:2506.10274) | §2 五维 taxonomy 完整复刻 + §5 + §6 |
| Zhang 2023 (arXiv:2303.13336) | §3 扩散三阶段引入分类 |
| Cui 2024 (arXiv:2410.03751) | §1 离散表示分类 + §4.1 SpeechLM 范式 |
| Xu Tan 2021 (arXiv:2106.15561) | §1 连续表示历史背景（mel/VAE latent） |

---

*2026-05-26 — 用 11 篇综述构建，对应分工表 §3.4*
