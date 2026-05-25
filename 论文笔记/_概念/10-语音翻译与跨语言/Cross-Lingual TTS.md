---
type: concept
aliases: [跨语言TTS, 跨语言语音合成, XTTS]
---

# Cross-Lingual TTS

## 定义
跨语言语音合成，使用说话人 A 的语言 L1 语音作为参考，合成该说话人说另一种语言 L2 的语音。核心目标是保留说话人音色同时消除外语口音（L2 accent）。

## 核心要点
1. 主要挑战：数据稀缺（同一说话人的多语言数据难以获取）、外语口音、音色保持
2. 传统方法：共享音素表示 + 说话人/语言解耦（adversarial disentanglement）
3. 现代方法：基于大规模数据的 codec language model + in-context learning（如 VALL-E X）
4. 关键评测维度：说话人相似度（ASV-Score）、口音评分（Accent Score）、可懂度（WER）

## 代表工作
- [[VALL-E-X]]: 零样本跨语言 TTS，基于 codec LM
- [[YourTTS]]: 基于 VITS 的多语言多说话人 TTS

## 相关概念
- [[S2ST]]
- [[Language ID]]
- [[Zero-shot TTS]]
