---
type: concept
aliases: [Turn-taking, 轮转]
---

# Turn-taking

## 定义
对话中谁说话、何时说话、何时停止的决策机制。

## 核心要点
1. Assistant turn-taking: 用户讲完后助手是否及时开始回答
2. User turn-taking: 助手讲话时用户开始说话，助手是否及时停下来听
3. 评测常用 Acc@K（第 K 个 token 时是否正确决策）

## 代表工作
- [[OmniFlatten]] 的 Table 4 显示其 turn-taking 大幅优于 [[Moshi]]
- [[Synchronization-Turn-Taking]]: 用因果 LSTM 探针测 [[Moshi]] 的轮替预测能力（[[EOI Prediction]] + [[Hold vs Non-Hold]]，production / perception 双视角）
- [[ModeratorLM]]: 首个角色条件化多方轮转模型，CoT 推理 + 角色 prompt 在多方场景 F1 达 0.76-0.79

## 相关概念
- [[OmniFlatten]]
- [[EOI Prediction]]
- [[Hold vs Non-Hold]]
- [[IPU]]
