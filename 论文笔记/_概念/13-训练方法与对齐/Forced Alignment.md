---
type: concept
aliases: [强制对齐, FA, Force Alignment]
---

# Forced Alignment

## 定义
将文本（phoneme / character / word）序列与音频在时间维度上对齐，确定每个文本单元对应的起止时间戳。输入为 waveform + text transcript，输出为时间对齐标注。

## 数学形式

$$
\text{FA}: (x_{1:T}, y_{1:N}) \rightarrow \{(y_n, t_n^{\text{start}}, t_n^{\text{end}})\}_{n=1}^{N}
$$

- $x_{1:T}$: 音频信号（采样率通常 16 kHz）
- $y_{1:N}$: 文本序列（phoneme 或 character 级别）
- $(t_n^{\text{start}}, t_n^{\text{end}})$: 第 $n$ 个文本单元的起止时间

## 核心要点
1. 传统方法基于 HMM-GMM（Kaldi/HTK），现代方法用 CTC / wav2vec 2.0 / Whisper 做对齐
2. Montreal Forced Aligner (MFA) 是目前 TTS 训练中最常用的开源工具
3. 对齐结果直接用于 [[Duration Predictor]] 训练数据的标注
4. 也用于数据清洗（如 GLM-TTS 中用于标点优化的字级时长统计）

## 代表工作
- MFA (Montreal Forced Aligner): 基于 Kaldi 的主流对齐工具
- [[GLM-TTS]]: 用于标点优化中的字符发音时长统计

## 评测/常见数字
- 对齐精度通常用边界误差衡量（boundary error），MFA 典型误差 < 20 ms
- 对齐失败率随数据质量和语言而异

## 相关概念
- [[Duration Predictor]]
- [[CER]]
- [[WER]]
