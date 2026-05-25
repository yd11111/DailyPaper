---
title: "NVSpeech: An Integrated and Scalable Pipeline for Human-Like Speech Modeling with Paralinguistic Vocalizations"
method_name: "NVSpeech"
authors: [Huan Liao, Qinke Ni, Yuancheng Wang, Yiheng Lu, Haoyue Zhan]
year: 2025
venue: arXiv
arxiv_id: "2508.04195"
pdf_path: "assets/papers/NVSpeech.pdf"
library_source: "高德文献库"
source_topic: "Dataset"
tags: [classic, dataset, tts]
created: 2026-05-22
---

# NVSpeech: Paralinguistic-Aware Speech Pipeline

## 📌 一句话

港中深团队提出将**非语言声音**（笑声、叹气、犹豫等 paralinguistic vocalizations）系统性地纳入语音识别和生成全流程的 pipeline，包括数据标注、ASR 识别、TTS 合成三个环节的一体化解决方案。

## 🛠 核心方法

**输入 → 输出**: speech with paralinguistics ↔ text with paralinguistic tags

**架构组件**（按 pipeline 顺序）:
1. **Paralinguistic-aware ASR**: 识别并标注语音中的非语言声音事件
2. **Tagged Text Representation**: 用特殊 tag 在文本中标记 paralinguistic events 的类型和位置
3. **Paralinguistic-aware TTS**: 从含 tag 的文本生成包含自然非语言声音的语音
4. **Scalable Data Pipeline**: 从大规模数据中自动发现和标注 paralinguistic events

**关键创新**: 首次将 paralinguistic vocalizations 从"噪声"提升为"特征"，在 ASR 和 TTS 两端同时处理，打通了完整的 human-like speech 建模链条。

## 🖼 架构图

![Figure 2: Paralinguistic-aware speech recognition and generation pipeline overview](https://ar5iv.labs.arxiv.org/html/2508.04195/assets/x2.png)

## 📊 关键结果 / 评测

- ASR (SenseVoice): open-domain CER 3.79%, paralinguistic F1 0.85
- TTS (CosyVoice2): in-domain CER 7.51%（12.8% 相对降低），UTMOS 2.67
- 人类偏好胜率: 78.7% (CosyVoice) / 75.4% (CosyVoice2)
- Naturalness MOS: 4.0 ± 0.16

## 💡 借鉴意义（一句话）

做全双工对话 / 自然 TTS 的人关注——如何让合成语音包含 filler、笑声等 paralinguistic 是提升自然度的关键缺失环节。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2508.04195
- PDF: [[assets/papers/NVSpeech.pdf|本地 PDF]]
- 源目录: `Dataset/NVSpeech.pdf`
