---
title: "GSRM: Generative Speech Reward Model for Speech RLHF"
method_name: "GSRM"
authors: [Maohao Shen, Tejas Jayashankar, Osama Hanna, Naoyuki Kanda, Yancheng Wang, Kateřina Žmolíková]
year: 2026
venue: arXiv
arxiv_id: "2602.13891"
pdf_path: "assets/papers/GSRM.pdf"
library_source: "高德文献库"
source_topic: "RLHF"
tags: [classic, rlhf, speech-llm]
created: 2026-05-19
---

# GSRM: Generative Speech Reward Model

## 📌 一句话

Meta Superintelligence Labs + MIT 2026 年 2 月新工作。把语音 naturalness 评估从「raw audio → 标量分数」改造成**可解释的 CoT 推理过程**（提取声学特征 → 链式推理 → 输出评分），可作为 speech LLM 的 online RLHF reward model。

## 🛠 核心方法

**输入 → 输出**: 语音音频 → 结构化 naturalness 评估（含 reasoning 链 + 标量分）

**架构组件**（按数据流顺序）:
1. **Acoustic Feature Synthesis**: 把 raw audio 分解成命名的声学维度。摘要列了 6 类：Pitch Level、Pitch Slope、Pitch Range、Pitch Variability、Intensity Level、Intensity Variability，每类有清晰定义和分档（Low / Normal / High）
2. **CoT Reasoning Synthesis**: 基于上一步抽出的特征做 feature-grounded chain-of-thought 推理，生成 explainable judgment
3. **Generative Reward Output**: 输出结构化评估（包含中间推理 + 最终评分），可作为 RLHF 的 reward 信号
4. **训练数据**: 31k 专家评分数据集 + OOD 真实 user-assistant 语音交互 benchmark

**关键创新**: 不像传统 MOS predictor 黑盒输出标量分数，GSRM 把评估拆成**可命名的声学维度 + 链式推理**两阶段，做到 explainable judgment；且直接对接 speech LLM 的 online RLHF 训练循环。

## 🖼 架构图

![Fig.2 (p5): GSRM CoT Synthesis Framework——Acoustic Feature Synthesis（顶部 6 类声学维度）+ CoT Reasoning Synthesis（底部链式推理）](assets/papers/figs/GSRM_fig1.png)

## 📊 关键结果 / 评测

- **数据集**: 31k expert ratings（自建）+ OOD 真实交互 benchmark
- **model-human correlation 接近 human inter-rater consistency**（具体 PCC 数字看正文）
- **下游验证**: 作为 RLHF online reward 改进 speech LLM naturalness；test-time scaling（K 增大）单调提升 PCC

## 💡 借鉴意义（一句话）

做 Speech RLHF / TTS 评测 / Speech LLM 对齐的人**必读**——把「黑盒 MOS predictor」改造成「可解释 reward model」是当前 speech alignment 最缺的关键件，且 Meta 出品工程可信度高。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2602.13891
- PDF: [[assets/papers/GSRM.pdf|本地 PDF]]
- 源目录: `RLHF/GSRM-meta-mit-2026-01.pdf`
