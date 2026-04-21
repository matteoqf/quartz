---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [RAG, 检索增强]
status: 精修
updated: 2026-04-21
---

# 💡 RAG检索增强生成

> **本质**：通过检索外部知识库获取相关文档，将检索结果注入LLM上下文，解决模型知识过时和幻觉问题。

---

## 🔬 发现历程
2020年Lewis等人提出RAG，将BERT式检索器与BART生成器结合。ChatGPT后RAG成为企业知识库的主流方案，2023年有大量RAG框架涌现（LangChain、LlamaIndex等）。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| Embedding模型 | text-embedding-ada-002, BGE, E5等 |
| 向量数据库 | Pinecone/Milvus/Chroma，核心是近似最近邻搜索(ANN) |
| Chunk大小 | 典型256-1024 token，需平衡语义完整性和召回率 |
| Top-K | 检索返回相关文档数，通常3-10个 |

## ⚠️ 误区排雷
- ❌ 误区一：RAG能完全解决幻觉 → 正确的是：RAG只是降低幻觉概率，生成器仍可能误解检索结果
- ❌ 误区二：Embedding模型不重要 → 正确的是：检索质量决定上限，差的Embedding导致再好生成器也无用

## 🎯 行动要点
- 混合检索：向量检索+关键词检索（BM25）结合，兼顾语义和精确匹配
- 重排序：检索后用Cross-Encoder重新排序，提升Top-K质量
- 迭代RAG：多轮检索-生成，逐步深化对复杂问题的回答

## 📐 关联
- [[LLM_大语言模型.md]] — RAG增强的生成对象
- [[Context_Window_上下文窗口.md]] — 检索结果注入上下文的窗口限制
- [[Inference.md]] — RAG系统的推理优化
