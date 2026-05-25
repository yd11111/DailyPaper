---
type: concept
aliases: [Spectral Clustering, 谱聚类]
---

# Spectral Clustering

## 定义
基于图拉普拉斯矩阵特征分解的聚类方法，将数据点映射到低维特征空间后再做 K-Means。在语音领域广泛用于说话人聚类（speaker diarization）和数据处理 pipeline 中的说话人分组。

## 核心要点
1. 构建相似度图（如余弦相似度），计算归一化拉普拉斯矩阵
2. 取前 k 个最小特征值对应的特征向量，形成低维表示
3. 在低维空间中执行 K-Means 聚类
4. 在说话人分类任务中优于直接 K-Means，能捕捉非线性结构

## 代表工作
- [[FireRedTTS]]: 数据 pipeline 中对说话人 embedding 做谱聚类 + 迭代合并（cos sim > 0.8）

## 相关概念
- [[Speaker Diarization]]
- [[ECAPA-TDNN]]
