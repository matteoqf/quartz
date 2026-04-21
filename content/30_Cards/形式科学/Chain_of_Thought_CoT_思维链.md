---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [推理, 思维链]
status: 精修
updated: 2026-04-21
---

# 💡 Chain of Thought思维链

> **本质**：通过在回答前显式展示推理步骤，将复杂问题分解为可执行的中间步骤，提升LLM的推理能力。

---

## 🔬 发现历程
2022年Wei等人首次提出CoT，在GSM8K数学题上验证有效。同年Self-Consistency通过采样多数投票进一步提升。2023年ToT（Tree of Thoughts）扩展到树状推理。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| Few-shot CoT | 在prompt中提供推理示例 |
| Zero-shot CoT | 加"让我们一步一步思考"触发 |
| Self-Consistency | 多次采样+多数投票，计算成本高 |
| 适用场景 | 数学、逻辑、代码等结构化推理任务 |

## ⚠️ 误区排雷
- ❌ 误区一：CoT适合所有任务 → 正确的是：创意写作、情感分析等任务CoT反而可能降低效果
- ❌ 误区二：CoT推理一定正确 → 正确的是：模型可能产生"看似合理但错误的推理链"，需验证中间步骤

## 🎯 行动要点
- 复杂推理任务先尝试Few-shot CoT：在prompt中展示2-3个完整推理示例
- 验证链：让模型在推理后自我检查"I need to verify this step"
- 代码先行：对于数学问题，让模型先生成Python代码执行，再基于结果推理

## 📐 关联
- [[Transformer.md]] — Transformer是CoT能力的基础
- [[涌现.md]] — 复杂推理是涌现能力之一
- [[LLM_大语言模型.md]] — CoT在LLM上最有效
