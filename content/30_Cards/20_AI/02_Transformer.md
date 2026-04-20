---
category: AI大模型
priority: ⭐⭐⭐⭐
tags: [核心概念]
updated: 2026-04-21
---

# Transformer

Transformer是2017年由Google在论文《Attention Is All You Need》中提出的深度学习架构，它彻底改变了自然语言处理（NLP）领域的发展方向。在此之前，序列建模主要依赖RNN（循环神经网络）和LSTM（长短期记忆网络），这些架构存在长距离依赖难以捕获、并行计算效率低等问题。

## 核心组件

Transformer的核心是自注意力机制（Self-Attention），它允许输入序列中的每个位置同时关注序列中的所有其他位置，从而有效解决长距离依赖问题。具体来说，自注意力通过计算Query（查询）、Key（键）和Value（值）三个向量之间的点积相似度来确定注意力权重。

Transformer的整体结构由编码器（Encoder）和解码器（Decoder）组成。编码器负责将输入序列映射为连续的向量表示，解码器则基于编码器的输出和已生成的内容逐步预测下一个输出 token。现代大模型（如GPT系列）通常只使用解码器部分，这种架构被称为"因果Transformer"或"GPT架构"。

## 技术优势

相比RNN，Transformer具有并行计算效率高、路径长度短（任意两点之间的信息传递路径为O(1)）、可扩展性强等优势。这些特性使得训练更大规模的模型成为可能，也奠定了现代大语言模型的技术基础。

> **核心必背**：Transformer以自注意力机制替代RNN，核心组件是Query、Key、Value的计算，支持并行、高效处理长距离依赖，是现代LLM的主流架构基础。

---

*相关词条：Attention Mechanism、Neural Network、LLM*
