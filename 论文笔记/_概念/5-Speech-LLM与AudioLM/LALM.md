---
type: concept
aliases: [Large Audio Language Model, 大型音频语言模型, Audio LLM]
---

# LALM

## 定义

Large Audio Language Model 的简称——以 LLM 为 backbone、接受**通用音频**（语音 + 环境声 + 音乐）作为输入、输出文本的模型族。与只处理语音的 [[Speech LLM]] 不同，LALM 关注 audio captioning、AQA、声音事件理解等更广任务；与 [[Whisper]] 一类纯 ASR 不同，LALM 强调多任务的对话与推理能力。

## 核心要点

1. **输入侧**：通常 audio encoder（[[CLAP]] / [[BEATs]] / [[Whisper encoder]] 等）+ projection layer
2. **backbone**：冻结或部分微调的 LLM（[[LLaMA]] / [[Qwen]] / [[Mistral]] 一类）
3. **输出侧**：默认仅文本；若加 TTS 头则升级为 speech-out 模型
4. **典型代表**：[[Audio-Flamingo]]、[[SALMONN]]、[[Qwen-Audio]]、[[Qwen2-Audio]]、[[Pengi]]
5. **典型任务**：audio captioning、AQA、AAC、speech translation、声音事件理解
6. **典型短板**：选择性听觉（鸡尾酒会）、多说话人理解、长 form 音频推理仍弱

## 代表工作

- [[Audio-Flamingo]]：NVIDIA few-shot ICL LALM
- [[SALMONN]]：USTC 双 encoder + Q-Former + LLM
- [[Qwen2-Audio]]：阿里 audio + speech 通用 LALM
- [[MUSA]]：专评 LALM 选择性听觉的 benchmark

## 评测/常见数字

- AudioCaps captioning: SOTA LALM CIDEr ~80
- AQA SOTA 接近 70% acc，但带 distractor 后掉到 < 50%

## 相关概念

- [[Speech LLM]]
- [[Audio Codec]]
- [[Audio-Flamingo]]
- [[SALMONN]]
- [[Qwen-Audio]]
