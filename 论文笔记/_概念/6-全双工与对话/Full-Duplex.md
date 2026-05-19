---
type: concept
aliases: [Full-Duplex, 全双工, 全双工对话]
---

# Full-Duplex

## 定义

模型 **同时进行感知和响应**（边听/看边说）的交互范式，区别于 turn-based / half-duplex 的"轮流"模式。关键诉求是允许 user 中途打断 ([[Barge-In|barge-in]])、模型可主动发声、感知信号能即时影响正在生成的内容。

## 核心要点

1. **核心 vs turn-based**:
   - turn-based: perception 与 response 切成交替阶段，I/O 阻塞
   - full-duplex: 持续耦合，token 级并行
2. **关键评测指标**:
   - First-Packet Latency（首包延迟）
   - Barge-in Reaction Time（打断反应时间）
   - End-to-End Latency
   - 流畅度 / 中断容忍度 / 主动行为质量
3. **典型实现思路**:
   - **双流建模** (如 [[Moshi]]): 用户流 + 助手流分别建模，时间帧对齐
   - **时分多路复用** (如 [[Omni-Flow]] / [[MiniCPM-o 4.5]]): 把多模态输入输出统一序列化到共享时间轴
   - **VAD-driven turn-taking**: 工程拼接，不算真正 full-duplex
4. **挑战**:
   - 文本与语音生成时间不一致 → 需 [[TAIL]] 等时延对齐机制
   - 模型自主决定 when/whether to output（避免抢话/沉默）
   - 训练数据需要时间戳对齐的多模态 stream

## 代表工作

- [[Moshi]]: Kyutai，双流建模 + Mimi codec，160 ms 端到端延迟
- [[MiniCPM-o 4.5]]: 9B 端侧 + [[Omni-Flow]] + [[TAIL]]
- [[GLM-4-Voice]]: 智谱端到端语音对话
- [[dGSLM]]: 离散 GSLM 双说话人对话
- [[VITA]]: 腾讯多模态交互
- [[SyncLLM]]: 实时双向 LLM
- [[LiveCC]] / [[StreamingVLM]]: vision-only 流式

## 评测/常见数字

| 模型 | 首包延迟 | 端到端延迟 | 备注 |
|---|---|---|---|
| Moshi | ~160 ms | — | 双流 codec |
| MiniCPM-o 4.5 (RTX 4090, INT4) | 0.58 s (64 frame) | RTF 0.21 | 端侧 |

## 相关概念

- [[Omni-Flow]]
- [[TAIL]]
- [[Barge-In]]
- [[Turn-Taking]]
- [[VAD]]
- [[Proactive Behavior]]
