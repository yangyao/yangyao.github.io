---
layout: post
title: "Dell XPS 9360 (i5-7200U) 安装黑苹果手记"
date: 2026-04-27 10:00:00
tags:
- hackintosh
- xps9360
- opencore
- dell
- macos
categories:
- engineering
---

家里有一台闲置很久的 Dell XPS 9360，i5-7200U，13 寸 1080p 屏幕。家里设备已经很全了——MacBook、iPad、NAS、软路由、HomeServer 一个不少。这台 XPS 放着也是落灰，不如装个黑苹果折腾一下。

不是为了实用，纯粹是想玩。

## 准备工作

黑苹果的安装流程在 2026 年已经比早年成熟很多了，核心工具就两个：

1. **OCLP (OpenCore Legacy Patcher)**：在 Mac 上制作 macOS 安装 U 盘
2. **OpCore Simplify**：在 Windows 下根据硬件自动生成专属 EFI 配置

流程是这样的：先用 Mac 上的 OCLP 把 macOS 镜像写入 U 盘，然后切到 Windows 用 OpCore Simplify 检测 XPS 9360 的硬件（CPU、显卡、声卡等），导出一份定制的 EFI 文件夹。最后用 DiskGenius 把 EFI 文件夹拖进 U 盘的 EFI 分区，理论上就能引导了。

## U 盘引导踩坑

理论归理论，实际开机按 F12 进启动菜单——看不到 U 盘。

### 第一轮：BIOS 设置

先排查 BIOS 设置。XPS 9360 需要调整这几项：

- **Secure Boot** → Disabled（关键，否则不认 OpenCore）
- **Boot List Option** → UEFI
- **Fast Boot** → Thorough（防止跳过 USB 初始化）
- **Enable Legacy Option ROMs** → 勾选

改完保存重启，F12 还是不行。

### 第二轮：Legacy 模式的诱惑

抱着试试的心态把 Boot List Option 改成了 Legacy——这回 U 盘出现了！但选中后立刻启动失败。

原因很简单：OCLP 和 OpCore Simplify 制作的 EFI 都是面向 UEFI 的（GPT 分区表 + FAT32 ESP），Legacy 模式需要的是 MBR 分区 + 专门的引导程序（DuetPkg）。两者不匹配，自然起不来。

结论很明确：**必须坚持 UEFI 模式。**

### 第三轮：换思路——EFI 直接写入硬盘

折腾 U 盘启动实在不行，换个思路：既然硬盘的 ESP 分区是正常的，为什么不把 OpenCore 的 EFI 直接放进去？

用 DiskGenius 查看硬盘 ESP 分区，里面有三个文件夹：

- `Boot`：UEFI 默认引导项
- `Dell`：Dell 硬件诊断工具
- `Microsoft`：Windows 引导文件

都不用动。只需要把 OpCore Simplify 导出的 EFI 里的 `OC` 文件夹复制进去就行。

操作步骤：

1. 在 DiskGenius 中右键 ESP 分区 → 分配盘符
2. **先备份**整个 EFI 文件夹到桌面
3. 把 U 盘 EFI 里的 `OC` 文件夹复制到硬盘 ESP 的 `EFI` 目录下
4. 不动 `Boot`、`Dell`、`Microsoft`，Windows 引导完全不受影响

然后在 BIOS 里手动添加启动项：指向 `\EFI\OC\OpenCore.efi`，设为第一启动项。

保存退出，重启——终于看到了 OpenCore 的引导菜单。

## Kernel Panic：Broadcom 网卡驱动崩溃

成功进入 OpenCore 菜单，选择 macOS 安装盘，安装过程顺利。但第二天开机时卡在了进度条，强制重启后出现了五国文字（kernel panic）。

崩溃报告的关键信息：

```
Kernel Extensions in backtrace:
   com.apple.driver.AirPort.BrcmNIC(1400.1.1)
```

根因是 Broadcom 无线网卡驱动在加载时 crash 了。这在 XPS 9360 黑苹果里比较常见，不是系统损坏，纯粹是驱动问题。

解决方案：

1. **Reset NVRAM**：进入 OpenCore 引导菜单，按空格键显示隐藏工具，选择 Reset NVRAM
2. **OCLP Post-Install Root Patch**：进系统后运行 OCLP 的后安装补丁，它会自动更新 Wi-Fi 驱动

两步做完，网卡恢复正常，启动也稳定了。

## HiDPI：13 寸 1080p 屏幕的字体问题

macOS 装好后第一个感受是——字太小了。13 寸 1080p 在 macOS 下的默认缩放和 Windows 完全不同，UI 元素会偏小。

解决办法是开启 HiDPI 模式，让系统用更高分辨率渲染再缩放回来。用社区的一键脚本：

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
```

脚本会提示选择显示器型号和分辨率。我选了 1424x802——比原生 1080p 小一些，但字体明显变大，而且这个分辨率专门修复了睡眠唤醒后屏幕缩小的问题。

开启后在「系统设置 → 显示器」里切换到新的 HiDPI 分辨率即可。

副作用是会略微增加显卡负担，在 XPS 9360 的 Intel HD 620 上偶尔能感觉到轻微卡顿，但不影响日常使用。

## 最终使用体验

折腾完之后，这台 XPS 9360 上跑着 macOS（系统型号被识别为 MacBookPro14,1），日常使用体验：

**流畅度**：i5-7200U 虽然是双核四线程，但跑 macOS 日常任务（浏览器、文档、终端）完全够用。毕竟 macOS 对硬件的优化比 Windows 好，老机器反而跑得比预期流畅。

**续航**：换了一块淘宝 150 元的新电池，日常使用大概 4-5 小时，作为备用机完全合格。

**兼容性**：HiDPI 开启后偶尔有睡眠唤醒问题（唤醒后屏幕不亮），需要重新开合一次屏幕盖。Wi-Fi 和蓝牙正常，触控板手势基本可用但不如真 Mac 流畅。

**不足**：13 寸非触屏、1080p 分辨率、USB 接口少（只有两个 USB-C），这些硬件短板是系统软件解决不了的。

## 小结

这台 XPS 9360 的黑苹果之旅，踩坑主要集中在两个地方：

1. **U 盘 UEFI 引导不识别**：最后绕过 U 盘，直接把 OpenCore EFI 合并到硬盘 ESP 分区解决
2. **Broadcom 网卡驱动 kernel panic**：重置 NVRAM + OCLP Root Patch 解决

都不是什么高深的问题，但如果你也在 XPS 9360 上装黑苹果，这两个坑大概率会遇到。希望这篇记录能帮你省点时间。
