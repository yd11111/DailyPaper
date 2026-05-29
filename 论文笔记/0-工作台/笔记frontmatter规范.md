---
title: 笔记 frontmatter 规范
type: 工作台
purpose: 标准化论文/概念/地图笔记的 frontmatter，使每篇笔记天然带"它会更新哪张地图"的信息
created: 2026-05-26
last_updated: 2026-05-26
status: draft-v1
---

# 笔记 frontmatter 规范

## 0. 用途与生效范围

把每篇笔记的 frontmatter 标准化，让 `DailyPapers → 论文层 → 概念层 → 地图层` 的回流可以在结构化数据上做（而不是靠 Claude 每次重新读全文判断）。

**生效范围与执行节奏**：
- **新论文笔记**：从规范生效之日起，paper-reader / daily-papers-notes 写入的笔记必须按此规范填 frontmatter
- **历史论文笔记（45+ 篇 TTS + 其他领域）**：**不强制立即回填**。挑高重要性论文（如 [[VALL-E]] / [[CosyVoice]] 系列 / [[NaturalSpeech3]] / [[StepAudio2.5]]）优先补；普通笔记按需补
- **地图笔记 (`0-*`)**：本规范不强制——地图笔记有自己的更松散结构

## 1. 论文笔记 frontmatter 标准（最重要）

### 1.1 完整模板

```yaml
---
title: "完整论文标题"
method_name: "ShortName"                 # 用于 wikilink，模型名/方法名
authors: [Author1, Author2]
year: 2026
venue: arXiv / ICLR 2026 / EMNLP 2025 / ...
arxiv_id: "2510.06927"
pdf_path: "assets/papers/ShortName.pdf"  # 可选

# === 领域标签（必填）===
domain: TTS                              # 一级领域，单选
subdomain: speech-llm                    # 二级领域，可选
tags: [tts, speech-llm, controllable-tts]

# === 知识地图联动字段（必填）===
routes: [codec-lm-tts, instruction-tts]  # 技术路线，多选，见 §2
problems: [zero-shot-cloning, evaluation]# 解决/触及的核心问题，多选，见 §3
representations: [acoustic-token]        # 涉及的表示空间，多选，见 §4
related_maps:                            # 应反哺的地图笔记
  - "[[TTS-技术路线图]]"
  - "[[TTS-评测体系]]"
related_surveys:                         # 关联综述（可选）
  - "[[ControllableTTS-Survey]]"

# === 评估与状态（必填）===
evidence_level: medium                   # high / medium / low，见 §5
maturity: emerging                       # mature / emerging / exploratory，见 §6
last_repositioned: 2026-05-26            # 最近一次重新评估的日期

# === 回流状态（自动维护）===
map_backfilled: false                    # 是否已回填到地图
backfilled_at:                           # 回填完成时间

# === 通用元数据 ===
created: 2026-05-26
last_updated: 2026-05-26
---
```

### 1.2 最小可用版本

如果一时不想填全，最少必须填：

```yaml
---
title: ""
method_name: ""
year: 2026
domain: TTS
routes: [...]
problems: [...]
related_maps: [...]
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-26
---
```

其他字段可后续补。

## 2. `routes` 枚举值（技术路线标签）

### TTS 路线

| 值 | 含义 | 对应地图章节 |
|---|---|---|
| `acoustic-feature-tts` | Mel / Spectrogram 中介路线 | [[TTS-技术路线图]] 路线 1 |
| `end-to-end-latent-tts` | VAE/Flow 端到端 | [[TTS-技术路线图]] 路线 3 |
| `codec-lm-tts` | 离散 codec token + LM | [[TTS-技术路线图]] 路线 2 |
| `diffusion-flow-tts` | 连续 latent + Diffusion/Flow | [[TTS-技术路线图]] 路线 4 |
| `tokenizer-free-tts` | 跳过 codec 的连续 AR | [[TTS-技术路线图]] 路线 5 |
| `controllable-tts` | 可控合成（条件输入） | [[TTS-技术路线图]] §控制范式 |
| `instruction-tts` | 自然语言指令控制 | [[TTS-技术路线图]] §控制范式（策略 4） |
| `speech-llm-tts` | LLM-native TTS / 统一 SpeechLM | [[TTS-SpeechLM-Dialogue关系]] 位置 ② |
| `streaming-tts` | 流式低延迟 | [[TTS-核心挑战]] 挑战 3 |
| `dialogue-tts` | 对话系统中的 TTS | [[TTS-技术路线图]] §对话系统中的 TTS |

### ASR 路线（待 ASR-领域总览 定稿后补）

| 值 | 含义 |
|---|---|
| `ctc-asr` | CTC 路线 |
| `rnnt-asr` | RNN-T 路线 |
| `encoder-decoder-asr` | Encoder-Decoder（Whisper 类） |
| `llm-asr` | LLM-based ASR |
| `streaming-asr` | 流式 ASR |

### Codec / Tokenizer 路线（待补）

| 值 | 含义 |
|---|---|
| `rvq-codec` | Residual VQ |
| `fsq-codec` | Finite Scalar Quantization |
| `ssl-distilled-codec` | SSL 蒸馏混合 codec |
| `multi-domain-codec` | 多域（speech+music+audio） |

## 3. `problems` 枚举值（核心问题标签）

| 值 | 含义 | 对应地图章节 |
|---|---|---|
| `zero-shot-cloning` | 零样本说话人克隆 | [[TTS-核心挑战]] 挑战 1 |
| `speaker-similarity` | 说话人相似度提升（SECS/SIM-o） | 挑战 1 |
| `prosody-control` | 韵律控制 | 挑战 2 |
| `emotion-style-control` | 情感与风格控制 | 挑战 2 |
| `long-form-stability` | 长文本稳定性 | 挑战 2 / 挑战 3 |
| `latency` | 延迟（首包/RTF） | 挑战 3 |
| `streaming` | 流式生成能力 | 挑战 3 |
| `multilinguality` | 多语言能力 | 挑战 1 (Multilingual Cloning) |
| `evaluation` | 评估方法论 | 挑战 6 |
| `instruction-following` | 指令跟随能力 | 挑战 2 |
| `data-scale` | 数据规模问题 | 挑战 4 |
| `codec-design` | Codec/Token 设计 | 挑战 5 |
| `safety-watermark` | 安全/水印/防伪 | 挑战 7 |
| `dialogue-integration` | 对话系统集成 | 挑战 8 |
| `interrupt-handling` | 打断处理 | 挑战 8 |
| `state-consistency` | 多轮状态保持 | 挑战 8 |
| `hidden-state-driven` | 隐状态驱动 TTS | 挑战 8 |

## 4. `representations` 枚举值（表示层标签）

| 值 | 含义 | 对应地图章节 |
|---|---|---|
| `mel-spectrogram` | Mel 频谱 | [[TTS-表示层地图]] §1 |
| `linear-spectrogram` | 线性频谱 | §1 |
| `continuous-latent` | 自训练 VAE latent | §1 |
| `semantic-token` | SSL 离散 token（HuBERT/WavLM 等） | §1 / §5.1 |
| `acoustic-token` | 神经 codec token（EnCodec/DAC/SoundStream） | §1 / §5.1 |
| `mixed-token` | 语义+声学混合 token（SpeechTokenizer/Mimi/X-Codec） | §5.1 |
| `audio-token` | 通用音频 token（笼统称呼） | §1 |
| `tokenizer-free-continuous` | 跳过 token 的连续值预测 | §5.2 |
| `unified-token-space` | LLM-native 统一 token 空间 | §1 |
| `factorized-latent` | 因子化解耦 latent（FACodec 类） | §2 辅助维度 |

## 5. `evidence_level`（证据强度）

| 值 | 判定标准 |
|---|---|
| `high` | 有大规模独立第三方评测验证，且 codebase 开源可复现 |
| `medium` | 单一团队声称的指标，无独立验证，但方法学可信 |
| `low` | 仅 demo / blog post，或评测协议不透明 |

**判定时务必区分**："方法很优秀" ≠ "证据 high"。Position #11 (#11) 指出大量论文跨论文不可比，缺乏 community-agreed protocol——所以**多数 2024-2026 论文应保守标 medium**。

## 6. `maturity`（技术路线成熟度）

| 值 | 判定标准 |
|---|---|
| `mature` | 工业系统大量部署，社区共识形成 |
| `emerging` | 已有标志性工作，但还在快速演化 |
| `exploratory` | 单点探索，尚无第二个独立验证 |

参考点：
- HiFi-GAN / FastSpeech 2 → mature
- VALL-E / CosyVoice 系列 → emerging（虽广泛使用但 codec 设计未收敛）
- Tokenizer-free（MELLE/VoxCPM）→ exploratory（[[TTS-趋势判断]] 趋势 2 表述为"从单点变活跃"，仍未脱 exploratory）
- LLM-native TTS（Qwen3-TTS/StepAudio2.5）→ exploratory ~ emerging 边界

## 7. 论文正文区块（在 frontmatter 之外）

paper-reader 写入论文笔记的正文要额外包含**两个固定区块**（与现有「📌 一句话 / 🛠 核心方法 / ...」并列）：

### 区块 A：在知识地图中的定位

```md
## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]
- **技术路线**：[[TTS-技术路线图]] §<路线名/章节>
- **核心问题**：[[TTS-核心挑战]] §<挑战名>；[[TTS-评测体系]] §<指标名>
- **表示层位置**：[[TTS-表示层地图]] §<表示类型>
- **在 SpeechLM/对话框架内的位置**：[[TTS-SpeechLM-Dialogue关系]] 位置 ① / ② / ③ / ④（按需）
- **相邻工作**：[[ModelA]] / [[ModelB]] / [[ModelC]]
```

### 区块 B：后续重估

```md
## 🔄 后续重估

- **2026-05-26**：初读。认为其主要贡献在 X，未解决 Y。
- **<日期>**：<触发原因>——比如读了 [[ModelZ]] 后重新认为其...
- **<日期>**：...
```

每次重估都加一行，**不删旧条目**——重估的演进本身就是知识。

## 8. 概念笔记 frontmatter（简化版）

```yaml
---
title: "Voice Cloning 术语标准化"
type: 概念
domain: TTS
tags: [voice-cloning, terminology, ...]
created: 2026-05-26
last_updated: 2026-05-26
based_on:                                # 该概念来源的综述/论文
  - "Azzuni-2025-arXiv:2505.00579"
related_maps:                            # 应反哺哪些地图
  - "[[TTS-核心挑战]]"
  - "[[TTS-评测体系]]"
---
```

## 9. 地图笔记 frontmatter（已有结构，不强制改）

地图笔记（`0-*`）保留各自现有结构。建议补一个字段：

```yaml
last_review: 2026-05-26                  # 最近一次基于新论文复审的日期
review_trigger:                          # 触发复审的事件
  - "11 篇综述综合 (2026-05-26)"
```

## 10. paper-reader skill 改造（待办）

要让规范真正生效，需要改 `paper-reader` skill：

- [ ] 在 prompt 模板里加 routes/problems/representations 字段（参考本规范的枚举值）
- [ ] 在论文笔记模板里加「🗺️ 在知识地图中的定位」和「🔄 后续重估」两个区块
- [ ] 在 paper-reader 完成笔记保存后，自动把该论文追加到 [[待回填地图]]
- [ ] 在 daily-papers-notes（批量笔记）同步加入这些字段

详见 [[0-架构与重构方案]] §下一步实施计划。

## 11. 相关文档

- [[待回填地图]] — 工作台，新论文进入后写入这里等待人工 review 回填
- [[0-架构与重构方案]] — 0-* 目录的重构提案
- [[TTS-11篇综述综合-2026-05]] §6 — 本次工作的执行索引

---

*v1，2026-05-26 — 首版规范。后续随实际使用调整字段和枚举值。*
