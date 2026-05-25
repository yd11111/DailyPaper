---
title: TTS 趋势判断（2025 年中）
type: 阶段综述
domain: TTS
tags: [trends, tts, forecast, 2025]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 趋势判断（2025 年中）

> 基于本笔记库 39 篇 TTS 论文 + 相关 Codec / Speech-LLM / 全双工论文的综合判断。每个趋势标注**确信度**和**证据来源**。

## 趋势 1: Codec LM 路线确立为主流范式

**确信度**: 高 | **方向**: 已确立

2023 年 [[VALL-E]] 开创 Codec LM 范式后，2024-2025 几乎所有新系统都采用了这条路线或其变体：

- [[CosyVoice]] / [[CosyVoice2]] / [[CosyVoice3]]（阿里）
- [[SeedTTS]]（字节）
- [[IndexTTS2]]（B 站）
- [[GLM-TTS]]（智谱）
- [[FireRedTTS]]（小红书）
- [[Qwen3-TTS]]（阿里）

纯 mel 路线（[[FastSpeech2]] 系）和纯端到端路线（[[VITS]] 系）在 2025 年已少有新的重量级工作。

**但**: Codec LM 并非终局——tokenizer-free 路线（[[VoxCPM]]、[[LatentLM]]）正在挑战"语音必须先离散化"这一前提。

## 趋势 2: 训练数据从十万级跃升到百万级

**确信度**: 高 | **方向**: 加速中

| 时间 | 代表系统 | 数据量 |
|------|---------|--------|
| 2024 H1 | [[CosyVoice]] | 17.2 万小时 |
| 2024 H2 | [[CosyVoice2]] | 未公开（推测 30-50 万 h） |
| 2025 Q1 | [[VoxCPM]] | 180 万小时 |
| 2025 Q2 | [[CosyVoice3]] | 100 万小时 |
| 2025 Q2 | [[Qwen3-TTS]] | 500 万小时 |

**关键观察**:
- 数据规模在 12 个月内增长了 10-30 倍
- 但 [[GLM-TTS]] 用 10 万小时精筛数据达到可比效果，说明**数据质量的边际收益可能高于数据数量**
- 开源数据集（[[Emilia]] 10 万 h）与工业界（100-500 万 h）的差距在拉大，这对学术研究者不利

**预判**: 2025 下半年可能出现 1000 万小时级的系统，但 diminishing returns 效应会逐渐显现。

## 趋势 3: LLM-native TTS 正在浮现

**确信度**: 中 | **方向**: 早期但势头明确

从"用专门的声学模型做 TTS"到"直接用通用 LLM 做 TTS"的转变正在发生：

| 阶段 | 代表 | 特征 |
|------|------|------|
| 独立声学模型 | [[FastSpeech2]]、[[VITS]] | TTS 是独立系统 |
| Codec LM（专用 LM） | [[VALL-E]]、[[CosyVoice]] | 专门训练的 LM 做 TTS |
| 通用 LLM 微调 | [[GLM-TTS]]（GLM-4）| 在通用 LLM 上加 TTS 能力 |
| **LLM 直出** | [[Qwen3-TTS]] | 通用 LLM 直接输出 speech token |
| 统一 Speech LLM | [[StepAudio2.5]]、[[Moshi]] | 理解+生成在一个模型中 |

**意味着什么**:
- TTS 可能不再是独立产品，而是 LLM 的一个"模态"——就像 LLM 已经可以生成图像一样
- 对 TTS 研究者的影响：核心竞争力从"设计更好的声学模型"转向"设计更好的 speech token / 更好的多模态训练策略"

**不确定性**: 通用 LLM 做 TTS 的质量能否真正达到专用系统的水平？[[Qwen3-TTS]] 的结果初步积极，但还需要更多独立验证。

## 趋势 4: Flow Matching 取代 Diffusion 成为主要生成范式

**确信度**: 高 | **方向**: 已确立

2024-2025 的新系统几乎全部转向 [[Flow Matching]]：

- [[CosyVoice]] / [[CosyVoice2]] / [[CosyVoice3]]: Flow Matching 解码器
- [[F5-TTS]]: DiT + Flow Matching
- [[SemaVoice]]: Flow Matching on semantic latent
- [[GLM-TTS]]: 流式 Flow Matching
- [[NaturalSpeech3]]: 从 diffusion 转向 Flow Matching

**原因**: Flow Matching 训练更稳定（直接回归 vector field，不需要 noise schedule 设计）、推理更快（可用 1-step 或 few-step ODE solver）。

## 趋势 5: 流式 / 低延迟成为必备能力

**确信度**: 高 | **方向**: 从锦上添花变为硬需求

2024 之前的 TTS 论文很少讨论延迟；2025 年几乎每篇工业系统论文都报告流式能力：

- [[CosyVoice2]]: chunk-aware causal attention
- [[GLM-TTS]]: streaming decoder
- [[VoxCPM]]: 自回归连续预测天然支持流式

**驱动力**: 全双工对话系统（[[Moshi]]、[[OmniFlatten]]）要求 TTS 延迟 < 200ms，否则对话体验不可接受。

**未解**: 流式生成在 chunk 边界的不连续问题（能量跳变、韵律断裂）仍需更好的解决方案。

## 趋势 6: Tokenizer 设计仍是开放战场

**确信度**: 高 | **方向**: 无共识，百花齐放

| 方案 | 代表 | 思路 |
|------|------|------|
| RVQ 多层 | [[EnCodec]]、[[SoundStream]] | 经典但序列长 |
| RVQ 简化 | [[SPEAR-TTS]]（语义+声学） | 减少层数 |
| FSQ | [[CosyVoice2]] | 有限标量量化替代 VQ |
| 语义 token | [[HuBERT]] token | SSL 模型出的离散单元 |
| 跨模态 | [[FireRedTTS]] | 借用图像 VQ |
| 无 tokenizer | [[VoxCPM]] | 直接建模连续值 |

**关键问题**: tokenizer 的选择直接决定了系统的上限（信息瓶颈），但目前没有公认的"最优 tokenizer"——不同路线在不同场景下各有优劣。

## 趋势 7: 中国团队成为 TTS 研究的主力

**确信度**: 高 | **方向**: 已确立

2024-2025 年 TTS 领域的主要突破几乎全部来自中国团队：

| 团队 | 代表系统 |
|------|---------|
| 阿里通义 | [[CosyVoice]] 系列、[[Qwen3-TTS]] |
| 字节 | [[SeedTTS]]、Seed-TTS-eval |
| 智谱 | [[GLM-TTS]] |
| 面壁智能 | [[VoxCPM]] |
| 小红书 | [[FireRedTTS]] 系列 |
| B 站 | [[IndexTTS2]] |
| 阶跃星辰 | [[StepAudio2.5]] |
| 微软亚研 | [[VALL-E]] 系列、[[NaturalSpeech]] 系列 |

Google（AudioLM、SoundStorm）和 Meta（Voicebox）在 2023 年有重要贡献，但 2024-2025 年的发文节奏明显放缓。

## 趋势 8: 开源与闭源的博弈

**确信度**: 中 | **方向**: 开源在追赶但差距在拉大

| 阵营 | 代表 | 优势 | 劣势 |
|------|------|------|------|
| 开源 | [[F5-TTS]]、[[GPT-SoVITS]]、[[Fish-Speech]]、[[IndexTTS2]] | 社区活跃、可复现、可定制 | 数据量受限（最多 10 万 h 级开源数据） |
| 闭源/半开源 | [[Qwen3-TTS]]、[[SeedTTS]]、[[CosyVoice3]] | 百万级私有数据、完整工业 pipeline | 不可复现、不公开训练细节 |

**预判**: 开源社区的竞争力越来越依赖于开源数据集的规模。[[Emilia]] 是目前最大的开源 TTS 数据集（10 万 h），但与工业界 100-500 万 h 的差距意味着开源系统在数据维度上很难追平。

## 半年后回看清单

> 2025 年底回来检验这些预判：

- [ ] Codec LM 仍是主流？还是 tokenizer-free 路线崛起？
- [ ] 最大训练数据量是否突破 1000 万小时？diminishing returns 了吗？
- [ ] 是否出现了一个被广泛接受的 TTS benchmark（类似 ImageNet 的地位）？
- [ ] LLM-native TTS 是否已经在质量上匹配甚至超越专用系统？
- [ ] 流式 TTS 首包延迟是否突破 100ms？
- [ ] 是否有重要的非中国团队新系统？

---

## 我的整体判断

TTS 在 2024-2025 经历了一次**范式收敛**：从 2023 年的路线百花齐放（mel / E2E / Codec LM / Diffusion / Flow），收敛到 **Codec LM + Flow Matching** 作为主干、LLM-native 作为未来方向。

这个领域目前的主要瓶颈**不是模型架构**（主流架构已经足够好），而是：
1. **训练数据**（质和量）
2. **评测标准化**（缺少公认 benchmark）
3. **流式工程**（理论可行但工程细节多）

对研究者而言，最有价值的方向可能不是提出又一个新 TTS 架构，而是解决上述瓶颈问题——尤其是 tokenizer 设计和评测标准化。

---

*最后更新: 2026-05-25*
