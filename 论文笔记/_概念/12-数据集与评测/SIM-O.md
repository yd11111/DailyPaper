---
type: concept
aliases: [SIM-O, Speaker Similarity Original, 说话人相似度]
---

# SIM-O

## 定义
Speaker Similarity to Original，衡量合成语音与原始参考音频的说话人相似度。通过 speaker encoder 提取 embedding 后计算余弦相似度。

## 数学形式

$$
\text{SIM-O} = \cos\big(\text{Emb}(y_\text{ref}),\, \text{Emb}(\hat{y})\big)
$$

## 核心要点
1. 是 zero-shot TTS 的核心评测指标
2. 常用 speaker encoder: WavLM-TDNN、ECAPA-TDNN、Resemblyzer
3. 0.6+ 通常被认为是较好的相似度
4. 也常写作 SECS (Speaker Encoder Cosine Similarity)

## 代表工作
- [[VALL-E]]: 首次大规模使用 SIM-O 评测 zero-shot TTS
- [[CosyVoice]]: Seed-TTS-eval 上的 SIM-O 评测

## 相关概念
- [[MOS]]
- [[WER]]
- [[SECS]]
