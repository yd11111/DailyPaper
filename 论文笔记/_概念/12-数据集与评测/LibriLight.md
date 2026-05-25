---
type: concept
aliases: [LibriLight, Libri-Light]
---

# LibriLight

## 定义
Meta 发布的大规模无标注英文有声书语音数据集，约 60,000 小时，来源于 LibriVox 公共领域有声书。设计用于自监督语音预训练和大规模语音生成模型训练。

## 核心要点
1. 规模约 60K 小时，比 LibriSpeech（960h）大约 60 倍
2. 无标注数据，需要 ASR 模型做伪标注（pseudo-labeling）
3. VALL-E 和 VALL-E X 均使用 LibriLight 作为英文训练数据
4. 常用 Kaldi ASR（在 LibriSpeech 上训练）做伪转录
5. 是目前大规模英文 TTS/Speech LLM 训练的重要数据源

## 代表工作
- [[VALL-E]]: 使用 LibriLight 60K 小时训练
- [[VALL-E-X]]: 使用 LibriLight 作为英文训练数据

## 相关概念
- [[LibriSpeech]]
- [[LibriTTS]]
