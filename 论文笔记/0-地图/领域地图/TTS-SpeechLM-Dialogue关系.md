---
title: TTS ↔ SpeechLM / Dialogue 关系图
type: 领域地图
domain: TTS
tags: [speechlm, dialogue, omni, audio-llm, full-duplex, tts]
created: 2026-05-26
last_updated: 2026-05-26
based_on: [Cui-2024-arXiv:2410.03751, Yang-2025-arXiv:2502.19548, Ji-2024-arXiv:2411.13577, AudioLM-2025-arXiv:2501.15177]
---

# TTS ↔ SpeechLM / Dialogue 关系图

> ⚠️ **未 verify 警告**（2026-05-26 添加）：本笔记 2026-05-26 新建，**主要基于 Cui 2024 / Yang 2025 / Ji 2024 / Audio-LM #14 四篇综述的 HTML 章节摘录综合**。具体 cell（"X 模型属于 Cui 框架的位置 ②"、"Y 模型在 Ji 11 子类中归为 ZZ"等）的归类是**基于综述对该模型的描述**做的二阶判断，**未按新工作流 §11 三层 verify**——大量"X 是 Y 类"的归类应该回到该模型论文 §X / GitHub 重新核实。**结构层（4 视角 + 4 位置 + 关系图）保留**，**具体模型归类需要 verify**。详见 [[方法论复盘-2026-05-26-知识地图建设]]。

---

## 0. 核心问题

**TTS 是独立产品，还是 SpeechLM 的输出模态？**

这个问题在 2024 年之前不需要问——TTS 就是独立产品。但 2024 起，VALL-E / SpeechGPT / Moshi / Mini-Omni / Qwen-Audio / StepAudio2.5 等系统把语音理解、生成、对话能力压缩进同一个模型，TTS 的边界开始模糊。

本图把 11 篇综述提供的多个视角拼在一起，给 TTS 在更大语音 AI 生态中的位置一个明确的坐标。

## 1. TTS 视角的内部演化（用户已有笔记综合）

参考 [[TTS-代表模型谱系]] 的"演化脉络"：

| 阶段 | 时间 | TTS 的定位 |
|---|---|---|
| 独立 TTS | ~2021 前 | FastSpeech2 / VITS / NaturalSpeech 是独立合成系统 |
| Codec LM TTS | 2023-2024 | VALL-E / CosyVoice 仍是独立 TTS，但用 LM 做序列建模 |
| LLM-native TTS | 2026 | Qwen3-TTS 用通用 LLM 直出 speech token，开始模糊 TTS 与 LLM 边界 |
| 统一 Speech LLM | 2026 | StepAudio2.5 把 ASR + TTS + 对话压成一个模型 |

**变化的本质**：TTS 从"独立产品"演化为"基础模型的一种模态能力"。

## 2. SpeechLM 视角（来源：Cui 2024 #3）

### 2.1 SpeechLM 的定义（Cui 2024 §I）

"An autoregressive foundation model that processes and generates speech end-to-end, utilizing contextual understanding for coherent sequence generation."

形式：$\mathbf{M}^{\text{out}} = SpeechLM(\mathbf{M}^{\text{in}};\theta)$，M 为多模态序列，每元素可为音频样本或文本 token。

**明确排除**：纯文本 LM（称为 TextLM）。

### 2.2 端到端 vs ASR+LLM+TTS 级联（Cui 2024 §II）

| 维度 | 级联（ASR+LLM+TTS） | 端到端 SpeechLM |
|---|---|---|
| 信息损失 | 副语言信息（音高/音色）在文本中间态丢失 | 直接建模语音 token，副语言保留 |
| 延迟 | ASR/TTS 顺序执行延迟叠加 | 单模型推理，可流式 |
| 累积错误 | ASR 转写错误 → LLM → TTS 放大 | 无中间错误放大 |

**Cui 立场**：级联"suffers from inherent limitations"，SpeechLM 是"a promising alternative"。

### 2.3 SpeechLM 三阶段管线（Cui 2024 Figure 4 复刻）

```
SpeechLM
├── Speech Tokenizer
│   ├── 语义理解目标   (HuBERT / Wav2vec 2.0 / WavLM / USM)
│   ├── 声学生成目标   (EnCodec / SoundStream)
│   └── 混合目标       (SpeechTokenizer / Mimi)
├── Language Model     (Transformer / LLaMA / Qwen2 / OPT / Mixtral)
└── Vocoder
    ├── GAN-based      (HiFi-GAN / BigVGAN / MelGAN)
    ├── Codec Decoder  (EnCodec decoder)
    └── 其他           (Flow / Diffusion / AR / VAE)
```

**TTS 在框架中被重新定义**：不再是独立的"声学模型 + 声码器"系统，而是 LM 输出端的 **"Token-to-Speech Vocoder"**——只负责把 LM 预测的 token 还原为波形。这是 SpeechLM 视角下 TTS 最大的范式变化。

## 3. LLM-Speech 集成视角（来源：Yang 2025 #5）

Cui 2024 关注"SpeechLM 是什么"，Yang 2025 关注"**如何把语音送进 LLM**"——这是工程决策维度的补充。

### 3.1 三类集成（Yang 2025 §3-5）

| 类别 | 子类 | 代表 | TTS 含义 |
|---|---|---|---|
| **Text-based**（文本接口）| Cascaded | AudioGPT / HuggingGPT | LLM 调用外部 TTS 工具 |
| | LLM Rescoring | — | N-best 假设重评 |
| | LLM GER | HyPoradise / GenTranslate | N-best → 重生成转写 |
| **Latent-representation-based**（隐层表示）| Conv Downsampling | SLAM-ASR / Seed-ASR | 卷积降采样后注入 LLM |
| | CTC Compression | — | 用 CTC 预测压缩帧 |
| | **Q-Former** | SALMONN / Qwen2-Audio | Transformer 查询将变长输入映射为定长 |
| | Other | Qwen2-Audio | 线性投影 / 随机下采样 |
| **Audio-token-based**（音频 token）| Semantic | TWIST / Spirit-LM / SpeechGPT | S3M + k-means 离散化 |
| | Acoustic | LauraGPT | 神经编解码器码本 |
| | **Semantic + Acoustic** | AudioPaLM / **Moshi** | 两阶段或联合建模 |

### 3.2 三类的 TTS 适用性（Yang 2025 §6）

**Yang 2025 的条件式结论**：
- 资源充足 / 需要实时 → **隐层表示 / 音频 token** 优于文本接口
- 资源有限 / 需可解释 → **文本接口** 更优
- 语音仅作输入 → **隐层表示** 最深度集成
- 语音也需输出 → **音频 token** 更合适（"generating speech from latent representations remains challenging"）

**我的归纳**：对 TTS 来说，**只有 Audio-token 类才能真正输出语音**。文本接口类只能调用外部 TTS，隐层表示类几乎不涉及语音生成。所以 LLM-native TTS（[[Qwen3-TTS]] / [[StepAudio2.5]]）必然走 Audio-token 路线。

## 4. 对话系统视角（来源：Ji 2024 #8 WavChat）

Ji 2024 的分类细于 Cui 2024，**单独把对话系统拆出 11 个子类**。

### 4.1 级联系统 5 子类（Ji 2024 §3）

分类标准：core LM 能否直接理解和生成语音表示。**不能则为级联**。

| 子类 | 说明 | 代表 |
|---|---|---|
| 原始三段式 | ASR → LLM(ChatGPT) → TTS，仅保留文本智能 | AudioGPT |
| +副语言感知 | 引入情感/风格向量辅助 LLM 生成带风格标注文本 | ParalinGPT / Spoken-LLM / E-chat |
| +语音直入理解 | 冻结 Whisper 编码器 + adapter 直接送入 LLM，输出仍为文本 | Qwen2-Audio / SALMONN |
| +多模态理解 | 图像/视频/音频编码器统一接入 LLM | VITA / Baichuan-Omni |
| +可训练 TTS 解码 | LLM 内嵌流式 TTS 模块但仍先生成文本内容 | Llama 3.1 语音版 |

### 4.2 端到端系统 6 子类（Ji 2024 §4）

| 子类 | 核心思路 | 代表 |
|---|---|---|
| 纯语音 AR | 双轨数据 self-/cross-attention，无文本依赖 | **dGSLM** |
| 文本→语音序列拼接 | 先生成文本 token 再生成语音 token | SpeechGPT / GLM-4-Voice / EMOVA |
| 隐状态投影 | LLM hidden states 经投影层/解码器输出语音 | PSLM / LLaMA-Omni / IntrinsicVoice / Freeze-Omni |
| 交错拼接 | 词级 interleaving，需精确对齐 | Spirit-LM |
| **文本-语音并行生成** | 同时输出文本与声学 token | **Moshi** / Mini-Omni |
| 多阶段对齐后推理无文本 | 训练含文本，推理纯语音 | OmniFlatten / SyncLLM |

### 4.3 全双工底座（Ji 2024 §5，对全双工方向高价值）

**Survey 原文**："the system should be able to listen and speak simultaneously"。

| 维度 | 说明 |
|---|---|
| 半双工 | 用户说完→系统回复，无法实时打断（传统文本对话） |
| **全双工** | 系统与用户可同时说话、实时中断 |
| 中断（Interrupt） | 用户不满可打断；系统也可主动打断澄清意图 |
| 填充词（Fillers） | "okay" / "haha" 等增强自然感 |

**Moshi 是全双工的标志性突破**：通过并行建模用户流和系统流，"eliminates the need for explicit speaker turns"。

### 4.4 TTS 在对话系统中的四维重定义（我的归纳，基于 Ji 2024 §6）

| 维度 | 含义 | 代表实现 |
|---|---|---|
| **低延迟流式** | 不能等全文本输入完才合成 | Freeze-Omni: NAR prefill + AR generate；CosyVoice2 chunk-aware |
| **可被打断** | 全双工架构下随时停止当前解码 | Moshi 实时监测用户流 |
| **状态保持** | 多轮中维持音色/风格一致 | Moshi depth Transformer 在 token 级保持声学连贯 |
| **从文本驱动到隐状态驱动** | TTS 不再接受文本，而是 LLM 的 hidden states | LLaMA-Omni / IntrinsicVoice |

**这是已有 [[TTS-技术路线图]] / [[TTS-核心挑战]] 缺失的关键视角**。对应分工表中"新增挑战 8"的来源。

### 4.5 Ji 2024 的批判性观察

- **端到端 ≠ 真端到端**：多数所谓端到端模型训练阶段仍重度依赖文本对齐，推理时才省略文本，是"训练时级联、推理时端到端"
- **全双工评估缺失**：缺乏系统化 benchmark 来度量 interrupt latency、overlap handling 质量
- **数据瓶颈**：高质量双轨/全双工对话数据极度稀缺（dGSLM 仅用 Fisher 语料）

## 5. Audio-LM vs SpeechLM 边界（来源：Audio-LM #14 vs Cui #3）

### 5.1 范围对比

| 维度 | Cui 2024 SpeechLM | Audio-LM 2025 |
|---|---|---|
| 覆盖域 | **Speech only**（ASR / TTS / 对话） | **Speech + Music + Sound** 三域统一 |
| 主要任务 | 理解 + 生成 + 对话 | 分类 / 检索 / AAC / AQA + 少量生成 |
| TTS 含量 | TTS 是核心生成任务之一 | TTS 仅 Table II 一笔带过（VALL-E / Seed-TTS） |
| 重点架构 | tokenizer + LM + vocoder 三阶段管线 | 4 类架构（Two Towers / Two Heads / One Head / Cooperated）|

**Audio-LM 作者明确表态**（#14 §相关工作）：把 Cui 等 SpeechLM survey 列为"narrow focus"，指出"fails to capture broader developments and cross-domain synergies"。

### 5.2 ALM 四类架构（Audio-LM #14 Fig 4）

| 类型 | 特征 | 代表 |
|---|---|---|
| **Two Towers** | 双编码器 + 对比对齐，晚期交互 | CLAP |
| **Two Heads** | 双编码器 + 语言模型解码 | Pengi / Qwen-Audio |
| **One Head** | 单一编码器处理双模态 | CALM |
| **Cooperated Systems** | LLM 作 agent 协调多模型 | AudioGPT |

### 5.3 对 TTS 的实际意义

**Audio-LM 视角下 TTS 几乎不是研究重点** —— TTS 在四类架构中要么作为生成头（Two Heads / One Head 的副产品），要么作为 LLM 调用的外部工具（Cooperated）。**真正以 TTS 为目标的工业系统几乎都还在 SpeechLM 范畴内**（CosyVoice / Qwen3-TTS / SeedTTS / StepAudio2.5）。

## 6. TTS 在不同框架下的 4 个位置

综合 §2-§5 给出的多视角，TTS 在更大语音 AI 生态中有**四种共存的位置**：

| 位置 | TTS 的形态 | 代表系统 | 关键约束 |
|---|---|---|---|
| ① 独立产品 | 独立 TTS 系统，可单独调用 | [[CosyVoice]] / [[F5-TTS]] / [[IndexTTS2]] / [[VoxCPM]] / [[Qwen3-TTS]] (作 API) | 零样本 + 多语言 + 流式 |
| ② SpeechLM 输出模块 | LM 输出 token，专用 vocoder 还原波形 | [[SeedTTS]] / [[Moshi]] (TTS 部分) / SpeechGPT (token-to-speech 头) | LM 与 TTS 紧耦合 |
| ③ Audio-LM 子任务 | 多任务音频 LLM 中的一个生成任务 | Pengi / Qwen-Audio (TTS 头) | TTS 不优化到极致 |
| ④ Dialogue Agent 组件 | 对话系统中实时响应的语音输出 | Mini-Omni / Freeze-Omni / OmniFlatten | 强约束：低延迟 + 可被打断 + 状态保持 |

**关键观察**：**四种位置同时存在且不互斥** —— 同一个工业系统可能既提供独立 TTS API（位置 ①），又服务对话产品（位置 ④）。CosyVoice 是位置 ① 的代表，但 CosyVoice 的 token 也被嵌到对话系统里成为位置 ②。

## 7. 关系图的可视化

```
                       ┌──────────────────────────────┐
                       │     大型 Audio-Language       │
                       │   ┌────────────────────────┐ │
                       │   │  SpeechLM (Cui 2024)   │ │
                       │   │ ┌─────────────────┐    │ │
                       │   │ │ Spoken Dialogue │    │ │
                       │   │ │  (Ji 2024)      │    │ │
                       │   │ │ ┌───────────┐   │    │ │
                       │   │ │ │ Full-duplex│  │    │ │
                       │   │ │ └───────────┘   │    │ │
                       │   │ └─────────────────┘    │ │
                       │   └────────────────────────┘ │
                       │  Audio-LM #14：含 music+sound│
                       └──────────────────────────────┘
                                   ↑
                       TTS 在哪一层运行？
                                   ↑
        ┌───────────────┬─────────────────┬──────────────┐
        ① 独立 TTS     ② SpeechLM 输出   ③ ALM 子任务   ④ Dialogue 组件
        CosyVoice      Moshi/SeedTTS     Pengi/QwenA   Freeze-Omni
        F5-TTS         SpeechGPT         (TTS 头)      Mini-Omni
        IndexTTS2      
        VoxCPM
        Qwen3-TTS(API)
```

## 8. 现状判断与未来分化

**当前共存的四种位置在未来 12-24 月可能分化**（我的判断）：

| 用例 | 主导走向 | 依据 |
|---|---|---|
| 工业 voice cloning / configurable TTS（B2B/B2C 单点 API） | 持续 ① | CosyVoice/Qwen3-TTS 作 API 的成熟度 + 客户需要单点调用 |
| AI agent / 实时对话产品 | 走 ② / ④ | StepAudio2.5 / Mini-Omni 类设计已经把 TTS 集成 |
| 通用多模态 LLM | 走 ③（但 TTS 不会成为主竞争点）| Audio-LM #14 趋势已经表明 TTS 在 ALM 里被边缘化 |
| 录音棚级 / 专业配音 | 留在 ① | 需要极致质量 + 多次迭代，不适合 LLM-native |

**关键不确定性**：
- LLM-native TTS（Qwen3-TTS / StepAudio2.5）能否在质量上**全面匹配**专用系统 ①？目前缺乏独立第三方大规模评测
- 全双工对话（Moshi 类）的工业化落地速度
- Audio-LM 的多任务能力会不会反向"侵蚀" SpeechLM 的市场

## 9. 与其他笔记的关系

- [[TTS-领域总览]] §与相邻领域的关系 — 简单速览版，本笔记是详细版
- [[TTS-技术路线图]] — 模型架构视角的 5 条路线（不展开对话/SpeechLM 框架）
- [[TTS-核心挑战]] §挑战 8（待新增）— "对话系统重定义 TTS"
- [[TTS-代表模型谱系]] — 各模型在本图哪个位置的对照
- [[TTS-11篇综述综合-2026-05]] §3.5 — 本笔记的分工表入口

## 10. 主要来源

| 综述 | 贡献到本笔记的哪节 |
|---|---|
| Cui 2024 (arXiv:2410.03751, ACL 2025) | §2 SpeechLM 完整视角 + §6 位置 ② |
| Yang 2025 (arXiv:2502.19548, ACL 2025 Findings) | §3 集成三分类 |
| Ji 2024 (arXiv:2411.13577) WavChat | §4 对话系统 11 子类 + 全双工 + 四维重定义 + §6 位置 ④ |
| Audio-LM 2025 (arXiv:2501.15177) | §5 ALM vs SpeechLM 边界 + §6 位置 ③ |
| [[ControllableTTS-Survey]] (Xie 2024) | §1 LLM-native TTS 的可控性视角 |

---

*2026-05-26 — 用 11 篇综述构建，对应分工表 §3.5*
