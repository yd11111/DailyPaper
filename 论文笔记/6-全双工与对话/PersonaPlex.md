---
title: "PersonaPlex: Voice and Role Control for Full Duplex Conversational Speech Models"
method_name: "PersonaPlex"
authors: [Rajarshi Roy, Jonathan Raiman, Sang-gil Lee, Teodor-Dumitru Ene, Robert Kirby]
year: 2026
venue: arXiv
arxiv_id: "2602.06053"
pdf_path: "assets/papers/PersonaPlex.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, full-duplex]
created: 2026-05-22
---

# PersonaPlex: Voice & Role Control for Full Duplex Speech Models

## 📌 一句话

NVIDIA 提出的全双工对话模型扩展——在 [[Moshi]] 基础上加入 **Hybrid System Prompt**（文本 prompt + voice cloning prompt），实现对全双工模型的角色和声音控制，无需重新训练。

## 🛠 核心方法

**输入 → 输出**: user speech + system prompt + voice prompt → agent speech (full-duplex)

**架构组件**（按架构层次）:
1. **Moshi Backbone**: 全双工语音对话基础架构
2. **Hybrid System Prompt**: 文本 prompt（角色/风格指令）+ voice prompt（说话人音色 cloning）
3. **Dual-stream AR**: 继承 Moshi 的双流建模

**关键创新**: 首次为全双工对话模型引入**可控的角色和声音**——通过 hybrid prompt 机制，用户可以在不重训模型的情况下切换对话角色（如客服/助手/角色扮演）和声音。

## 🖼 架构图

![Figure 1: PersonaPlex — Hybrid System Prompt + Moshi-based duplex 架构](https://ar5iv.labs.arxiv.org/html/2602.06053/assets/x1.png)

## 📊 关键结果 / 评测

- Voice cloning: speaker similarity 显著提升
- Role control: 角色一致性 + 对话质量保持

## 💡 借鉴意义（一句话）

做全双工对话的人关注——PersonaPlex 解决了全双工模型的"个性化"问题，hybrid prompt 是低成本定制的可行方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2602.06053
- PDF: [[assets/papers/PersonaPlex.pdf|本地 PDF]]
- 源目录: `TTS-LLM/PERSONAPLEX- VOICE AND ROLE CONTROL FOR FULL DUPLEX CONVERSATIONAL SPEECH MODELS.pdf`
