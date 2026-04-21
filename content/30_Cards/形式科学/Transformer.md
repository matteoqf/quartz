---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [AI架构, 深度学习]
status: 精修
updated: 2026-04-21
---

# 💡 Transformer

> **本质**：用自注意力机制替代RNN，实现全局并行计算的长距离依赖建模。

---

## 🔬 发现历程
2017年Google Brain团队在论文《Attention is All You Need》中首次提出。Vaswani等研究者发现，在机器翻译任务上，完全使用注意力机制的模型远超RNN+注意力的混合方案。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| 核心运算 | Q·K^T / √dk → Softmax → V |
| Multi-Head | 典型8-16头，每头独立学习不同语义空间 |
| 位置编码 | Sinusoidal或Rotary Embedding |
| 上下文窗口 | 典型4K-128K token |

## ⚠️ 误区排雷
- ❌ 误区一：Transformer是RNN的改进 → 正确的是：Transformer完全抛弃了RNN结构，并行性是本质差异
- ❌ 误区二：注意力机制是Transformer的核心 → 正确的是：残差连接+层归一化+前馈网络同样关键

## 🎯 行动要点
- 理解Q/K/V投影机制：Query决定查询意图，Key定义被检索特征，Value承载实际信息
- 掌握FFN的门控机制：1/3参数量的MLP是知识存储单元
- 注意因果掩码：语言模型需要防止未来信息泄露

## 📐 关联
- [[Attention_Mechanism.md]] — 自注意力的数学形式
- [[LLM_大语言模型.md]] — Transformer是LLM的架构基础
- [[Chain_of_Thought_CoT_思维链.md]] — Transformer对链式推理的支持
