---
id: Fnm教程
aliases: [fnm, Fast Node Manager]
tags: [工具, Node.js, 版本管理]
created: 2026-04-17
updated: 2026-04-17
---

# Fnm 完全指南

> Fnm (Fast Node Manager) 是一个基于 Rust 编写的 Node.js 版本管理器，比 nvm 快 5-10 倍，支持 `.node-version` 和 `.nvmrc` 自动切换。
> 版本：fnm 1.39.0 | macOS Apple Silicon

---

## 目录

## 1. 概述
## 2. 安装
## 3. 核心命令
## 4. 版本管理
## 5. 环境配置
## 6. 与 yabai / skhd 集成
## 7. 常见问题 Q&A
## 8. 命令速查表

---

## 1. 概述

### 核心特性

| 特性 | 说明 |
|------|------|
| **极速** | Rust 编写，比 nvm 快 5-10 倍 |
| **多版本** | 同时安装和管理多个 Node.js 版本 |
| **自动切换** | 根据 `.node-version` / `.nvmrc` 自动切换 |
| **跨平台** | macOS / Linux / Windows (WSL) |
| **Shell 兼容** | Zsh / Bash / Fish / PowerShell |
| **Corepack** | 支持内置 Corepack 自动启用 |

### 工作原理

```
~/.fnm/
├── node-versions/       # 各版本 Node.js
│   ├── 18.20.0/
│   ├── 20.14.0/
│   └── 25.9.0/
├── aliases/             # 版本别名（如 lts/*, latest）
└── multinode.toml       # 多版本配置
```

### 你的环境

- fnm：1.39.0
- 安装路径：`/opt/homebrew/bin/fnm`
- 当前 Node.js：v25.9.0

---

## 2. 安装

### Homebrew（推荐）

```bash
brew install fnm
```

### 手动安装

```bash
# macOS / Linux
curl -fsSL https://fnm.vercel.app/install | bash

# 或通过 npm（不推荐）
npm install -g fnm
```

### Shell 配置

**Zsh（推荐）：**

```bash
# 在 ~/.zshrc 添加
fnm env --use-on-cd > ~/.zshrc_fnm
# 或手动添加
eval "$(fnm env --use-on-cd)"
```

**重启生效：**

```bash
source ~/.zshrc
```

---

## 3. 核心命令

### fnm list-remote

列出所有可安装的 Node.js 版本。

```bash
fnm list-remote
fnm list-remote --node-dist-mirror https://nodejs.org/dist  # 使用官方源
```

### fnm list

列出本地已安装的 Node.js 版本。

```bash
fnm list
fnm list --all  # 包含别名
```

输出示例：

```
v18.20.0
v20.14.0
v25.9.0
* v25.9.0 (default)
```

### fnm install

安装指定版本的 Node.js。

```bash
fnm install 22           # 安装最新版 22.x
fnm install 20.14.0      # 安装精确版本
fnm install --lts        # 安装最新版 LTS
fnm install --latest     # 安装最新版
fnm install 18 --corepack-enabled  # 启用 Corepack
```

### fnm use

切换当前 Shell 的 Node.js 版本。

```bash
fnm use 20              # 使用 20.x 最新版
fnm use 20.14.0         # 使用精确版本
fnm use --local        # 读取 .node-version 文件切换
fnm use default         # 切换到默认版本
```

### fnm current

显示当前使用的 Node.js 版本。

```bash
fnm current
# 输出: v25.9.0
```

### fnm default

设置默认版本（全局生效）。

```bash
fnm default 20.14.0     # 设置默认版本
fnm default lts/*      # 设置默认版本为 LTS
```

### fnm exec

在指定版本下执行命令（不切换全局版本）。

```bash
fnm exec --using=20 node --version    # 用 Node 20 执行
fnm exec --using=18 npm test          # 用 Node 18 运行测试
```

### fnm alias / unalias

为版本设置别名。

```bash
fnm alias 20.14.0 work     # 设置别名
fnm alias 18.20.0 legacy   # 设置别名
fnm unalias work           # 移除别名
```

### fnm uninstall

卸载指定版本。

```bash
fnm uninstall 18.20.0     # 卸载指定版本
fnm uninstall --unused    # 卸载未使用的版本
```

### fnm completions

生成 Shell 自动补全脚本。

```bash
fnm completions --shell zsh > ~/.zfunc/_fnm   # Zsh
fnm completions --shell bash > /etc/bash_completion.d/fnm  # Bash
```

### fnm env

打印 fnm 运行所需的环境变量。

```bash
fnm env                      # 打印配置
fnm env --use-on-cd         # 包含自动切换逻辑
```

---

## 4. 版本管理

### 版本命名规则

| 格式 | 示例 | 说明 |
|------|------|------|
| 精确版本 | `20.14.0` | 指定具体版本 |
| 主版本 | `20` | 20.x 最新版 |
| 次版本 | `20.14` | 20.14.x 最新版 |
| LTS 别名 | `lts/*` | 最新 LTS 版本 |
| 最新版 | `latest` | 最新稳定版 |

### .node-version 文件

在项目根目录创建 `.node-version` 文件，进入目录时自动切换：

```bash
echo "20.14.0" > .node-version
cd  # 自动切换到 20.14.0
```

### .nvmrc 文件（兼容 nvm）

```bash
echo "20" > .nvmrc
cd  # 自动切换到 20.x 最新版
```

### fnm 配置文件

`~/.config/fnm/config.toml`：

```toml
node-dist-mirror = "https://nodejs.org/dist"
fnm-dir = "/Users/matteo/.fnm"
log-level = "info"
arch = "arm64"
version-file-strategy = "local"   # local 或 recursive
corepack-enabled = true
resolve-engines = true
```

---

## 5. 环境配置

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `FNM_DIR` | `~/.fnm` | fnm 根目录 |
| `FNM_NODE_DIST_MIRROR` | nodejs.org/dist | Node.js 镜像 |
| `FNM_LOGLEVEL` | info | 日志级别 |
| `FNM_ARCH` | arm64 | 架构 |
| `FNM_VERSION_FILE_STRATEGY` | local | 版本文件策略 |
| `FNM_COREPACK_ENABLED` | false | 启用 Corepack |

### 代理配置

```bash
export FNM_NODE_DIST_MIRROR=https://npmmirror.com/mirrors/node
```

### 与 pnpm / corepack 配合

```bash
# 全局启用 Corepack
corepack enable

# 某个版本启用
fnm use 20
corepack enable
corepack prepare pnpm@latest --activate
```

---

## 6. 与 yabai / skhd 集成

Fnm 主要负责 Node.js 版本管理

但 fnm 与 [Node.js教程](Node.js教程.md) / [pnpm教程)(pnpm教程.md) 配合使用：

- `fnm use 20` → 自动切换 Node 版本
- `pnpm install` → 使用对应版本的包管理器
- `yabai -m query --windows` → 查看窗口信息

相关教程：[Node.js教程)(Node.js教程.md)(pnpm教程.md) | 

---

## 7. 常见问题 Q&A

**Q: 进入目录后版本没自动切换？**

```bash
# 确保 shell 配置了 --use-on-cd
eval "$(fnm env --use-on-cd)"

# 检查 .node-version 或 .nvmrc 文件是否存在
cat .node-version
```

**Q: 如何查看所有可用版本？**

```bash
fnm list-remote | grep "v20"
```

**Q: 如何在不同项目使用不同版本？**

```bash
# 项目 A
echo "20.14.0" > project-a/.node-version

# 项目 B
echo "18.20.0" > project-b/.node-version

# 进入时自动切换
cd project-a  # 自动用 20.14.0
cd project-b  # 自动用 18.20.0
```

**Q: 如何加速安装？**

```bash
# 使用国内镜像
export FNM_NODE_DIST_MIRROR=https://npmmirror.com/mirrors/node
fnm install 20
```

**Q: 如何完全卸载？**

```bash
# 卸载 fnm
brew uninstall fnm

# 删除配置和数据
rm -rf ~/.fnm
rm ~/.zshrc_fnm
```

---

## 8. 命令速查表

| 场景 | 命令 |
|------|------|
| 安装 Node 20 | `fnm install 20` |
| 安装 LTS | `fnm install --lts` |
| 切换版本 | `fnm use 20` |
| 列出已安装 | `fnm list` |
| 列出可安装 | `fnm list-remote` |
| 设置默认 | `fnm default 20` |
| 查看当前 | `fnm current` |
| 执行命令 | `fnm exec --using=20 node --version` |
| 卸载版本 | `fnm uninstall 18` |
| 生成补全 | `fnm completions --shell zsh` |
| 查看帮助 | `fnm <subcommand> --help` |
