---
type: concept
aliases: [Echo Return Loss Enhancement, 回声回损增强]
---

# ERLE (Echo Return Loss Enhancement)

## 定义
回声回损增强，衡量回声消除系统抑制回声的能力，单位 dB。值越高表示回声抑制越彻底。

## 数学形式

$$
\text{ERLE}(\text{dB}) = 10 \log_{10} \frac{\sum_n y^2(n)}{\sum_n \hat{s}^2(n)}
$$

其中 $y(n)$ 是含回声的麦克风信号，$\hat{s}(n)$ 是处理后的输出信号（理想情况下只含近端语音）。

## 核心要点
1. 主要在 single-talk far-end (ST-FE) 和 double-talk (DT) 场景下分别评估
2. 典型数值：优秀系统 > 50 dB，DeepVQE 达 65.7 dB
3. 高 ERLE 不一定意味着好的主观质量——过度抑制可能损伤近端语音

## 代表工作
- [[LMPAN]]: DT ERLE 47.15 dB
- [[DeepVQE]]: DT ERLE 65.7 dB

## 相关概念
- [[AEC]]
- [[AECMOS]]
- [[PESQ]]
