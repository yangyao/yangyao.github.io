---
layout: post
title: "Kubuntu 远程桌面方案选型：从 xrdp 黑屏到 RustDesk"
date: 2026-04-09 21:30:00
tags:
- linux
- kubuntu
- rdp
- kde
- rustdesk
categories:
- engineering
---

最近要把一台 Kubuntu（Ubuntu 24.04, KDE Plasma 5.27）从 Mac 远程连上去。本以为是个很简单的事，结果一路踩坑，最终选了 RustDesk 才算解决。记录一下整个过程。

## 问题：xrdp 连 Kubuntu 黑屏

装好 xrdp，从 Mac 用 RDP 连过去，登录界面正常，输入密码——齿轮转完直接黑屏。

追了一圈，根因很明确：

**KDE Plasma 的 kwin_x11 和 plasmashell 是单例进程，同一用户只能运行一个。** 物理显示器 (:0) 已经占着了，xrdp 在 :10 上创建的新会话永远拿不到窗口管理器。

这不是配置问题，是架构决定的。去 xrdp wiki 查 KDE 支持状态，写着 `TODO`——根本没有官方指南。

用新用户测试验证了这个结论：另一个用户连 RDP 能正常出桌面，因为新用户没有 kwin/plasmashell 冲突。

## 中间方案：x11vnc + xrdp 代理

既然不能创建新会话，那就共享已有的物理桌面。架构：

```
Mac → xrdp (3389/RDP) → libvnc.so → x11vnc (5910/VNC) → Display :0
```

x11vnc 直连物理显示器的 X 会话，xrdp 通过 VNC 后端代理转发给 Mac。Mac 端仍然用 RDP 协议连接。

这个方案能用，但**体验很差**——本质上画面经过了 VNC 编码（截图→压缩→传帧），再被 xrdp 包装成 RDP，延迟明显，操作卡顿。

### 关键配置

x11vnc 的 auth 文件路径由 SDDM 动态生成（如 `/run/sddm/xauth_VdAUQG`），每次重启都变。需要脚本动态查找：

`/usr/local/bin/x11vnc-start.sh`：

```bash
#!/bin/bash
AUTH_FILE=$(find /run/sddm/ -name 'xauth_*' -print -quit 2>/dev/null)
if [ -z "$AUTH_FILE" ]; then
    AUTH_FILE=$(find /tmp -name 'sddm-auth-*' -print -quit 2>/dev/null)
fi
if [ -z "$AUTH_FILE" ]; then
    exit 1
fi
exec /usr/bin/x11vnc -display :0 -auth "$AUTH_FILE" \
    -rfbport 5910 -shared -forever -nopw \
    -threads -cursor -ncache 10 \
    -o /var/log/x11vnc.log
```

踩过一个坑：最初把 `-auth /run/sddm/auth*` 直接写在 systemd 的 `ExecStart` 里，但 **systemd 不展开 glob**，导致服务启动失败。

## 最终方案：RustDesk + 自建服务器

VNC 的体验太差，换 RustDesk。RustDesk 不依赖 xrdp，自己处理屏幕捕获和编码，跟桌面环境无关。

关键是：**RustDesk 默认走公共服务器**（`rs-ny.rustdesk.com`），同一局域网也不直连。对于要完全内网隔离的场景，需要自建服务器。

### 自建 RustDesk Server

在 Kubuntu 机器上同时跑 hbbs（ID 注册服务）和 hbbr（中继服务）：

```bash
# 下载
wget https://github.com/rustdesk/rustdesk-server/releases/latest/download/rustdesk-server-linux-amd64.zip
unzip rustdesk-server-linux-amd64.zip
sudo mkdir -p /usr/local/bin/rustdesk-server
sudo cp amd64/hbbs amd64/hbbr /usr/local/bin/rustdesk-server/
sudo chmod +x /usr/local/bin/rustdesk-server/hbbs /usr/local/bin/rustdesk-server/hbbr
```

`/etc/systemd/system/hbbs.service`：

```ini
[Unit]
Description=RustDesk ID/Rendezvous Server
After=network.target

[Service]
Type=simple
WorkingDirectory=/usr/local/bin/rustdesk-server
ExecStart=/usr/local/bin/rustdesk-server/hbbs
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/hbbr.service`：

```ini
[Unit]
Description=RustDesk Relay Server
After=network.target

[Service]
Type=simple
WorkingDirectory=/usr/local/bin/rustdesk-server
ExecStart=/usr/local/bin/rustdesk-server/hbbr
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable hbbs hbbr
sudo systemctl start hbbs hbbr
```

hbbs 启动后会在工作目录生成 `id_ed25519.pub`，这个就是服务器的公钥 Key。

### 客户端配置

Mac 和 Kubuntu 两端的 RustDesk 都要指向自建服务器。在 RustDesk 设置 → 网络中：

- **ID 服务器**: `192.168.2.217`
- **Key**: `cat /usr/local/bin/rustdesk-server/id_ed25519.pub` 的输出内容

所有流量走局域网，不经过任何外部服务器。

## 附带修的两个问题

### 1. 系统自动休眠

远程机器不能休眠，不然 SSH 都连不上。

`/etc/systemd/sleep.conf.d/disable-sleep.conf`：

```ini
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowSuspendThenHibernate=no
AllowHybridSleep=no
```

`/etc/systemd/logind.conf.d/disable-lid-sleep.conf`：

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
IdleAction=ignore
```

### 2. DPMS 自动关显示器

HDMI 显示器一段时间无操作后显示"无信号"，是 X11 的 DPMS 省电模式。

临时恢复：`DISPLAY=:0 xset dpms force on`

永久禁用，在 KDE 自启动里加一个脚本 `~/.config/autostart/disable-dpms.desktop`：

```ini
[Desktop Entry]
Type=Application
Name=Disable DPMS
Exec=bash -c 'xset s off; xset -dpms; xset s noblank'
X-GNOME-Autostart-enabled=true
```

## 方案对比

| | xrdp 新会话 | x11vnc + xrdp 代理 | RustDesk |
|---|---|---|---|
| KDE 兼容 | 不行，单例限制 | 行（共享物理桌面） | 行（独立捕获） |
| 协议 | RDP | RDP→VNC→X | 自有协议 |
| 体验 | 黑屏 | 卡顿 | 流畅 |
| 部署复杂度 | 低 | 高（x11vnc+xrdp+auth） | 中（自建服务器） |
| 内网隔离 | 是 | 是 | 需自建服务器 |

## 总结

xrdp + KDE Plasma 5 是死路——不是配置能解决的。x11vnc 代理能凑合但体验差。RustDesk 自建服务器是最干净的方案：跟桌面环境无关，协议高效，完全内网可控。

如果以后升级到 KDE Plasma 6，可以考虑 KRDP（KDE 官方 RDP 服务器），但那是以后的事了。
