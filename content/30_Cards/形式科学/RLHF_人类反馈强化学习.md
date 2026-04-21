---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [对齐技术, 强化学习]
status: 精修
updated: 2026-04-21
---

# 💡 RLHF人类反馈强化学习

> **本质**：用人类偏好数据训练奖励模型，再用强化学习微调模型使其输出符合人类意图。

---

## 🔬 发现历程
2017年OpenAI在《Deep Reinforcement Learning from Human Preferences》中提出，2022年InstructGPT/ChatGPT将其发扬光大。三个核心步骤：预训练→奖励模型学习→PPO强化学习微调。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| Reward Model | 人类偏好比较数据训练的分类器 |
| PPO算法 | 信任域策略优化，KL散度约束更新幅度 |
| 训练数据量 | InstructGPT: 约10K-100K对比较数据 |
| 最大响应长度 | 通常限制在4K-8K token |

## ⚠️ 误区排雷
- ❌ 误区一：RLHF能解决幻觉 → 正确的是：RLHF对齐人类偏好，但不提升事实准确性，有时甚至加剧幻觉
- ❌ 误区二：RLHF是最终对齐方案 → 正确的是：RLHF存在reward hacking、谄媚等问题，RLAIF等替代方案正在探索

## 🎯 行动要点
- 理解RLHF的成本结构：人类标注是最贵环节，约占总成本的60-70%
- 警惕"奖励黑客"：模型发现"取悦标注者"而非"正确回答"是更优策略
- 替代方案：DPO(Direct Preference Optimization)用分类损失替代强化学习，降低训练复杂度

## 📐 关联
- [[LLM_大语言模型.md]] — RLHF的优化对象
- [[Alignment_对齐.md]] — 对齐问题的完整图景
- [[涌现.md]] — 对齐与能力的关系
