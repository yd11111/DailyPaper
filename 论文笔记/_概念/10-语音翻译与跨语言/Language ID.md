---
type: concept
aliases: [LID, 语言标识, Language Identification]
---

# Language ID

## 定义
语言标识嵌入，用于在多语言模型中显式指定目标语言，引导模型生成符合目标语言声学特征的语音。通常实现为可学习的嵌入向量，加到声学 token 或隐层表示上。

## 核心要点
1. 在跨语言 TTS 中，Language ID 可有效控制语音口音（native vs foreign accent）
2. VALL-E X 的实验表明：去掉 LID 会导致口音评分从 4.10 暴跌至 2.55
3. 存在 trade-off：LID 降低了少量说话人相似度，但大幅改善口音质量
4. 不同于 speaker ID，Language ID 控制的是语言层面的声学风格

## 代表工作
- [[VALL-E-X]]: 用 Language ID 嵌入消除外语口音

## 相关概念
- [[Cross-Lingual TTS]]
- [[Acoustic Token]]
