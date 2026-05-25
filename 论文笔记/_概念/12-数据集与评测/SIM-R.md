---
type: concept
aliases: [Speaker Similarity Reconstructed, 重建相似度]
---

# SIM-R

## 定义

Speaker Similarity - Reconstructed，衡量合成语音与重合成（resynthesized）参考 prompt 的说话人余弦相似度。与 SIM-O（与原始参考 prompt 比较）不同，SIM-R 消除了 codec 重建误差的影响，更纯粹地反映生成模型的说话人建模能力。

## 数学形式

$$
\text{SIM-R} = \cos(\mathbf{e}_{\text{syn}}, \mathbf{e}_{\text{recon}})
$$

- $\mathbf{e}_{\text{syn}}$: 合成语音的说话人 embedding
- $\mathbf{e}_{\text{recon}}$: 将参考 prompt 通过同一 codec 重合成后的说话人 embedding
- 常用 WavLM-TDCNN 提取说话人 embedding

## 核心要点
1. SIM-R 通常高于 SIM-O，因为消除了 codec 重建误差
2. SIM-O 和 SIM-R 结合使用可以区分生成模型误差和 codec 误差
3. 0.6+ 算较好，0.7+ 算优秀

## 代表工作
- [[NaturalSpeech3]]: SIM-R 0.76（SOTA）

## 相关概念
- [[SIM-O]]
- [[SECS]]
- [[Speaker Encoder]]
