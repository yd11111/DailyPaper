---
type: concept
aliases: [Bidirectional and Auto-Regressive Transformers]
---

# BART

## 定义
Facebook/Meta 提出的 denoising autoencoder 预训练框架，采用 encoder-decoder 架构。预训练时对输入施加多种噪声（token 删除、句子打乱、span masking 等），训练模型恢复原始文本。

## 核心要点
1. 结合了 BERT（双向编码）和 GPT（自回归解码）的优势
2. 预训练噪声包括：token masking、token deletion、text infilling、sentence permutation、document rotation
3. 微调时可用于生成式任务（摘要、翻译）和判别式任务
4. 其 denoising pretraining 范式被 SPEAR-TTS 借鉴用于 semantic token 序列

## 代表工作
- Lewis et al. (2020): "BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension"
- [[SPEAR-TTS]]: 借鉴 BART/T5 的 denoising pretrain，在 semantic token 上做 token deletion + reconstruction

## 相关概念
- [[T5]]
- [[BERT]]
- [[Encoder-Decoder Transformer]]
