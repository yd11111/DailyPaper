---
title: "CoT-ST: Enhancing LLM-based Speech Translation with Multimodal Chain-of-Thought"
method_name: "CoT-ST"
authors: [Yexing Du, Ziyang Ma, Yifan Yang, Keqi Deng, Xie Chen, Bo Yang]
year: 2024
venue: arXiv
arxiv_id: "2409.19510"
pdf_path: "assets/papers/CoT-ST.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, translation, speech-llm]
created: 2026-05-22
---

# CoT-ST: Multimodal Chain-of-Thought for Speech Translation

## 📌 一句话

将 LLM 的 Chain-of-Thought 推理引入语音翻译——先 ASR 转录源语音（intermediate reasoning step），再基于转录文本做翻译，用多模态 CoT 解决语音翻译中的多义词消歧等困难问题。

## 🛠 核心方法

**输入 → 输出**: source speech → transcription (CoT step) → target text translation

**架构组件**（按数据流顺序）:
1. **Frozen Speech Encoder ([[Whisper]])**: 提取语音特征
2. **Q-Former Projection**: 将语音特征映射到 LLM 的 embedding 空间
3. **Frozen LLM**: 执行 CoT 推理——先生成 ASR 转录，再基于转录做翻译
4. **Sequential CoT**: ASR → MT 两步串行，中间转录结果可辅助消歧

**关键创新**: 把 NLP 的 CoT 范式自然地映射到语音翻译——ASR 转录就是"中间推理步骤"，帮助 LLM 消解源语音中的歧义（如多义词）。

## 🖼 架构图

![Figure 2: CoT-ST model architecture — Whisper encoder + Q-Former + frozen LLM with ASR→MT CoT](https://ar5iv.labs.arxiv.org/html/2409.19510/assets/x2.png)

## 📊 关键结果 / 评测

- BLEU (FLEURS 6×12, LLM-SRT-32B): 24.6 avg（vs SeamlessM4T-V2 20.2）
- BLEU (FLEURS 15×14, 210 directions): 21.4 avg（vs SeamlessM4T-V2 18.8）
- BLEU (CoVoST-2 En→X, LLM-SRT-7B): 39.1 avg（vs Qwen2-Audio 37.9）
- Ablation: 移除 SRT stage 导致 -4.9 BLEU（最大单阶段影响）

## 💡 借鉴意义（一句话）

做 Speech LLM / 语音翻译的人值得关注——CoT 思路简洁有效，可推广到其他语音理解任务。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2409.19510
- PDF: [[assets/papers/CoT-ST.pdf|本地 PDF]]
- 源目录: `SSL/2409.19510v1.pdf`
