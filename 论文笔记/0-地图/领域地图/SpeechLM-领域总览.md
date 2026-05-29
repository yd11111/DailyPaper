---
title: SpeechLM 领域总览（占位）
type: 领域地图
domain: SpeechLM
tags: [overview, speechlm, audio-llm, omni, full-duplex, domain-map, placeholder]
created: 2026-05-26
last_updated: 2026-05-26
status: placeholder
---

# SpeechLM 领域总览

> 🚧 **占位文档**。SpeechLM/Audio-LM/对话系统的核心关系已在 [[TTS-SpeechLM-Dialogue关系]] 完整建立，本文先以 stub 形式存在，等需要从 SpeechLM 自身视角组织（而非 TTS 视角）时再扩展。

## 领域定义与范围

Speech Language Model（SpeechLM）= 端到端处理和生成语音的自回归基础模型（参 Cui 2024 #3 定义）。本领域是 **TTS / ASR / 对话** 的统一上层框架，**全双工 / Omni / Audio LLM** 也都收归于此。

**用户长期关注的方向**（参 MEMORY.md user_research_focus）：全双工、Omni、Audio LLM、SpeechLM —— 大部分在本领域。

## 与已有笔记的关系

| 视角 | 文档 |
|------|------|
| **TTS 视角看 SpeechLM**（TTS 作为 SpeechLM 输出模块）| [[TTS-SpeechLM-Dialogue关系]] §2 / §6 |
| **对话系统视角看 SpeechLM**（11 子类分类 + 全双工） | [[TTS-SpeechLM-Dialogue关系]] §4 |
| **LLM-Speech 集成视角**（三类集成方式）| [[TTS-SpeechLM-Dialogue关系]] §3 |
| **Audio-LM vs SpeechLM 边界**（speech-only vs 跨域）| [[TTS-SpeechLM-Dialogue关系]] §5 |
| **SpeechLM 自身视角**（不限定 TTS / 对话角度）| 本文（待扩展）|

短期可以**只看 [[TTS-SpeechLM-Dialogue关系]]**，因为绝大多数 SpeechLM 讨论都已经在那张关系图覆盖。

## 当前主流 SpeechLM 速览

详见 [[TTS-SpeechLM-Dialogue关系]] §2.3 / §4。简表（按 Cui 2024 / Yang 2025 / Ji 2024 整合）：

### 端到端 SpeechLM

| 模型 | 路线 | 特征 |
|---|---|---|
| GSLM | 纯语音 AR | 首个 SpeechLM 概念验证 |
| AudioLM | 分层 token 级联 | 语义 + 声学两阶段 |
| SpeechGPT | LLaMA + HuBERT | 链式模态微调 |
| AudioPaLM | PaLM-2 + token | 验证 scaling 收益 |
| Spirit-LM | LLaMA-2 + interleaved | 词级 interleaving |
| **Moshi** | Mimi + 并行生成 | **全双工标志** |
| GLM-4-Voice | Whisper VQ + GLM-4-9B | 中英文对话 |
| Mini-Omni | Whisper + Qwen2 + SNAC | 并行 7 层 token |
| Freeze-Omni | 冻结 LLM + adapter | 训练成本低 |
| OmniFlatten / SyncLLM | 训练含文本，推理纯语音 | 间接端到端 |
| LauraGPT | Conformer + Qwen + EnCodec | 工业级 |
| [[StepAudio2.5]] | 统一 Speech LLM | ASR + TTS + 实时交互 |

### Audio LLM（跨域：speech + music + sound）

| 模型 | 特征 |
|---|---|
| Qwen-Audio / Qwen2-Audio | 多任务 Audio LLM |
| SALMONN | 双编码器 + Q-Former |
| LTU | LLaMA + 大规模音频问答 |
| GAMA | Audio Q-Former + 多层聚合 |
| Pengi | "frames all audio tasks as text-generation" |
| Audio Flamingo 1-3 | in-context learning |
| AudioGPT | LLM agent 调度音频模型 |

## 四大研究维度

```
SpeechLM 四大维度
├── 表示选择              ← [[TTS-表示层地图]]
│   语义/声学/混合 token + 因果性
├── 模型架构
│   端到端 vs 级联 / 自回归 vs masked / 单流 vs 多流
├── 训练范式
│   预训练 → 指令微调 → DPO/RLHF
└── 评估能力              ← [[TTS-评测体系]]
    理解 + 生成 + 对话 + 多模态
```

## 核心挑战（与 TTS 相关挑战 + 独有挑战）

**与 [[TTS-核心挑战]] 共享**：
- 挑战 3 流式低延迟
- 挑战 4 数据规模与质量
- 挑战 5 Codec/Token 设计
- 挑战 6 评估方法论
- 挑战 8 对话系统重定义 TTS（实际是 SpeechLM 视角问题）

**SpeechLM 独有**（待扩展为独立笔记）：
- **跨模态 instruction following 评测**：缺乏统一基准
- **长上下文音频理解**：当前模型多在 30s 级，跨分钟级表现差
- **多任务能力 vs 单任务专用模型的差距**：通用 SpeechLM 在单一任务上常弱于专用模型
- **真端到端 vs 训练时级联**（Ji 2024 #8 警示）
- **全双工评估缺失**：interrupt latency 等无标准化指标

## 与相邻领域的关系

```
                    Audio-LM（跨域：含 music / sound）
                            ↕ 范围扩展
                    SpeechLM（speech-only：ASR + TTS + 对话）
                    ┌────────┼────────┐
                    ↓        ↓        ↓
                  ASR      TTS    Dialogue
                                     ↓
                                  全双工（Full-duplex）
                                     ↓
                                Omni（多模态对话）
```

## 待完善清单

按 [[0-架构与重构方案]] §6 实施时优先级：

- [ ] 读最新 SpeechLM / Audio LLM 综述（Cui 2024 已读，可补 Yang 2025 / Audio-LM 2025 之外的新综述）
- [ ] 写 `0-地图/技术路线/SpeechLM-技术路线图.md`（端到端 / 级联 / 隐层 / token 四类的详细对比）
- [ ] 写 `0-地图/核心问题/SpeechLM-核心挑战.md`（独有挑战部分）
- [ ] 写 `0-代表模型谱系/SpeechLM-代表模型谱系.md`（按训练范式 + 表示层组织）
- [ ] 单独建 `0-地图/领域地图/全双工-领域总览.md`（用户长期关注，应独立成线）
- [ ] 单独建 `0-地图/领域地图/Omni-领域总览.md`
- [ ] 把现有 `5-Speech-LLM与AudioLM/` 下论文按 [[笔记frontmatter规范]] 加 routes/problems 标签

## 相关笔记

- [[TTS-SpeechLM-Dialogue关系]] — SpeechLM 完整全景（本领域主要参考）
- [[TTS-表示层地图]] — SpeechLM 的底层基座（Tokenizer）
- [[TTS-核心挑战]] 挑战 8 — 对话系统重定义 TTS
- [[TTS-趋势判断]] 趋势 3 — LLM-native TTS / 统一 Speech LLM
- [[StepAudio2.5]] — 用户长期关注的代表系统
- [[待回填地图]] — 新 SpeechLM 论文进入后的回流入口

---

*占位 v1，2026-05-26 — 短期可只看 [[TTS-SpeechLM-Dialogue关系]]。等"全双工 / Omni" 独立成线时再单独建笔记。*
