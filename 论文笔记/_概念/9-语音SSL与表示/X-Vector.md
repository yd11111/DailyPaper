---
type: concept
aliases: [x-vector, Speaker Embedding, 说话人嵌入]
---

# X-Vector

## 定义
从语音中提取的固定维度说话人特征向量，用于说话人识别/验证和 TTS 音色控制。最初由 Snyder et al. (2018) 提出，现泛指各类 speaker embedding。

## 核心要点
1. 将变长语音压缩为固定维度向量，表征说话人身份
2. CosyVoice 中用 CAM++ 提取 x-vector 注入 LLM 和 CFM
3. 常用模型：ECAPA-TDNN、CAM++、ERes2Net、WavLM-TDNN

## 代表工作
- [[CosyVoice]]: x-vector 注入实现音色-语义-韵律解耦

## 相关概念
- [[CAM++]]
- [[ERes2Net]]
