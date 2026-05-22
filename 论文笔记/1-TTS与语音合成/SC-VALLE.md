---
title: "SC VALL-E: Style-Controllable Zero-Shot Text to Speech Synthesizer"
method_name: "SC-VALLE"
authors: [Daegyeom Kim, Seongho Hong, Yong-Hoon Choi]
year: 2024
venue: IEEE Access
arxiv_id: null
pdf_path: "assets/papers/SC-VALLE.pdf"
library_source: "高德文献库"
source_topic: "ControllableTTS"
tags: [classic, tts]
created: 2026-05-22
---

# SC-VALLE: Style-Controllable Zero-Shot TTS

## 📌 一句话

在 [[VALL-E]] 框架上增加风格控制能力，通过 style embedding 实现 zero-shot 条件下的说话风格（语速/音量/情感等）可控合成。

## 🛠 核心方法

**输入 → 输出**: text + style prompt + speaker prompt → speech waveform

**架构组件**（按数据流顺序）:
1. **Style Encoder**: 从 style reference 音频抽取 style embedding
2. **AR Codec LM (VALL-E backbone)**: 自回归预测 speech tokens，条件为 text + speaker prompt + style embedding
3. **NAR Codec LM**: 非自回归补全剩余 RVQ 层
4. **Codec Decoder**: speech tokens → waveform

**关键创新**: 把 style 控制从传统参数化（F0/duration/energy）升级到 embedding 级别，直接注入 VALL-E 的 LM，实现 zero-shot speaker × arbitrary style 的组合。

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 目标：在保持 VALL-E zero-shot speaker cloning 能力的同时增加风格可控性

## 💡 借鉴意义（一句话）

做可控 TTS 的人可参考其 style embedding 注入 codec LM 的方案，但此方向已有更新的工作（如 CosyVoice 的 instruct 模式）。

## 🔗 链接

- PDF: [[assets/papers/SC-VALLE.pdf|本地 PDF]]
- 源目录: `ControllableTTS/SC-VALLE.pdf`
