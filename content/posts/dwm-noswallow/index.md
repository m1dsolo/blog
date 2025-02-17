---
title: 'dwm 中防止特定窗口被 swallow 的方法'
date: '2025-02-17'
tags: ['dwm', 'X11', '窗口管理']
draft: false
description: '本文介绍如何在 dwm 窗口管理器的 swallow 补丁基础上，实现选择性防止特定窗口被嵌入终端的功能。通过简单的文件通信机制，使用 noswallow 脚本即可控制窗口的 swallow 行为。'
keywords: [
  "dwm",
  "swallow",
  "窗口管理",
  "X11",
  "suckless",
  "平铺式窗口管理器",
]
---

## 前言

dwm 是 X11 下的平铺式窗口管理器，用 C 语言编写，全部代码不到 2k 行，非常轻量级。它提供基本的窗口管理功能，额外的功能可以通过修改源码或打补丁实现。

[suckless 官网](https://dwm.suckless.org/patches/) 提供了大量补丁，其中 [swallow](https://dwm.suckless.org/patches/swallow/) 是一个常用的补丁。

**swallow 的作用：** 启动新应用程序时，将其嵌入到已有的终端窗口中，而不是创建新窗口。这样可以减少窗口数量，保持桌面整洁。

![swallow 效果](https://m1dsolo.xyz/images/dwm-noswallow/swallow.gif)

*swallow 补丁演示（图片来源：Luke Smith）*

**本文解决的问题：** 有时我们**不希望**某些窗口被 swallow（如调试游戏时需要看到终端输出），本文介绍如何在 swallow 基础上实现选择性禁用。

---

## 1. 需求分析

swallow 固然好用，但有些场景下我们**不希望**窗口被嵌入：

- **开发 debug 时**：需要看到终端输出的 debug 信息
- **运行长时间任务时**：需要独立窗口查看进度

我尝试过 [dynamicswallow patch](https://dwm.suckless.org/patches/dynamicswallow/)，但功能过于重量级。于是我决定在 swallow 补丁基础上修改，实现**简单的选择性禁用**。

**实现效果：** 通过 `noswallow` 脚本启动应用，该窗口就不会被 swallow。

![noswallow 效果](https://m1dsolo.xyz/images/dwm-noswallow/noswallow.gif)

---

## 2. 实现原理

核心思路：**文件通信**。

dwm 和终端之间需要通信，最简单的方式是检查文件是否存在。

### 2.1 修改 dwm.c

在 `dwm.c` 的 `swallow` 函数开始处添加检查：

```c
void
swallow(Client *p, Client *c)
{
    // 原有检查
    if (c->noswallow || c->isterminal)
        return;
    if (c->noswallow && !swallowfloating && c->isfloating)
        return;
    
    // 新增：检查 /tmp/noswallow 文件
    if (!access("/tmp/noswallow", F_OK))
        return;
    
    // ... 原有 swallow 逻辑
}
```

**原理：**
- `access("/tmp/noswallow", F_OK)` 检查文件是否存在
- 存在则返回 0，跳过 swallow
- 不存在则继续 swallow 逻辑

**重新编译：**
```bash
cd dwm
sudo make clean install
```

### 2.2 创建 noswallow 脚本

```bash
#!/bin/sh
# /usr/local/bin/noswallow

# 1. 创建标记文件
touch /tmp/noswallow

# 2. 执行命令（带参数）
"$@"

# 3. 删除标记文件
rm /tmp/noswallow
```

**赋予执行权限：**
```bash
sudo chmod +x /usr/local/bin/noswallow
```

**使用方法：**
```bash
# 正常启动（会被 swallow）
game

# 防止 swallow
noswallow game
```

---

## 3. 配置步骤

### 3.1 应用补丁

1. 下载 swallow 补丁：
```bash
cd ~/dwm
wget https://dwm.suckless.org/patches/swallow/dwm-swallow-6.3.diff
patch < dwm-swallow-6.3.diff
```

2. 修改 `dwm.c`，添加 noswallow 检查（见上文）

3. 重新编译：
```bash
sudo make clean install
```

### 3.2 安装 noswallow 脚本

```bash
sudo tee /usr/local/bin/noswallow >/dev/null <<'EOF'
#!/bin/sh
touch /tmp/noswallow
"$@"
rm /tmp/noswallow
EOF

sudo chmod +x /usr/local/bin/noswallow
```

### 3.3 验证配置

```bash
# 1. 重启 dwm（Mod+Shift+q）
# 2. 测试正常启动（应被 swallow）
st

# 3. 测试 noswallow 启动（应独立窗口）
noswallow st
```

---

## 4. 常见问题

### 4.1 编译失败

**错误：** `undefined reference to 'access'`

**解决：** 在 `dwm.c` 顶部添加：
```c
#include <unistd.h>
```

### 4.2 noswallow 不工作

**检查清单：**
- [ ] `/tmp/noswallow` 文件权限正确
- [ ] 脚本有执行权限（`chmod +x`）
- [ ] dwm 已重新编译并重启
- [ ] swallow 补丁已正确应用

### 4.3 文件未及时删除

如果 `noswallow` 脚本被中断（如 Ctrl+C），`/tmp/noswallow` 可能残留。

**解决：** 手动删除：
```bash
rm /tmp/noswallow
```

**改进脚本（自动清理）：**
```bash
#!/bin/sh
trap "rm /tmp/noswallow" EXIT
touch /tmp/noswallow
"$@"
```

---

## 5. 扩展用法

### 5.1 永久禁用特定程序

创建别名（`~/.bashrc` 或 `~/.zshrc`）：

```bash
alias game="noswallow game"
alias steam="noswallow steam"
```

### 5.2 临时切换模式

```bash
# 禁用 swallow 模式
export NOSWALLOW=1

# 恢复 swallow 模式
unset NOSWALLOW
```

修改 noswallow 脚本支持环境变量：
```bash
#!/bin/sh
if [ "$NOSWALLOW" = "1" ]; then
    touch /tmp/noswallow
    trap "rm /tmp/noswallow" EXIT
fi
"$@"
```

---

## 总结

**优点：**
- 实现简单（几行代码）
- 无需额外依赖
- 灵活控制（按需禁用）

**缺点：**
- 需要重新编译 dwm
- 仅适用于已安装 swallow 补丁的用户

**适用场景：**
- dwm 用户 + swallow 补丁
- 需要选择性禁用 swallow
- 追求轻量级方案

---

## 参考资源

- [dwm swallow patch](https://dwm.suckless.org/patches/swallow/)
- [dwm dynamicswallow patch](https://dwm.suckless.org/patches/dynamicswallow/)
- [我的 dwm 配置](https://github.com/m1dsolo/dotfiles)
- [Arch Wiki - dwm](https://wiki.archlinux.org/title/Dwm)
