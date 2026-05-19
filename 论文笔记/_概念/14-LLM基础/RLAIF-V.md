---
type: concept
aliases: [RLAIF-V]
---

# RLAIF-V

## 定义

清华 NLP 组提出的 **基于开源 AI 反馈** 的多模态对齐方法，用开源 LLM 给视觉响应打偏好标签替代昂贵人工标注，专注降低 [[Hallucination|幻觉]]。被 [[MiniCPM-V]] / [[MiniCPM-o]] 系列广泛使用。

## 核心要点

1. **AI 反馈**: 用 open-source LLM 评估 response 准确性，标注 chosen/rejected 对
2. **专攻 multimodal hallucination**: 改善视觉一致性 (HallusionBench / MMHal)
3. **可迁移性**: [[MiniCPM-o 4.5]] 发现 RLAIF-V 在图文上学到的反幻觉能力可迁移到 omni-modal full-duplex streaming 场景
4. **配套算法**: 通常以 [[DPO]] 风格或与 [[GRPO]] 联合使用

## 代表工作

- RLAIF-V 原论文: Yu et al. 2024
- [[MiniCPM-V]] / [[MiniCPM-o 4.5]]: 实际部署

## 相关概念

- [[Hallucination]]
- [[DPO]]
- [[GRPO]]
- [[RLHF]]
