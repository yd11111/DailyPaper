---
title: "FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot"
method_name: "FireRedTTS2"
authors: [Kun Xie, Feiyu Shen, Junjie Li, Fenglong Xie, Xu Tang, Yao Hu]
year: 2025
venue: arXiv
arxiv_id: "2509.02020"
pdf_path: "assets/papers/FireRedTTS2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# FireRedTTS-2: Long Conversational Speech Generation

## 📌 一句话

[[FireRedTTS]] 的续作——聚焦**长对话语音生成**（podcast / chatbot），解决多轮对话中的 speaker consistency、turn-taking、长文本合成等工业痛点。

## 🛠 核心方法

**输入 → 输出**: multi-turn dialogue text → multi-speaker speech

**架构组件**:
1. **对话理解**: 解析多轮对话结构
2. **Multi-speaker 合成**: 多说话人语音生成
3. **长文本处理**: 支持 podcast 级别的长文本合成

**关键创新**: 针对**长对话场景**优化——解决了传统 TTS 在长文本 / 多轮对话中 speaker 漂移、韵律不一致的问题。

## 📊 关键结果 / 评测

- 支持 podcast 级别长度的多人对话生成
- Speaker consistency 在长对话中保持稳定

## 💡 借鉴意义（一句话）

做对话 TTS / podcast 生成的人关注——FireRedTTS-2 针对长对话的实际问题给出了解决方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2509.02020
- PDF: [[assets/papers/FireRedTTS2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/fireredtts2.pdf`
