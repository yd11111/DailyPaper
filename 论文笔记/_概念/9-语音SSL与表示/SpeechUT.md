---
type: concept
aliases: [SpeechUT, Unified-Modal Speech-Unit-Text Pre-Training]
---

# SpeechUT

## 定义
微软提出的统一模态语音-单元-文本预训练框架。使用隐藏单元（hidden units）作为语音和文本之间的桥梁，通过联合预训练实现语音理解和生成能力。

## 核心要点
1. 架构包含三个组件：语音编码器、语义编码器、语义解码器
2. 语音侧通过 masked prediction 预训练，文本侧通过 seq2seq 翻译预训练
3. VALL-E X 中对 SpeechUT 做了改进：将 clustering-based hidden unit 替换为音素（phoneme）
4. 可通过 CTC + cross-entropy 联合微调实现 ASR/ST 多任务

## 代表工作
- [[VALL-E-X]]: 使用改进的 SpeechUT 作为 S2ST 翻译模块

## 相关概念
- [[HuBERT]]
- [[CTC]]
- [[ASR]]
