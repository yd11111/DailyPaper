---
type: concept
aliases: [TIMIT Corpus, DARPA TIMIT]
---

# TIMIT

## 定义
DARPA 资助的标准语音识别评测数据集（Garofolo et al., 1993），包含 630 名美式英语说话人的录音，附有音素级标注。常用于音素识别（PER）评测。

## 核心要点
1. 标准划分：训练 3696 句，核心测试 192 句
2. 评测指标：Phone Error Rate (PER)
3. WaveNet 在 TIMIT 上取得 18.8 PER（直接从原始波形训练的最佳结果）
4. 规模较小，现代 ASR 更常用 [[LibriSpeech]] 等大规模数据集

## 评测/常见数字
- 经典 HMM-GMM: ~25% PER
- DNN-HMM: ~20% PER
- WaveNet (raw waveform): 18.8% PER

## 代表工作
- [[WaveNet]]: 直接从原始波形的音素识别

## 相关概念
- [[LibriSpeech]]
