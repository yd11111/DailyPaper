---
type: concept
aliases: [MRTE, Multi-Reference Timbre Encoder]
---

# MRTE

## 定义
GPT-SoVITS 中用于融合内容编码和音色信息的模块。以 SSL 内容编码为 Query、文本编码为 Key/Value 做 4 头 Cross-Attention，再加上全局说话人嵌入的残差连接。

## 数学形式

$$
\text{MRTE}(\mathbf{h}_{\text{ssl}}, \mathbf{h}_{\text{text}}, \mathbf{g}) = \text{Conv}_{1\times1}\left(\text{MHA}(\mathbf{h}_{\text{ssl}}, \mathbf{h}_{\text{text}}, \mathbf{h}_{\text{text}}) + \mathbf{h}_{\text{ssl}} + \mathbf{g}\right)
$$

## 核心要点
1. 通过 [[Cross-Attention]] 融合语言内容和文本语义
2. 残差连接保留原始 SSL 编码和全局说话人嵌入
3. 内置消融模式：mode 1 去除 cross-attention（仅音色），mode 2 去除 SSL（仅文本）
4. 输出供 VITS 的 Prior Encoder 使用

## 代表工作
- [[GPT-SoVITS]]: MRTE 是 SoVITS 阶段的核心融合模块

## 相关概念
- [[Cross-Attention]]
- [[Speaker Encoder]]
- [[ContentVec]]
