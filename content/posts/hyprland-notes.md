+++
title = 'Hyprland 安装以及配置记录'
date = 2025-02-07T20:58:30+08:00
draft = false
toc = true
tocBorder = true
+++

安装过程不记录，在 ArchWiki 上有详细的安装过程。本文仅记录 Hyprland 安装之后的配置过程，以及一些问题的解决方法。

## 应用程序

- Kvantum: GTK, QT 主题统一工具
- Nwg-Look: GTK 外观配置工具
- Waybar: 状态栏
- Wofi: 应用启动器
- Thunar: 文件管理器

可通过包管理器安装以上包含的所有应用。

```sh
paru -S hyprland hyprtheme hyprpicker waybar wofi nwg-look kvantum thunar
```

## 网络管理

在 Arch Linux 安装阶段需要安装网络管理工具，特别是笔记本等通过 WiFi 连接网络的设备。推荐使用 Iwd[^1] 或 NetworkManager[^2]，对于这两者，我更推荐 Iwd，因为它更轻量级。需要通过图形界面来管理网络建议使用 NetworkManager。

```sh
pacman -S iwd
```

## XWayland[^3] 缩放后显示模糊

XWayland 的缩放和 DPI 设置不支持 HiDPI[^4]，因此需要强制将 XWayland 的缩放设置为 1x。

```sh
# ~/.config/hypr/hyprland.conf
xwayland {
  force_zero_scaling = true
}
```

## Chromium 系应用启用 Wayland

Chromium 系应用默认使用 XWayland，因此需要手动启用 Wayland。

```sh
# ~/.config/chrome-beta-flags.conf
--enable-features=UseOzonePlatform
--ozone-platform=wayland
--enable-wayland-ime

# ~/.config/chrome-flags.conf
--enable-features=UseOzonePlatform
--ozone-platform=wayland
--enable-wayland-ime

# ~/.config/code-flags.conf
--ozone-platform=wayland
--enable-wayland-ime
--gtk-version=4
--ignore-gpu-blocklist
--enable-features=TouchpadOverscrollHistoryNavigation
```

## 电源控制

```sh
# ~/.config/hypr/hyprland.conf
bind = SUPER, ESCAPE. exec, wlogout
```

## Secure Boot

与 Ubuntu 和 Fedora 等默认支持 Secure Boot 的发行版不同，Arch Linux 默认不提供 Secure Boot 支持，需要手动签名来启用。

依照官方 Wiki 进行签名过于繁琐，且在后续使用期间可能会因为软件的修改导致签名失效无法启动等问题，不建议采取官方的解决方案，推荐使用 [SbCtl](https://github.com/Foxboron/sbctl) 来简化签名过程。

```sh
pacman -S sbctl
```

在进行签名之前要在 BIOS 中将 Secure Boot 设置为 Setup Mode。

```sh
# 检查当前安全启动状态
sbctl status

# 创建用户自定义密钥
sbctl create-keys

# 注册密钥
sbctl enroll-keys

# 检查当前镜像卡签名状态
# 该命令会列出所有需要签名的镜像，所有的镜像签名后才能够启用 Secure Boot
sbctl verify

# 进行签名
sbctl sign -s <filename>
```

签名完成之后建议使用 `sbctl verify` 再次验证签名状态，如果有未签名的镜像，则需要重新进行签名。

```goat
+--------------------------------------------------------------------------------------+
| +------------+-----------------------+------+-----------------+------+-------------+ |
| | Work Space |                       | Time |                 | Tray | Information | |
| +------------+-----------------------+------+-----------------+------+-------------+ |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                      Hyprland                                        |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
|                                                                                      |
+--------------------------------------------------------------------------------------+
```

---

大部分内容和配置来自于 [Arch Wiki](https://wiki.archlinux.org/title/Main_page) 和 [Hyprland Wiki](https://wiki.hyprland.org/)。了解更多请参阅以上维基。

_本文长期记录安装 Hyprland 后续的配置和问题解决方法。_

{data-content="footnotes"}

[^1]: iNet Wireless Daemon: 由 Intel 开发的无线网络守护进程，支持 WiFi 6E 和 WiFi 7。

[^2]: NetworkManager: 网络管理器，支持多种网络连接方式。

[^3]: XWayland: X11 的 Wayland 兼容层，允许运行 X11 应用。

[^4]: HiDPI: 高分辨率显示器，通常指 4K 或更高分辨率的显示器。
