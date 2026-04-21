---
id: yabai教程
aliases: [yabai, yabai窗口管理器]
tags: [工具, 极客, 窗口管理, 平铺窗口管理器, macOS]
created: 2026-04-20
updated: 2026-04-20
---

# yabai 完全指南

> macOS 平铺式窗口管理器 — 用键盘高效管理窗口布局
> 版本：v7.1.18

---

## 目录

[[#1-工具概述]]
[[#2-安装与初始化]]
[[#3-配置文件详解]]
[[#4-配置项参考]]
[[#5-窗口操作命令]]
[[#6-信号系统]]
[[#7-skhd-快捷键配置]]
[[#8-应用规则]]
[[#9-查询命令]]
[[#10-常见问题]]

---

## 1. 工具概述

yabai 是 macOS 上的平铺式窗口管理器，通过二叉空间分割（BSP）算法自动排列窗口，支持：

- **BSP 布局**：窗口自动分割为树状结构，无需手动调整
- **多显示器支持**：每个显示器独立管理 spaces
- **浮动窗口**：指定应用（视频播放器、设置面板）保持浮动
- **鼠标交互**：配合 skhd 实现键盘为主、鼠标为辅的操作
- **信号系统**：窗口/空间变化时触发自定义脚本

当前状态：1 个显示器，主显示器 2560x1440，10 个 Space，4 个活跃窗口。

---

## 2. 安装与初始化

### 2.1 安装

```bash
brew install koekeishiya/formulae/yabai
brew install koekeishiya/formulae/skhd
```

### 2.2 SIP 处理

macOS 13.4+ 推荐使用无 SIP 注入模式；旧版或 Intel Mac 需关闭 SIP：

```bash
# 重启按 Cmd+R 进入恢复模式
csrutil disable
```

### 2.3 签名加载与自启

```bash
# 每次启动加载签名模块
sudo yabai --load-sa

# brew services 自启
brew services start yabai
brew services start skhd
```

### 2.4 基础验证

```bash
yabai --version              # 查看版本
yabai -m query --spaces      # 查看所有 space
yabai -m query --windows     # 查看所有窗口
yabai -m query --displays    # 查看显示器
```

---

## 3. 配置文件详解

### 3.1 ~/.yabairc

位置：`~/.yabairc`（sh 脚本，yabai 启动时执行）

```sh
#!/usr/bin/env sh
sudo yabai --load-sa

# --- 布局模式 ---
yabai -m config layout bsp                      # BSP 二叉空间分割
yabai -m config window_placement second_child    # 新窗口在当前窗口右侧/下方

# --- 焦点跟随鼠标 ---
yabai -m config focus_follows_mouse autofocus     # 鼠标移入窗口时自动聚焦

# --- 间距（像素）---
yabai -m config top_padding    5
yabai -m config bottom_padding 5
yabai -m config left_padding   5
yabai -m config right_padding  5
yabai -m config window_gap     5

# --- 鼠标操作 ---
yabai -m config mouse_modifier alt   # 按住 alt 启用鼠标窗口操作
yabai -m config mouse_action1 move   # alt + 左键拖拽移动窗口
yabai -m config mouse_action2 resize # alt + 右键拖拽缩放窗口

# --- 应用规则（浮动窗口）---
yabai -m rule --add app="^系统设置$" manage=off
yabai -m rule --add app="^计算器$" manage=off
yabai -m rule --add app="^访达$" manage=off
yabai -m rule --add app="^活动监视器$" manage=off
yabai -m rule --add app="^mpv$" manage=off layer=above floating=on
yabai -m rule --add app="^IINA$" manage=off layer=above floating=on
```

### 3.2 ~/.skhdrc

位置：`~/.skhdrc`（skhd 启动时读取）

常用快捷键速览：

| 快捷键              | 功能                         |
| ------------------- | ---------------------------- |
| `alt - return`      | 打开 Ghostty 终端            |
| `alt - b`           | 打开 Chrome                  |
| `alt - s`           | 打开 Spotify                 |
| `alt - w`           | 打开微信                     |
| `alt - o`           | 打开 Obsidian                |
| `alt - t`           | 打开 Telegram                |
| `alt - h/j/k/l`     | 切换窗口焦点（左/下/上/右）  |
| `shift + alt - h/l` | 将窗口向左/右交换位置        |
| `shift + alt - 1~0` | 将窗口移动到桌面 1~10 并跳转 |
| `alt - f`           | 切换窗口全屏                 |
| `ctrl + alt - r`    | 重启 skhd 服务               |

完整配置见 [[skhd教程]]

---

## 4. 配置项参考

### 4.1 布局模式

```bash
yabai -m config layout bsp    # 二叉空间分割（默认）
yabai -m config layout float  # 自由浮动
```

### 4.2 新窗口放置

```bash
yabai -m config window_placement first_child   # 新窗口在左侧/上方
yabai -m config window_placement second_child  # 新窗口在右侧/下方（默认）
```

### 4.3 焦点跟随鼠标

```bash
yabai -m config focus_follows_mouse autofocus  # 自动跟随
yabai -m config focus_follows_mouse autoraise  # 跟随且提升窗口
yabai -m config focus_follows_mouse off       # 关闭（默认）
```

### 4.4 间距配置

```bash
yabai -m config window_gap     5    # 窗口间隙
yabai -m config top_padding    5    # 顶部外边距
yabai -m config bottom_padding 5    # 底部外边距
yabai -m config left_padding   5    # 左侧外边距
yabai -m config right_padding  5    # 右侧外边距
```

### 4.5 鼠标操作

```bash
yabai -m config mouse_modifier alt   # 可选：alt / shift / ctrl / cmd
yabai -m config mouse_action1 move   # 左键：移动
yabai -m config mouse_action2 resize # 右键：缩放
yabai -m config mouse_follows_focus on  # 聚焦窗口时鼠标跟随
```

### 4.6 其他配置

```bash
yabai -m config split_ratio 0.5        # 分割比例（0.3-0.7）
yabai -m config auto_balance on        # 自动平衡窗口大小
yabai -m config border on              # 窗口边框（需额外安装）
yabai -m config shadow on             # 阴影
yabai -m config mouse_drop_action swap # 鼠标放下窗口时的行为
```

---

## 5. 窗口操作命令

### 5.1 聚焦

```bash
yabai -m window --focus west   # 焦点向左
yabai -m window --focus south # 焦点向下
yabai -m window --focus north # 焦点向上
yabai -m window --focus east  # 焦点向右
yabai -m window --focus prev  # 上一个窗口
yabai -m window --focus next  # 下一个窗口
yabai -m window --focus first # 第一个窗口
yabai -m window --focus last  # 最后一个窗口
```

### 5.2 移动（交换位置）

```bash
yabai -m window --warp west   # 窗口向左交换位置
yabai -m window --warp east   # 向右交换
yabai -m window --warp south  # 向下交换
yabai -m window --warp north  # 向上交换
```

### 5.3 缩放

```bash
yabai -m window --toggle zoom-fullscreen  # 全屏/退出全屏
yabai -m window --toggle native-fullscreen # 原生全屏
yabai -m window --toggle parent           # 聚焦父节点
yabai -m window --toggle smaller          # 缩小
yabai -m window --toggle larger           # 放大
```

### 5.4 空间操作

```bash
yabai -m window --space 1   # 将窗口移到桌面 1
yabai -m space --focus 1   # 聚焦桌面 1
yabai -m window --space 2 && yabai -m space --focus 2  # 移动并跳转
```

### 5.5 旋转与翻转

```bash
yabai -m space --rotate 90   # 当前 space 旋转 90°
yabai -m space --rotate 180  # 旋转 180°
yabai -m space --mirror x-axis # 左右翻转
yabai -m space --mirror y-axis # 上下翻转
```

### 5.6 栈模式（窗口堆叠）

```bash
yabai -m window --stack next  # 将窗口加入当前窗口的栈
yabai -m window --focus stack.prev  # 栈中上一个
yabai -m window --focus stack.next  # 栈中下一个
```

### 5.7 关闭与最小化

```bash
yabai -m window --close   # 关闭窗口
yabai -m window --minimize # 最小化到 Dock
```

---

## 6. 信号系统

yabai 可以在窗口/space 变化时触发脚本。

### 6.1 常用信号

```bash
yabai -m signal --add event=window_focused action="echo focused"
yabai -m signal --add event=space_changed action="echo space changed"
yabai -m signal --add event=application_activated action="echo app activated"
yabai -m signal --add event=display_changed action="echo display changed"
yabai -m signal --add event=mission_control_enter action="echo mission control"
yabai -m signal --add event=window_created action="echo window created"
yabai -m signal --add event=window_destroyed action="echo window destroyed"
```

### 6.2 信号示例

```bash
# 当 space 切换时，更新状态栏
yabai -m signal --add event=space_changed action="sketchybar --trigger space"

# 当应用激活时，打印日志
yabai -m signal --add event=application_activated action="logger 'app activated'"
```

### 6.3 信号管理

```bash
yabai -m signal --list     # 列出所有信号
yabai -m signal --remove <id>  # 移除信号
```

---

## 7. skhd 快捷键配置

### 7.1 安装

```bash
brew install koekeishiya/formulae/skhd
brew services start skhd
```

### 7.2 语法

```
<MODIFIER> - <KEY> : <COMMAND>
```

- `alt` = Option 键
- `shift` = Shift 键
- `ctrl` = Control 键
- `cmd` / `super` = Command 键
- `hyper` = Ctrl+Shift+Cmd+Alt（需 Karabiner-Elements）

### 7.3 完整 ~/.skhdrc 示例

```sh
# === 打开应用 ===
alt - return : open -a "Ghostty.app"
alt - b      : open -a "Google Chrome.app"
alt - s      : open -a "Spotify.app"
alt - w      : open -a "WeChat.app"
alt - o      : open -a "Obsidian.app"
alt - t      : open -a "Telegram.app"
alt - r      : open -a "App Cleaner 9.app"
alt - z      : open -a "极空间.app"
alt - q      : open -a "QuarkCloudDrive.app"
alt - e      : open ~

# === yabai 窗口聚焦 ===
alt - h : yabai -m window --focus west
alt - j : yabai -m window --focus south
alt - k : yabai -m window --focus north
alt - l : yabai -m window --focus east

# === yabai 窗口移动 ===
shift + alt - h : yabai -m window --warp west
shift + alt - l : yabai -m window --warp east

# === 移动到桌面并跳转 ===
shift + alt - 1 : yabai -m window --space 1; yabai -m space --focus 1
shift + alt - 2 : yabai -m window --space 2; yabai -m space --focus 2
shift + alt - 3 : yabai -m window --space 3; yabai -m space --focus 3
shift + alt - 4 : yabai -m window --space 4; yabai -m space --focus 4
shift + alt - 5 : yabai -m window --space 5; yabai -m space --focus 5
shift + alt - 6 : yabai -m window --space 6; yabai -m space --focus 6
shift + alt - 7 : yabai -m window --space 7; yabai -m space --focus 7
shift + alt - 8 : yabai -m window --space 8; yabai -m space --focus 8
shift + alt - 9 : yabai -m window --space 9; yabai -m space --focus 9
shift + alt - 0 : yabai -m window --space 10; yabai -m space --focus 10

# === 全屏切换 ===
alt - f : yabai -m window --toggle zoom-fullscreen

# === 重启 skhd ===
ctrl + alt - r : brew services restart skhd
```

---

## 8. 应用规则

### 8.1 基础规则

```bash
# 浮动（不参与平铺）
yabai -m rule --add app="^计算器$" manage=off

# 置顶（总在其它窗口之上）
yabai -m rule --add app="^mpv$" manage=off layer=above floating=on

# 原生全屏应用
yabai -m rule --add app="^Safari$" native-fullscreen on
```

### 8.2 常用规则速查

| 应用       | 规则                                 | 说明             |
| ---------- | ------------------------------------ | ---------------- |
| 系统设置   | `manage=off`                         | 设置窗口保持浮动 |
| 计算器     | `manage=off`                         | 保持浮动         |
| mpv        | `manage=off layer=above floating=on` | 浮动置顶（视频） |
| IINA       | `manage=off layer=above floating=on` | 浮动置顶（视频） |
| 访达       | `manage=off`                         | 部分情况浮动     |
| 活动监视器 | `manage=off`                         | 保持浮动         |

### 8.3 管理规则

```bash
yabai -m rule --list           # 列出所有规则
yabai -m rule --add app="^App$" ...  # 添加规则
yabai -m rule --remove app="^App$"   # 移除规则
```

---

## 9. 查询命令

```bash
# 查询当前所有窗口
yabai -m query --windows

# 查询所有 space
yabai -m query --spaces

# 查询所有显示器
yabai -m query --displays

# 查询当前 space 的窗口
yabai -m query --windows --space

# 查询当前聚焦窗口
yabai -m query --windows --focused

# jq 格式化输出
yabai -m query --windows | jq '.[] | {app:.app, title:.title}'
```

---

## 10. 常见问题

### Q1: yabai 无法加载（SIP 阻止）

**A**：macOS 13.4+ 用户可使用注入模式，或关闭 SIP：

```bash
# 恢复模式执行
csrutil disable
```

### Q2: skhd 快捷键不生效

**A**：

1. 检查 skhd 是否运行：`brew services list | grep skhd`
2. 重启服务：`brew services restart skhd`
3. 查看错误日志：`cat ~/.skhd.log`

### Q3: 新窗口没有出现在期望位置

**A**：检查 `window_placement` 配置，默认为 `second_child`（右侧/下方）。

### Q4: 鼠标操作无效

**A**：确保 `mouse_modifier` 设置正确：

```bash
yabai -m config mouse_modifier alt
```

### Q5: 如何让某个应用始终保持浮动？

**A**：在 `~/.yabairc` 添加规则：

```bash
yabai -m rule --add app="^AppName$" manage=off
```

### Q6: 如何动态调整窗口间隙？

**A**：通过信号触发动态调整：

```bash
yabai -m signal --add event=space_changed \
    action="yabai -m config window_gap 10"
```

---

## 附录：命令速查表

| 操作         | 命令                                          |
| ------------ | --------------------------------------------- |
| 聚焦左窗口   | `yabai -m window --focus west`                |
| 聚焦右窗口   | `yabai -m window --focus east`                |
| 移动窗口向左 | `yabai -m window --warp west`                 |
| 全屏切换     | `yabai -m window --toggle zoom-fullscreen`    |
| 关闭窗口     | `yabai -m window --close`                     |
| 移动到桌面 N | `yabai -m window --space N`                   |
| 查看所有窗口 | `yabai -m query --windows`                    |
| 查看所有桌面 | `yabai -m query --spaces`                     |
| 添加浮动规则 | `yabai -m rule --add app="^Name$" manage=off` |
| 重启 yabai   | `brew services restart yabai`                 |

---

## 相关
