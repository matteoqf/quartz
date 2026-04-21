---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [Token, 计量单位]
status: 精修
updated: 2026-04-21
---

# 💡 Token

> **本质**：文本处理的最小单元，介于字符和词之间，LLM以此为单位进行概率计算和费用结算。

---

## 🔬 发现历程
GPT-3前使用BPE(Byte Pair Encoding)分词，后续模型多采用SentencePiece等更先进分词器。Tokenization将文本转换为整数序列输入模型。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| 中文token效率 | 约1汉字=1.3-2.0 token |
| 英文token效率 | 约1单词=1.3-1.5 token |
| 1000 token | 约等于750个英文单词 |
| 上下文限制 | 以token计数，不是字符或词 |

## ⚠️ 误区排雷
- ❌ 误区一：Token直接等于字数 → 正确的是：中英文差异巨大，1K token中文约500-700字，英文约750词
- ❌ 误区二：Tokenization不影响模型能力 → 正确的是：分词粒度决定模型对词义的理解深度，过粗或过细都会影响表达能力

## 🎯 行动要点
- API计费按input+output token总和计算，注意两者的单价的差异
- 压缩prompt可显著降低成本：去除冗余、使用更简洁的表达
- 代码token效率最高：代码的规律性使BPE分词器效率接近100%

## 📐 关联
- [[LLM_大语言模型.md]] — Token是LLM的基本输入单位
- [[Context_Window_上下文窗口.md]] — 窗口以token计量
- [[Inference.md]] — 推理按token计费
