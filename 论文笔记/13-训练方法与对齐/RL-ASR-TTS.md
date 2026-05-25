---
title: "Explore the Reinforcement Learning for the LLM based ASR and TTS system"
method_name: "RL-ASR-TTS"
authors: [Changfeng Gao, Yabin Li, Keyu An, Zhifu Gao, Zhihao Du, Han Zhao, Xiangang Li]
year: 2025
venue: arXiv
arxiv_id: "2509.18569"
pdf_path: "assets/papers/RL-ASR-TTS.pdf"
library_source: "高德文献库"
source_topic: "RLHF"
tags: [classic, rlhf, tts, asr]
created: 2026-05-22
---

# RL-ASR-TTS: RL for LLM-based Speech Systems

## 📌 一句话

阿里通义探索将强化学习（GRPO/DPO）系统性地应用到 **LLM-based ASR 和 TTS** 两个方向——ASR 端用 CER 做 reward 降低识别错误，TTS 端用 MOS/SIM 做 reward 提升合成质量。

## 🛠 核心方法

**输入 → 输出**: LLM-based ASR/TTS system → RL-finetuned system

**架构组件**（按数据流顺序）:
1. **Baseline LLM-ASR/TTS**: 基于 CosyVoice / FunASR 等通义自有模型
2. **GRPO (Group Relative Policy Optimization)**: 采样 K 个候选 → reward 排序 → 相对优势优化
3. **ASR Reward**: CER / WER 自动评估
4. **TTS Reward**: MOS predictor + speaker similarity
5. **Training Framework**: 统一 RL 训练流程同时适配 ASR 和 TTS

**关键创新**: 首次在**同一框架**内同时对 LLM-based ASR 和 TTS 做 RL，揭示两个方向的共性（LLM backbone）和差异（reward 设计）。

## 🖼 架构图

![Figure 1: RL training framework for LLM-based ASR/TTS](https://ar5iv.labs.arxiv.org/html/2509.18569/assets/img/rl_framework2.png)

## 📊 关键结果 / 评测

- ASR WER (short): 9.71（GRPO, vs baseline 10.25, -5.3%）
- ASR WER (long): 6.03（vs 6.35, insertion errors 2.72→0.86）
- TTS WER 中文: 3.330（CosyVoice2-0.5B + GRPO, vs 4.280 baseline, -22%）
- TTS WER 英文: 5.279（vs 6.074 baseline）

## 💡 借鉴意义（一句话）

做 Speech RLHF / CosyVoice fine-tuning 的人**必读**——通义官方给出的 RL 训练 recipe，可直接参考 reward 设计和 GRPO 超参。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2509.18569
- PDF: [[assets/papers/RL-ASR-TTS.pdf|本地 PDF]]
- 源目录: `RLHF/RL-tongyi-25-9.pdf`
