---
type: concept
aliases: [End-of-IPU Prediction, EOI Prediction, IPU 结束预测]
---

# EOI Prediction (End-of-IPU Prediction)

## 定义

**End-of-IPU Prediction**：在帧级时间轴上预测某一帧是否是当前 [[IPU]] 的最后一帧（即紧接着是 $\geq \tau$ 的静音）。是 [[Turn-taking]] 研究中刻画"说话即将结束"信号的细粒度任务。

## 数学形式

给定每帧 hidden state $\mathbf{h}_t$ 与帧级二值标签 $y_t \in \{0, 1\}$（仅 IPU 最后一帧为 1），训练分类器：

$$
\hat{y}_t = \sigma\big(f(\mathbf{h}_{t-\delta})\big)
$$

其中 $\delta \geq 0$ 是因果延迟（让探针只能看 $\delta$ 秒前的状态）。损失为帧级 BCE，评测用 AUC-ROC 随 $\delta$ 变化的曲线。

## 核心要点
1. **极不平衡**：正样本极稀疏（每 IPU 仅 1 帧），AUC 比 F1 更稳健
2. **比 turn-shift 更紧迫**：EOI 是即时的声学事件，决定模型何时插话
3. **延迟扫描**：$\delta$ 增大、AUC 仍高 → hidden state **提前**编码了 EOI 信号
4. **production vs perception**：自我预测 vs 对对方预测两个视角刻画不同认知过程

## 代表工作
- [[Synchronization-Turn-Taking]]: 在两个 [[Moshi]] 互相对话的 hidden state 上用 [[Causal LSTM]] 探针，扫描 $\delta \in [0, 1920]$ ms，证明 EOI 信号在 production 与 perception 视角都被提前编码

## 相关概念
- [[IPU]]
- [[Turn-taking]]
- [[Hold vs Non-Hold]]
- [[Causal LSTM]]
