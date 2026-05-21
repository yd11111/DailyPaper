---
type: concept
aliases: [FLEURS, Few-shot Learning Evaluation of Universal Representations of Speech]
---

# Fleurs

## 定义

Google 2022 年发布的多语种 ASR / TTS / LID 评测集 (Conneau et al., SLT 2022)。覆盖 **102 语种**，每语种约 12 小时（≈ 3,000 utterance），全部来自 [[FLoRes-101]] 多语种平行文本的朗读版本——也就是同一文本平行翻译到所有 102 语种，再由母语者朗读。

## 核心要点

1. **真正多语种平行**：同一份英文短文翻译成 102 种语言并朗读，可做 cross-lingual 直接比较
2. **few-shot 定位**：设计目标是 few-shot / zero-shot 评测，不是大训练集
3. **三任务**：ASR、speech translation (X→Eng)、LID
4. **WER 测试基线**：[[Whisper-large-v3]] 平均 WER ≈ 24%，[[MMS-1B]] fine-tune-all ≈ 18%
5. 与 [[Common Voice]] 互补：CV 数据多但语种少 / FLEURS 语种多但每种数据少

## 代表工作

- 原论文 SLT 2022 (arXiv 2205.12446)
- 所有 multilingual ASR 论文（[[Whisper]]、[[MMS]]、[[Seamless]]、[[SBPN]]）的标准评测之一

## 评测/常见数字

- 102 语种平均 WER：MMS-1B ~18%, Whisper-large-v3 ~24%
- Speech translation BLEU：SeamlessM4T-Large v2 平均 ~26

## 相关概念

- [[Common Voice]]
- [[Whisper]]
- [[MMS]]
- [[SeamlessM4T]]
