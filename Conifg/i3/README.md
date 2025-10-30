# 如何使用 i3 配置文件

首先你需要备份位于`.config/i3`文件夹中的配置文件和位于`/etc/i3status.conf`文件

## 使用配置文件前需要的软件包

在使用配置文件前请安装下列软件包，这样你将获得最好的体验  

| 软件名 |         用途         |
| :----: | :------------------: |
| picom  |  半透明化合成渲染器  |
| scrot  |     屏幕截图软件     |
|  feh   | 壁纸实现及图片预览器 |
| thunar |    文件资源管理器    |

## 使用配置文件

- 将这里面的`i3status.conf`放到`/etc`目录下
- 将`config`、`i3bar.conf`、`net-speed.sh`、`wallpaper`放置到用户目录下的`.config/i3`下面
- 赋予`net-speed.sh`和位于`wallpaper`中的`wallpaper.sh`运行权限
