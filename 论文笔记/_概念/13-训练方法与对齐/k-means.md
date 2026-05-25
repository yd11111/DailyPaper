---
type: concept
aliases: [K-Means, k-means clustering, K-Means 聚类]
---

# k-means

## 定义
经典无监督聚类算法，将 N 个数据点分成 K 个簇，每个点属于距其最近的聚类中心。在语音领域常用于将自监督模型（HuBERT/w2v-BERT）的连续特征离散化为 semantic token。

## 核心要点
1. 输入：SSL 模型某一层的连续特征向量
2. 对特征做 mean-variance normalization 后运行 k-means
3. 聚类中心的索引即为离散 token
4. K 值决定码本大小（常用 500-2000），影响信息保留粒度

## 代表工作
- [[AudioLM]]: 用 w2v-BERT + k-means 提取 semantic token
- [[SPEAR-TTS]]: w2v-BERT 第 7 层 + K=512 k-means 生成 semantic token
- [[HuBERT]]: 训练目标本身就依赖 k-means 聚类的离散标签

## 相关概念
- [[Semantic Token]]
- [[w2v-BERT]]
- [[HuBERT]]
- [[VQ]]
