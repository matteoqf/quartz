---
category: 06应用
priority: ⭐⭐⭐⭐
tags: [AI Agent, 自动化]
status: 精修
updated: 2026-04-21
---

# 💡 Agent智能体

> **本质**：能够自主感知环境、规划路径、执行行动的AI系统，在复杂任务中无需人类持续干预。

---

## 🔬 发现历程
2023年GPT-4发布后Agent概念爆发。OpenAI的ChatGPT Plugins、Anthropic的Tool Use、BabyAGI、AutoGPT等开源项目涌现。2024年Operator、Claude Computer Use等计算机控制Agent出现。

## 📊 关键组件
| 组件 | 功能 |
|------|------|
| 感知 | 环境信息输入（视觉、文本、API数据）|
| 规划 | 任务分解、优先级排序、自我反思 |
| 行动 | 调用工具、操作界面、生成输出 |
| 记忆 | 短期上下文+长期经验积累 |

## ⚠️ 误区排雷
- ❌ Agent=完全自主 → 正确的是：当前Agent仍需人类设定目标、约束和边界
- ❌ Agent已能替代所有工作 → 正确的是：复杂推理、长程规划仍需人类监督

## 🎯 行动要点
- 红队测试：在受控环境中测试Agent行为边界
- 权限控制：Agent的API权限应遵循最小权限原则
- 失败恢复：设计Agent的异常处理和人工接管机制

## 📐 关联
- [[LLM_大语言模型.md]] — Agent的大脑
- [[RAG_检索增强生成.md]] — Agent的知识检索能力
- [[AIGC_生成式AI.md]] — Agent可以调用AIGC工具
