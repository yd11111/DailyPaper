---
type: concept
aliases: [GLM-4-Voice, GLM-Voice]
---

# GLM-4-Voice

## 定义
智谱 2024 年发布的端到端语音对话模型，9B 参数，基于 GLM-4 backbone。

## 核心要点
1. 支持中英双语
2. Speech tokenizer 输出 12.5 Hz 离散 token
3. 三阶段 streaming TTS（reduce latency）

## 代表工作
- [[OmniFlatten]] 与之对比，怀疑 GLM-4-Voice 训练数据泄露了测试集

## 相关概念
- [[OmniFlatten]]
