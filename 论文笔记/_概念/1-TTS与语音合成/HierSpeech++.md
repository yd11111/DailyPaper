---
type: concept
aliases: [HierSpeech2, HierSpeech-plus-plus]
---

# HierSpeech++

## 定义

基于层级语音合成的零样本 TTS 系统，通过分层的语音表示（self-supervised + acoustic）实现多说话人语音合成。

## 核心要点
1. 分层 speech 表示，先生成 SSL 特征再生成声学特征
2. 支持零样本多说话人合成
3. 在 LibriSpeech 上的 WER 较高（6.33），说话人相似度一般（Sim-O 0.51）

## 代表工作
- 本身即是代表工作

## 评测/常见数字
- LibriSpeech test-clean: WER 6.33, Sim-O 0.51, CMOS -0.41（相对 NaturalSpeech 3）

## 相关概念
- [[VITS]]
- [[Zero-shot TTS]]
- [[SSL Speech Representation]]
