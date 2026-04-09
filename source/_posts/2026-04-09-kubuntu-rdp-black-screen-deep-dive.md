---
layout: post
title: "从黑屏到真相：Kubuntu RDP 黑屏的完整排查与修复"
date: 2026-04-09 19:50:00
tags:
- linux
- kubuntu
- rdp
- kde
- troubleshooting
categories:
- engineering
---

最近在折腾一台 Kubuntu 机器，想从 Mac 通过 RDP 远程连接。本以为装个 xrdp 就完事了，没想到一连接就掉进了一个深坑：**输入密码后黑屏**。

这篇文章记录了完整的排查过程。不是那种"照着抄就行"的教程，而是一个极客式的追问：**为什么会黑屏？到底是哪个组件在拦？KDE 的桌面会话模型到底是什么机制？xrdp 的工作原理又是什么？** 只有搞清楚这些问题，才能从根上解决问题，而不是靠玄学配置碰运气。

## 问题现象

用 Mac 的 Microsoft Remote Desktop 连接 Kubuntu 机器（`192.168.2.217`），xrdp 登录界面正常出现，输入用户名密码后：

1. 屏幕上出现一个小齿轮图标开始旋转（这是 KDE Plasma 的会话加载动画）
2. 齿轮转完之后，屏幕直接变黑
3. 有时能看到鼠标光标，有时连鼠标都没有
4. 物理显示器上的桌面一切正常

## 第一层：机器是不是挂了？

最开始连不上的时候，SSH 也连不上。按了一下物理电源键，机器又活了。

这很可疑。先查是不是休眠了：

```bash
ssh yangyao@192.168.2.217 "systemctl status sleep.target"
```

果然，机器在无人操作时会自动挂起（suspend）。对于一台要远程访问的机器来说，这是不可接受的。

### 修复：禁用休眠

创建 `/etc/systemd/sleep.conf.d/disable-sleep.conf`：

```ini
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowSuspendThenHibernate=no
AllowHybridSleep=no
```

创建 `/etc/systemd/logind.conf.d/disable-lid-sleep.conf`，防止合盖触发休眠：

```ini
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
IdleAction=ignore
```

```bash
sudo systemctl restart systemd-logind
```

机器不再自动休眠了。但 RDP 的黑屏问题依旧。**休眠只是第一层表象，不是根因。**

## 第二层：黑屏的时候，到底发生了什么？

现在 SSH 能连上了，可以在 RDP 连接的同时观察远程机器上的进程状态。

先看 xrdp 的日志：

```bash
cat ~/.xorgxrdp.%s.log
```

再看当前所有的 Xorg 进程：

```bash
ps aux | grep Xorg
```

输出类似这样：

```
root     1355  /usr/lib/xorg/Xorg -auth /run/sddm/xauth_VdAUQG ... vt2    # 物理显示器 :0
yangyao  2301  /usr/lib/xorg/Xorg :10 -auth .Xauthority -config xrdp/xorg.conf  # xrdp 创建的 :10
```

关键发现：**xrdp 在 display :10 上成功启动了一个新的 Xorg 进程**。X 服务器本身是正常的。

那问题出在 X 服务器之上的层面。继续看窗口管理器和桌面：

```bash
ps aux | grep kwin
ps aux | grep plasmashell
```

结果：

```
yangyao  1500  kwin_x11       # 只在 :0 上运行
yangyao  1510  plasmashell    # 只在 :0 上运行
```

**kwin_x11 和 plasmashell 只在物理显示 :0 上运行，在 xrdp 创建的 :10 上根本没有启动。**

没有窗口管理器 = 没有任何窗口能被渲染 = 黑屏。鼠标光标是 Xorg 本身提供的，所以有时能看到。

## 第三层：为什么 kwin 和 plasmashell 不启动？

这才是问题的核心。让我深挖一下。

查看 `startwm.sh`——这是 xrdp 创建新会话后执行的桌面启动脚本：

`/etc/xrdp/startwm.sh`：

```sh
#!/bin/sh
if test -r /etc/profile; then
    . /etc/profile
fi
if test -r ~/.profile; then
    . ~/.profile
fi
export QT_X11_NO_MITSHM=1
test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession
```

这个脚本会调用 `/etc/X11/Xsession`，后者最终会启动 KDE Plasma 会话。问题是：**KDE Plasma 的核心组件能不能为同一个用户启动第二个实例？**

手动测试一下：

```bash
# 在 :10 的环境中尝试启动 kwin
DISPLAY=:10 kwin_x11 &
```

观察日志，发现 kwin_x11 检测到已经有实例在运行，**直接退出了**。

这不是 bug。这是 KDE Plasma 的设计：

> **kwin_x11 和 plasmashell 都是单例进程（singleton）。同一用户只能运行一个实例。**

这个设计在物理桌面场景下完全合理——谁会需要两个窗口管理器？但在远程桌面场景下，这就变成了致命限制：物理显示器 (:0) 已经占据了 kwin 和 plasmashell 的唯一名额，xrdp 新建的 :10 会话永远拿不到窗口管理器。

## 第四层：官方怎么说？

作为一个较真的极客，我不能只凭观察就下结论。我要找官方文档来验证。

先查 xrdp 的官方文档。xrdp wiki 上有一个 [Desktop Support](https://github.com/neutrinolabs/xrdp/wiki/DE-Desktop-Support) 页面，列出了各种桌面环境的支持情况。

找到 KDE 那一栏，赫然写着：

> **TODO**

是的，KDE 部分是空的。xrdp 项目组根本没有为 KDE 桌面提供官方的配置指南。

再去看 KDE 这边。KDE Plasma 6 引入了一个叫 **KRDP** 的官方 RDP 服务器组件，专门为 KDE 会话设计。但我的系统是：

```
Ubuntu 24.04 LTS
KDE Plasma 5.27.12  （Plasma 5 的最后一个版本）
```

KRDP 需要 Plasma 6+，**在 Plasma 5.27 上不可用**。

Ubuntu 24.10 和 25.04 才自带 Plasma 6 + KRDP。

到这里，真相完全清晰了：

1. xrdp 通过创建新 X 会话实现远程桌面
2. KDE Plasma 的核心组件是单例，同一用户不能启动两个
3. 物理显示器已经占用了唯一的 kwin/plasmashell 实例
4. xrdp 没有官方的 KDE 支持（wiki 上写的 TODO）
5. KDE 自己的解决方案（KRDP）需要 Plasma 6，我的系统不满足

**这不是配置问题，而是架构层面的不兼容。**

## 第五层：验证根因——换一个用户试试

为了彻底确认"单例"这个根因，我创建了一个新用户 `rdpuser`，用这个用户的凭据通过 xrdp 登录。

```bash
sudo useradd -m rdpuser
sudo passwd rdpuser
```

用 RDP 以 rdpuser 身份连接——**桌面完美呈现**。kwin_x11 和 plasmashell 都在 :12 上正常启动了。

这彻底证实了：**问题不是 xrdp 不支持 KDE，而是同一个用户不能同时拥有两个 KDE 会话。**

## 最终方案：共享物理桌面，而非创建新会话

既然不能为同一用户创建第二个 KDE 会话，那思路就应该反过来：**不创建新会话，而是把已有的物理桌面共享出去。**

架构变成：

```
Mac RDP Client → xrdp (3389) → libvnc.so → x11vnc (5910) → 物理 Display :0
```

x11vnc 是一个 VNC 服务器，它直接连接到物理显示器的 X 会话，把画面以 VNC 协议输出。xrdp 通过 libvnc.so 模块作为 VNC 客户端连接到 x11vnc，然后以 RDP 协议转发给 Mac 客户端。

这样，Mac 端仍然使用 RDP 协议连接，但实际上看到的是物理桌面。

### 第一步：安装 x11vnc

```bash
sudo apt install x11vnc
```

### 第二步：创建 x11vnc 启动脚本

x11vnc 需要一个 X11 的 auth 文件才能连接到 display :0。这个文件由 SDDM（KDE 的显示管理器）在登录时生成，路径是随机的（如 `/run/sddm/xauth_VdAUQG`）。如果硬编码路径，重启后路径变了就又会失败。

所以需要一个脚本来动态查找：

`/usr/local/bin/x11vnc-start.sh`：

```bash
#!/bin/bash
# 动态查找 SDDM 为 display :0 生成的 auth 文件
AUTH_FILE=$(find /run/sddm/ -name 'xauth_*' -print -quit 2>/dev/null)
if [ -z "$AUTH_FILE" ]; then
    AUTH_FILE=$(find /tmp -name 'sddm-auth-*' -print -quit 2>/dev/null)
fi
if [ -z "$AUTH_FILE" ]; then
    exit 1
fi
exec /usr/bin/x11vnc -display :0 -auth "$AUTH_FILE" \
    -rfbport 5910 -shared -forever -nopw \
    -o /var/log/x11vnc.log
```

每个参数的作用：

- `-display :0` — 连接到物理显示器
- `-auth "$AUTH_FILE"` — 用 SDDM 的 auth 文件获得访问权限
- `-rfbport 5910` — 监听在 5910 端口（VNC 协议）
- `-shared` — 允许多个客户端同时连接
- `-forever` — 客户端断开后不退出，继续等待新连接
- `-nopw` — 不需要 VNC 密码（认证由 xrdp 处理）

### 第三步：创建 systemd 服务

`/etc/systemd/system/x11vnc.service`：

```ini
[Unit]
Description=x11vnc server for console sharing
After=display-manager.service

[Service]
Type=simple
ExecStart=/usr/local/bin/x11vnc-start.sh
ExecStop=/usr/bin/killall x11vnc
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo chmod +x /usr/local/bin/x11vnc-start.sh
sudo systemctl daemon-reload
sudo systemctl enable x11vnc
sudo systemctl start x11vnc
```

这里有一个之前踩过的坑：最初直接在 systemd 的 `ExecStart` 里写 `-auth /run/sddm/auth*`，用 glob 模式匹配 auth 文件。但 **systemd 不支持在 ExecStart 中展开 glob**，所以路径匹配不到文件，服务一直启动失败。这就是为什么改用脚本动态查找的原因。

### 第四步：配置 xrdp 代理到 x11vnc

`/etc/xrdp/xrdp.ini`：

```ini
[Globals]
ini_version=1
fork=true
port=3389
tcp_nodelay=true
tcp_keepalive=true
security_layer=negotiate
crypt_level=high
certificate=
key_file=
ssl_protocols=TLSv1.2, TLSv1.3
autorun=

allow_channels=true
allow_multimon=true
bitmap_cache=true
bitmap_compression=true
bulk_compression=true
max_bpp=32
new_cursors=true
use_fastpath=both

blue=009cb5
grey=dedede

ls_width=350
ls_height=430
ls_bg_color=dedede
ls_logo_x_pos=55
ls_logo_y_pos=50
ls_label_x_pos=30
ls_label_width=65
ls_input_x_pos=110
ls_input_width=210
ls_input_y_pos=220
ls_btn_ok_x_pos=142
ls_btn_ok_y_pos=370
ls_btn_ok_width=85
ls_btn_ok_height=30
ls_btn_cancel_x_pos=237
ls_btn_cancel_y_pos=370
ls_btn_cancel_width=85
ls_btn_cancel_height=30

[Logging]
LogFile=xrdp.log
LogLevel=INFO
EnableSyslog=true

[Channels]
rdpdr=true
rdpsnd=true
drdynvc=true
cliprdr=true
rail=true
xrdpvr=true
tcutils=true

[console]
name=Console (Physical Desktop)
lib=libvnc.so
ip=127.0.0.1
port=5910
username=ask
password=ask
```

关键改动是 **只保留了 `[console]` 这一个会话定义**，去掉了原来的 `[Xorg]` 和 `[Xvnc]`。这样新连接不会尝试创建新的 X 会话（那会失败），而是直接连接到 x11vnc 共享的物理桌面。

- `lib=libvnc.so` — 使用 VNC 后端（而非 xup/Xorg 后端）
- `ip=127.0.0.1` — 连接本机的 x11vnc
- `port=5910` — x11vnc 的监听端口

### 第五步：sesman.ini 的调整

`/etc/xrdp/sesman.ini` 中也做了几处修改：

```ini
KillDisconnected=true        # 原值 false — 断开连接后立即终止会话
DisconnectedTimeLimit=5      # 原值 0   — 5 秒后清理断开的会话
Policy=UBC                   # 原值 Default — 按 User+BitPerPixel+Connection 分配会话
```

这些改动确保断开 RDP 后不会有残留进程。

```bash
sudo systemctl restart xrdp
```

### 连接成功

Mac 上用 Microsoft Remote Desktop 连接 `192.168.2.217`，输入用户名密码——**物理桌面的完整 KDE Plasma 环境出现了**。不是新会话，不是黑屏，就是那台机器上真实的桌面。

## 整个排查链路回顾

```
黑屏
 → 第一层：机器休眠了？ → 是，修了。但 RDP 还是黑屏。
  → 第二层：Xorg 启动了吗？ → 是的，:10 上有 Xorg。但黑屏。
   → 第三层：窗口管理器启动了吗？ → 没有。kwin_x11 和 plasmashell 只在 :0 上。
    → 第四层：为什么不在 :10 上启动？ → KDE 单例设计。同一用户只能有一个实例。
     → 第五层：官方怎么说？ → xrdp wiki KDE 部分写着 TODO。KRDP 需要 Plasma 6。
      → 第六层：换思路。不创建新会话，共享已有的物理桌面。
       → x11vnc 连 :0 → xrdp 代理 → Mac RDP 客户端。成功。
```

## 改动文件清单

| 文件 | 改动 | 目的 |
|---|---|---|
| `/etc/systemd/sleep.conf.d/disable-sleep.conf` | 新建 | 禁止系统休眠 |
| `/etc/systemd/logind.conf.d/disable-lid-sleep.conf` | 新建 | 禁止合盖休眠 |
| `/usr/local/bin/x11vnc-start.sh` | 新建 | 动态查找 SDDM auth 文件并启动 x11vnc |
| `/etc/systemd/system/x11vnc.service` | 新建 | x11vnc 的 systemd 服务，开机自启 |
| `/etc/xrdp/xrdp.ini` | 改写 | 只保留 `[console]` 会话，代理到 x11vnc |
| `/etc/xrdp/sesman.ini` | 修改 `KillDisconnected`, `DisconnectedTimeLimit`, `Policy` | 断开后及时清理会话 |
| `/etc/xrdp/startwm.sh` | 添加 `export QT_X11_NO_MITSHM=1` | 避免 Qt 的 MIT-SHM 共享内存问题（虽然最终没用上新会话） |

## 如果重来，什么是最短路径？

如果让我从零开始解决同样的问题，最短路径是：

1. 意识到 xrdp 默认是**创建新会话**，不是**共享已有会话**
2. 意识到 KDE Plasma 不支持同一用户同时拥有两个会话
3. 直接走"共享物理桌面"路线：x11vnc + xrdp libvnc.so 代理

前面花的大部分时间，其实是在验证"到底能不能让 xrdp 创建的新会话正常工作"。答案是不能——这不是配置能解决的，是 KDE 的架构决定的。

## 后记

这次排查给我最大的感触是：**黑屏这种症状太泛了，不能靠搜"KDE RDP 黑屏"来找答案。** 网上各种帖子说的解决方案千奇百怪——改 startwm.sh、装其他桌面环境、建新用户——大部分是在绕过问题而非解决问题。

只有一层一层往下追，从现象到进程到组件到设计理念，才能找到真正的根因。根因清楚了，解决方案反而是最简单的那个。
