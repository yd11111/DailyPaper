---
type: concept
aliases: [Contextual Listen-Attend-Spell, Contextual LAS]
---

# CLAS

## 定义

Google 2018 年提出的 ASR contextual biasing 经典方法 (Pundak et al., SLT 2018)。在 LAS / RNN-T 结构上加一个「bias encoder」，把动态关键词列表（用户联系人、地名等）编码成 attention key/value，再用一层 cross-attention 与解码器结合，在不重训声学模型的前提下让识别器对关键词更敏感。

## 数学形式

bias encoder 输出 $h^b_i = \text{BiasEnc}(z_i)$，其中 $z_i$ 是第 $i$ 个 bias phrase。解码器在每步生成 $y_t$ 时：
$$c^b_t = \sum_i \alpha^b_{t,i} h^b_i, \quad \alpha^b_{t,i} = \text{softmax}(s_t W h^b_i)$$
最终 logits 来自标准 LAS 状态与 $c^b_t$ 拼接。

## 核心要点

1. **不动声学模型**：bias 列表运行时给定，灵活更新
2. **attention bias**：用 query-key 相似度选择关键词
3. **加 `</bias>` token**：允许模型「主动决定」不 bias，避免强行替换
4. 与 [[CTC-WS-Streaming]] / [[NAM-Encoder]] / [[Trie-based biasing]] 并列为 contextual ASR 三大派系

## 代表工作

- Pundak et al., SLT 2018 (arXiv 1808.02480)
- Google Assistant 在生产环境的 contact biasing 用过其变种

## 评测/常见数字

- 在带 contact list 的语音指令测试集上，WER 相对降低 20–30%（vs no-bias LAS baseline）

## 相关概念

- [[CTC-WS-Streaming]]
- [[NAM-Encoder]]
- [[Trie-based biasing]]
- [[Conformer]]
