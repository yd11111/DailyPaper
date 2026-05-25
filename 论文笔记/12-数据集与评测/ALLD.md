---
title: "Audio Large Language Models Can Be Descriptive Speech Quality Evaluators"
method_name: "ALLD"
authors: [Chen Chen, Yuchen Hu, Siyin Wang, Helin Wang, Zhehuai Chen, Chao Zhang]
year: 2025
venue: ICLR 2025
arxiv_id: "2501.17202"
pdf_path: "assets/papers/ALLD.pdf"
library_source: "高德文献库"
source_topic: "eval"
tags: [classic, eval, speech-llm]
created: 2026-05-22
---

# ALLD: Audio LLMs as Descriptive Speech Quality Evaluators

## 📌 一句话

NTU + Cisco 提出 **ALLD (Audio LLM Descriptive evaluation)**：让 Audio LLM 不只输出 MOS 分数，而是生成**多维度描述性评估**（音质/韵律/清晰度等维度的文本分析），在 ICLR 2025 发表，证明 Audio LLM 可以做到接近人类的 descriptive evaluation。

## 🛠 核心方法

**输入 → 输出**: speech audio → multi-dimensional descriptive quality assessment + score

**架构组件**（按数据流顺序）:
1. **Audio LLM Backbone**: 基于现有 Audio LLM（如 Qwen-Audio）
2. **Multi-dimensional Meta Info**: 定义多个评估维度（naturalness / clarity / prosody / speaker identity 等）
3. **Descriptive Training**: 训练 Audio LLM 生成结构化的描述性评估文本
4. **Score Prediction**: 从描述性分析中推导出数值评分

**关键创新**: 把语音质量评估从"黑盒打分"升级为"可解释的多维度描述"——与 [[GSRM]] / [[SpeechJudge]] 方向一致，但走 Audio LLM 路线而非专用 reward model。

## 🖼 架构图

![Figure 2: ALLD framework and training examples with multi-dimensional meta info](https://ar5iv.labs.arxiv.org/html/2501.17202/assets/overview.jpg)

## 📊 关键结果 / 评测

- MOS 预测 MSE: 0.17（vs 前 SOTA CNN-SA-AP 的 0.23）
- A/B test 准确率: 98.6%
- LCC/SRCC: 0.93/0.93（in-domain MOS prediction）
- BLEU (descriptive): 25.84 (MOS) / 30.17 (A/B test)

## 💡 借鉴意义（一句话）

做 TTS 评测 / Audio LLM 的人**必读**——ALLD 证明 Audio LLM 有做 descriptive evaluator 的能力，可作为 RLHF reward 信号源。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2501.17202
- PDF: [[assets/papers/ALLD.pdf|本地 PDF]]
- 源目录: `eval/AUDIO_LARGE_LANGUAGE_MODELS_CAN_BEDESCRIPTIVE_SPEECH_QUALITY_EVALUATORS-ICLR.pdf`
