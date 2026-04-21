---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [注意力机制, 深度学习]
status: 精修
updated: 2026-04-21
---

# 💡 Attention Mechanism注意力机制

> **本质**：通过Query-Key-Value计算，让模型动态选择与当前任务最相关的输入信息。

---

## 🔬 发现历程
2014年Bahdanau等人在机器翻译中首次提出"加性注意力"，解决了RNN长距离依赖问题。2017年Transformer将其推广为Scaled Dot-Product Attention，成为主流。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| 计算方式 | Attention(Q,K,V) = Softmax(QK^T/√dk)V |
| 缩放因子 | √dk防止点积过大导致Softmax梯度消失 |
| Multi-Head | 16头×64维=1024维表示空间 |

## ⚠️ 误区排雷
- ❌ 误区一：注意力权重就是重要度 → 正确的是：注意力权重反映的是"预测当前位置时参考了哪些位置"，不等于语义重要性
- ❌ 误区二：注意力是透明可解释的 → 正确的是：后层注意力模式与前层高度耦合，浅层解释往往误导

## 🎯 行动要点
- 分析注意力图时，关注对角线模式（局部依赖）还是散射模式（长距离依赖）
- Attention sink现象：某些虚拟token（如BERT的[CLS]）劫持大量注意力，理解这一点对推理优化很重要
- 推理时可通过缓存K/V避免重复计算（PagedAttention、KV Cache）

## 📐 关联
- [[Transformer.md]] — 自注意力的应用
- [[Chain_of_Thought_CoT_思维链.md]] — 注意力在推理时的作用
- [[Context_Window_上下文窗口.md]] — 注意力的长度限制
