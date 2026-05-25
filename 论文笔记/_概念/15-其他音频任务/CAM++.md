---
type: concept
aliases: [CAM plus plus]
---

# CAM++

## 定义
Context-Aware Masking++ 说话人验证模型，由阿里 3D-Speaker 项目提出。用于提取说话人嵌入向量（speaker embedding），在说话人验证和 TTS 音色控制中广泛使用。

## 核心要点
1. 来自 3D-Speaker 项目的预训练说话人模型
2. 在 CosyVoice 中用于提取 x-vector 说话人嵌入
3. 与 ERes2Net 配合用于说话人相似度评估

## 代表工作
- [[CosyVoice]]: 用 CAM++ 提取说话人嵌入作为 LLM 和 CFM 的条件
- [[3D-Speaker]]: 原始项目

## 相关概念
- [[X-Vector]]
- [[ERes2Net]]
- [[3D-Speaker]]
