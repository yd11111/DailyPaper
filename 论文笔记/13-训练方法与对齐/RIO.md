---
title: "Robust Zero-Shot Text-to-Speech Synthesis with Reverse Inference Optimization"
method_name: "RIO"
authors: [Yuchen Hu, Chen Chen, Siyin Wang, Eng Siong Chng, Chao Zhang]
year: 2024
venue: arXiv
arxiv_id: "2407.02243"
pdf_path: "assets/papers/RIO.pdf"
library_source: "高德文献库"
source_topic: "RLHF"
tags: [classic, tts, rlhf]
created: 2026-05-22
---

# RIO: Reverse Inference Optimization for Zero-Shot TTS

## 📌 一句话

NTU 提出 **Reverse Inference Optimization**：在推理时通过反向优化 codec LM 的 latent，提升 zero-shot TTS 的鲁棒性（减少 word skip / repeat / mispronunciation），不需要重新训练模型。

## 🛠 核心方法

**输入 → 输出**: text + speaker prompt → robust speech (via inference-time optimization)

**架构组件**（按数据流顺序）:
1. **Zero-shot Codec LM**: 标准 AR + NAR codec language model（如 VALL-E）
2. **Reverse Inference**: 把生成的 speech tokens "反向"过 ASR 模型得到 text，与原始输入对比
3. **Optimization Loop**: 如果 ASR 转录与输入不匹配，调整 LM 的采样策略/latent，重新生成
4. **Test-time Only**: 不修改模型权重，纯推理时优化

**关键创新**: 把 TTS 的鲁棒性问题转化为**推理时搜索问题**——用 ASR 做 verifier，反复优化直到生成结果与输入 text 一致。思路类似 LLM 的 test-time compute scaling。

## 🖼 架构图

![Figure 1: RIO overview — zero-shot TTS + reverse inference optimization loop](https://ar5iv.labs.arxiv.org/html/2407.02243/assets/x1.png)

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 目标：显著降低 zero-shot TTS 的 word error（skip/repeat/mispronunciation）

## 💡 借鉴意义（一句话）

做 TTS 鲁棒性 / test-time optimization 的人值得关注——推理时用 ASR 做 verifier 的思路简洁有效，可应用到任何 codec LM TTS 系统。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2407.02243
- PDF: [[assets/papers/RIO.pdf|本地 PDF]]
- 源目录: `RLHF/Robust Zero-Shot Text-to-Speech Synthesis with Reverse Inference Optimization.pdf`
