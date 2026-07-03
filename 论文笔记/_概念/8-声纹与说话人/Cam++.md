---
type: concept
aliases: [CAM++, Cam++ Speaker Embedding]
---

# Cam++

## 定义
Cam++ 是一种说话人嵌入（speaker embedding）模型，用于提取说话人身份表示向量。在 TTS / VC 评测中常用于计算说话人相似度指标 SIM（Speaker Similarity），通过余弦相似度衡量生成语音与参考语音的说话人一致性。

## 核心要点
1. 输入为音频波形，输出为固定维度的说话人嵌入向量
2. 在评测中通过计算生成语音与参考语音嵌入的余弦相似度得到 SIM 分数
3. 与 WavLM-TDNN、ECAPA-TDNN、Resemblyzer 等属于同类工具

## 评测/常见数字
- SIM 值范围 [-1, 1]，TTS 零样本克隆优秀模型通常 >0.8
- 不同 speaker encoder 计算的 SIM 值不直接可比

## 代表工作
- [[UnifiedGuidanceFM]]: 使用 Cam++ 计算 SIM 指标

## 相关概念
- [[SIM-O]]
- [[WER]]
- [[Voice Conversion]]
