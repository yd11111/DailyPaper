---
type: concept
aliases: [Pixel RNN, Pixel Recurrent Neural Networks]
---

# PixelRNN

## 定义
van den Oord et al. (2016a) 提出的自回归图像生成模型，使用 LSTM 按光栅顺序逐像素生成图像。与 [[PixelCNN]] 同篇论文提出，但因 RNN 无法并行训练而较慢。

## 核心要点
1. 使用 Row LSTM 或 Diagonal BiLSTM 进行像素级自回归生成
2. 生成质量高但训练/推理慢
3. PixelCNN 是其卷积版本替代方案，训练更高效

## 代表工作
- [[WaveNet]]: 受 PixelRNN/PixelCNN 系列启发

## 相关概念
- [[PixelCNN]]
- [[Autoregressive Model]]
