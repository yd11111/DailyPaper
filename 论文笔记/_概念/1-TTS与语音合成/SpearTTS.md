---
type: concept
aliases: [Spear TTS, SPEAR-TTS, Speak Read and Prompt]
---

# SpearTTS

## 定义
Google 提出的两阶段 TTS 系统：S1 "reading"（text -> [[Semantic Token]]，用 [[w2v-BERT]] + [[k-means]] 提取）+ S2 "speaking"（semantic -> [[Acoustic Token]]，用 [[SoundStream]] [[RVQ]]）。通过 [[Backtranslation]] 仅需 15 分钟标注数据，是 [[VALL-E]] 之外的另一条离散 token TTS 路线。

## 核心要点
1. 两阶段解耦：S1 用少量标注数据学 text->semantic，S2 完全用无标注音频学 semantic->acoustic
2. [[Backtranslation]] + denoising pretraining 实现极低资源训练（15min 标注数据 CER 1.92%）
3. 3 秒参考音频的 example prompting 机制实现零样本声音克隆，无需 prompt 转录
4. MOS 4.96 vs GT 4.92；Speaker Sim 0.56 vs VALL-E 0.58（数据量差 240,000x）

## 评测/常见数字
- LibriSpeech test-clean CER: 1.92%（15min 标注数据 + backtranslation）
- MOS: 4.96 vs GT 4.92；vs VALL-E 3.35
- Speaker cosine similarity: 0.56（vs VALL-E 0.58，YourTTS 0.34）
- Speaker accuracy: 92.4% top-1（40 未见说话人，3 秒 prompt）

## 代表工作
- [[SPEAR-TTS]]: 本文（Kharitonov et al., 2023）
- [[CosyVoice]]: 在相同两阶段框架下证明监督式 semantic token 优于 k-means token
- [[AudioLM]]: SPEAR-TTS 的直接前身

## 相关概念
- [[Semantic Token]]
- [[Acoustic Token]]
- [[Backtranslation]]
- [[w2v-BERT]]
- [[SoundStream]]
- [[VALL-E]]
- [[Zero-shot TTS]]
