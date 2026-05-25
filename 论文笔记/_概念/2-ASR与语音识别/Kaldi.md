---
type: concept
aliases: [Kaldi ASR Toolkit]
---

# Kaldi

## 定义
开源语音识别工具箱，由 Daniel Povey 等人开发。长期作为学术界和工业界 ASR 系统的标准基础设施，支持 GMM-HMM、DNN-HMM、chain model 等多种声学建模方法。

## 核心要点
1. 提供完整的 ASR pipeline：特征提取、声学建模、语言建模、解码
2. 支持强制对齐（forced alignment）功能，广泛用于 TTS 数据准备
3. VALL-E / VALL-E X 使用 Kaldi 在 LibriSpeech 上训练的 ASR 模型对 LibriLight 做伪标注
4. 近年来逐渐被端到端框架（ESPnet、WeNet、NeMo）取代
5. 衍生项目 k2/Icefall 引入了 Zipformer 等现代架构

## 代表工作
- [[VALL-E-X]]: 使用 Kaldi 做 LibriLight 伪标注和 forced alignment

## 相关概念
- [[ASR]]
- [[Forced Alignment]]
- [[Conformer]]
