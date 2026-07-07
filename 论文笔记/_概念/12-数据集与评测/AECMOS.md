---
type: concept
aliases: [AEC MOS, Echo MOS]
---

# AECMOS

## 定义
微软提出的自动化回声消除质量评估指标，包含两个子指标：EMOS（echo annoyance，回声烦扰度）和 DMOS（other degradations，其他退化度）。两者平均得到 MOS_avg。评分范围 1-5，越高越好。

## 核心要点
1. 是 ICASSP AEC Challenge 的官方评测指标
2. 不需要参考信号（non-intrusive），适合盲测评估
3. EMOS 侧重回声残留程度，DMOS 侧重语音失真/噪声等其他退化
4. 典型数值：强系统 DT EMOS > 4.5，DT DMOS > 4.0

## 代表工作
- [[LMPAN]]: DT EMOS 4.63, DT DMOS 4.17, MOS_avg 4.49
- [[DeepVQE]]: DT EMOS 4.62, DT DMOS 4.02, MOS_avg 4.40

## 相关概念
- [[AEC]]
- [[ERLE]]
- [[MOS]]
- [[DNSMOS]]
