---
type: concept
aliases: [跳跃连接, Skip]
---

# Skip Connection

## 定义
将网络中间层的激活直接传递到后面的层（通常是输出层附近），绕过中间的多层变换。与 [[Residual Connection]] 不同，跳跃连接通常汇聚多层的输出到同一位置。

## 核心要点
1. WaveNet 中每个残差 block 输出一路跳跃连接，所有跳跃连接在最终输出前求和
2. U-Net 中也大量使用跳跃连接（编码器→解码器）
3. 有助于多尺度特征融合和梯度流

## 代表工作
- [[WaveNet]]: 参数化跳跃连接汇聚所有层的信息

## 相关概念
- [[Residual Connection]]
