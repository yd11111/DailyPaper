---
type: concept
aliases: [SR, 语音占比]
---

# Speech Ratio

## 定义

一段音频中被 [[VAD]] 判定为"语音"的帧数占总帧数的比例，用于衡量该段是否包含过多非语音内容（沉默、音乐、噪声、笑声等）。常用于 TTS / ASR 训练数据筛选。

## 数学形式

$$
\mathrm{SR}(x) = \frac{\sum_t \mathbb{1}[\mathrm{VAD}(x_t)=1]}{T}
$$

- $x_t$: 第 $t$ 帧
- $T$: 总帧数
- VAD: 常用 Silero VAD / PyAnnote VAD

## 核心要点

1. **数据筛选**: [[Raon-OpenTTS]] 用 SR < 0.79（21% 非语音）作为 15% 分位过滤阈值
2. **典型分布**: 高质量 audiobook 数据 SR > 0.9；YouTube / podcast 因有间隙容易低于 0.8
3. **与 [[DNSMOS]] / WER 的互补性**: SR 抓"有没有语音"，DNSMOS 抓"语音质量"，WER 抓"音文是否匹配"，三者覆盖不同失败模式

## 代表工作

- [[Raon-OpenTTS]]: 把 Speech Ratio 与 [[DNSMOS]] / [[Whisper]] WER 组合成 Combined 15% 过滤

## 相关概念

- [[VAD]]: 计算来源
- [[DNSMOS]]: 同为数据过滤的常用指标
