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

完成磁盘分区后，记得要为你新分区的磁盘进行格式化。你可以将磁盘理解为地皮，分区是在地皮上面划分地基，而格式化就是在地基上面盖仓库，而写入文件就是往仓库里搬东西。  

格式化磁盘可以使用mkfs命令，先前我们创建了3个分区，其中sda1要作为启动分区，sda2作为交换分区，sad3则作为我们的系统根分区。  
这里我们需要用到命令均在下面列出

```bash
    mkfs.fat -F 32
    mkswap
    swapon
    mkfs.ext4
```

mkfs.fat是用于给引导分区格式化使用的命令，-F 32是用于创建FAT32文件系统的参数，对于BIOS来说FAT文件系统是目前且唯一能读取的文件系统，如果格式化使用了其他文件系统进行格式化则有可能会导致操作系统无法引导或其他不可预料的问题  
mkswap是将分区设定为交换分区，swapon则是启用交换分区  
mkfs.ext4将分区格式化为Linux系统常用文件系统，ext文件系统目前有ext2 ext3和ext4，对于较新的Linux操作系统来说，使用ext4则会有相对较好的性能，如果想要使用其他文件系统如btfs、xfs等还需要其他配置，这里则使用ext4

```text
    root@archiso ~ # mkfs.fat -F 32 /dev/sda1
    mkfs.fat 4.2 (2021-01-31)
    root@archiso ~ # mkswap /dev/sda2
    Setting up swapspace uersion 1, size = 4 GiB(4294963200 bytes)
    nolabel, UUID=a43c3479-7f45-49f7-9e10-Zaaf116c709b
    
    root@archiso ~ # swapon /dev/sda2
    
    root@archiso ~ # mkswap /dev/sda3
    Creating filesystem with 4116992 4k blocks and 1030176 inodes
    Filesystem UUID:583a2157-11ec-4460-952a-ff4c798a80cZ
    Superblock backups stored on blocks:
        32768，98304，163840，229376，294912,819200，884736，1605632，2654208,
        4096000
    Allocating group tables : done
    Writinginode tables : done
    Creating .journal (16384 blocks) :done
    Writing superblocks and filesystem accounting information: done
    
    root@archiso ~ #
```

通过上述操作，已经完成了磁盘格式化及交换分区启用，接下来需要进行挂载磁盘为安装系统做准备

### 挂载分区

挂载分区前首先我们需要知道我们需要挂载的分区的具体位置，可以通过lsblk命令进行查看

```bash
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    sda      8:0    0    16G  0 disk
    ├─sda1   8:1    0   300M  0 part
    ├─sda2   8:2    0     4G  0 part [SWAP]
    └─sda3   8:3    0  11.7G  0 part
    sr0     11:0    1   1.1G  0 rom  /run/archiso/bootmnt
```

这里我们需要挂载sda1和sda3，其中sda1是我们的引导分区，需要挂载到"/boot"下面，sda3是我们的根分区需要挂载到"/"下面去，但由于我们在安装镜像提供的基础系统里，所以这里我们需要将sda1挂载到"/mnt/boot"而sda3挂载到"/mnt"下  
这里我们使用"mount"命令进行挂载，挂载到不存在的位置时可以通过携带"--mkdir"参数来创建并挂载到指定目录

![mount](../Images/mount.png)

由于Arch Linux并不会在安装镜像中提供任何安装包，所以接下来我们需要配置安装系统的镜像源来加快我们的下载速度

### 配置镜像源

Arch Linux使用pacman作为包管理器，对应配置文件在"/etc/pacman.conf"和"/etc/pacman.d/"当中，修改镜像源需要编辑位于"/etc/pacman.d/mirrorlist"文件，默认会提供几个镜像源，为了保证速度我们这边会将其都禁用并单独写入一个新的镜像源

您可以手动配置镜像源到上面的任意一个镜像服务器，也可以在文件中取消注释掉离你最近或者速度最快的服务器。  
您可以使用下列任意一个命令对配置文件进行编辑，对于初次尝试使用Linux的用户来说建议使用nano作为你的文本编辑器，如果你有使用Linux的经验，你可以选择你常用的文本编辑器进行编辑。

```bash
    #你可以选择任意一个适合你的文本编辑器对配置文件进行编辑
    #vim/vi文本编辑器为纯键盘操作编辑器，具有一定上手难度
    vim /etc/pacman.d/mirrors
    #nano文本编辑器
    #nano拥有着相对现代化的操作逻辑和快捷键，时候新手使用
    nano /etc/pacman.d/mirrors
```

![mirrors](../Images/mirrors.png)

这里我们使用中国科学技术大学作为我们的镜像源，完成配置后我们还需要进行一次系统更新以获取最新数据和安装包数据库，执行下列命令即可自动更新

```bash
    pacman -Syy
```

完成这些操作后就可以进行基础系统安装

### 安装基础系统

### 安装引导

### 配置基础环境

### 配置用户及sudo

## 后续配置及桌面环境
