---
title: "Adaptive Turn-Taking for Real-time Multi-Party Voice Agents"
method_name: "ModeratorLM"
authors: [Soumyajit Mitra, Prabhat Pandey, Abhinav Jain, Shanmukha Sahith, K V Vijay Girish]
year: 2026
venue: Interspeech 2026
arxiv_id: "2606.13544"
tags: [turn-taking, multi-party-conversation, speech-llm, role-playing, chain-of-thought, voice-agent, duplex]
note_tier: standard

# === 技术决策枚举 ===
lm_init_type: warm-start
multitask: true
post_training_type: none
streaming: true

# === 知识地图联动 ===
domain: Dialogue
subdomain: multi-party-turn-taking
routes: [e2e-duplex, speech-llm-tts]
problems: [interrupt-handling, dialogue-integration, evaluation]
representations: [audio-token]
related_maps:
  - "[[TTS-SpeechLM-Dialogue关系]]"
  - "[[TTS-核心挑战]]"
related_surveys: []
evidence_level: medium
maturity: exploratory
last_repositioned: 2026-06-30

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2606.13544/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2606.13544/paper.html"
figures_dir: "_resources/2606.13544/figures/"
github_local:
cached_at: 2026-06-30

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2606.13544v3
created: 2026-06-30
---

# 论文笔记：Adaptive Turn-Taking for Real-time Multi-Party Voice Agents

> **笔记分级**：standard（方法清晰、实验充分、值得精读）。分级标准见 `references/quality-standards.md §模板分级`。
>
> **结构说明**：本笔记分三层——**一、阅读层**（读懂论文所需，技术断言用核验后口径书写）/ **二、研究审计层**（核验来源、可信度、claim 审查、批判，独立容器）/ **三、知识系统层**（地图定位 + 重估日志）。三层互不混杂。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Amazon AGI; IIT Kharagpur |
| 日期 | June 2026 |
| 项目主页 | 未见公开 |
| 对比基线 | [[Moshi]] (Moshika), MP-Baseline |
| 链接 | [arXiv](https://arxiv.org/abs/2606.13544) / 未见开源代码 |

## 一句话总结

> 首个将显式角色条件融入多方语音对话轮转决策的 Speech LLM 系统，通过角色条件微调 + [[Chain-of-Thought]] 推理，在多方会话 [[Turn-taking]] 精度上较非角色基线提升 40%+ precision、70%+ recall。

---

# 一、阅读层（主文）

> 主文只承载"读懂这篇论文"所需的结论 + 证据链 + 关键图表。
> **口径**：所有具体技术断言用**核验后口径**书写。每条断言的 verify 来源 [§X] 不写在这里，统一收到「附录·核验结论」表。

## 核心贡献

1. **角色条件化轮转模型 ModeratorLM**：把显式角色（如"沉稳的倾听者"vs"强势的会议主持人"）注入 Speech LLM 的系统 prompt，使轮转决策与角色行为对齐
2. **推理增强变体 ModeratorLM-Think**：在轮转决策前插入 [[Chain-of-Thought]] 推理 trace，进一步提升角色一致性和召回率
3. **RolePlayConv 合成数据集**：约 75K 段多方语音对话（3-6 说话人），覆盖 125 种详细角色描述，含推理 trace 标注

## 问题背景

### 要解决的问题

多方语音对话（>2 人）中的轮转决策。双人场景的轮转靠停顿检测、VAD、句子完成标志即可，但多方场景存在重叠语音、动态 floor 竞争、协商式轮转分配，传统规则失效。

### 现有方法的局限

- [[Moshi]] 等全双工模型设计用于双人（dyadic）对话，直接用于多方场景 FP 率极高（0.47-0.66），无法区分"该我说"与"不该我说"
- 现有角色扮演语言代理（RLPA）聚焦文本对话中的人格模拟，未涉及实时语音轮转行为
- 缺少角色条件化的多方语音对话数据集

### 本文的动机

在多方场景中，用户对 voice agent 的期望随角色而变——有时需要主动主持，有时需要被动倾听。显式指定角色并让模型据此调节轮转行为，是比通用轮转模型更合理的建模方式。

## 方法详解

### 领域定位

ModeratorLM 属于 **多方语音对话轮转** 方向，与 [[Moshi]] 的全双工建模同属对话系统范畴，但核心差异在于：(1) 从双人扩展到多方，(2) 引入显式角色条件影响轮转策略，(3) 用 [[Chain-of-Thought]] 推理做轮转决策的显式推理链。

### 端到端数据流（先地图后街景）

ModeratorLM 的完整流水线：多通道音频输入 → **混音为单通道** → **Speech Encoder**（chunk 级语音嵌入）→ **线性投影层**（对齐到 LLM 嵌入空间）→ **Backbone LLM**（流式处理 chunk 序列 + 角色条件 system prompt + 文本转录）→ **输出**（turn-taking 控制 token + 可选文本回复 / 或空序列表示不接管）。

![[_resources/2606.13544/figures/fig1_moderatorlm_think.png]]

> **Figure 1**：ModeratorLM-Think 的 LLM 输入输出序列示例。Chunk 1 无推理产出；Chunk 2 产生推理 trace 但决定不接管；Chunk 3 推理后决定接管 floor 并生成回复。数据流从左到右：每个 chunk 的语音嵌入 + 转录文本按时序追加到 LLM 上下文中，LLM 在每个 chunk 后决策是否轮转。

下面逐个放大每个关键模块。

### Speech Encoder + 投影层

**为什么这样设计**：多方对话音频含重叠语音和动态 chunk 长度，需要编码器能独立处理变长 chunk 并输出固定维度嵌入。采用支持 variable lookahead 的 block-wise attention 编码器，训练时用 0.5-3 秒随机 chunk 长度提升对 chunk 时长变化的鲁棒性。

**怎么做**：多通道音频先混合为单通道，Speech Encoder 对每个 chunk 独立编码为 chunk 级嵌入，通过一个可训练的线性投影层映射到 LLM 的 embedding space。

### Backbone LLM（核心决策模块）

**为什么这样设计**：传统多方轮转用独立 [[VAD]] 模块 + 规则，但难以捕捉语义层面的轮转线索（如某人提到 agent 名字、话题转换到 agent 角色职责范围等）。将轮转决策完全委托给 Speech LLM 本身，利用 LLM 的语义理解能力做更精确的轮转判断。

**怎么做**：
- 角色通过 system prompt 注入（如 "You are a 42-year-old Indian CEO with confident and assertive communication style"）
- 每个 chunk 的语音嵌入 + 对应文本转录（含说话人标注）按时序追加到 LLM 上下文
- LLM 在每个 chunk 后输出两种信号之一：(a) turn-taking 控制 token + 文本回复，(b) 空序列（不接管）
- **ModeratorLM-Think** 变体在决策前先生成推理 trace（`<think>...</think>`），显式推理对话上下文与角色要求的匹配度

**具体例子**：给定角色"一个 21 岁的文学系学生，声音柔和、善于倾听"，当其他人在讨论热门话题时，ModeratorLM-Think 的推理 trace 可能是"他们正在热烈讨论，沉默让想法有空间发展，作为倾听者不需要打断"→ 输出空序列。但当有人直接问"你觉得呢？"时，推理变为"被直接问到了，角色设定虽是倾听者但被 address 时应该回应"→ 输出 turn-taking token + 回复。

### RolePlayConv 数据集构建

**为什么这样设计**：现有多方对话数据集（如 [[MELD]]）无角色条件标注，且大多是文本或固定场景。需要大规模、角色多样、含推理 trace 的多方语音对话数据。

**怎么做**：四阶段流水线——
1. **角色策划**：125 种详细角色描述（年龄/文化/沟通风格/职业等）
2. **对话生成**：Amazon Nova Pro LLM 生成 3-6 人对话，每轮限制 <15 词（模拟真实口语简短性）
3. **推理增强**：为所有 assistant 轮次 + 部分非 assistant 轮次生成推理 trace（stance selection + response planning + turn-taking considerations）
4. **语音合成**：[[Zonos]] TTS 逐轮合成，不同说话人池用于训练/测试分裂，沉默间隔从真实对话分布采样

总量约 75K 段对话，平均约 2 分钟/段。

### 训练流程

三阶段训练流水线：

**Stage 1 - 语音-LLM 对齐**：用约 90K 小时公开语音数据（[[VoxPopuli]] + [[MLS]] + [[Common Voice]] + [[People's Speech]]）做 ASR 任务，只更新投影层参数，其余冻结。目的：让投影层学会把语音嵌入映射到 LLM 能理解的空间。

**Stage 2 - 对话预训练**：在 [[AMI]] 和 [[Fisher]] 真实对话数据集上训练。通过轮换说话人模拟 assistant 角色：N 个参与者产生 N-1 个训练实例。目的：学习多方对话的基本轮转模式。

**Stage 3 - 角色条件微调**：专用 RolePlayConv 数据集微调，角色通过 system prompt 指定。Stage 2-3 均用 [[LoRA]] 微调（仅 13.4M 可训练参数），Speech Encoder 全程冻结。

Backbone LLM：ModeratorLM 用 Qwen3-4B-Instruct-2507，ModeratorLM-Think 用 Qwen3-4B-Thinking-2507。

### 推理流程

- 动态 chunk 分割（0.5-3s），在说话人边界处切分
- 每个 chunk 经 Speech Encoder → 投影 → 追加到 LLM 上下文
- LLM 基于累积上下文 + 角色 prompt 决策是否轮转
- 若轮转，生成文本回复（可接 streaming TTS 合成语音）
- ModeratorLM-Think 额外先采样推理 token（Temperature=0.7, TopP=0.8, TopK=20），再决策

## 关键结果

> 只列支撑主结论的核心表。完整表格见附录。

**核心证据**：Table 2 是全文最强证据，在两个评测集上同时对比 Moshi + MP-Baseline + ModeratorLM + ModeratorLM-Think。

| Model | NSF-1 @P | NSF-1 @R | NSF-1 @F1 | NSF-1 @FP | RPC @P | RPC @R | RPC @F1 | RPC @FP |
|-------|----------|----------|-----------|-----------|--------|--------|---------|---------|
| Moshi | 0.14 | 0.10 | 0.11 | 0.66 | 0.15 | 0.34 | 0.21 | 0.47 |
| MP-Baseline | 0.58 | 0.33 | 0.38 | 0.05 | 0.40 | 0.48 | 0.42 | 0.14 |
| ModeratorLM | 0.77 | 0.51 | 0.57 | 0.01 | 0.71 | 0.57 | 0.61 | 0.05 |
| **ModeratorLM-Think** | **0.81** | **0.74** | **0.76** | **0.01** | **0.79** | **0.82** | **0.79** | **0.03** |

**结论**：
- Moshi 在多方场景近乎失效（FP 0.47-0.66，precision 0.14-0.15），验证了双人模型不适用于多方的前提
- 角色条件微调（ModeratorLM vs MP-Baseline）大幅提升 precision（+0.19/+0.31）和 recall（+0.18/+0.09），同时降低 FP
- CoT 推理（ModeratorLM-Think vs ModeratorLM）在保持低 FP 的同时大幅提升 recall（+0.23/+0.25），F1 提升 0.19/0.18，说明显式推理帮助模型在"该说时说"的判断上显著改善

## 可复用的设计模式

1. **角色条件化轮转策略**：通过 system prompt 注入角色描述来调节对话行为，而非硬编码规则。适用于任何需要角色区分的语音对话场景。来自本文的角色条件微调设计。
2. **将轮转决策完全委托给 LLM**：不用独立 VAD 模块，而是让 Speech LLM 同时做语义理解 + 轮转决策。适用于需要语义级轮转判断的场景（如被直接 address 时才回应）。来自本文去除 VAD 的架构决策。
3. **CoT 推理增强决策质量**：在二分类决策（说/不说）前加入推理 trace，显著提升召回率且不损精度。适用于任何需要 reasoning 的流式决策场景。来自 ModeratorLM-Think 的设计。
4. **合成数据 + 多阶段训练弥补真实数据不足**：用 LLM 生成对话 → TTS 合成语音 → 角色条件微调，解决多方角色对话数据稀缺问题。适用于任何低资源对话场景。来自 RolePlayConv 构建流水线。

---

# 二、研究/审计层（附录）

> 独立容器：技术核验来源、可信度、claim 审查、批判都在这里，不与阅读层主线混杂。

## 📋 核验结论（技术元数据）

> 来源标注：`[已 verify §X / Eq.X / Tab.X / Fig.X]` 或 `[GitHub: <path>:<line>]`。

| 维度 | 核验后结论 | 来源 |
|------|-----------|------|
| LM 初始化 | warm-start from Qwen3-4B-Instruct (ModeratorLM) / Qwen3-4B-Thinking (ModeratorLM-Think) | [已 verify §3.1 "Qwen3-4B-Instruct-2507" / "Qwen3-4B-Thinking-2507"] |
| 训练 loss | Stage 1: ASR CE loss（仅投影层）; Stage 2-3: 对话+角色条件 LoRA 微调 loss | [已 verify §3.1 三阶段描述] |
| Tokenizer 架构 | Speech Encoder 产出 chunk 级嵌入 + 线性投影到 LLM space；文本转录作为 text token 并行输入 | [已 verify §2.1] |
| 多任务 | true：Stage 1 ASR 对齐 + Stage 2 对话预训练 + Stage 3 角色条件微调，但三阶段是顺序的非联合多任务 | [已 verify §3.1] |
| 训练数据 | Stage 1: ~90K h 公开语音(VoxPopuli+MLS+CommonVoice+People's Speech); Stage 2: AMI+Fisher; Stage 3: RolePlayConv ~75K 对话 | [已 verify §3.1] |
| 后训练 | 无（Stage 3 角色微调是 SFT，非 RLHF/DPO） | [已 verify §3.1 全文无 RL/DPO 描述] |
| 可训练参数 | 13.4M (LoRA) | [已 verify §3.1] |
| Speech Encoder | In-house，支持 variable lookahead block-wise attention | [已 verify §2.1 + §3.1] |

## 完整公式

论文中无显式数学公式——方法以系统架构和训练流程为主，无 loss 函数公式或算法伪代码。

## 完整图表

### Figure 1: ModeratorLM-Think 输入输出序列

![[_resources/2606.13544/figures/fig1_moderatorlm_think.png]]

**说明**：展示 ModeratorLM-Think 的三个连续 chunk 处理示例。Chunk 1 无推理产出（对话刚开始，无需决策）。Chunk 2 产生推理 trace 分析对话上下文但决定不轮转。Chunk 3 推理后决定接管 floor 并生成文本回复。核心信息：推理 trace 是 `<think>...</think>` 格式，轮转决策 token 和回复在推理后生成。

### Table 1: RolePlayConv 样例对话

角色设定："A 21-year-old Indian literature student with a soft, thoughtful voice who enjoys reading novels, sketching in quiet cafes, and listening more than speaking."

展示 4 位说话人 + assistant 的多方对话片段，每个 assistant 轮次包含 `[Thoughts]` 推理 trace。关键信息：推理 trace 包含 stance selection（当前应主动/被动）、response planning（如何回应）、turn-taking consideration（此时接管是否合理）。

### Table 2: 主实验结果

| Model | NOTSOFAR-1 @P | @R | @F1 | @A | @FP | @RM | RolePlayConv @P | @R | @F1 | @A | @FP | @RM |
|-------|------|------|------|------|------|------|------|------|------|------|------|------|
| Moshi | 0.14 | 0.10 | 0.11 | 0.21 | 0.66 | -- | 0.15 | 0.34 | 0.21 | 0.50 | 0.47 | -- |
| MP-Baseline | 0.58 | 0.33 | 0.38 | 0.69 | 0.05 | -- | 0.40 | 0.48 | 0.42 | 0.67 | 0.14 | -- |
| ModeratorLM | 0.77 | 0.51 | 0.57 | 0.77 | 0.01 | 0.08 | 0.71 | 0.57 | 0.61 | 0.76 | 0.05 | 0.14 |
| ModeratorLM-Think | **0.81** | **0.74** | **0.76** | **0.86** | **0.01** | **0.02** | **0.79** | **0.82** | **0.79** | **0.91** | **0.03** | **0.03** |

**说明**：NOTSOFAR-1 (NSF-1) 为真实会议录音；RolePlayConv 为合成数据测试集（零样本角色）。ModeratorLM-Think 在两个数据集上全面领先，FP 率极低（0.01-0.03），Reactive Miss Rate 仅 0.02-0.03。

### Table 3: LLM-as-a-Judge 角色忠实度

| Model | Turn-Taking [0,1] | Response [0,10] |
|-------|-------------------|-----------------|
| MP-Baseline | 0.58 | 4.6 |
| ModeratorLM | 0.68 | 6.9 |
| ModeratorLM-Think | **0.72** | **7.4** |

**说明**：Claude-Sonnet-3.5 作为 judge，评估轮转适当性和回复角色忠实度。与人类评估的 Spearman 相关 ρ = 0.87。ModeratorLM-Think 在两个维度均最优。

### Table 4: 角色定性对比

两种角色（权威型 vs 沉稳倾听型）给定相同对话上下文，产生截然不同的推理 trace 和轮转决策。权威角色决定打断以维持会议秩序；倾听角色认为"沉默让想法有空间"而选择不发言。关键信息：证明角色条件确实影响轮转行为，而非仅影响回复内容。

### Table 5: 消融实验 (RolePlayConv)

| Setup | ModeratorLM @P | @R | @A | ModeratorLM-Think @P | @R | @A |
|-------|------|------|------|------|------|------|
| Default (dynamic) | 0.71 | 0.57 | 0.76 | 0.79 | 0.82 | 0.91 |
| No Transcription | 0.42 | 0.14 | 0.57 | 0.39 | 0.42 | 0.57 |
| ASR Hypotheses | 0.68 | 0.56 | 0.76 | 0.75 | 0.80 | 0.90 |
| GT Thoughts | -- | -- | -- | 0.95 | 0.95 | 0.97 |
| Fixed (2s) | 0.88 | 0.78 | 0.88 | 0.82 | 0.82 | 0.91 |
| Turn-Fixed | 0.84 | 0.60 | 0.80 | 0.75 | 0.81 | 0.91 |

**说明**：
- **No Transcription** 导致严重退化（recall 从 0.57→0.14 / 0.82→0.42），说明转录文本是轮转决策的关键输入
- **ASR Hypotheses**（Kyutai-STT-2.6B, WER 6.7%）仅带来微小退化（-0.03/-0.02 precision），说明对 ASR 噪声有一定鲁棒性
- **GT Thoughts** 给 ModeratorLM-Think 带来近乎完美的性能（0.95/0.95/0.97），证明推理质量是性能瓶颈
- **Fixed 2s chunking** 使 ModeratorLM 性能大幅提升（+0.17/+0.21 P/R），但 ModeratorLM-Think 基本不变——说明无 CoT 的模型过拟合 chunk 长度信号，CoT 模型依靠推理 trace 而非 chunk 长度做决策

## 结果可信度分层

| 可信度 | 结果 | 理由 |
|--------|------|------|
| **中** | RolePlayConv 测试集上的 F1/Precision/Recall | 合成数据集（TTS 合成语音 + LLM 生成对话），与真实多方对话有 domain gap；但零样本角色 + 不同 LLM (QwQ-32B) 生成增加可信度 |
| **中** | NOTSOFAR-1 上的结果 | 真实会议录音，但 assistant 角色是后标注指定的（hybrid LLM-based ranking + human evaluation），非自然产生的角色；Teacher-forcing GT 上下文 |
| **中** | LLM-as-a-Judge 评估 | Claude-Sonnet-3.5 作为 judge，ρ=0.87 人类相关性较高，但评估规模仅 100 实例 |
| **低** | 与 Moshi 的对比 | Moshi 是 12.5 Hz 帧级预测的双人对话模型，用于多方场景本身不公平；±0.5s tolerance window 的设定是否合理未充分论证 |

## 核心 Claim 审查

1. **Paper Claim**："the first role-conditioned voice agent designed for multi-party conversational settings"
   **My Assessment**：在公开文献中确实未见先例将显式角色条件 + 语音模态 + 多方轮转结合的工作。但这一优先性声明的范围受限于"voice agent"——文本域已有相关工作（RoleLLM, CharacterGLM 等），只是未扩展到语音轮转。

2. **Paper Claim**："improved turn-taking precision by over 40% and recall by more than 70%"
   **My Assessment**：数字成立——在 RolePlayConv 上 ModeratorLM-Think vs MP-Baseline: precision 0.79 vs 0.40 (+97.5%), recall 0.82 vs 0.48 (+70.8%)。但 MP-Baseline 本身是弱基线（无角色条件的同架构模型），改进幅度受基线选择影响。

3. **Paper Claim**："explicit reasoning helps the model better interpret conversational context"
   **My Assessment**：GT Thoughts 实验（0.95/0.95/0.97）+ ModeratorLM-Think 对 chunk 策略不敏感（vs ModeratorLM 敏感）两个证据共同支撑此论点。但推理质量依赖于训练数据中的推理 trace 质量（LLM 生成），而非模型自身学会推理。

## 批判性思考

### 优点
1. **问题定义精准且务实**：多方语音轮转是真实产品痛点（如 Alexa 在多人家庭中误激活/误打断），角色条件化是合理的建模方式
2. **消融实验设计精到**：chunk 策略消融揭示了"模型过拟合 chunk 长度"的重要洞察，GT Thoughts 实验清晰定位了推理质量作为瓶颈
3. **双数据集评估**（合成 + 真实会议）增加了结论的鲁棒性
4. **CoT 使模型对 chunk 策略不敏感**是有价值的发现——意味着推理能力可以替代精细的音频分割

### 局限性
1. **依赖 GT 转录**：默认评估用 teacher-forcing GT 转录 + 说话人标注，实际部署需要实时 ASR + 说话人分离（speaker diarization），Table 5 "No Transcription" 的严重退化说明这是关键瓶颈
2. **合成数据 vs 真实对话 gap**：RolePlayConv 用 TTS 合成、对话由 LLM 生成、每轮 <15 词——真实多方对话有更复杂的重叠语音、backchannel、非语言线索，合成数据难以覆盖
3. **文本回复而非语音回复**：系统只生成文本，未端到端生成语音——离真正的 voice agent 还差 TTS 集成 + 延迟优化
4. **Speech Encoder 不开放**：in-house 编码器无法复现，对结果的可比性构成限制
5. **NOTSOFAR-1 的 assistant 角色定义不自然**：从真实会议中后标注 "哪个人是 assistant" 本身引入了评估偏差
6. **无延迟分析**：未报告推理延迟/RTF/首包延迟——对于 "real-time" 的声称缺少时间维度的实证

### 潜在改进方向
1. 集成实时 ASR + speaker diarization，评估端到端延迟
2. 在真实多方对话数据集（不仅是 NOTSOFAR 会议）上评测
3. 端到端语音生成（而非仅文本回复）
4. 探索更轻量的推理方案替代 full CoT（降低延迟）

### 可复现性评估
- [ ] 代码开源（未见）
- [ ] 预训练模型（未见）
- [x] 训练细节完整（三阶段流水线描述清晰）
- [ ] 数据集可获取（RolePlayConv 未见公开）
- [ ] Speech Encoder 可获取（in-house，不可复现）

---

# 三、知识系统层

## 🗺️ 在知识地图中的定位

- **所属领域**：[[Dialogue-领域总览]]（多方对话子方向）
- **技术路线**：[[TTS-SpeechLM-Dialogue关系]] 位置 ④（端到端语音交互系统），但当前仅做轮转决策（文本输出），未做语音生成——更准确说是位置 ③→④ 的过渡
- **核心问题**：[[TTS-核心挑战]] §挑战 8 对话重定义——多方轮转是双人全双工的自然延伸
- **相邻工作**：[[Moshi]]（双人全双工基线）/ [[OmniFlatten]]（双人轮转优化）/ [[IRAF]]（噪声鲁棒全双工）/ [[PersonaPlex]]

## 🔄 后续重估

- **2026-06-30**：初读。核心贡献在"角色条件化多方轮转"这一新问题定义，方法本身（LoRA 微调 + CoT）并不新颖。对 GT 转录的强依赖和缺少延迟分析是主要弱点。如果后续开源数据集 + 代码，复现价值会显著提升。作为 Interspeech 2026 论文，问题定义和实验设计水准合格，但离工业可用还有距离。

---

## 关联笔记

### 基于
- [[Moshi]]: 全双工对话基线，本文的核心对比对象
- [[Qwen3]]: backbone LLM 来源（Qwen3-4B-Instruct / Thinking）

### 对比
- [[OmniFlatten]]: 双人轮转优化，本文扩展到多方
- [[IRAF]]: 噪声鲁棒全双工，不同的研究角度

### 方法相关
- [[Chain-of-Thought]]: CoT 推理用于轮转决策
- [[LoRA]]: 参数高效微调
- [[Turn-taking]]: 核心概念
- [[VAD]]: 传统轮转方法，本文替代之

### 硬件/数据相关
- [[AMI]]: 对话预训练数据
- [[MFA]]: 强制对齐用于训练 chunk 分割

---

## 速查卡片

> [!summary] Adaptive Turn-Taking for Real-time Multi-Party Voice Agents
> - **核心**: 首个角色条件化多方语音轮转模型，通过 system prompt 注入角色影响轮转行为
> - **方法**: Speech Encoder + Qwen3-4B LoRA 微调 + 可选 CoT 推理；三阶段训练（ASR 对齐→对话预训练→角色微调）
> - **结果**: F1 0.76/0.79 (NSF-1/RPC)，较无角色基线提升 40%+ precision、70%+ recall；CoT 使模型对 chunk 策略不敏感（限定条件：GT 转录 teacher-forcing，非端到端）
> - **代码**: 未见开源

---

*笔记创建时间: 2026-06-30*
