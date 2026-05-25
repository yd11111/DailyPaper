---
type: concept
aliases: [ECAPA-TDNN, Emphasized Channel Attention Propagation and Aggregation TDNN]
---

# ECAPA-TDNN

## 定义
基于 TDNN（Time Delay Neural Network）的说话人验证模型，通过 Squeeze-and-Excitation 通道注意力、Res2Net 多尺度特征和 attentive statistics pooling 增强说话人表示能力。处理 Mel-Spectrogram 输入，输出固定维度的说话人 embedding。

## 核心要点
1. 在每个 TDNN 残差块中引入 SE（Squeeze-Excitation）通道注意力机制
2. 使用 Res2Net 结构实现多尺度特征聚合
3. 通过 attentive statistics pooling 将变长序列压缩为固定长度 utterance-level embedding
4. 在 VoxCeleb 等说话人验证任务上长期保持 SOTA

## 代表工作
- [[FireRedTTS]]: 用作声学编码器提取全局 utterance-level embedding（音色/风格/环境）
- [[CosyVoice]]: 说话人 embedding 提取

## 相关概念
- [[HuBERT]]
- [[WavLM]]
- [[3D-Speaker]]
