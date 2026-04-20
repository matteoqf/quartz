---
category: AI大模型
priority: ⭐⭐⭐⭐
tags: [进阶概念]
updated: 2026-04-21
---

# Context Window (上下文窗口)

## 一句话理解

AI一次性能够"看到"并处理的全部信息长度——相当于AI的工作记忆。窗口越大，能同时分析的信息越多，但计算成本也呈平方级增长。

## 核心必背

> **核心必背**：Context Window（上下文窗口）是Transformer模型在一次推理中能处理的最大Token数量，通常以K（千tokens）为单位。GPT-4o支持128K tokens，Claude 3支持200K tokens。窗口内的所有Token相互之间都可以做Attention——这既是优势（全局信息整合），也是瓶颈（Attention计算量=O(n²)，n是序列长度）。

## 为什么Context Window重要

1. **信息容量**：能同时处理多少内容（一部小说的长度？一篇论文？整个代码库？）
2. **长程依赖**：语言中相隔很远的词可能相互关联（如指代消解）
3. **应用场景**：多轮对话、长文档分析、代码补全、RAG检索增强等场景都受窗口大小制约

## 窗口大小的演进

| 模型 | 上下文窗口 |
|------|-----------|
| GPT-3 | 4K tokens |
| GPT-3.5 | 16K tokens |
| GPT-4 | 8K / 32K tokens |
| Claude 2 | 100K tokens |
| GPT-4o | 128K tokens |
| Claude 3.5 | 200K tokens |
| Gemini 1.5 | 1M tokens |

## Attention的计算困境

Attention机制的核心运算：对于长度为n的序列，需要计算n×n的注意力矩阵。

- 1K tokens：需要约100万次运算
- 100K tokens：需要约100亿次运算
- 1M tokens：需要约1万亿次运算

这就是为什么长上下文会大幅拖慢推理速度、增加显存占用。

## Context Window的限制与挑战

### 1. 位置编码外推（Position Encoding Extrapolation）
模型在训练时见过固定最大长度（如4K），如何在推理时泛化到更长的窗口？位置编码的方案至关重要——RoPE、ALiBi等编码方式都试图解决这个问题。

### 2. Lost in the Middle（中间丢失）
研究表明，即使窗口足够大，模型对"中间部分"的信息提取能力也较弱——对开头和结尾的信息记忆最好，中间的容易被忽略。

### 3. 内存与成本
- 显存占用 = Batch Size × Sequence Length² × Hidden Size × 4 bytes
- 推理成本随窗口大小线性增长（KV-Cache存储）

## 突破窗口限制的工程方法

- **RAG（检索增强生成）**：不把所有内容放窗口，而是检索相关片段
- **分层处理**：先摘要再分析
- **Streaming（流式）**：分段输入，累加处理
- **Sparse Attention**：只计算部分位置的相关性，而非全连接

## 实际影响

如果你的任务需要分析超过32K tokens的文档，普通的GPT-4无法一次性处理，必须切分或用支持更大窗口的模型（如Claude 3.5或GPT-4o）。这是选型时的重要考量。
