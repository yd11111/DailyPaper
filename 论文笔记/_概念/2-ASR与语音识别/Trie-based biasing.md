---
type: concept
aliases: [Trie-based ASR Biasing, Trie Biasing]
---

# Trie-based biasing

## 定义

ASR contextual biasing 的经典方法——把 bias phrase 列表组织成 prefix trie，在 beam search 解码过程中维护跨步的 trie 节点，让 hypothesis 命中关键词时获得 score boost。无需训练，可即插即用到 [[CTC]] / [[RNN-T]] / [[LAS]] 任意解码器。

## 数学形式

设 $\mathcal{T}$ 为 bias phrase trie。beam step $t$ 时，对每个 hypothesis $h$ 维护当前 trie 节点 $n_t(h)$。若新 token $y_t$ 在 $n_t$ 子节点中：
$$\text{score}(h\oplus y_t) \mathrel{+}= \lambda \cdot \mathbb{1}[y_t \in \text{children}(n_t)]$$
phrase 完整匹配时再加一次 reward。

## 核心要点

1. **零训练**：完全运行时方案，不动模型权重
2. **prefix trie**：O(1) 转移、O(|phrase|) 存储
3. **缺点**：beam search 解码结构耦合较深，对深度学习 ASR 优化器友好度差
4. **典型对比**：vs [[CLAS]] / [[NAM-Encoder]]（learned biasing） / [[CTC-WS-Streaming]]（CTC-domain biasing）
5. **常见在生产 ASR 框架**：Nvidia NeMo、ESPnet 等都内置 trie biasing

## 代表工作

- ESPnet / NeMo / Kaldi 的 contextual decoding 模块
- 各种语音助手生产系统的 contact-name biasing 实现

## 评测/常见数字

- 在 contact-name 测试集，WER 相对 no-bias baseline 降低 10–20%（不如 learned 方法激进但更鲁棒）

## 相关概念

- [[CLAS]]
- [[NAM-Encoder]]
- [[CTC-WS-Streaming]]
- [[Beam Search]]
