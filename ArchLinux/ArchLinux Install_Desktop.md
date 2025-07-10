# Arch Linux 桌面安装

对于新安装的操作系统来说，桌面环境是完全没有的，为此你需要安装许多软件包来搭建你的桌面环境。  
通常情况下你可以安装如 gnome 这样的桌面环境，并仅通过简单配置就能或一个相对现代化的桌面，又或者通过一些列折腾，最终获得一个你喜欢的你想要的桌面环境。  
本文将会通过一个相对复杂且折腾的例子讲解如何配置一个自己想要的桌面环境。

在正式进入安装桌面环境前，我们需要先配置后基础的东西`图形驱动`，我会在下列表格中列出图形驱动，你仅需要更具你的情况安装相应的包即可。

|     品牌     | 类型 |    DRM 驱动     |        OpenGL         |     Vulkan     |      DDX 驱动      |
| :----------: | :--: | :-------------: | :-------------------: | :------------: | :----------------: |
| AMD (ex-ATI) | 开源 | 包含在 Linux 中 |         mesa          |     amdvlk     | xf86-video-amdgpu  |
|     ATI      | 开源 | 包含在 Linux 中 |         mesa          |     amdvlk     |   xf86-video-ati   |
|    Intel     | 开源 | 包含在 Linux 中 |         mesa          |  vulkan-intel  |  xf86-video-intel  |
|    NVIDIA    | 开源 | 包含在 Linux 中 |         mesa          | vulkan-nouveau | xf86-video-nouveau |
|    NVIDIA    | 专有 |   nvidia-open   | nvidia 或 nvidia-open |  nvidia-utils  |    nvidia-utils    |

如果你不确定你的显卡是哪一个品牌的，你可以使用"lspci -d ::03xx"来查看你的设备中所拥有的显卡。

```bash
    lspci -d ::03xx
    #如果你是虚拟环境
    00:0f.0 VAG compatible controller: VMware SVGA II Adapter   #VMware 虚拟机显卡
    #如果你是实体机环境
    00:02.0 VGA compatible controller: Intel Corporation HD Graphics 630 (rev 04)   #CPU的核显
    01:00.0 3D controller: NVIDIA Corporation GP107M [GeForce GTX 1050 Mobile] (rev a1) #独立显卡
```

接下来只需要根据系统展示的内容安装`图形驱动`即可，如果你是虚拟机直接安装"mesa"包，如果你用 N 卡或者 A 卡又或者只有核显那么仅需要更具展示内容安装上表提供的软件包安装即可，通常情况下你只需要安装 OpenGL 和 DDX 驱动即可。

以下是常见的桌面环境，你可以在搜索引擎上搜索对应桌面环境的图片，挑选一个你所喜欢的，如果下列表格中没有列出你也可以查阅官方 wiki 进行配置。  
桌面环境都需要底层的渲染器（中间层）支持，通常来说在 Linux 上是使用 Xorg，当然最近几年也有新起的 Wayland。

需要注意的是 Xorg 是相对传统的中间层支持，但由于 Xorg 采用 C/S 架构（客户端-服务器架构），通讯路径、内存开销、内存安全以及软件构成架构等等存在或多或少的历史遗留问题，但 Xorg 胜在兼容性强、支持文档齐全这些优势。考虑到 Xorg 是从上世纪 90 年代就存在的中间层部分配置相对来说会比较复杂，以及存在部分内存安全性问题，如果可以请尽量选择使用 Wayland。

Wayland 作为新兴的中间层支持采用基于协议的结构，优化了 Xorg 的通讯路径较长、内存开销过大以及安全性的问题，但由于 Wayland 是新技术在一定程度上存在兼容性问题，或不支持 Xorg 的一些高级功能，又或者采用"XWaylangd"转义部分应用等问题，可能在一定程度上对于用户体验来说不会特别好。  
考虑到目前的 Linux 社区发展情况，使用 Wayland 是富有前瞻性的，如红帽的 RHEL 已经完全依赖于 Wayland，KDE、Gnome 已经明确将会迁移默认使用 Wayland 作为中间层。

本文将不会专注于其中一个中间层来讲解，而是将会同时讲解 2 种不同中间层配置和桌面环境搭建，你可以根据你的需求选择任意中间层来配置你的桌面环境，如果你考虑兼容性、稳定性那么 Xorg 是一个不错的选择，如果你喜欢更现代化、更强大、效率更好的中间层你可以选择 Wayland，又或者两个都安装在登录界面选择你想要的使用的中间层。

|    桌面环境名    | Xorg 支持 | Wayland 支持 | Wayland 支持所需软件包 |
| :--------------: | :-------: | :----------: | :--------------------: |
|  GNOME(窗口式)   |    是     |   是(默认)   |     xorg-xwayland      |
|   Xfce(窗口式)   |    是     |  实验性支持  |     xfwm4-wayland      |
|    i3(平铺式)    |    是     |  通过 Sway   |          sway          |
| Hyprland(平铺式) |    否     |      是      |           -            |

## 窗口式桌面环境

## 平铺式桌面环境
