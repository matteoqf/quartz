---
category: 01形式
priority: ⭐⭐⭐⭐
tags: [硬件, 算力]
status: 精修
updated: 2026-04-21
---

# 💡 GPU

> **本质**：拥有数千个小计算核心的并行处理器，擅长同时执行数百万次矩阵乘法，是深度学习训练的基础硬件。

---

## 🔬 发现历程
1999年NVIDIA推出GeForce 256时提出"GPU"概念，最初用于游戏图形渲染。2006年CUDA架构诞生，2012年AlexNet在GPU上训练引发深度学习革命，GPU成为AI基础设施。

## 📊 关键参数
| 参数 | 数值/标准 |
|------|-----------|
| NVIDIA A100 | 6912 CUDA核，80GB HBM2e，带宽2TB/s |
| H100 SXM | 16896 CUDA核，80GB HBM3，带宽3.35TB/s |
| RTX 4090 | 16384 CUDA核，24GB GDDR6X（消费级） |
| TFLOPS | A100 float16: 312 TFLOPS |

## ⚠️ 误区排雷
- ❌ 误区一：GPU显存=可用内存 → 正确的是：模型+优化器状态+梯度+激活值都需要显存，实际可加载模型远小于标称显存
- ❌ 误区二：买最好的GPU就能训练大模型 → 正确的是：单卡训练有天花板，大模型必须多卡并行（数据并行/张量并行/流水线并行）

## 🎯 行动要点
- 训练推荐H100/A100，推理可用H100/T4/L40S（成本效益不同）
- 显存带宽是关键瓶颈：HBM3 vs GDDR6X带宽差距达10倍
- 使用梯度累积突破单卡显存限制：大batch = micro_batch × gradient_accumulation_steps

## 📐 关联
- [[Compute_算力.md]] — GPU是算力的载体
- [[Transformer.md]] — Transformer在GPU上的计算模式
- [[量化.md]] — 通过降低精度节省显存
