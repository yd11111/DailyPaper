---
type: concept
aliases: [Length Regulator, 长度调节器, LR]
---

# Length Regulator

## 定义
FastSpeech 中提出的模块，根据 Duration Predictor 预测的音素时长，将音素侧隐状态序列按时长复制展开为 mel-spectrogram 侧序列，解决 phoneme 与 mel 之间的长度不匹配问题。

## 数学形式

$$
\mathcal{H}_{mel} = \mathcal{LR}(\mathcal{H}_{pho}, \mathcal{D}, \alpha)
$$

- $\mathcal{H}_{pho}$: 音素隐状态序列（长度 n）
- $\mathcal{D} = [d_1, ..., d_n]$: 音素时长序列
- $\alpha$: 语速控制因子

## 核心要点
1. 按 duration 将每个音素隐状态复制 $d_i$ 次，拼接成 mel 长度的序列
2. $\alpha > 1$ 放慢语速，$\alpha < 1$ 加快语速
3. 可通过增大空格字符的 duration 在词间插入停顿
4. 机制极其简洁，但效果显著

## 代表工作
- [[FastSpeech]]: 首次提出 Length Regulator
- [[FastSpeech 2]]: 继承并沿用

## 相关概念
- [[Duration Predictor]]
- [[FastSpeech]]
- [[Non-Autoregressive]]
