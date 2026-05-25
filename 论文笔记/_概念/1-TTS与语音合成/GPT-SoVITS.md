---
type: concept
aliases: [GPT-SoVITS, GPTSoVITS]
---

# GPT-SoVITS

## 定义
中文社区最流行的开源零样本 TTS 方案，结合 GPT 风格的自回归 token 预测与 SoVITS（So-VITS-SVC 的变体）声码器，支持少样本语音克隆。

## 核心要点
1. 结合 AR 语义 token 预测和 VITS 声码器，两阶段架构
2. 中文社区生态最活跃的开源 TTS 项目之一
3. 在 LibriSpeech test-clean 上 WER 5.13%，SS 0.405（较 CosyVoice 2 有明显差距）

## 代表工作
- [[CosyVoice2]]: 作为基线对比

## 相关概念
- [[VITS]]
- [[VALL-E]]
