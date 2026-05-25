---
title: "STITCH: Simultaneous Thinking and Talking with Chunked Reasoning for Spoken Language Models"
method_name: "STITCH"
authors: [Cheng-Han Chiang, Xiaofei Wang, Linjie Li, Chung-Ching Lin, Kevin Lin]
year: 2026
venue: ICLR 2026
arxiv_id: null
pdf_path: "assets/papers/STITCH.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, full-duplex, speech-llm]
created: 2026-05-22
---

# STITCH: Simultaneous Thinking and Talking for SLMs

## 📌 一句话

微软提出的 **Spoken Language Model** 推理方法——让模型**边想边说**（simultaneous thinking and talking），用 chunked reasoning 将内部文本推理与语音输出交错进行，解决了"先想完再说"导致的延迟问题。

## 🛠 核心方法

**输入 → 输出**: speech input → interleaved text reasoning + speech output

**架构组件**（按推理流程）:
1. **Speech Encoder**: 用户语音 → token
2. **Chunked Reasoning**: 将推理分成小 chunk，每个 chunk 先生成文本推理再生成对应语音
3. **Interleaved Generation**: text chunk ↔ speech chunk 交替生成

**关键创新**: **Chunked reasoning**——不像 [[Moshi]] 的 Inner Monologue 那样完全并行，而是按 chunk 交替"想一段→说一段"，在推理质量和延迟之间取得更好的平衡。

## 📊 关键结果 / 评测

- 发表于 ICLR 2026
- 匿名投稿（ICLR 2026），具体数字见论文 Table 1-3

## 💡 借鉴意义（一句话）

做 Speech LLM / 全双工对话的人关注——STITCH 的 chunked reasoning 是"实时推理 + 语音输出"的一个中间方案。

## 🔗 链接

- PDF: [[assets/papers/STITCH.pdf|本地 PDF]]
- 源目录: `TTS-LLM/STITCH- SIMULTANEOUS THINKING AND TALKING WITH CHUNKED REASONING FOR SPOKEN LANGUAGE MODELS.pdf`
