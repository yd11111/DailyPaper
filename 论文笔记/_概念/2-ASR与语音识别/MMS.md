---
type: concept
aliases: [Massively Multilingual Speech, Meta MMS]
---

# MMS

## 定义

Meta 2023 年发布的 Massively Multilingual Speech 项目 (Pratap et al.)。基于 [[wav2vec 2.0]] 自监督预训练 + CTC 微调，覆盖 ASR（1100+ 语种）、TTS（1100+ 语种）、语种识别（4000+ 语种），是当前覆盖语种最多的开源多语种语音工具栈。

## 核心要点

1. **数据爆破**：基于宗教文本朗读语料（圣经多语种朗读 / MMS-lab），把语种规模从百级别推到千级别
2. **adapter 化**：基模型共享、每个语种一个 LoRA-like adapter，可低成本新增语种
3. **TTS / ASR 双向**：同一份 SSL 表示同时支持识别与合成
4. **ASR 性能**：在 FLEURS 上覆盖 102 语言，WER 一般优于 [[Whisper-large-v2]]
5. **TTS 限制**：合成质量是「能听清」级别，不如商业 TTS 自然

## 代表工作

- 原论文 (arXiv 2305.13516)
- [[SBPN]]：尼日利亚语 ASR 蒸馏中作为强基线参考

## 评测/常见数字

- FLEURS 102 语言平均 WER：MMS-1B（fine-tune all）≈ 18%，Whisper-large-v2 ≈ 24%
- MMS-zeroshot 在 1107 语种上 LID 准确率 > 94%

## 相关概念

- [[Whisper]]
- [[Fleurs]]
- [[Common Voice]]
- [[wav2vec 2.0]]
