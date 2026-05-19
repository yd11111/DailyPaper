---
type: concept
aliases: [Modality Alignment, 模态对齐]
---

# Modality Alignment

## 定义
把单模态（如纯文本）LLM 通过有监督微调，使其学会另一种模态（如语音）的输入/输出，建立模态间的对应关系。

## 核心要点
1. 通常用配对数据（如 speech-text pair）做 SFT
2. 在 speech-text 场景下，常用 ASR + TTS 双任务联合训练
3. 是构建多模态/Omni LLM 的第一步

## 代表工作
- [[OmniFlatten]] 阶段 1：用 ASR + TTS 把 [[Qwen2-0.5B]] 从文本 LLM 改造成 speech-text 多模态 LLM

## 相关概念
- [[OmniFlatten]]
