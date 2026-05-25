---
type: concept
aliases: [MusicLM]
---

# MusicLM

## 定义
Google 提出的文本条件音乐生成模型，将 AudioLM 的分层 token 建模框架扩展到 text-to-music 场景，引入 MuLan 做文本-音乐对齐。

## 核心要点
1. 继承 AudioLM 的 semantic → acoustic 分层建模
2. 用 MuLan embedding 做文本条件
3. 生成高质量、长时间（数分钟）的音乐
4. 支持文本描述、旋律哼唱等多种条件

## 代表工作
- [[AudioLM]]: MusicLM 的前置框架

## 相关概念
- [[AudioLM]]
- [[Semantic Token]]
- [[Acoustic Token]]
