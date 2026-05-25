---
type: concept
aliases: [Transducer, RNN-T, RNN-Transducer, RNNT]
---

# Transducer

## 定义
端到端 ASR 的主流架构之一（RNN-Transducer / Conformer-Transducer），由编码器（encoder）、预测网络（prediction network）和联合网络（joint network）三部分组成，天然支持流式识别。

## 核心要点
1. 编码器处理音频特征，预测网络处理已识别 token 历史，联合网络融合两者做输出决策
2. 训练使用 Transducer loss（RNN-T loss），基于 forward-backward 算法
3. 天然支持流式推理，无需等待整段音频
4. 现代实现常用 Conformer 替代 RNN 作为编码器（Conformer-Transducer）

## 代表工作
- [[FireRedTTS]]: 数据 pipeline 中使用两遍 Transducer-based ASR 做批量转写
- Zipformer-Transducer (k2/Icefall)

## 评测/常见数字
- LibriSpeech test-clean WER: ~2-3%（Conformer-Transducer）
- 支持流式和非流式两种模式

## 相关概念
- [[ASR]]
- [[Conformer]]
- [[Whisper]]
