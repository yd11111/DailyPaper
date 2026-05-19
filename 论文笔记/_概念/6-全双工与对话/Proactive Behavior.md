---
type: concept
aliases: [Proactive Behavior, 主动行为, Proactive Interaction]
---

# Proactive Behavior

## 定义

模型在没有显式 user request 的情况下，基于持续多模态观测主动发起行为（如 reminder、scene description、comment）。是从 **reactive (request-driven)** 转向 **context-driven** 交互范式的关键能力。

## 核心要点

1. **触发不依赖 user query**: 模型从 evolving 多模态环境本身判断何时行动。
2. **典型场景**: long-horizon assistance、ambient interaction、real-time scene description、proactive reminding。
3. **实现机制（[[MiniCPM-o 4.5]] / [[Omni-Flow]]）**:
   - out-stream 在每个 chunk 内自主决定输出 `[listen]` 还是真正的内容 token
   - 不需要外置 [[VAD]] 或 turn-taking 模块
   - Listen-Speak 控制头将 "是否说话" 与 "说什么" 解耦
4. **训练支撑**: 需要 task-specific 的 proactive 监督数据（人工构造 continuous scene description / proactive reminding 等场景）。
5. **当前局限**: 大多数模型主动行为仍简单，缺乏 long-horizon planning / 自我发起的复杂助理行为。

## 代表工作

- [[MiniCPM-o 4.5]]: 首个明确把 proactive 作为目标能力的开源 9B Omni-LLM

## 相关概念

- [[Omni-Flow]]: 提供 proactive 行为的统一建模框架
- [[Full-Duplex]]
- [[VAD]]
- [[Turn-Taking]]
