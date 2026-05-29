---
title: TTS 评测体系
type: 专题对比
domain: TTS
tags: [evaluation, metrics, benchmark, tts]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 评测体系

> ⚠️ **未 verify 警告**（2026-05-26 添加）：本文档 2026-05-26 修订新增"评估方法论的结构性陷阱"7 子节 + 负责任评估三层框架。其中具体反例（如 LibriSpeech test-clean 子集 1234/40/1127 条、SIM-o 算法在 VALL-E vs VALL-E 2 不一致等）来自 Position #11 (arXiv:2510.06927) §Appendix A/B 摘录，**未直接 WebFetch verify Appendix 原文表格**。**结构层（7 维评测 + 主客观指标 + 三层框架）保留**，引用 Position #11 的具体数字需要 §X verify。详见 [[方法论复盘-2026-05-26-知识地图建设]]。

---

## 评测维度全景

TTS 评测需要覆盖多个正交维度，单一指标无法全面衡量系统质量。

| 维度 | 测什么 | 主要指标 | 典型 benchmark |
|------|--------|---------|---------------|
| **语音质量** | 自然度、清晰度、无伪影 | MOS / UTMOS / DNSMOS | LJSpeech / LibriTTS |
| **说话人相似度** | 与参考音色的匹配度 | SIM-O / SECS / EER | Seed-TTS-eval / LibriSpeech |
| **内容准确性** | 发音正确、无漏字多字 | WER / CER (ASR 转录) | 自定义 hard case 集 |
| **韵律自然度** | 停顿、重音、语调 | MOS (韵律) / F0 RMSE | 需专家评审 |
| **多语言** | 跨语言泛化 | 各语言独立 WER/MOS | Seed-TTS-eval (zh/en) |
| **鲁棒性** | 长文本、特殊符号、重复 | 成功率 / 完成率 | EmergentTTS-Eval |
| **效率** | 推理速度、资源占用 | RTF / 首包延迟 / 显存 | 自测（硬件绑定） |

## 核心指标详解

### 主观指标

#### [[MOS]] (Mean Opinion Score)

- **定义**: 人工评分 1-5 分（1=差, 5=优秀），取均值
- **优点**: 最直接反映人耳感知
- **缺点**: 成本高（需 20+ 评审员 × 数百条）、不可复现（不同评审池结果不同）、存在评分膨胀（新系统普遍被评 4.0+）
- **变体**: CMOS（对比 MOS）、DMOS（退化 MOS）
- **使用情况**: [[CosyVoice3]]、[[Qwen3-TTS]]、[[VoxCPM]] 等所有主要系统都报告 MOS

#### [[MUSHRA]] (MUltiple Stimuli with Hidden Reference and Anchor)

- **定义**: 在 0-100 范围内同时对多个系统评分，包含隐藏的参考和锚点
- **优点**: 对比性强，评审一致性更高
- **缺点**: 设计复杂（需要合适的锚点）
- **使用情况**: [[StepAudio2.5]] 报告了 MUSHRA

### 客观指标 — 智能度

#### [[WER]] / [[CER]] (Word/Character Error Rate)

- **定义**: 用 ASR 模型转录合成语音，与原文计算编辑距离
- **依赖**: ASR 模型选择（[[Whisper]] large-v3 是目前最常用的）
- **注意**: WER 低不代表听感好，只代表"说对了字"
- **典型值**: 顶级系统在 LibriSpeech test-clean 上 WER < 3%

#### Seed-TTS-eval WER/CER

- **定义**: 字节 SeedTTS 提出的标准化评测集，包含中英文 hard case
- **优点**: 有公开评测集、困难样本（绕口令、长句、多语码切换）
- **使用情况**: [[CosyVoice2]]、[[CosyVoice3]]、[[IndexTTS2]] 广泛采用

### 客观指标 — 说话人相似度

#### [[SIM-O]] (Speaker Similarity - Objective)

- **定义**: 用说话人验证模型（如 WavLM-TDNN / ECAPA-TDNN）计算参考音频与合成音频的余弦相似度
- **范围**: 0-1，越高越好
- **注意**: 不同 speaker encoder 给出的绝对值不可比；应使用相同 encoder 做系统间对比
- **典型值**: 零样本场景 SIM-O > 0.70 算可用，> 0.80 算优秀

#### [[SECS]] (Speaker Embedding Cosine Similarity)

- 同 SIM-O 的变体命名，本质相同

### 客观指标 — 自动 MOS

#### [[UTMOS]] (Universal TTS MOS Predictor)

- **定义**: 训练的 MOS 预测模型，输入合成语音输出预测 MOS
- **优点**: 免费、可复现、快速
- **缺点**: 与真实 MOS 相关性有限（尤其在非英语和极端质量区间）
- **典型值**: 4.0+ 表示质量不错

#### [[DNSMOS]] (Deep Noise Suppression MOS)

- **定义**: 微软提出的自动 MOS，侧重噪声和信号质量
- **用途**: 评估语音增强后的质量，TTS 中较少单独使用

### 效率指标

#### [[RTF]] (Real-Time Factor)

- **定义**: 生成 1 秒语音所需的计算时间。RTF < 1 表示实时，RTF < 0.1 表示 10 倍实时
- **注意**: 严重依赖硬件（A100 vs V100）、batch size、序列长度
- **典型值**: AR 模型 RTF 0.3-1.0；NAR 模型 RTF 0.01-0.1

#### 首包延迟 (First-packet Latency)

- **定义**: 从输入文本到开始输出第一个音频帧的时间
- **对流式系统关键**: [[CosyVoice2]] ~200ms、[[GLM-TTS]] ~300ms
- **包含**: 文本前端处理 + 第一个 chunk 的模型推理时间

## 主要 Benchmark 与数据集

### 通用评测

| Benchmark | 来源 | 规模 | 特点 |
|-----------|------|------|------|
| LJSpeech test | 公开 | ~100 句 | 单说话人英文，最老的标准集 |
| LibriTTS test-clean | 公开 | ~500 句 | 多说话人英文，清洁录音 |
| LibriTTS test-other | 公开 | ~500 句 | 多说话人英文，嘈杂录音 |
| Seed-TTS-eval | 字节 | 中英各 ~100 句 | 困难样本、标准化流程 |
| [[EmergentTTS-Eval]] | 学术 | ~200 句 | 长文本/数字/特殊格式鲁棒性 |
| [[TTSDS2]] | 学术 | 框架 | 多维度自动化评测框架 |

### 专项评测

| 评测 | 测什么 | 代表使用 |
|------|--------|---------|
| [[SUPERB]] | 语音通用任务（含 TTS 相关） | 语音 SSL 模型评测 |
| [[Dynamic-SUPERB]] | 动态新增任务 | 语音 LLM 评测 |
| [[SpeechJudge]] | LLM-as-judge 自动评测 | 避免人工 MOS 的替代方案 |
| [[Emilia]] | 大规模 TTS 数据集 + 评测 | 开源 TTS 系统标准化训练和评测 |

### 2026 年新增评测方向

| 方向 | 代表 | 测什么 | 现状 |
|------|------|--------|------|
| **指令跟随评测** | [[Qwen3-TTS]]、[[StepAudio2.5]] | 可控性/情感一致性/指令遵循率 | 萌芽期，缺乏标准化 |
| **Speech RLHF** | [[GSRM]]（2026） | 用 generative speech reward model 做自动奖励评分 | 替代人工 MOS 的新方向 |
| **开放数据评测** | [[Raon-OpenTTS]]（2026） | 强调可复现：开放数据 + 开放模型 + 开放评测 | KRAFTON/首尔大/KAIST 推动 |

**趋势**：随着 Instruction TTS（[[FlexiVoice]]）和 LLM-native TTS（[[Qwen3-TTS]]）的出现，传统的 WER + SIM-O + MOS 三件套不再足够。"模型是否遵循了自然语言指令"成为新的评测维度，但目前缺乏标准化的指令跟随评测集。

## 各系统评测对比矩阵

> 以下数据均来自各论文自报，**不同论文的评测设置不完全可比**。仅作参考。

### 英文零样本 TTS

| 系统 | WER↓ | SIM-O↑ | MOS↑ | 评测集 |
|------|------|--------|------|--------|
| [[VALL-E]] | 5.9% | 0.580 | 3.8 | LibriSpeech |
| [[NaturalSpeech2]] | - | 0.612 | 4.2 | LibriSpeech |
| [[CosyVoice]] | 4.3% | 0.735 | 4.1 | Seed-TTS-eval |
| [[CosyVoice2]] | 2.5% | 0.778 | - | Seed-TTS-eval |
| [[F5-TTS]] | 2.4% | 0.682 | - | Seed-TTS-eval |
| [[IndexTTS2]] | 2.1% | 0.701 | - | Seed-TTS-eval |

### 中文零样本 TTS

| 系统 | CER↓ | SIM-O↑ | 评测集 |
|------|------|--------|--------|
| [[CosyVoice]] | 3.2% | 0.758 | Seed-TTS-eval zh |
| [[CosyVoice2]] | 1.8% | 0.796 | Seed-TTS-eval zh |
| [[CosyVoice3]] | 0.95% | 0.805 | Seed-TTS-eval zh |
| [[IndexTTS2]] | 1.3% | 0.812 | Seed-TTS-eval zh |

## 评估方法论的结构性陷阱

> 本节基于 Position paper *Towards Responsible Evaluation for TTS* (arXiv:2510.06927, 2025) + Azzuni 2025 *Voice Cloning Survey* + Mousavi 2025 *Discrete Audio Tokens* 的具体反例。**这些不是个别工作的疏忽，而是当前 TTS 评估的系统性问题**。

### 陷阱 1：LibriSpeech test-clean 子集碎片化

**Position #11 §Appendix A 实证**：同一个"LibriSpeech test-clean"被不同论文切成完全不同的子集来报 WER：

| 论文 | 评测条数 |
|---|---|
| [[VALL-E]] | 1234 条 |
| [[NaturalSpeech3]] | 40 条 |
| [[F5-TTS]] | 1127 条 |

**后果**：WER 数值在不同子集上**绝对值不可比**——40 条的 WER 可能因子集偏差被显著低估或高估。即使两个工作"都在 LibriSpeech test-clean 上评测"，结果仍不可直接横比。

**建议**：报告 WER 时必须**精确列出**使用的样本数 + 选择方式（随机？前 N 条？特定 utterance ID 列表？）。

### 陷阱 2：SIM-o 计算方式不统一

**Position #11 §Appendix B 实证**：同一篇团队的不同代号工作在 SIM-o 计算上都不一致：

| 论文 | SIM-o 计算方式 |
|---|---|
| [[VALL-E]] | **排除 prompt 片段** 后算 cosine similarity |
| VALL-E 2 | **包含 prompt 片段** 算 cosine similarity |

**后果**：VALL-E 2 报告的 SIM-o 比 VALL-E 高，可能**完全是协议差异**而非真实能力提升。任何跨论文 SIM-o 比较都需要核对计算协议。

**建议**：报告 SIM-o 时必须明确：(1) speaker encoder 是哪个；(2) prompt 是否被纳入计算；(3) 参考音频和合成音频的对齐方式。

### 陷阱 3：跨论文 SECS 不可比（speaker encoder 不统一）

**Azzuni 2025 #6 §VI 警示**：不同论文使用不同 speaker encoder 报告 SECS / SIM-o：

| Speaker Encoder | 典型使用工作 |
|---|---|
| X-vector | 早期工作 |
| GE2E | YourTTS 类 |
| ECAPA-TDNN | VALL-E / NaturalSpeech 系列 |
| TitaNet-L | 部分较新工作 |
| WavLM-TDNN | 2024+ 主流 |

**后果**：论文 A 用 ECAPA-TDNN 报 SECS=0.78，论文 B 用 GE2E 报 SECS=0.82 —— **不能说论文 B 更好**。

详见 [[Voice-Cloning术语标准化]] §SECS 跨论文不可比警示。

### 陷阱 4：WER 用作 RL reward 的危险

**Position #11 §3.1 警示**：把 WER 作为 RL 训练的 reward 会"**collapse prosodic variance into monotone output**"——模型为了刷低 WER 会牺牲韵律自然度，输出单调机械。

**后果**：报告 WER 持续下降的论文需要警惕——是否同时牺牲了其他维度？

**建议**：用 RL 优化 TTS 时必须报告多维度 trade-off（韵律 / 情感 / 自然度），而非只追求 WER。

### 陷阱 5：DNSMOS 的域外滥用

**Position #11 §3.1 实证**：[[DNSMOS]] 训练在**语音增强**数据集上（去噪、去混响后的语音），但被大量论文用于评估**合成语音**质量。这是典型的 domain shift——预测器在分布外的输入上**没有可信度**。

**后果**：DNSMOS 数值在合成语音上的绝对值可能严重偏离真实人耳感知。

**建议**：用 DNSMOS 仅作参考，不作为决定性指标。同时报告 UTMOS（专门为 TTS 训练）或人工 MOS。

### 陷阱 6：推理任务定义不一致

**Position #11 §Appendix 实证**：即使"Continuation" 这一标准任务的定义都不统一：

| 论文 | Continuation 任务定义 |
|---|---|
| [[VALL-E]] | 用 **前 3 秒** 做 prompt，合成剩余 |
| E2 TTS | 用 **最后 3 秒** 做 prompt |

**后果**：不同 prompt 位置对应不同难度（开头 vs 结尾的声学特征不同），评测难度系数不同。

**建议**：报告 Continuation / Cross-sentence 等任务时必须明确 prompt 的具体位置和长度。

### 陷阱 7：MOS 的天花板效应 + 不可迁移性

**Position #11 §3.2 主张**：
- **天花板效应**：现代高质量 TTS 系统间 MOS 分数饱和（普遍 4.0+），**无法区分**真正的质量差异
- **不可迁移性**：不同研究间 MOS 分数直接对比"meaningless"——评分者池、评分指导、播放设备都不同
- **报告不透明**：虽有 ITU-T P.808 标准，多数研究仅名义参照而无实际遵守

**建议**：
1. 报告 MOS 时必须含：评分人数、评分指导原文、播放条件、统计显著性检验（p-value）
2. 跨论文比较时优先用 CMOS（对比 MOS）而非绝对 MOS
3. 对最先进系统间的对比应考虑 **Audio Turing Test** 等更具区分力的协议（Position #11 推荐方向）

## 负责任评估三层框架

**Position #11 §4 提出**，应作为知识库内的评估方法论标尺：

| 层级 | 关注 | 具体要求 |
|---|---|---|
| **L1 保真与准确** | 系统是否真的好 | 更鲁棒 / 更具区分力 / 更全面的主客观评分方法论；如 SP-MCQA、Audio Turing Test |
| **L2 可比性 / 标准化 / 可迁移性** | 跨论文是否可比 | 标准化基准（prompt list 公开 / 子集明确）；透明报告（评测协议细节）；可迁移指标（LLM-as-a-Judge）|
| **L3 治理 / 公平 / 安全** | 系统是否负责任 | 训练数据来源披露 + 授权；群体差异审计（按口音 / 性别 / 年龄分层报告）；防伪 / 水印 / 可追溯性纳入标准评估 |

**当前 TTS 论文绝大多数只触及 L1，少数到 L2，L3 几乎全部缺失**（我的归纳，基于 11 篇综述综合）。

## 跨域 Codec 评测的补充（Mousavi 2025 #7 提供）

Mousavi 2025 跑通的几个跨域 codec benchmark 可作为 **TTS 表示层选型的间接指标**：

| Benchmark | 测什么 | 与 TTS 的关系 |
|---|---|---|
| **Codec-SUPERB** | 重建质量跨域评估 | codec 重建上界 ≈ TTS LM 输出还原音质天花板 |
| **VERSA** | 多维度音频质量 | 同上 |
| **DASB** | 下游判别 + 生成任务（含 TTS / ASR / 增强 / 源分离）| 直接测 codec 对 TTS 性能的影响 |
| **SALMon** | 声学语言建模质量 | 测 codec 对 LM 学习的友好度 |

详见 [[TTS-表示层地图]] §6 评估表示质量的方法。

## 评测陷阱与红旗

### 常见不可靠做法

1. **Cherry-pick baseline**: 只和弱基线比，避开同期强竞品
2. **不同评测集**: A 系统报 LibriSpeech 结果，B 系统报 Seed-TTS-eval 结果，直接对比无意义
3. **MOS 评分膨胀**: 不同评审池、不同评分指导导致 MOS 绝对值不可跨论文比较
4. **只报最好指标**: 报 WER 低但不报 SIM-O，或反之
5. **缺少显著性检验**: MOS 差 0.1 是否有统计显著性？多数论文不报 p-value
6. **测试集泄露**: 用 LibriSpeech 的数据同时训练和测试
7. **子集碎片化**: 名义同一评测集但实际样本数 / 选择方式不同（见陷阱 1）
8. **协议未透明**: SIM-o 是否含 prompt、Continuation 用前/后段 prompt 等关键细节缺失（见陷阱 2、6）

### 如何读懂一篇论文的评测

1. 看**评测集是否标准化**（Seed-TTS-eval > 自建集）
2. 看**baseline 是否公平**（同数据量、同模型大小？）
3. 看**是否报了多个维度**（WER + SIM-O + MOS 三者缺一不可）
4. 看**是否有 ablation**（证明每个模块的贡献）
5. 看**数字的量级**（WER 从 3% 降到 2.5% vs 从 10% 降到 5%，意义完全不同）

## 评测工具推荐

| 工具 | 用途 | 获取方式 |
|------|------|---------|
| Whisper large-v3 | WER/CER 计算 | `pip install openai-whisper` |
| WavLM-TDNN | SIM-O 计算 | SpeechBrain |
| UTMOS | 自动 MOS | HuggingFace |
| pesq / polqa | 客观语音质量 | ITU 标准实现 |
| pysepm / pysptk | 频谱分析 | pip |

---

*最后更新: 2026-05-25*
