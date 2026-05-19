---
type: concept
aliases: [MiniCPM-V, MiniCPM-V 4.5]
---

# MiniCPM-V

## 定义

OpenBMB / 清华 NLP 组的端侧多模态视觉语言模型系列。专注 **GPT-4V level on-device performance**，强 OCR / 文档理解 / 多图。是 [[MiniCPM-o]] omni 系列的视觉部分基础。

## 系列演进

| 版本 | 关键点 |
|---|---|
| MiniCPM-V 2.x | 早期视觉对话 |
| MiniCPM-V 4.5 | 强化 OCR、文档、视频；[[MiniCPM-o 4.5]] 直接基于其 pretrained checkpoint |

## 核心架构

- **视觉编码器**: [[SigLIP ViT]] + [[LLaVA-UHD]] 切片 + [[Resampler]]
- **Backbone**: 通常配合 Qwen 系列文本 backbone

## 相关概念

- [[MiniCPM-o]]: omni 扩展版
- [[LLaVA-UHD]]
- [[CapsFusion]]: 数据 pipeline 组件
- [[RLAIF-V]]: 反幻觉训练
