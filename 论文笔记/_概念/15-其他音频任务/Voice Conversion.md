---
type: concept
aliases: [Voice Conversion, VC, 语音转换]
---

# Voice Conversion

## 定义
语音转换，将一段语音的说话人身份转换为目标说话人，同时保留语言内容和韵律。区别于 TTS（从文本生成）和 SVC（歌声转换）。

## 核心要点
1. 核心挑战：解耦内容信息与说话人信息
2. 方法流派：encoder-decoder、GAN-based、diffusion-based、disentangled representation
3. [[VITS]] 多说话人模式天然支持 VC：通过 [[Normalizing Flow]] 的正逆变换实现说话人切换

## 代表工作
- [[VITS]]: 利用 flow 的可逆性实现 VC（附录 D）
- so-vits-svc: 基于 VITS 的歌声转换

## 相关概念
- [[VITS]]
- [[Normalizing Flow]]
- [[SIM-O]]
