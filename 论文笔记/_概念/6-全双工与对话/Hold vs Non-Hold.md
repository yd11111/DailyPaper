---
type: concept
aliases: [Hold vs Non-Hold, Turn-Management Prediction, 续说 vs 让出]
---

# Hold vs Non-Hold

## 定义

对话中的二元轮替管理决策：在某个 [[IPU]] 边界上，当前说话人是**续说（Hold）**还是**让出话语权（Non-Hold）**。比 [[EOI Prediction]] 粒度更粗，刻画的是宏观对话方向决策。

## 数学形式

在每个 [[IPU]] 边界事件 $e_i$（出现在时刻 $t_i$）上，给定边界后两人是否再开口、谁先开口：

$$
y_i =
\begin{cases}
\text{Hold} & \text{若原说话人在}\ \tau_{\text{pause}}\ \text{内续说} \\
\text{Non-Hold} & \text{若对方先开口（含轻度重叠）}
\end{cases}
$$

常见清洗规则（[[Synchronization-Turn-Taking]]）：
- 排除 pause > 1 s 的样本（太长，不算自然轮替）
- 排除 overlap > 240 ms 的样本（强重叠不属于"干净换手"）
- 轻度 overlap (≤240 ms) 计为 Non-Hold

训练同 [[EOI Prediction]]：因果延迟 $\delta$ + 因果 LSTM + BCE + AUC-ROC。

## 核心要点
1. **稀疏离散事件**：每段对话只有几十个 IPU 边界，正负样本比 EOI 更平衡但总量小
2. **衰减比 EOI 慢**：在大延迟下 AUC 下降更慢 → 宏观决策可能在 IPU 开始时就形成
3. **对决策强度敏感**：人为偏置（如降低 [[Moshi]] 的 PAD logit）会破坏 perception 侧的可预测性
4. **production / perception**：分别测自我决策与对对方决策的预判

## 代表工作
- [[Synchronization-Turn-Taking]]: 在 [[Moshi]] 仿真对话上以 80 ms 静音阈值切 [[IPU]]，按上述规则清洗标签，用 [[Causal LSTM]] 探针扫描 $\delta \in [0, 1920]$ ms

## 相关概念
- [[Turn-taking]]
- [[EOI Prediction]]
- [[IPU]]
- [[Backchannel]]
