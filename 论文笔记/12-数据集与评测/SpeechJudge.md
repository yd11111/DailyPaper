---
title: "SpeechJudge: Towards Human-Level Judgment for Speech Naturalness"
method_name: "SpeechJudge"
authors: [Xueyao Zhang, Chaoren Wang, Huan Liao, Ziniu Li, Yuancheng Wang]
year: 2025
venue: arXiv
arxiv_id: "2511.07931"
pdf_path: "assets/papers/SpeechJudge.pdf"
library_source: "高德文献库"
source_topic: "eval"
tags: [classic, eval]
created: 2026-05-22
---

# SpeechJudge: Human-Level Speech Naturalness Judgment

## 📌 一句话

港中深 + ByteDance 联合构建的**语音自然度评估数据集 + 评估模型**框架：99K 对 TTS 对比样本（来自 17 个 zero-shot TTS 系统）+ 人类标注偏好 + 基于 CoT 推理的 Generative Reward Model (GRM)，在 pairwise preference 准确率上达到人类水平。

## 🛠 核心方法

**输入 → 输出**: speech pair + text → pairwise preference judgment（含 CoT 推理过程）

**架构组件**（按数据流顺序）:
1. **SpeechJudge-Data**: 99K speech pairs，来自 17 个 zero-shot TTS 系统在 200+ speakers 上的合成结果，人工标注 pointwise MOS + pairwise preference
2. **SpeechJudge-Bench**: 多维度评测集，分 naturalness / text accuracy / speaker similarity / emotion / 总体 5 个维度
3. **SpeechJudge-GRM (Generative Reward Model)**: 两阶段训练——(a) SFT: Gemini-2.5-Flash 做 teacher 生成 CoT rationale，student 学推理过程；(b) RL: GRPO 对齐人类偏好
4. **Multi-turn Evaluation**: 支持 pointwise scoring 和 pairwise comparison 两种评估模式

**关键创新**: 把 LLM 的 CoT + RLHF 范式完整迁移到语音评估领域——不再黑盒出分，而是先"推理"（分析音质/韵律/口音等维度）再判断，与 [[GSRM]] 思路一致但数据规模更大。

## 🖼 架构图

![Figure 4: SpeechJudge-GRM 训练框架——Stage 1: SFT with teacher CoT → Stage 2: RL with GRPO](https://arxiv.org/html/2511.07931/img/grm.png)

## 📊 关键结果 / 评测

- Pairwise preference accuracy: GRM 达到 human inter-annotator agreement 水平
- 覆盖 17 个 TTS 系统（包括 CosyVoice / XTTS / F5-TTS 等主流系统）
- 99K 标注对，规模远超已有 TTS 评测数据集

## 💡 借鉴意义（一句话）

做 TTS 评测 / RLHF reward model 的人**必读**——SpeechJudge 是目前规模最大的 TTS pairwise preference 数据集，GRM 可直接作为 TTS RLHF 的 reward model。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2511.07931
- PDF: [[assets/papers/SpeechJudge.pdf|本地 PDF]]
- 源目录: `eval/SpeechJudge.pdf`
