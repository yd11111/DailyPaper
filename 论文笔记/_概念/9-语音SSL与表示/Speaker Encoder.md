---
type: concept
aliases: [Speaker Encoder, 说话人编码器, Speaker Embedding]
---

# Speaker Encoder

## 定义
从语音中提取固定维度的说话人身份向量（speaker embedding）的模型，用于说话人验证、语音克隆中的音色条件化。

## 核心要点
1. 常见架构：ECAPA-TDNN、X-Vector、LSTM-based、ERes2Net/ERes2NetV2
2. 输出向量通常 L2 归一化后用余弦相似度比较
3. 在 TTS 中作为全局条件注入解码器，控制生成音色
4. 训练通常使用大规模说话人分类任务 + metric learning

## 代表工作
- [[GPT-SoVITS]]: V2Pro 使用 ERes2NetV2 提取 20480 维说话人嵌入
- [[CosyVoice 2]]: 使用说话人编码器做零样本音色控制

## 评测/常见数字
- SIM-O / SECS 指标用于评测说话人相似度（0.6+ 算好）

## 相关概念
- [[X-Vector]]
- [[ERes2Net]]
- [[CAM++]]
