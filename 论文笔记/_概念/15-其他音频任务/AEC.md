---
type: concept
aliases: [Acoustic Echo Cancellation, 声学回声消除, Echo Cancellation]
---

# AEC (Acoustic Echo Cancellation)

## 定义
声学回声消除，从麦克风信号中去除由扬声器播放的远端信号经房间声学路径传播后产生的回声成分。是全双工通信和对话系统的基础前端处理模块。

## 数学形式

$$
y(n) = s(n) + e(n) + n(n)
$$

其中 $y$ 为麦克风信号，$s$ 为近端语音，$e = r * h$ 为回声（远端信号 $r$ 与回声路径响应 $h$ 的卷积），$n$ 为噪声。AEC 的目标是从 $y$ 中估计并消除 $e$。

## 核心要点
1. 传统方法基于自适应滤波（如 [[NLMS]]、RLS），建模线性回声路径
2. 现代神经网络方法可以处理非线性回声（设备失真、音效处理等）
3. 关键挑战：double-talk（双讲）场景下同时有近端语音和回声，需要避免误消近端语音
4. 评测指标：[[ERLE]]（Echo Return Loss Enhancement）、[[AECMOS]]

## 代表工作
- [[DeepVQE]]: 端到端联合 AEC + NS + DRB
- [[LMPAN]]: 轻量多路径对齐 AEC+NS
- FADI-AEC: 基于扩散的 AEC
- ICASSP AEC Challenge (2021-2023): 标准化评测

## 相关概念
- [[NLMS]]
- [[ERLE]]
- [[AECMOS]]
- [[全双工]]
- [[VAD]]
