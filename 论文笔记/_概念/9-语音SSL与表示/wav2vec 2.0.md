---
type: concept
aliases: [wav2vec2, wav2vec 2.0, w2v2]
---

# wav2vec 2.0

## 定义

Meta/FAIR 提出的自监督语音表示学习模型。通过对连续语音特征做量化后进行对比学习 (contrastive learning)，学习通用语音表示。在 ASR fine-tuning 后可达极低 WER；也广泛用作冻结特征提取器。

## 数学形式

训练目标：
$$
\mathcal{L} = -\log \frac{\exp(\mathrm{sim}(c_t, q_t)/\kappa)}{\sum_{q'\in Q_t} \exp(\mathrm{sim}(c_t, q')/\kappa)}
$$

- 输入：16kHz 原始波形
- CNN encoder → 连续特征 → Transformer → 上下文化表示 $c_t$
- 量化模块：Gumbel-Softmax 产生离散 code $q_t$
- 输出维度：768 (Base) / 1024 (Large)，帧率 50 Hz (20ms)

## 核心要点

1. 预训练阶段无需标注数据——纯自监督
2. 量化模块引入离散瓶颈，迫使模型学习语言信息
3. 可加 CTC head 做 ASR fine-tuning（Libri-Light 10min 标注即可达合理 WER）
4. CTC posterior 是低维（词表大小 ~29-72）的丰富内容表示
5. 常被用作冻结特征提取器（如 SR-FD 中用其 CTC head 输出做分布匹配）

## 代表工作

- Baevski et al. (2020): wav2vec 2.0 原文
- [[SR-FD]]: 用冻结 wav2vec 2.0 CTC 作为内容特征提取器
- XLS-R / XLSR-53: 多语种扩展版

## 评测/常见数字

- LibriSpeech test-clean WER: ~1.8% (Large + LM)
- 无 LM: ~2.3% (Large)
- 常用作下游任务（SID / ER / ASR）的 frozen feature extractor

## 相关概念

- [[HuBERT]]
- [[WavLM]]
- [[CTC]]
- [[Whisper]]
