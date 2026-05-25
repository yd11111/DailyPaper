---
title: TTS 趋势判断（2026 年中）
type: 阶段综述
domain: TTS
tags: [trends, tts, forecast, 2026]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 趋势判断（2026 年中）

> 基于本笔记库 39 篇 TTS 论文 + 18 篇 2026 年新增工作 + 相关 Codec / Speech-LLM / 全双工论文的综合判断。每个趋势标注**确信度**和**证据来源**。

## 上期（2025 年中）预判回顾

先检验上一轮趋势判断的准确性：

| 预判 | 结果 | 评分 |
|------|------|------|
| Codec LM 仍是主流？ | ✅ 仍然是，但 Tokenizer-free 路线加速崛起 | 基本正确 |
| 最大训练数据量突破 1000 万 h？ | ❌ 未突破，[[Qwen3-TTS]] 500 万 h 是目前已知最大 | 过于乐观 |
| 出现公认 TTS benchmark？ | ⚠️ Seed-TTS-eval 采用率最高但仍非"ImageNet 级"公认标准 | 进展有限 |
| LLM-native TTS 匹配专用系统？ | ✅ [[Qwen3-TTS]] 初步证实，但独立验证仍不充分 | 趋势正确 |
| 流式首包延迟突破 100ms？ | ❌ 200ms 仍是主流水平，<100ms 未见可靠报告 | 过于乐观 |
| 出现重要的非中国团队新系统？ | ⚠️ [[Raon-OpenTTS]]（韩国）出现，但影响力有限；Google/Meta 继续沉寂 | 部分正确 |

**反思**：上一轮对"突破性进展"的时间节点普遍乐观了 6-12 个月。技术路线的判断（Codec LM 主流、LLM-native 浮现）基本准确，但工程落地速度（延迟、数据规模天花板）比预期慢。

---

## 趋势 1: Codec LM + Flow Matching 仍是工业主流

**确信度**: 高 | **方向**: 稳定，但不再独大

2025 年中判断"范式收敛到 Codec LM + Flow Matching"仍然成立。2026 年主要工业系统继续使用这一架构：

- [[CosyVoice3]]（阿里，100 万 h）
- [[GLM-TTS]]（智谱，10 万 h 精筛）
- [[IndexTTS2]]（B 站，100 层 AR）

**但 2026 的变化是**：Codec LM 不再是"唯一可选"——LLM-native 路线（趋势 3）和 Tokenizer-free 路线（趋势 2）开始分流关注度和资源。"主流"≠"唯一"。

## 趋势 2: Tokenizer-free / 连续 AR 从探索变为活跃方向

**确信度**: 中高 | **方向**: 加速增长

这是 2026 年最显著的技术演变。从 2024 年 MELLE 的单点探索，到 2026 年已有 5+ 篇工作：

| 模型 | 年份 | 团队 | 思路 |
|------|------|------|------|
| [[MELLE]] | 2024 | 微软 | 首个无 VQ 的 AR TTS，连续 mel + latent sampling |
| [[VoxCPM]] | 2025 | 面壁智能 | LLM 直接预测连续 mel 帧，180 万 h |
| [[FELLE]] | 2025 | 微软 | MELLE + token-wise Flow Matching 精修 |
| [[CLEAR]] | 2025 | 港中大/华为 | 连续 latent AR，专注低延迟 |
| [[SemaVoice]] | 2026 | — | SFM 对齐 + patch-wise diffusion head |

**核心优势**：避开 codec 设计这个仍无共识的"开放战场"（见趋势 6）。

**尚未证实**：连续 AR 的 scaling 能力是否匹配离散 token 路线。VoxCPM 180 万 h 是目前最大规模实验，但与 Qwen3-TTS 500 万 h 仍有差距。训练稳定性（连续值回归的方差问题）也需要更多验证。

## 趋势 3: LLM-native TTS 从"浮现"变为"已验证"

**确信度**: 高 | **方向**: 已验证，但"全面取代专用系统"尚早

2025 年中判断"LLM-native TTS 正在浮现"——到 2026 年中，这一趋势已从浮现变为初步验证：

| 阶段 | 代表 | 年份 | 特征 |
|------|------|------|------|
| 独立声学模型 | [[FastSpeech2]]、[[VITS]] | 2019-21 | TTS 是独立系统 |
| 专用 Codec LM | [[VALL-E]]、[[CosyVoice]] | 2023-24 | 专门训练的 LM 做 TTS |
| 通用 LLM 微调 | [[GLM-TTS]]（GLM-4） | 2024 | 在通用 LLM 上加 TTS 能力 |
| **LLM 直出** | [[Qwen3-TTS]] | 2026 | 通用 LLM 直接输出 speech token，500 万 h |
| **统一 Speech LLM** | [[StepAudio2.5]] | 2026 | ASR + TTS + 实时交互在一个模型中 |

**意味着什么**：
- TTS 正在从独立产品变成 LLM 的一个"模态能力"
- 核心竞争力从"声学模型架构"转向"speech token 设计"和"多模态训练策略"
- 对专用 TTS 系统的需求可能不会消失（延迟、定制化场景），但通用场景会被 LLM-native 蚕食

**不确定性**：[[Qwen3-TTS]] 和 [[StepAudio2.5]] 的质量细节尚未被充分独立验证。"LLM 理解语义 → 自动推断韵律"的上限也不清楚。

## 趋势 4: 自然语言指令可控（Instruction TTS）浮现

**确信度**: 中 | **方向**: 早期，ICLR 2026 有标志性工作

2026 年出现了从"给条件"到"说人话"的可控性范式转变：

| 控制方式 | 代表 | 要求 | 年份 |
|---------|------|------|------|
| 显式条件（duration/pitch） | [[FastSpeech2]] | 需要 F0 标注 | 2020 |
| 参考音频 | [[MegaTTS]]、[[CosyVoice]] | 需要参考音频 | 2023-24 |
| **自然语言描述** | [[FlexiVoice]] | 只需文字描述 | 2026 (ICLR) |
| **LLM 语义理解** | [[Qwen3-TTS]] | 自动推断 | 2026 |

[[FlexiVoice]]（ICLR 2026）明确验证了"用文字描述替代参考音频"的可行性。这意味着：
- 零样本克隆可能不需要参考音频——描述就够了
- 可控性的评测需要新维度（指令遵循率）

**尚未验证**：自然语言指令的控制精度是否足以满足专业场景（录音棚级配音、情感剧本演绎）。

## 趋势 5: Flow Matching 地位稳固

**确信度**: 高 | **方向**: 已确立，无挑战者

2025 年的判断完全成立。2026 年的新系统继续使用 Flow Matching 作为核心生成范式。没有出现能替代 Flow Matching 的新生成模型。

唯一值得注意的变化是 **SSM 的出现**：[[MambaVoiceCloning]]（ICLR 2026）用 Mamba SSM 替代 Transformer 做 diffusion TTS 的条件路径，线性复杂度。这不是替代 Flow Matching，而是替代 Flow Matching 里的 Transformer backbone——两者是正交的。

## 趋势 6: Tokenizer 设计仍是开放战场

**确信度**: 高 | **方向**: 仍无共识，但 ICLR 2026 有多篇新探索

2025 年中判断"百花齐放，尚无共识"不变，且 ICLR 2026 集中出现了一波 codec 新探索：

| 方案 | 代表 | 年份 | 思路 |
|------|------|------|------|
| RVQ 多层 | [[EnCodec]]、[[SoundStream]] | 2022 | 经典但序列长 |
| FSQ | [[CosyVoice2]] | 2024 | 有限标量量化替代 VQ |
| 无 tokenizer | [[VoxCPM]] | 2025 | 直接建模连续值 |
| **动态帧率** | [[FlexiCodec]] | 2026 (ICLR) | 帧率随内容复杂度自适应 |
| **噪声鲁棒** | [[StableToken]] | 2026 (ICLR) | 对输入噪声更鲁棒的 tokenizer |
| **Diffusion Autoencoder** | [[ScalingSpeechTokenizers]] | 2026 (ICLR) | 用 diffusion autoencoder 替代 VQ |

**关键问题演进**：2025 年问的是"哪种 tokenizer 最好"，2026 年开始有人问"是否还需要 tokenizer"（Tokenizer-free 路线的崛起使这个问题本身被质疑）。

## 趋势 7: 中国团队继续主导，但韩国团队出现

**确信度**: 高 | **方向**: 格局基本不变

2024-2026 年 TTS 领域的主力仍然是中国团队（阿里、字节、智谱、面壁、小红书、B 站、阶跃星辰、微软亚研）。

**2026 年变化**：
- [[Raon-OpenTTS]]（KRAFTON / 首尔大 / KAIST 联合）是 2026 年少见的非中国团队重要工作，强调开放数据 + 开放模型
- Google、Meta 在 TTS 领域的发文节奏继续放缓（AudioLM/SoundStorm/Voicebox 后未见重量级后续）
- ICLR 2026 出现了多篇 codec 和可控 TTS 的工作（FlexiCodec、StableToken、FlexiVoice、MambaVoiceCloning），来源更多元

## 趋势 8: Speech RLHF 进入语音领域

**确信度**: 中 | **方向**: 早期，但方向清晰

[[GSRM]]（2026）提出 generative speech reward model，将 RLHF 对齐技术引入语音领域：

- 训练一个可以对生成语音打分的 reward model
- 用 RL 优化 TTS 模型使其输出被 reward model 评分更高
- 潜在用途：替代人工 MOS、改善韵律自然度、提升主观质量

这与 [[SeedTTS]]（2024）的 RL 后训练思路一脉相承，但 GSRM 提供了更通用的 reward model 框架。

**尚未验证**：speech reward model 的泛化能力（跨语言、跨系统）；RL 训练是否稳定；与人工评价的真实相关性。

## 趋势 9: 开源与闭源差距继续拉大

**确信度**: 中高 | **方向**: 差距在扩大

| 阵营 | 代表（2026） | 数据规模 | 优势 | 劣势 |
|------|------------|---------|------|------|
| 开源 | [[F5-TTS]]、[[GPT-SoVITS]]、[[Fish-Speech]]、[[IndexTTS2]]、[[Raon-OpenTTS]] | 最多 10 万 h 级（[[Emilia]]） | 社区活跃、可复现、可定制 | 数据量受限 |
| 闭源/半开源 | [[Qwen3-TTS]]、[[SeedTTS]]、[[CosyVoice3]]、[[VoxCPM]]、[[StepAudio2.5]] | 100-500 万 h | 完整工业 pipeline | 不可复现 |

**2026 的新变量**：[[Raon-OpenTTS]] 明确以"开放数据 + 开放模型"为卖点，说明学术社区开始有意识地对抗数据壁垒。但 Emilia 10 万 h vs 工业界 500 万 h 的鸿沟短期内无法弥合。

---

## 半年后回看清单

> 2026 年底回来检验这些预判：

- [ ] Tokenizer-free 路线是否出现了与 Codec LM 正面对比胜出的案例？
- [ ] LLM-native TTS 是否已有独立第三方大规模评测验证？
- [ ] Instruction TTS（FlexiVoice 后续）是否有更多工作跟进？
- [ ] SSM（Mamba）在 TTS 中是否有后续发展，还是一篇就沉了？
- [ ] Speech RLHF 是否被主流系统采用？
- [ ] 是否出现了千万小时级训练的系统？
- [ ] 流式 TTS 首包延迟是否突破 100ms？
- [ ] 开源最大数据集是否突破 100 万 h？

---

## 我的整体判断

TTS 在 2025-2026 经历了**从范式收敛到多路线并存**的转变：

- 2024-2025：收敛到 Codec LM + Flow Matching
- 2026：Codec LM 仍是主流，但 Tokenizer-free（连续 AR）和 LLM-native 两条新路线加速发展，不再是"一条路线统治一切"

这个领域目前的主要瓶颈**已从模型架构转向三个方向**：
1. **评测标准化 + 指令跟随评测**：Instruction TTS 和 LLM-native TTS 需要新的评测维度，传统三件套不够了
2. **训练数据的质与量**：数据规模的 diminishing returns 效应开始显现（GLM-TTS 10 万 h ≈ CosyVoice3 100 万 h），数据质量 > 数据量的共识正在形成
3. **Tokenizer 设计 vs Tokenizer-free**：这个根本性分歧可能需要 1-2 年才能见分晓

对研究者而言，2026 年最有价值的方向可能是：
- **Instruction TTS / 可控性**（ICLR 2026 已有多篇，方向明确但仍在早期）
- **Speech RLHF / 自动评测**（替代人工 MOS 的刚需）
- **Tokenizer-free 路线的 scaling 实验**（需要更多大规模验证）

---

*最后更新: 2026-05-25*
