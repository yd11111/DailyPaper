---
type: concept
aliases: [Mini-Omni, Mini-Omni 2]
---

# Mini-Omni

## 定义

清华开源端到端语音对话模型系列。把语音生成放在 [[LLM Backbone|backbone]] 内部直接吐 speech token，是 "backbone-direct speech generation" 范式的代表。

## 核心要点

1. **Backbone 直出 speech token**：与 [[MiniCPM-o 4.5]] 把 speech token 下放给轻量 decoder 的设计相反
2. **效率代价**: backbone 必须每秒约 25 step 解码（语速），影响吞吐
3. **能力代价**: 直接训练 speech token 输出会拖累 backbone 的核心语言能力（[[Catastrophic Forgetting]]）
4. **代表 "thinking while speaking" 流式生成思路**

## 在 [[MiniCPM-o 4.5]] 论文中的位置

- 反例对照：被 §2 明确指出"backbone direct generation 会显著影响效率与语言能力"
- 流式 TTS 范式比较：[[TAIL]] vs Mini-Omni 的 large text lead 策略

## 相关概念

- [[Step-Audio]]: 同思路的另一代表
- [[MiniCPM-o]]: 反向设计
- [[Speech Token Decoder]]
