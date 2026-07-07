---
type: concept
aliases: [Voice Bench]
---

# VoiceBench

## 定义
基于 LLM 的语音助手评测基准（Chen et al. 2024），包含 AlpacaEval、CommonEval、IFEval、SD-QA、AdvBench 五个子集，覆盖指令跟随、常识问答、方言问答、安全拒答等维度。支持 s2t 和 s2s 评测模式。

## 核心要点
1. AlpacaEval / CommonEval 测试通用指令跟随能力
2. IFEval 测试格式约束型指令跟随（如 "include the phrase 'My answer is no.'"）
3. SD-QA 测试方言语音理解
4. AdvBench 测试对不安全请求的拒答能力
5. 常与 [[OpenAudioBench]] 组合使用，两者得分归一化至 0-100 后取平均

## 代表工作
- [[FlexiSLM]]: 在此 benchmark 上展示了跨帧率的鲁棒性
- Chen et al. 2024 (arXiv:2410.17196): VoiceBench 原论文

## 相关概念
- [[OpenAudioBench]]
- [[LibriSpeech]]
