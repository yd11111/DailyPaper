---
type: concept
aliases: [Dynamic Time Warping, 动态时间规整]
---

# DTW

## 定义
Dynamic Time Warping，一种衡量两个时间序列相似度的算法，允许非线性对齐以处理速度差异。在语音领域常用于计算两段语音的 pitch（F0）轮廓距离。

## 数学形式

$$
\text{DTW}(X, Y) = \min_{\pi} \sum_{(i,j) \in \pi} d(x_i, y_j)
$$

- $X, Y$: 两个时间序列
- $\pi$: 对齐路径（满足单调递增约束）
- $d(\cdot, \cdot)$: 逐帧距离函数（通常为欧氏距离）

## 核心要点
1. 在 TTS 评测中，DTW 距离常用于衡量合成语音与参考语音的韵律相似度（pitch contour matching）
2. DTW 值越小表示韵律越接近参考
3. 比直接计算帧级 F0 RMSE 更鲁棒，因为它容忍说话速度差异

## 代表工作
- [[MegaTTS2]]: 用 pitch 的 DTW 距离评估 prosody 建模质量

## 相关概念
- [[Prosody Transfer]]
- [[MOS]]
- [[WER]]
