---
title: "EmergentTTS-Eval: Evaluating TTS Models on Complex Prosodic, Expressiveness, and Linguistic Challenges Using Model-as-a-Judge"
method_name: "EmergentTTS-Eval"
authors: [Ruskin Raj Manku, Yuzhi Tang, Xin Wang]
year: 2025
venue: arXiv
arxiv_id: "2505.23009"
pdf_path: "assets/papers/EmergentTTS-Eval.pdf"
library_source: "高德文献库"
source_topic: "eval"
tags: [classic, eval, tts]
created: 2026-05-22
---

# EmergentTTS-Eval: Complex TTS Evaluation Benchmark

## 📌 一句话

针对 TTS 系统在**复杂语言现象**（外来词、数字、缩写、疑问句韵律、情感表达等）上的评测 benchmark，用 Model-as-a-Judge（LLM 自动评分）方案大规模评估 TTS 的 emergent abilities。

## 🛠 核心方法

**输入 → 输出**: TTS outputs on challenging test cases → multi-dimensional evaluation scores

**架构组件**（按流程顺序）:
1. **Challenge Categories**: 多类困难测试场景——外来词发音 / 数字读法 / 缩写展开 / 疑问韵律 / 情感表达等
2. **Data Refinement Pipeline**: 多层精炼 test cases 确保难度和区分度
3. **Model-as-a-Judge**: 用 LLM 做自动评分，替代大规模人工 MOS 评估
4. **Depth-Refinement**: 逐层增加评估细粒度（粗 → 精）

**关键创新**: 聚焦传统 MOS 评测忽略的**长尾 / 困难场景**——现有 TTS 在简单句子上已趋近人类，但在外来词/复杂韵律上差距仍然巨大。

## 📊 关键结果 / 评测

- Win-rate 范围: 8.90% (Suno Bark) → 65.17% (GPT-4o-audio Ballad)
- 评委一致性: Kendall's W 0.97
- 人-模型对齐: Spearman ρ = 90.5%
- Text normalization 影响: Complex Pronunciation win-rate 51.69% → 76.74%

## 💡 借鉴意义（一句话）

做 TTS 评测的人值得关注——EmergentTTS-Eval 补充了 MOS 评测缺失的"困难场景"维度，适合做 TTS 系统的压力测试。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2505.23009
- PDF: [[assets/papers/EmergentTTS-Eval.pdf|本地 PDF]]
- 源目录: `eval/EmergentTTS-Eval.pdf`
