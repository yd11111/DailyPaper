---
title: "CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training"
method_name: "CosyVoice3"
authors: [Zhihao Du, Changfeng Gao, Yuxuan Wang, Fan Yu, Tianyu Zhao]
year: 2025
venue: arXiv
arxiv_id: null
pdf_path: "assets/papers/CosyVoice3.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts, speech-llm]
created: 2026-05-22
---

# CosyVoice 3: In-the-wild Speech Generation

## 📌 一句话

阿里通义实验室 CosyVoice 系列第三代，核心改进在于 **scaling up**（数据从 170K→500K 小时）+ **post-training**（SFT + RLHF 对齐人类偏好），同时保持流式推理能力，目标是 in-the-wild 场景下的高鲁棒性语音生成。

## 🛠 核心方法

**输入 → 输出**: text → streaming speech waveform（支持 zero-shot voice cloning）

**架构组件**（按数据流顺序）:
1. **LLM-based Token Predictor**: 自回归预测 speech tokens（沿袭 CosyVoice 1/2 的 LLM backbone）
2. **Flow Matching Decoder**: 从 speech tokens → mel spectrogram，用 [[flow matching]] 做高质量转换
3. **Streaming Chunk Mechanism**: 支持 chunk-level 流式生成，低延迟
4. **Post-training Pipeline**: SFT 用高质量对齐数据微调 + RLHF（基于 MOS 反馈）进一步优化自然度
5. **500K 小时数据**: 相比 CosyVoice 2 的 170K 小时扩大近 3 倍

**关键创新**: 在语音生成领域首次系统性地引入 **post-training 范式**（SFT → RLHF），将 LLM alignment 经验迁移到 TTS；同时通过数据 scaling 验证了"更多数据 = 更好泛化"在语音域同样成立。

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 定性目标：in-the-wild 场景鲁棒性（噪声/口音/非标准文本）
- 已开源模型和推理代码

## 💡 借鉴意义（一句话）

做 TTS / Speech LLM 对齐的人**必读**——CosyVoice 3 是目前公开信息最详尽的"TTS + post-training"工程实践，post-training pipeline 可复用到其他 TTS 系统。

## 🔗 链接

- PDF: [[assets/papers/CosyVoice3.pdf|本地 PDF]]
- 源目录: `TTS-LLM/CosyVoice3_0.pdf`
