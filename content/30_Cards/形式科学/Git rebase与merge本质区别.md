---
id: git-rebase-vs-merge
title: Git Rebase与Merge本质区别
type: definition
tags:
  - Git
  - 版本控制
  - 形式科学
created: 2026-04-21
---

# Git Rebase与Merge本质区别

## 核心定义

**Merge（合并）**：将两条或多条分支的历史**合并**为一个新的提交，保留完整的分支历史轨迹，所有提交节点保持原样。

**Rebase（变基）**：将一组提交**重新应用**到另一个基准点之上，本质上是"复制"提交并改变其父节点，从而重写提交历史。

---

## 关键区别

| 维度 | Merge | Rebase |
|------|-------|--------|
| **历史表现** | 分叉-汇聚结构，产生 merge commit | 线性历史，将提交"嫁接"到目标分支末端 |
| **是否改写历史** | 不改写已有历史 | 改写提交Hash和历史 |
| **提交结构** | 保留所有原始提交节点 | 可能改变提交顺序或消除中间提交 |
| **适用场景** | 合并特性分支、公共分支协作 | 整理本地提交、保持线性历史 |

---

## 本质机制

### Merge
- 找到两个分支的**最近公共祖先（LCA）**
- 将目标分支自LCA以来的所有变更作为补丁应用
- 生成一个**新的 merge commit**，拥有两个父节点

### Rebase
- 取出当前分支自分割点以来的**每个提交**
- 逐一重新应用到目标分支的末端
- 每一次replay都会产生**新的提交对象**（Hash改变）
- 最终结果是一条线性的提交链

---

## 典型工作流

```
A---B---C  (feature)
     \
      D---E  (main)

# Merge
git checkout main
git merge feature
结果: A---B---C---M  (M是merge commit)
           \      /
            D---E

# Rebase
git checkout feature
git rebase main
结果: A---B---D---E---C'
                  (C'是C的重新应用)
```

---

## 风险提示

- **Rebase不要用于公共/已推送分支**——会改写历史，导致协作者的历史错乱
- Merge适合保留完整的分支演化轨迹
- Rebase适合个人分支整理，追求干净的线性历史
