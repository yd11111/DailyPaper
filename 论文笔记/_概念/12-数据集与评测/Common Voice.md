---
type: concept
aliases: [CommonVoice, Mozilla Common Voice]
---

# Common Voice

## 定义

Mozilla 主导的众包多语种语音数据集（2017 起持续滚动），用户朗读句子贡献语音 + 接受/拒绝投票。截至 2024 年覆盖 **100+ 语种**、累计 30,000+ 小时验证语音，是开源世界最重要的多语种 ASR 训练 / 评测语料之一。

## 核心要点

1. **众包朗读**：内容偏 read speech，远离 in-the-wild / 对话场景
2. **逐版本滚动**：CV 17.0 / 18.0 数据持续增长，版本号必须报告
3. **覆盖广**：从英语到大量低资源语种（约鲁巴、豪萨等）
4. **标签弱**：accent / age / gender 字段为自报，可信度差
5. 在低资源 ASR 场景常与 [[Fleurs]] 配对使用，互补

## 代表工作

- 原论文 LREC 2020 (arXiv 1912.06670)
- [[MMS]]、[[Whisper]]、[[SBPN]]、[[AfriHuBERT]] 等多语种 ASR/SSL 模型基本都用 CV 作为评测之一

## 评测/常见数字

- CV 16.0 (En): test WER for Whisper-large-v3 ≈ 8.5%
- Yoruba / Hausa 等低资源语种 verified hours 通常 < 100h

## 相关概念

- [[Fleurs]]
- [[LibriSpeech]]
- [[MMS]]
- [[Whisper]]
