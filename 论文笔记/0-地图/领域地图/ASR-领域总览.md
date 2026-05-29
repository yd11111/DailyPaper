---
title: ASR 领域总览（占位）
type: 领域地图
domain: ASR
tags: [overview, asr, domain-map, placeholder]
created: 2026-05-26
last_updated: 2026-05-26
status: placeholder
---

# ASR 领域总览

> 🚧 **占位文档**。结构与 [[TTS-领域总览]] 同构，等待按相同方法（读综述 + 综合现有论文笔记）填实。当前为最小骨架。

## 领域定义与范围

自动语音识别（Automatic Speech Recognition, ASR）将语音波形转化为文本序列。现代 ASR 的核心追求已从"听清字"演进到 **多语种泛化 / 噪声鲁棒 / 流式低延迟 / 长音频上下文 / 与 LLM 融合** 等多维同时达标。

本笔记库覆盖范围（待充实）：

| 层级 | 说明 | 代表笔记 |
|------|------|---------|
| 前端 | VAD、音频分块、特征提取 | — |
| 声学模型 | encoder 架构（Conformer / Whisper backbone） | [[Whisper]]（如有） |
| 解码 | CTC / RNN-T / Attention / LLM-based | — |
| 后处理 | LLM rescoring / GER | — |
| 整体系统 | 端到端工业级 ASR | [[Seed-ASR]]（如有）/ [[Qwen2-Audio]] |

## 四大技术路线（待精化）

```
┌──────────────────────────────────────────────────────────────────────┐
│                      ASR 技术路线                                     │
├──────────────┬───────────────┬──────────────┬────────────────────────┤
│  CTC         │  RNN-T        │  Enc-Dec     │  LLM-based ASR        │
├──────────────┼───────────────┼──────────────┼────────────────────────┤
│  Wav2Vec 2   │  传统流式      │  Whisper      │  SLAM-ASR             │
│  HuBERT-CTC  │  TDNN-RNNT    │  USM          │  Qwen2-Audio          │
│              │  Conformer-T  │  Seed-ASR     │  SALMONN              │
│              │               │               │  Phi-Audio            │
└──────────────┴───────────────┴──────────────┴────────────────────────┘
```

## 关键演进节点（待填）

| 年份 | 里程碑 | 意义 |
|------|--------|------|
| 2016 | DeepSpeech 2 | 端到端 CTC 工业落地 |
| 2018 | Conformer | Transformer + Conv 混合架构 |
| 2020 | Wav2Vec 2.0 | 自监督预训练 |
| 2022 | Whisper | 大规模弱监督多语言 ASR |
| 2024 | Seed-ASR | 工业级 LLM-ASR 融合（待笔记） |
| 2024 | Qwen2-Audio / SALMONN | 多任务 Audio LLM 含 ASR |

## 核心挑战（待精化）

1. 长音频（>5 min）的上下文一致性
2. 多语种 / 方言泛化
3. 噪声鲁棒性
4. 实时性 / 流式
5. 与 LLM 的集成方式（Cascaded / Latent / Token，见 [[TTS-SpeechLM-Dialogue关系]] §3）
6. 评测标准化（与 TTS 共享 [[TTS-评测体系]] §评估方法论的结构性陷阱 中的多数问题）

## 与相邻领域的关系

```
TTS（1-TTS）← shared codec/SSL → ASR（2-ASR）
    ↕                                    ↕
SpeechLM（5-Speech-LLM）  ←─同框架管线─→  
                                         ↕
                                语音 SSL（9-SSL）
```

详见 [[TTS-SpeechLM-Dialogue关系]] §3（LLM-Speech 集成视角中 ASR 的位置）。

## 待完善清单

按 [[0-架构与重构方案]] §6 阶段 B 实施时优先级：

- [ ] 找到对应综述（Whisper 后的 ASR / Speech Foundation Model）→ 仿 [[TTS-11篇综述综合-2026-05]] 做综合
- [ ] 写 `0-地图/技术路线/ASR-技术路线图.md`
- [ ] 写 `0-地图/核心问题/ASR-核心挑战.md`
- [ ] 写 `0-地图/专题对比/ASR-评测体系.md`
- [ ] 写 `0-地图/领域地图/ASR-代表模型谱系.md`
- [ ] 把现有 `2-ASR与语音识别/` 下论文按 [[笔记frontmatter规范]] 加 routes/problems 标签

## 相关笔记

- [[TTS-SpeechLM-Dialogue关系]] §2 / §3 — ASR 在 SpeechLM/Cascaded/Audio-token 框架内的位置
- [[TTS-评测体系]] §评估方法论的结构性陷阱 — ASR 共享其中的 WER 陷阱（WER ≠ 感知可懂度）
- [[待回填地图]] — 新 ASR 论文进入后的回流入口

---

*占位 v1，2026-05-26 — 等扩到完整版。*
