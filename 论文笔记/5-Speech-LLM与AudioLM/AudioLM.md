---
title: "AudioLM: a Language Modeling Approach to Audio Generation"
method_name: "AudioLM"
authors: [Zalán Borsos, Raphaël Marinier, Damien Vincent, Eugene Kharitonov, Olivier Pietquin]
year: 2023
venue: TMLR
arxiv_id: "2209.03143"
pdf_path: "assets/papers/AudioLM.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# AudioLM: Language Modeling for Audio Generation

## 📌 一句话

Google 提出的**音频语言模型**——将音频生成重新定义为语言建模问题，用分层 token（semantic token from [[w2v-BERT]] + acoustic token from [[SoundStream]]）做自回归生成，无需文本标注即可生成语义连贯且高保真的语音/音乐。

## 🛠 核心方法

**输入 → 输出**: audio prompt → continued audio (speech/music)

**架构组件**（按层次结构）:
1. **Semantic Tokens**: [[w2v-BERT]] → k-means 量化，捕捉语义/语言内容
2. **Coarse Acoustic Tokens**: [[SoundStream]] RVQ 前几层，捕捉说话人和韵律
3. **Fine Acoustic Tokens**: SoundStream RVQ 后几层，补充音频细节
4. **Hierarchical LM**: 三阶段自回归——semantic → coarse acoustic → fine acoustic

**关键创新**: 首次提出 **semantic + acoustic 两级 token 分层建模**的思路——semantic token 保证语义连贯性，acoustic token 保证音频质量。这一"粗到细"的分层思路被后续几乎所有 Speech LLM 继承。

## 🖼 架构图

![Figure 2: AudioLM — 分层 token 建模（semantic → coarse → fine acoustic）](https://ar5iv.labs.arxiv.org/html/2209.03143/assets/x2.png)

## 📊 关键结果 / 评测

- Speech continuation: 人类评估者无法区分真实 vs 生成语音（50% 猜测概率）
- Piano continuation: 保持曲式结构和旋律连贯性
- 无需文本标注，纯音频训练

## 💡 借鉴意义（一句话）

做 Speech LLM / Audio Generation 的人**必读**——AudioLM 奠定了"语义 token + 声学 token 分层建模"的基本范式，[[VALL-E]] / [[SoundStorm]] / [[MusicLM]] 均直接继承此架构。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2209.03143
- PDF: [[assets/papers/AudioLM.pdf|本地 PDF]]
- 源目录: `TTS-LLM/AuioLM.pdf`
