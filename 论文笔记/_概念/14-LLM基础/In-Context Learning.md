---
type: concept
aliases: [ICL, 上下文学习, 提示学习]
---

# In-Context Learning

## 定义

模型在推理时通过输入中的示例（prompt / demonstration）来完成新任务，无需更新模型参数。GPT-3 首次系统展示了这一能力。在语音领域，VALL-E 等模型通过 acoustic prompt 实现说话人音色的 in-context learning。

## 核心要点

1. 无需 fine-tune，仅通过 prompt 中的少量示例即可泛化
2. 能力随模型规模和数据规模增长而增强（scaling law）
3. 语音领域的 ICL：给定 3 秒 reference audio 作为 prompt，模型自动学习说话人身份、情感、声学环境

## 代表工作

- GPT-3 (Brown et al., 2020): NLP 中首次大规模展示 ICL
- [[VALL-E]]: 语音领域首次实现 ICL，3 秒 prompt 即可 zero-shot TTS
- [[VALL-E-X]]: 将 ICL 扩展到跨语言场景，涌现 code-switch 合成能力
- [[Zero-shot TTS]]: ICL 在 TTS 中的具体应用

## 相关概念

- [[Zero-shot TTS]]
- [[VALL-E]]
- [[GPT]]
