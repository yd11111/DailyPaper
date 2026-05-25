---
type: concept
aliases: [w2v-BERT, wav2vec-BERT, w2v-BERT XL]
---

# w2v-BERT

## 定义
结合 contrastive learning（wav2vec 2.0 风格）和 masked language modeling（BERT 风格）的自监督语音预训练模型。对音频波形提取表示后同时做对比学习和 MLM 训练。

## 核心要点
1. 0.6B 参数，Conformer-based 架构
2. 中间层（MLM 模块第 7 层）embeddings 经 k-means 量化后可作为高质量 semantic token
3. 输出 25 Hz 帧率（每 40 ms 一帧），1024 维
4. 归一化为零均值单位方差后再做 k-means 能显著提升语音判别力

## 代表工作
- [[AudioLM]]: 用 w2v-BERT 第 7 层 + k-means (K=1024) 提取 semantic token，码率 250 bps，ABX within-speaker 6.7%
- [[w2v-BERT 2.0]]: 升级版，用于 USM 等大规模多语种语音模型

## 评测/常见数字
- ABX within-speaker: 6.7%（w2v-BERT XL, layer 7, K=1024, 250 bps）
- ViSQOL 重建质量: 1.1（不可逆重建，仅承载语义）

## 相关概念
- [[HuBERT]]
- [[WavLM]]
- [[SSL Speech Representation]]
- [[Semantic Token]]
