# Arch Linux 安装

## Arch Linux 官方提供快速安装脚本

虽然官方提供了自动安装脚本，但在本文中不会过多介绍和讲解如何使用。  
如果需要使用请自行点击下面的链接访问对应页面查看。  
自动安装脚本介绍页面（英文）: <https://wiki.archlinux.org/title/Archinstall>  
自动安装脚本介绍页面（中文）: <https://wiki.archlinuxcn.org/wiki/Archinstall>  
自动安装脚本使用页面（英文）: <https://archinstall.archlinux.page/installing/guided.html>

## Arch Linux 手动安装

手动安装 Arch Linux 一直是官方鼓励使用的方式，这样你不仅可以知道在安装过程中出现的各种问题，同时也可以根据需要自定义需要安装的软件包。  
在正式安装操作系统前，你需要知道 GPT 分区和 MBR 分区的特点，这不仅影响你后面引导操作系统的方式，同时也影响你系统的安全性。  
在本文中不会过多赘述 GPT 分区和 MBR 分区的特点，以及如何影响操作系统，如果需要可以点[这里](https://sizuku.privatecloud.fun:14444/Na-Sizuku/sizuku-study-doc/-/blob/master/Extend/Disk%20Partition.md)

### 准备系统安装镜像

Arch Linux 系统镜像你可以从官方网站获取，也可以通过镜像服务器获取，通常情况下推荐从镜像服务器获取，因为下载的速度更快。  
你可以从这里获取完整的镜像服务器[列表](https://archlinux.org/download/)，并根据自身所处的位置进行选择。  
如果你在国内，可以参考下面列表里展示的几个镜像服务器，并根据自身情况进行选择。

|                    站点(国内各大企业)                    | 最大速率(MB/s) |                       站点(国内各大高校)                        | 最大速率(MB/s) |
| :------------------------------------------------------: | :------------: | :-------------------------------------------------------------: | :------------: |
|   [阿里云](https://mirrors.aliyun.com/archlinux/iso/)    |      1000      | [中国科学技术大学](https://mirrors.ustc.edu.cn/archlinux/iso/)  |      1000      |
|  [华为](https://mirrors.huaweicloud.com/archlinux/iso/)  |      1000      | [清华大学](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/) |      1000      |
| [腾讯](https://mirrors.cloud.tencent.com/archlinux/iso/) |     10000      |      [北京大学](https://mirrors.pku.edu.cn/archlinux/iso/)      |     10000      |
|      [网易](https://mirrors.163.com/archlinux/iso/)      |     20000      |                                -                                |       -        |

下载好系统镜像后记得下载 sha256sum.txt 用于对比文件 hash 值，这不仅仅是避免文件在下载的时候被修改，同时也是避免下载后的文件损坏。  
对于 Windows 操作系统你可以使用下面的命令进行校验。

```powershell
CertUtil -hashfile <文件名/文件路径> sha256
```

对于 Linux 操作系统你可以直接使用 sha256sum 命令进行校验。  
如果你已使用 Arch Linux 作为你的操作系统，为什么还要来看安装指北，你应该去看官方 Wiki。

```bash
sha256sum -c sha256sums.txt
```

当然你也可以跳过文件校验制作系统镜像 U 盘或光盘。

### 验证网络连接

Arch Linux 官方提供的镜像不包含任何软件包，这也意味着安装操作系统需要全程联网。  
对于有线连接的设备镜像在启动后将会自动使用 DHCP 获取网络，如果你在特定的网络环境下，你可能需要配置固定 IP 来连接网络。

```bash
ip addr add <IP地址>/<子网掩码前缀> brd <广播地址> dev <接口名字>
ip addr add 192.168.1.201/24 brd 192.168.1.255 dev ens192
```

上述命令是用于配置静态 IP 地址，需要注意静态 IP 地址不能和 DHCP 获取的 IP 地址一致，不然会提示地址已经使用。  
这里没用使用 net-tools 包里的 ifconfig 命令，这是由于 net-tools 包存在的一些问题，目前官方镜像不提供使用 net-tools 提供的命令如 ifconfig 等，转而使用的是 iproute2 提供的命令，对应的命令改变将在下表体现。

| 弃用的命令 |                                      替换命令                                       |
| :--------: | :---------------------------------------------------------------------------------: |
|    arp     |                                 ip n (ip neighbor)                                  |
|  ifconfig  |                     ip a (ip addr), ip link, ip -s (ip -stats)                      |
|  iptunnel  |                                      ip tunnel                                      |
|  iwconfig  |                                         iw                                          |
|   nameif   |                                  ip link, ifrename                                  |
|  netstat   | ss, ip route (for netstat-r), ip -s link (for netstat -i), ip maddr (for netstat-g) |
|   route    |                                   ip r (ip route)                                   |

如果你使用 WiFi 作为网络接入，你将需要使用 iwd 包的 iwctl 进入 WiFi 连接 CLI 界面。

```bash
station <设备名称> scan
station <设备名称> get-networks
station <设备名称> connect <网络名称>
```

![iwctl界面](../Images/iwctl.png)

在完成网络配置后一定要记得 ping 一下，比如 baidu.com

### 磁盘分区

自 2012 年后的计算机主板基本都支持了 EFI 引导，在磁盘分区中将优先考虑使用 GPT 分区而不优先考虑使用 MBR 分区。  
如果你不确定你的电脑是否支持 EFI 引导，你可以通过下列命令进行验证。

```bash
cat /sys/firmware/efi/fw_platform_size
```

需要注意的是如果你的 BIOS 没有设置 EFI 引导，命令可能无法执行。  
对于磁盘分区，推荐使用 fdisk 进行操作，当然你也可以根据需要选择其他自己熟悉分区工具，本文将使用 fdisk 作为演示。  
在安装镜像中执行 lsblk 用于查看计算机中磁盘详细信息，如安装了几块磁盘，有哪些磁盘已经分区，磁盘挂载点等等。  
![lsblk](../Images/lsblk.png)  
在演示中使用的磁盘为 sda，实际安装中还请参考 lsblk 命令执行结果，并`根据情况`修改命令的**参数**。  
使用 fdisk 对 sda 磁盘进行修改，当键入命令后将会进入 fdisk CLI 界面。

```bash
fdisk /dev/<磁盘名称>
```

![fdisk](../Images/fdisk.png)  
默认情况下 fdisk 会自动创建一个 DOS 分区表既 MBR 分区表，如果你的系统已经支持了 EFI 引导，那么请输入 g 并回车执行创建 GPT 分区表。  
如果不知道怎么操作请根据提示键入 m 并回车执行，fdisk 将会告诉你可用的操作。  
![make GPT partition](../Images/fdisk-g.png)  
接下来你可以根据你的需要进行相应的分区工作，如果看不动英文你可以参考[此处](https://wiki.archlinuxcn.org/wiki/Fdisk#%E5%88%9B%E5%BB%BA%E5%88%86%E5%8C%BA%E8%A1%A8%E5%92%8C%E5%88%86%E5%8C%BA)，这里是 ArchLinux 中文社区 Wiki，讲述了如何操作 fdisk 进行磁盘分区。

你可以根据下表来创建分区和规划大小，默认情况下/home 和/root 是直接在根分区(/)下面创建的，如果你需要将用户家目录(/home)进行独立分区，你可以根据需要酌情减少根分区大小。  
如果你使用 Arch Linux 作为你的主力操作系统，推荐将用户家目录分区进行独立，这样当你需要重新安装操作系统的时候，用户数据将不会丢失。  
如果你不清楚 Linux 文件结构的特点，那么你可以参考该视频[100 秒解释 Linux 系统目录结构](https://www.bilibili.com/video/BV1y3411t7Tz/)，如果你在海外可以参考[Linux directories in 100 seconds](https://youtu.be/42iQKuQodW4)

| 系统挂载点 |              分区类型              |         推荐大小         |                                        用途                                         |
| :--------: | :--------------------------------: | :----------------------: | :---------------------------------------------------------------------------------: |
|   /boot    | EFI 系统分区(EFI system partition) |        300Mb 起步        |    用于给 EFI 引导准备的特殊分区，如果使用 MBR+传统 BIOS 引导可以则不需要该分区     |
|   [SWAP]   |     Linux 交换分区(Linux swap)     | 4G 起步，最好为 2 的倍数 | 当系统内存不够用时作为虚拟内存使用，如果内存大于 8G 可酌情考虑，大于 16G 则可不需要 |
|     /      |  Linux 文件系统(Linux filesystem)  |      最少 20G 起步       |                   作为系统的根分区，最好和/boot 分区在同一磁盘下                    |
|   /home    |  Linux 文件系统(Linux filesystem)  |   可以根据需要进行创建   |           作为用户家目录的挂载点，如果不单独分区则需要加大系统根分区大小            |
|   /root    |  Linux 文件系统(Linux filesystem)  |   可以根据需要进行创建   |    作为 root 特权用户的家目录挂载点，如果不单独进行分区则需要加大系统根分区大小     |

作为演示，下面是一个已经完成分区的磁盘，由于演示系统作为服务器使用，故此没有创建如/home 等专用分区，根分区也并没用根据推荐进行创建。

```bash
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0    16G  0 disk
├─sda1   8:1    0   300M  0 part
├─sda2   8:2    0     4G  0 part
└─sda3   8:3    0  11.7G  0 part
sr0     11:0    1   1.1G  0 rom  /run/archiso/bootmnt
```
