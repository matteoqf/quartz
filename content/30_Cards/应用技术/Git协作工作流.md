---
type: procedure
tags:
  - Git
  - 协作
  - 版本控制
created: 2026-04-21
modified: 2026-04-21
---

# Git协作工作流

## 概述

本文档描述团队日常开发中基于Git的标准协作流程，适用于功能分支工作流（Feature Branch Workflow），是GitHub Flow / GitFlow的简化实践。

---

## 1. 分支模型

```
main (生产分支)
  └── develop (开发分支，可选)
        ├── feature/xxx
        ├── feature/yyy
        └── fix/zzz
```

- `main`：始终保持可发布状态，受保护（protected），合并需Pull Request + 审查
- `feature/`：从 `main` 或 `develop` 创建的短期功能分支
- `fix/`：从 `main` 创建的紧急修复分支
- `release/`：发布准备分支（可选）

---

## 2. 协作流程详解

### 2.1 开发准备

```bash
# 克隆仓库（首次）
git clone <repository-url>

# 同步最新代码
git checkout main
git pull origin main

# 创建功能分支
git checkout -b feature/用户登录功能
```

### 2.2 日常开发

```bash
# 查看当前状态
git status

# 添加修改到暂存区
git add <文件名>          # 单个文件
git add .                # 当前目录所有修改

# 提交（提交信息遵循 conventional commits）
git commit -m "feat: 添加用户登录功能"

# 推送到远程
git push -u origin feature/用户登录功能
```

**提交信息规范（Conventional Commits）**：

| 类型     | 说明               |
|----------|--------------------|
| `feat`   | 新功能             |
| `fix`    | 修复               |
| `docs`   | 文档               |
| `style`  | 格式（不影响代码） |
| `refactor` | 重构             |
| `test`   | 测试               |
| `chore`  | 构建/工具          |

格式：`type: 简短描述（50字以内）`

### 2.3 同步最新代码（定期执行）

```bash
# 切回主分支
git checkout main
git pull origin main

# 切换回功能分支，将main的更新合并进来
git checkout feature/用户登录功能
git merge main

# 如有冲突，解决冲突后：
git add .
git commit -m "merge: 合并main最新代码"
```

### 2.4 代码审查与合并

1. **发起 Pull Request（PR）**

   在GitHub/GitLab等平台操作：
   - Push 完成后，平台会提示创建 PR
   - 选择目标分支（通常为 `main`）
   - 填写 PR 描述：做了什么、为什么做、如何测试
   - 指定至少一名 Reviewer

2. **审查者操作**

   ```bash
   # 拉取他人分支进行审查
   git fetch origin
   git checkout -b pr/123 origin/feature/用户登录功能

   # 审查代码，必要时在本地测试
   # 提出修改意见或 approve
   ```

3. **合并 PR**

   审查通过后：
   - Squash and merge（推荐）：将多个提交压缩为一个
   - Create a merge commit：保留所有提交历史
   - Rebase and merge：保持线性历史

### 2.5 合并后的收尾

```bash
# 删除已合并的远程分支
git push origin --delete feature/用户登录功能

# 删除本地分支
git branch -d feature/用户登录功能

# 切回主分支继续开发
git checkout main
git pull origin main
```

---

## 3. 常用命令速查

| 场景               | 命令                                         |
|--------------------|----------------------------------------------|
| 创建分支           | `git checkout -b <branch>`                    |
| 切换分支           | `git checkout <branch>`                      |
| 查看所有分支       | `git branch -a`                              |
| 查看远程分支       | `git branch -r`                              |
| 删除本地分支       | `git branch -d <branch>`                     |
| 删除远程分支       | `git push origin --delete <branch>`          |
| 查看提交历史       | `git log --oneline --graph --all`             |
| 查看未提交的修改   | `git diff`                                   |
| 查看暂存区修改     | `git diff --cached`                          |
| 撤销暂存           | `git reset HEAD <file>`                      |
| 撤销工作区修改     | `git checkout -- <file>`                     |
| 查看远程仓库地址   | `git remote -v`                              |
| 追加远程仓库       | `git remote add <name> <url>`                |
| 储藏工作进度       | `git stash`                                  |
| 恢复储藏           | `git stash pop`                              |

---

## 4. 常见冲突解决

### 4.1 合并时发生冲突

```bash
git merge main
# 输出：Auto-merging xxx.md
#       CONFLICT (content): Merge conflict in xxx.md

# 查看冲突文件
git status
```

### 4.2 解决冲突

打开冲突文件，搜索 `<<<<<<<` / `=======` / `>>>>>>>` 标记，手动合并：

```
<<<<<<< HEAD
当前分支的内容
=======
被合并分支的内容
>>>>>>> main
```

保留正确版本，删除标记，保存文件。

### 4.3 标记冲突已解决

```bash
git add <解决冲突的文件>
git commit -m "merge: 解决与main的冲突"
```

---

## 5. 最佳实践

1. **高频小提交**：每完成一个独立可测试的改动就提交，便于追溯和回滚
2. **先拉取再开发**：每天开始工作前先 `git pull origin main`
3. **功能分支要勤合并 main**：避免积累大量冲突
4. **PR 描述要清晰**：让 reviewer 一眼看懂改动目的
5. **不要直接 push main**：始终通过 PR 合并
6. **使用 `.gitignore`**：排除 `node_modules/`、`*.pyc`、`.env` 等不需要版本控制的文件
7. **保护敏感信息**：即使误提交，也使用 `git filter-branch` 或 BFG Repo-Cleaner 清理

---

## 6. 紧急修复流程

```
main ----●----●----●----●  (hotfix分支从这切出)
              ↑
         git checkout -b fix/紧急bug
         修复 → 测试 → PR → 合并到main → 删除分支
```

```bash
git checkout main
git pull origin main
git checkout -b fix/修复登录崩溃

# 修复代码，提交
git commit -m "fix: 修复登录页崩溃问题"

# 立即发起PR，通知团队快速审查
git push -u origin fix/修复登录崩溃
```

---

## 7. 关联

- [[Git基础操作]]：Git基本命令参考
- [[Git团队协作规范]]：团队代码规范与约定
- [[CI_CD工作流]]：持续集成与部署

---

*最后更新：2026-04-21*
