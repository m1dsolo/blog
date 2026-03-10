---
title: '局域网 WoL 远程开机配置指南（Arch Linux + 技嘉主板）'
date: '2026-03-09'
tags: ['Linux', 'WoL', '远程开机']
draft: false
description: '本文介绍如何在 Arch Linux 上配置 Wake-on-LAN（WoL）远程开机功能，涵盖技嘉 B650M 主板 BIOS 设置、网卡配置、systemd 服务创建，以及笔记本远程唤醒台式机的完整流程。'
keywords: [
  "Wake-on-LAN",
  "WoL",
  "远程开机",
  "Arch Linux",
  "技嘉主板",
  "局域网唤醒",
]
---

## 前言

Wake-on-LAN（简称 WoL）是一种通过网络信号远程唤醒计算机的技术。

**实现效果：** 笔记本和台式机在同一局域网下，台式机连接网线，笔记本可以把台式机开机。

**本文环境：**
- 主板：技嘉 B650M AORUS（BIOS 版本 F33b）
- 系统：Arch Linux
- 网卡：有线网络（WoL 需要网线连接）

---

## 1. 主板 BIOS 配置

**主板型号：** 技嘉 B650M AORUS（BIOS 版本 F33b）

**BIOS 设置：**

1. 开机按 **Del** 进入 BIOS
2. 进入 `Settings → IO Ports`
3. 配置以下选项：

| 选项 | 设置 | 说明 |
|------|------|------|
| `ErP Support` | **Disabled** | 必须关闭，否则关机后网卡断电 |
| `Network Stack` | **Enabled** | 启用网络栈 |

4. 按 **F10** 保存并退出

**注意：** `ErP` 是欧盟节能标准，开启后关机时主板会完全断电，导致 WoL 失效。

---

## 2. 获取网卡信息

进入台式机 Arch Linux，执行：

```bash
ip addr
```

输出示例：
```bash
2: enp12s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 10:ff:e0:ce:62:08 brd ff:ff:ff:ff:ff:ff
    altname enx10ffe0ce6208
    inet 192.168.1.10/24 brd 192.168.1.255 scope global dynamic noprefixroute enp12s0
```

**记录两个关键信息：**
- 网卡名：`enp12s0`
- MAC 地址：`10:ff:e0:ce:62:08`

---

## 3. 安装工具

台式机：
```bash
sudo pacman -S ethtool
```

笔记本
```bash
sudo pacman -S wakeonlan
```

- `ethtool`：查看和配置网卡参数
- `wakeonlan`：发送 WoL 魔术包

---

## 4. 配置 WoL（临时生效）

```bash
# 开启 WoL（替换为你的网卡名）
sudo ethtool -s enp12s0 wol g

# 检查状态
sudo ethtool enp12s0 | grep "Wake-on"
```

**期望输出：**
```
Wake-on: g
```

如果显示 `Wake-on: d` 表示 WoL 已禁用。

---

## 5. 配置 WoL（永久生效）

创建 systemd 服务模板：

```bash
sudo tee /etc/systemd/system/wol@.service >/dev/null <<'EOF'
[Unit]
Description=Enable Wake-on-LAN for interface %i
Requires=network.target
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/ethtool -s %i wol g

[Install]
WantedBy=multi-user.target
EOF
```

启用服务（替换为你的网卡名）：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wol@enp12s0.service
```

**验证：**
```bash
systemctl status wol@enp12s0.service
```

---

## 6. 测试 WoL

**在笔记本上执行：**

```bash
# 发送 WoL 魔术包（替换为台式机 MAC）
wakeonlan 10:ff:e0:ce:62:08
```

**输出：**
```
Sending magic packet to 255.255.255.255:9 with payload 10:ff:e0:ce:62:08
Hardware addresses: <total=1, valid=1, invalid=0>
Magic packets: <sent=1>
```

**验证台式机是否开机：**
```bash
ping -c 3 192.168.1.10
```

---

## 7. 常见问题

### 8.1 WoL 不工作

**检查清单：**
1. BIOS 中 `ErP` 是否关闭
2. 关机后网卡指示灯是否亮着
3. 网线是否连接正常
4. 是否在同一局域网内

### 8. 外网 WoL 方案

---

## 8. 外网 WoL 方案

WoL 默认只能在局域网内生效。如需外网开机，有以下方案：

### 8.1 路由器端口转发

1. 登录路由器管理页面
2. 配置端口转发规则：
   - 外部端口：`9`（UDP）
   - 内部 IP：台式机 IP（如 `192.168.1.10`）
   - 内部端口：`9`（UDP）
3. 配置 DDNS（动态域名解析）
4. 外网发送 WoL：
```bash
wakeonlan -i 你的公网 IP 10:ff:e0:ce:62:08
```

**注意：** 部分运营商禁用 UDP 9 端口，可改用其他端口（如 7777）。

### 8.2 云服务器中转

通过云服务器 SSH 隧道转发：

```bash
# 1. SSH 反向隧道（笔记本 → 云服务器）
ssh -R 7777:localhost:9 yang@云服务器 IP

# 2. 在云服务器上发送 WoL
wakeonlan -i 云服务器内网 IP 10:ff:e0:ce:62:08
```

### 8.3 路由器自带 WoL 功能

部分路由器（如 OpenWrt、梅林）自带 WoL 功能，可直接在路由器页面发送魔术包。

---

## 9. 发布前检查清单

- [ ] BIOS 中 `ErP Support` 已关闭
- [ ] 关机后网卡指示灯亮着
- [ ] 网线连接正常
- [ ] systemd 服务已启用（`systemctl status wol@enp12s0.service`）
- [ ] 笔记本和台式机在同一局域网
- [ ] MAC 地址正确

---

## 总结

**配置流程：**
1. BIOS 关 ErP、开 WoL
2. 获取网卡 MAC 地址
3. 用 ethtool 开启 WoL
4. 创建 systemd 服务永久生效
5. 用 wakeonlan 测试

**优点：**
- 无需额外硬件
- 配置简单
- 同一局域网内秒开机

**缺点：**
- 需要台式机插网线
- 外网访问需要额外配置

---

## 参考资源

- [Arch Wiki - Wake-on-LAN](https://wiki.archlinux.org/title/Wake-on-LAN)
- [技嘉主板 WoL 配置](https://www.gigabyte.com/support)
- [ethtool 手册](https://man.archlinux.org/man/ethtool.8)
