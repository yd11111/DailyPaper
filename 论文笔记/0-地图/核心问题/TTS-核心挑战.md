---
title: TTS 核心挑战
type: 核心问题
domain: TTS
tags: [challenges, tts, open-problems]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 核心挑战

以下是 TTS 领域当前尚未完全解决的核心问题。每个问题标注**紧迫度**（当前工业/研究中的阻塞程度）和**进展阶段**。

> ⚠️ **未 verify 警告**（2026-05-26 添加）：本文档 2026-05-26 修订时新增的内容（挑战 1 Azzuni 四级、挑战 2 Xie 五策略、挑战 5 Mousavi 五维、挑战 6 Position #11 重写、挑战 8 对话重定义等）含具体技术对照 cell，部分基于综述章节摘录 + 已有论文笔记的二阶推断，**未按新工作流 §11 三层 verify**。**结构层（8 项挑战 + 优先级矩阵）保留**，具体 cell 中的模型归类、SECS 数值、评测协议反例等数字 / 命名细节需要核实原文。详见 [[方法论复盘-2026-05-26-知识地图建设]]。

---

## 挑战 1: 零样本说话人克隆（Zero-shot Speaker Cloning）

**紧迫度**: ⭐⭐⭐⭐⭐ | **阶段**: 已有初步方案，但鲁棒性不足

### 问题定义

给定一段 3-10 秒的参考音频，生成保持该说话人音色、说话风格的任意文本语音。核心难点在于从极短参考中分离出足够的说话人信息。

### 术语标准化（Azzuni 2025 #6）

Voice cloning 子领域的术语长期混乱。Azzuni 2025 提出**四级标准化定义**，区分轴是 `是否微调 × 数据量`：

| 类别 | 定义 | 代表 |
|---|---|---|
| Speaker Adaptation | 需微调，数据量不限 | AdaSpeech 系列 |
| Few-Shot Cloning | ≤5 min 参考，需微调 | Attentron、UnitSpeech、USAT |
| **Zero-Shot Cloning** | **推理时无梯度更新**，依赖专用模块（speaker encoder 等） | [[VALL-E]]、[[NaturalSpeech3]]、StyleTTS 2、HierSpeech++ |
| Multilingual Cloning | 跨语言保持说话人特征 | [[YourTTS]]、[[XTTS]] |

详见 [[Voice-Cloning术语标准化]]。下文按此分类组织当前方案。

### 当前方案谱系（按 Azzuni 四级归类）

| 类别 | 方案 | 代表 | 思路 | 局限 |
|------|------|------|------|------|
| Zero-Shot | In-context Learning | [[VALL-E]]、[[CosyVoice]] | 把参考音频 token 作为 prompt 前缀 | 依赖 codec 质量；短参考下音色漂移 |
| Zero-Shot | 说话人编码器 | [[XTTS]] (兼 Multilingual) | 训练专门的 speaker encoder 提取 embedding | embedding 维度有限，丢失细粒度信息 |
| Zero-Shot | 解耦表征 | [[MegaTTS2]]、[[NaturalSpeech3]] | 显式分离 content/timbre/prosody | 分离不完全导致信息泄漏 |
| Zero-Shot | Flow Matching 条件 | [[F5-TTS]] | 参考音频直接拼接到输入 | 需要 fill-in-the-middle 训练策略 |
| Multilingual | 多语言 VITS 扩展 | [[YourTTS]] | 多语言多说话人统一架构 | 较旧；SECS 较低（~0.34） |

### 真实零样本能力上限（关键警示）

**Azzuni 2025 表 III 数据**：即使最先进的零样本系统，SECS 仍**与真实语音存在可察觉差距**：

| 系统 | 报告 SECS | 评测集 |
|---|---|---|
| [[NaturalSpeech3]] | ~0.67 | LibriSpeech zero-shot |
| VALL-E 2 | ~0.64 | LibriSpeech zero-shot |
| [[YourTTS]] | ~0.34 | VCTK zero-shot（旧基线）|
| 真实说话人 vs 自己 | ~0.85+ | 上限参考 |

**因此**：[[TTS-代表模型谱系]] 等笔记里的 "零样本 ✓" 标记应理解为"声称支持"而非"达到接近真实"。"零样本克隆已成熟"是被高估的判断。同时跨论文 SECS 不可比（不同 speaker encoder），详见 [[Voice-Cloning术语标准化]] §SECS 跨论文不可比警示。

### 2026 年新进展

- **连续 AR 路线**: [[SemaVoice]]（2026）通过 SFM 对齐 + patch-wise diffusion head 在连续空间做零样本克隆，避开 codec 信息瓶颈
- **无参考音频克隆**: [[FlexiVoice]]（ICLR 2026）证明可用自然语言描述控制说话风格，完全不需要参考音频，开辟了"描述性克隆"新方向
- **LLM-native 克隆**: [[Qwen3-TTS]] 利用通用 LLM 的 in-context learning 能力做零样本，500 万小时数据

### 未解问题

- **跨语言克隆**: 参考音频是中文，生成英文——音色保持但发音模式如何处理？[[VALL-E-X]]、[[YourTTS]] 有探索但效果仍不稳定
- **极短参考 (<3s)**: 当参考只有 1-2 秒时，所有方法的 [[SIM-O]] 都显著下降
- **说话人一致性**: 长文本生成时，音色在段落间漂移（尤其 AR 模型）
- **安全**: 零样本克隆的滥用防护（deepfake 检测、水印）

### 相关笔记

[[VALL-E]] → [[CosyVoice]] → [[CosyVoice2]] → [[CosyVoice3]] 展示了这条路线的持续演进

---

## 挑战 2: 韵律与表达性控制（Prosody & Expressiveness）

**紧迫度**: ⭐⭐⭐⭐ | **阶段**: 粗粒度可控，细粒度仍困难

### 可控性范式的演进框架（Xie 2024 #2）

Xie 2024 把可控 TTS 的控制策略整理为**渐进路径**——这是理解此挑战的总框架：

| 阶段 | 控制策略 | 输入 | 代表 |
|---|---|---|---|
| 1 | **Style Tagging** | 离散标签 / 连续信号 / 潜变量修改 | 早期 prosody 控制 |
| 2 | **Reference Speech Prompt** | 少量参考音频做零样本克隆 | [[VALL-E]] / [[CosyVoice]] / [[MegaTTS]] |
| 3 | **Natural Language Descriptions** | 自然语言描述属性（如 PromptTTS）| PromptTTS / TextrolSpeech |
| 4 | **Instruction-Guided Synthesis** | 统一指令格式驱动 | InstructTTS / [[FlexiVoice]] / VoxInstruct |
| 5 | **Instruction-Guided Editing** | 指令驱动语音编辑 | 编辑特定属性 / 段落级修改 |

可控维度（Xie 2024 §2）：韵律 (Prosody) / 音色 (Timbre) / 情感 (Emotion) / 风格 (Style) / 语言 (Language) / 环境 (Environment) **六维**。

详见 [[ControllableTTS-Survey]]。本挑战聚焦韵律 / 表达性子维度。

### 问题定义

控制生成语音的情感、语速、停顿、重音、语调曲线等韵律属性。理想状态是既支持全局风格控制（"用开心的语气说"）也支持字/词级细粒度标注。

### 当前方案

| 粒度 | 方案 | 代表 | 状态 |
|------|------|------|------|
| 全局情感 | 情感 embedding 条件 | [[EmotionThinker]] | 粗粒度可用 |
| 语速 | duration 缩放 | [[FastSpeech2]] | mel 路线成熟 |
| 音高 | pitch contour 输入 | [[FastSpeech2]]、[[MegaTTS]] | 需要 F0 标注 |
| 细粒度韵律 | 韵律 latent 建模 | [[MegaTTS2]]、[[NaturalSpeech3]] | 分解质量不稳定 |
| 自然韵律 | 文本理解驱动 | [[Qwen3-TTS]] | LLM 理解文本语义→自动推断韵律 |
| **自然语言指令** | NL 描述控制 | [[FlexiVoice]]（ICLR 2026） | 用文字描述风格，不需要参考音频 |

### 2026 年新进展

- **Instruction TTS 浮现**: [[FlexiVoice]]（ICLR 2026）验证了"用自然语言描述替代参考音频"的可行性，将可控性从"给条件"转向"说人话"
- **LLM 语义理解驱动韵律**: [[Qwen3-TTS]] 展示通用 LLM 的语义理解能力可以自动推断合适的韵律，无需显式韵律标注
- **Speech RLHF**: [[GSRM]]（2026）提出 generative speech reward model，将 RLHF 对齐技术引入语音领域，有望在韵律自然度上带来突破

### 未解问题

- **韵律-内容纠缠**: 改变韵律时常常扭曲内容（换重音位置导致发音变化）
- **长文本韵律连贯**: 段落级、篇章级的韵律规划（不只是句级）
- **跨文化韵律**: 不同语言的韵律规则差异大，如何做到多语言统一建模
- **可控性 vs 自然性**: 过度控制导致不自然，如何在可控和自然之间找平衡

---

## 挑战 3: 流式与低延迟（Streaming & Low Latency）

**紧迫度**: ⭐⭐⭐⭐⭐ | **阶段**: 200-500ms 首包可达，<100ms 仍是开放问题

### 问题定义

在用户输入文本后，以极低延迟（首包延迟）开始输出语音。对全双工对话系统尤其关键——用户说完话到系统开始回应的延迟直接影响对话自然度。

### 延迟分解

```
总延迟 = LLM推理首token延迟 + TTS首包延迟 + 网络传输
         (~100-300ms)          (~100-500ms)    (~50ms)
```

### 当前方案

| 方案 | 代表 | 首包延迟 | 代价 |
|------|------|---------|------|
| Chunk-aware AR | [[CosyVoice2]] | ~200ms | 需要 chunk 边界处理 |
| Streaming LM + Flow | [[GLM-TTS]] | ~300ms | 流式 Flow Matching 推理 |
| 非自回归 | [[FastSpeech2]] | <50ms | 不支持零样本 |
| Diffusion 加速 | [[NaturalSpeech2]] + DDIM | ~500ms | 质量-速度 tradeoff |

### 2026 年新进展

- **SSM 替代 Transformer**: [[MambaVoiceCloning]]（ICLR 2026）用 Mamba SSM 做 diffusion TTS 的条件路径，线性复杂度有望降低长序列推理延迟
- **统一 Speech LLM 流式**: [[StepAudio2.5]]（2026）在统一基座中同时实现 ASR + TTS + 实时交互，流式是核心设计目标

### 未解问题

- **流式 + 高质量的 tradeoff**: chunk 级生成在边界处会有不连续（能量突变、韵律断裂）
- **RTF < 0.1 + 高质量**: 实时率低于 0.1（即 10 倍实时）是工业要求，但最高质量的模型通常 RTF > 0.3
- **全双工中断恢复**: 用户 barge-in 时，TTS 需要立即停止并可能从断点续说
- **端侧部署**: 手机端 TTS 需要模型压缩到 <100MB，延迟 <50ms

---

## 挑战 4: 训练数据规模与质量（Data Scale & Quality）

**紧迫度**: ⭐⭐⭐⭐ | **阶段**: 量在爆发，质的标准仍在探索

### 当前数据规模

| 系统 | 训练数据量 | 数据来源 |
|------|-----------|---------|
| [[Qwen3-TTS]] | 500 万小时 | 私有（多语言） |
| [[VoxCPM]] | 180 万小时 | 私有 |
| [[CosyVoice3]] | 100 万小时 | 私有（从 CosyVoice 的 17 万小时一路扩大） |
| [[GLM-TTS]] | 10 万小时 | 10 万小时严格筛选 |
| [[IndexTTS2]] | 5.5 万小时 | [[Emilia]] 开源数据集 |

### 核心问题

- **数据质量 vs 数量**: [[GLM-TTS]] 用 10 万小时精筛数据达到与 100 万小时系统可比的效果，说明"更多数据"不等于"更好模型"
- **数据处理 pipeline**: SNR 过滤、说话人聚类、文本-语音对齐、去混响——每一步都影响最终质量
- **开源数据缺口**: [[Emilia]]（100K h）是目前最大的开源 TTS 数据集，但与工业界 100-500 万小时的规模差距巨大
- **多语言数据不平衡**: 中英文数据充足，但小语种（东南亚、非洲语言）严重不足
- **数据合规**: 训练数据的版权、隐私、consent 问题日益突出

---

## 挑战 5: Codec / Token 设计（Speech Tokenization）

**紧迫度**: ⭐⭐⭐⭐ | **阶段**: 百花齐放，尚无共识最优方案

### 问题定义

如何将连续语音信号转化为离散/连续的中间表征，使之既保留语音质量又适合 LLM 建模。

### 设计空间（Mousavi 2025 #7 五维 taxonomy 替代旧二分）

Mousavi 2025 明确反对"语义 vs 声学"二分（认为 insufficient），提出五维分类——这套替代了上一版本的简单维度表：

| 维度 | 选项 | 关键判断 |
|------|------|---|
| **架构** | CNN / CNN+RNN / Transformer / CNN+T | CNN+T 是 2024+ 主流 |
| **量化** | K-means / RVQ / SVQ / GVQ / FSQ / MSRVQ / CSRVQ / PQ | 工业战场在 RVQ vs FSQ vs SVQ 三选一 |
| **训练范式** | 分离 vs 联合；目标 Recon/VQ/GAN/Feat/Diff/MP | 多数仍分离训练 |
| **流式能力** | 因果卷积 / 因果注意力 / 帧率 | Mimi 12.5fps 因果 T 是新基准 |
| **目标领域** | 单域 (Speech/Music/Audio) vs 多域 | speech-only 通常质量更高 |

**辅助维度**：解耦（FACodec 分 4 路）、语义蒸馏（SpeechTokenizer/Mimi 首层 SSL）。

详见 [[TTS-表示层地图]] §2。

### 当前探索

- [[SoundStream]] / [[EnCodec]]: RVQ 8-12 层，开山之作但序列太长
- [[DAC]]: 改进 RVQ 量化质量
- [[SNAC]]: 多尺度 RVQ，不同层不同帧率
- [[FSQ]]: 有限标量量化替代向量量化（[[CosyVoice2]] 使用）
- [[RepCodec]]: 利用 SSL 表征做 codec
- [[VoxCPM]]: 完全跳过 tokenization，直接建模连续 mel

### 2026 年新进展

- **Codec 新探索（ICLR 2026）**: [[FlexiCodec]] 动态帧率适配不同内容复杂度；[[StableToken]] 噪声鲁棒 tokenizer；[[ScalingSpeechTokenizers]] 用 diffusion autoencoder 替代 VQ
- **Tokenizer-free 路线加速**: [[MELLE]]→[[FELLE]]→[[CLEAR]]→[[SemaVoice]]，5+ 篇工作证明跳过 tokenization 是可行方向，从根本上回避了 codec 设计问题

### 未解问题

- **语义信息 vs 声学细节的最优平衡点**在哪里？
- **codec 对下游 LM 的友好性**如何量化？
- **codec 训练与 TTS 训练联合优化**是否可行？（目前几乎都是分开训练）
- **Tokenizer-free 路线是否会使 codec 设计问题本身过时？** 这取决于连续 AR 路线的 scaling 能力能否匹配离散 token 路线

---

## 挑战 6: 评估方法论的结构性危机（Evaluation Methodology Crisis）

**紧迫度**: ⭐⭐⭐⭐ | **阶段**: 上一版本表述为"评测标准化"——经 Position paper *Towards Responsible Evaluation for TTS* (arXiv:2510.06927, #11) 检视后**升级为结构性危机**

### 问题定义（升级版）

Position #11 揭示的问题比"缺乏标准"更严重：**现有的自动指标本身就有结构性缺陷 + 跨论文不可比是默认状态而非例外**。

**具体证据**（详见 [[TTS-评测体系]] §评估方法论的结构性陷阱）：

| 指标 / 协议 | 结构性问题 |
|---|---|
| **WER** | ASR 自身错误 + 与感知非单调对应；用作 RL reward 会 "collapse prosodic variance into monotone" |
| **SIM-o / SECS** | 不同 speaker encoder（ECAPA-TDNN / GE2E / X-vector / TitaNet-L）报告值不可比；同团队 VALL-E vs VALL-E 2 的 SIM-o 算法都不一致 |
| **MOS** | 天花板效应（高质量系统饱和 4.0+）+ 跨论文不可迁移 + 报告不透明 |
| **DNSMOS** | 训练在语音增强数据，被滥用于合成评估——典型 domain shift |
| **LibriSpeech test-clean** | 子集碎片化反例：[[VALL-E]] 1234 条 / [[NaturalSpeech3]] 40 条 / [[F5-TTS]] 1127 条 |
| **Continuation 任务** | 定义不一致：VALL-E 用前 3s prompt，E2 TTS 用最后 3s |

### 现有指标

| 类型 | 指标 | 测什么 | 问题 |
|------|------|--------|------|
| 客观-智能度 | [[WER]] (ASR↓) | 内容清晰度 | 依赖 ASR 模型选择 |
| 客观-音色 | [[SIM-O]] / [[SECS]] | 说话人相似度 | 不同 speaker encoder 结果差异大 |
| 客观-质量 | [[UTMOS]] / [[DNSMOS]] | 自动 MOS | 与人工 MOS 相关性有限 |
| 主观 | [[MOS]] | 综合自然度 | 成本高、不可复现、评分者偏差 |
| 主观 | [[MUSHRA]] | 质量对比 | 需要锚点设计 |
| 效率 | [[RTF]] | 实时率 | 硬件依赖 |

### 标准化努力

- [[SUPERB]]: 语音通用 benchmark，但 TTS 任务覆盖有限
- [[Emilia]]: 标准化数据集 + 评测 pipeline
- [[TTSDS2]]: 专门的 TTS 评测框架
- [[SpeechJudge]]: LLM-as-judge 做自动评测
- Seed-TTS-eval: 字节提出的标准化评测集（未单独发论文）
- **2026 新增 — 指令跟随评测**: [[Qwen3-TTS]]、[[StepAudio2.5]] 等 Instruction TTS / Speech LLM 系统需要新的评测维度——可控性、情感一致性、指令遵循率，传统的 WER + SIM-O + MOS 三件套不再足够
- **2026 新增 — Speech RLHF**: [[GSRM]] 提出 generative speech reward model，可作为自动评测器替代人工 MOS

### 未解问题

- **缺少公认的"ImageNet for TTS"**: 没有一个所有人都用的标准评测集
- **主观 vs 客观的鸿沟**: 自动指标与人耳感知仍有显著差距
- **多维度评测**: 质量、相似度、韵律、可控性应该如何加权比较？
- **跨语言评测**: 不同语言的评测标准应该统一还是分开？

详见 [[TTS-评测体系]]

---

## 挑战 7: 安全与伦理（Safety & Ethics）

**紧迫度**: ⭐⭐⭐ | **阶段**: 意识提升中，技术方案早期

### 问题定义

零样本 TTS 使得语音 deepfake 成本极低。如何在推进技术的同时防范滥用？

### 当前探索

- **语音水印**: 在生成语音中嵌入不可感知的水印，用于溯源
- **Deepfake 检测**: 训练分类器区分真人语音与合成语音
- **声纹授权**: 要求说话人 consent 才能使用其声纹
- **合成声明**: 在生成语音中强制加入"本内容由 AI 生成"提示

### 未解问题

- 水印的鲁棒性（抗压缩、抗重编码）
- 检测器的泛化性（对未见过的 TTS 系统）
- 法律框架（各国对语音合成的立法不同）
- 开源模型的滥用防控

---

## 挑战 8: 对话系统重定义 TTS（Dialogue Redefining TTS）

**紧迫度**: ⭐⭐⭐⭐ | **阶段**: 标志性系统出现（Moshi/Mini-Omni/Freeze-Omni），但工程实现还在迭代

### 问题定义

Ji 2024 *WavChat* (#8) 揭示：**当 TTS 作为对话系统的组件而非独立产品时，要求维度完全变了**。传统 TTS 追求自然度 + 准确度；对话系统中的 TTS 还要满足**四个新约束**。

### 四维重定义

| 维度 | 含义 | 代表实现 |
|---|---|---|
| **低延迟流式** | 不能等全文本输入完才合成 | Freeze-Omni: NAR prefill + AR generate；[[CosyVoice2]] chunk-aware；[[GLM-TTS]] streaming |
| **可被打断** | 全双工架构下 TTS 生成随时停止并切换 | [[Moshi]] 实时监测用户流；OmniFlatten 推理无文本 |
| **状态保持** | 多轮中维持音色 / 风格一致 | Moshi 的 depth Transformer 在 token 级保持声学连贯 |
| **从文本驱动到隐状态驱动** | TTS 不再接受文本，而是 LLM 的 hidden states | LLaMA-Omni / IntrinsicVoice / PSLM |

### 全双工的底层要求（Ji 2024 §5）

**Survey 原文**："the system should be able to listen and speak simultaneously"。这意味着：
- **因果性硬约束**：非因果 SSL tokenizer（HuBERT/Wav2vec）需"因果蒸馏"或 chunk 化（如 [[Mimi]] 把 WavLM 蒸馏成因果）
- **流式 codec 必须**：12.5fps 级低帧率 + 因果架构（Mimi 是当前新基准）
- **数据极度稀缺**：高质量双轨/全双工对话数据，dGSLM 仅用 Fisher 语料

### Ji 2024 的关键警示

- **端到端 ≠ 真端到端**：多数所谓端到端模型训练阶段仍重度依赖文本对齐，**推理时才省略文本**，是"训练时级联、推理时端到端"
- **全双工评估缺失**：缺乏系统化 benchmark 度量 interrupt latency / overlap handling 质量
- 数据瓶颈短期难突破

### 与其他挑战的关系

- 与挑战 3（流式低延迟）部分重叠，但**侧重对话场景而非单点 TTS**
- 与挑战 7（安全）连接：全双工系统的 deepfake 风险更大
- 与 [[TTS-SpeechLM-Dialogue关系]] 完整对应；本挑战是其在"核心问题"维度的浓缩

### 未解问题

- **interrupt latency 标准化基准**：缺乏公认评测协议
- **TTS 状态保持的量化**：多轮音色漂移如何度量
- **训练数据获取**：高质量双轨对话数据合成可否替代真人数据
- **隐状态驱动的可控性**：从文本→TTS 到 hidden states→TTS 后，instruction TTS 还能否生效

---

## 挑战优先级矩阵

| 挑战 | 工业紧迫 | 学术价值 | 2026 热度 |
|------|---------|---------|----------|
| 零样本克隆 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 🔥🔥🔥 |
| 流式低延迟 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 🔥🔥🔥 |
| Token 设计 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 🔥🔥🔥🔥（含 Tokenizer-free 路线的崛起） |
| 韵律控制 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 🔥🔥🔥（Instruction TTS 推动） |
| 训练数据 | ⭐⭐⭐⭐ | ⭐⭐⭐ | 🔥🔥 |
| 评估方法论危机 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 🔥🔥🔥（Position #11 升级紧迫度 + 指令跟随评测 + Speech RLHF 推动） |
| 安全伦理 | ⭐⭐⭐ | ⭐⭐ | 🔥 |
| 对话系统重定义 TTS | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 🔥🔥🔥🔥（Moshi / Mini-Omni 推动） |

---

*最后更新: 2026-05-25*
