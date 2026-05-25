---
type: concept
aliases: [反向翻译, Back-Translation, BT]
---

# Backtranslation

## 定义
源自机器翻译的半监督学习技术：先训练反向模型（target -> source），用它将大量无标注目标域数据翻译回源域，生成合成配对数据来扩增正向模型的训练集。在语音领域用于减少 text-speech 配对数据需求。

## 核心要点
1. 利用单语/无标注数据生成伪配对，是经典的数据增广方法
2. 在 TTS 中特别有效，因为 text-to-speech 是一对多映射（同一文本可对应不同语音），而反向 speech-to-text 是相对简单的多对一映射
3. 需要先有少量真实配对数据训练反向模型，再用反向模型转录大量无标注数据

## 代表工作
- [[SPEAR-TTS]]: 在 TTS 中首次系统性应用 backtranslation，将 semantic token -> text 的反向模型用于生成合成配对数据，仅需 15 分钟真实标注
- Sennrich et al. (2016): NMT 领域提出 backtranslation 的经典论文

## 相关概念
- [[T5]]
- [[Semantic Token]]
- [[Zero-shot TTS]]
