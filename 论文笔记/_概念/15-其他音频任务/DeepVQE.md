---
type: concept
aliases: [Deep Voice Quality Enhancement]
---

# DeepVQE

## 定义
端到端的联合声学回声消除（AEC）、噪声抑制（NS）和去混响（DRB）模型。通过单一网络同时处理三种退化，避免级联系统的误差累积。是 AEC 领域的重要基线模型。

## 核心要点
1. 端到端方法：直接从 mic + ref 输入到增强输出，无需 LAEC 前处理
2. 参数量 0.82M，MACs 315M——效果好但计算量较大
3. AEC Challenge 2023 盲测集上 DT ERLE 达 65.7 dB
4. DeepVQE-S 是轻量版本，常作为端侧部署的效果上界

## 评测/常见数字
- DT EMOS: 4.62, DT DMOS: 4.02, MOS_avg: 4.40
- DT ERLE: 65.7 dB

## 代表工作
- Evgenii et al.: DeepVQE 原始论文
- [[LMPAN]]: 以 DeepVQE 为主要对比基线

## 相关概念
- [[AEC]]
- [[ERLE]]
- [[AECMOS]]
