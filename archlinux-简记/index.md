# Archlinux 简记


<!-- # Archlinux 简记 -->

## 1. 无图形安装

### 检查安装模式

检测是否为 `uefi 方式` 安装，若为 `uefi 方式` 则以下命令将输出一堆东西

```bash
ls /sys/firmware/efi/efivars
```

### 更改字体

默认 liveCD 字体可能过小

```bash
setfont /usr/local/kbd/consolefonts/LatGrkCyr-12x22.psfu.gz
```

### 禁用 reflector 服务

禁用 reflector 服务，以方便自定义源

```bash
systemctl stop reflector.service
```

### 设置镜像源

```bash
vim /etc/pacman.d/mirrorlist
```

生效的是**首行**，因此添加中科大或者清华的放在最上面

```text
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

### 连接网络

```bash
iwctl                           # 执行iwctl命令，进入交互式命令行
device list                     # 列出设备名，比如无线网卡看到叫 wlan0
station wlan0 scan              # 扫描网络
station wlan0 get-networks      # 列出网络 比如想连接YOUR-WIRELESS-NAME这个无线
station wlan0 connect YOUR-WIRELESS-NAME # 进行连接 输入密码即可
exit                            # 成功后exit退出
```

设置完成后测试

```bash
ping baidu.com
```

### 更新系统时钟

```bash
timedatectl set-ntp true        # 将系统时间与网络时间进行同步
timedatectl status              # 检查服务状态
```

### 分区（Btrfs）

这里文件采用 `Btrfs` 文件系统，分区根据个人情况

|/|/home|/efi|Swap|
|---|---|---|---|
| | |500M|>= 内存 x 60%|

由于使用 `Btrfs` 文件系统，这里 `/home` 与 `/` 将位于同一分区

```bash
lsblk                           # 显示当前分区情况
cfdisk /dev/sdx                 # 对安装 archlinux 的磁盘分区
fdisk -l                        # 分区完成后复查磁盘情况
```

### 格式化

```bash
mkfs.vfat  /dev/sdax            # 格式化 efi 分区
mkfs.btrfs -L myArch /dev/sdxn  # 格式化 / 与 /home 所在分区
mkswap /dev/sdxn                # 格式化 Swap 分区
```

<u>双系统中原本 Windows 已经存在 `efi` 分区的情况下，不需要再格式化了，后续挂载即可</u>

### 创建子卷

为创建子卷，先将 `/` 与 `/home` 所在分区挂载至 `/mnt`

```bash
mount -t btrfs -o compress=zstd /dev/sdxn /mnt
```

```bash
df -h # 查看挂载状态
```

```bash
btrfs subvolume create /mnt/@     # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
```

查看子卷情况

```bash
btrfs subvolume list -p /mnt
```

子卷创建完成后卸载

```bash
umount /mnt
```

### 挂载

<font color='red'>先挂根目录</font>

```bash
mount -t btrfs -o subvol=/@,compress=zstd /dev/sdxn /mnt # 挂载 / 目录
mkdir /mnt/home # 创建 /home 目录
mount -t btrfs -o subvol=/@home,compress=zstd /dev/sdxn /mnt/home # 挂载 /home 目录
mkdir -p /mnt/boot/efi # 创建 /boot/efi 目录
mount /dev/sdxn /mnt/boot/efi # 挂载 /boot/efi 目录
swapon /dev/sdxn # 挂载交换分区
```

查看挂载情况

```bash
df -h
free -h # 查看 Swap 情况
```

### 安装

```bash
pacstrap /mnt base base-devel linux linux-firmware
pacstrap /mnt dhcpcd iwd vim
```

### 生成 fstab 文件

```bash
genfstab -U /mnt > /mnt/etc/fstab
cat /mnt/etc/fstab # 复查一下 /mnt/etc/fstab 确保没有错误
```

### 切换系统

```bash
arch-chroot /mnt
```

### 主机与主机名

```bash
vim /etc/hostname           # 修改主机名
vim /etc/hosts              # 修改主机映射
```

```bash
# /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux
```

### 时间与时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #设置时区 
hwclock --systohc                                       # 硬件时间设置
```

### Locale

[wiki-archlinuxcn](https://wiki.archlinuxcn.org/wiki/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%E6%9C%AC%E5%9C%B0%E5%8C%96)
上不推荐直接修改 `/etc/locale.conf` 方式修改 locale

> 不推荐在 `/etc/locale.conf` 里把全局的 LANG locale 设置成中文 `LANG=zh_CN.UTF-8`，因为 TTY 下没有 CJK 字体，这样设置会导致 TTY 中显示豆腐块（除非你使用的内核打了 cjktty 补丁能绘制中文字体，比如linux-lilyCNRepo）。
>
> 每个用户单独的 locale 可以在 ~/.bashrc、~/.xinitrc 或 ~/.xprofile 中设置：  
>
> `.bashrc`：每次使用终端时会应用此处的设置。  
> `.xinitrc`：每次使用 startx 或 SLiM 来启动 X 窗口系统时会应用此处的设置。  
> `.xprofile`：每次使用 GDM 等显示管理器时会应用此处的设置。  

可以在用户环境变量下配置

```bash
# ~/.xinitrc 或 ~/.xprofile 中如下配置且放在 WM 启动前即可
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```

以下做法来自 <https://archlinuxstudio.github.io/ArchLinuxTutorial/#/rookie/basic_install?id=_12%e8%ae%be%e7%bd%ae-locale-%e8%bf%9b%e8%a1%8c%e6%9c%ac%e5%9c%b0%e5%8c%96>

编辑 `/etc/locale.gen`，去掉 `en_US.UTF-8 UTF-8` 以及 `zh_CN.UTF-8 UTF-8` 行前的注释符号 `#`

```bash
vim /etc/locale.gen                         # 去除相关注释
locale-gen                                  # 生成 locale

echo 'LANG=en_US.UTF-8'  > /etc/locale.conf # 向 /etc/locale.conf 导入内容
```

### 安装微码

```bash
pacman -S intel-ucode # Intel
pacman -S amd-ucode # AMD
```

### 引导

```bash
# 安装引导程序相应包
pacman -S grub efibootmgr os-prober 

# 安装 GRUB 到 EFI 分区
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ARCH

# 编辑 /etc/default/grub 文件
vim /etc/default/grub 
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT` 这一行，进行如下修改：

- 去掉这行中最后的 `quiet` 参数
- 把 `loglevel` 的数值从 `3` 改成 `5`。这样是为了后续如果出现系统错误，方便排错
- 加入 `nowatchdog` 参数，这可以显著提高开关机速度

添加新的一行 `GRUB_DISABLE_OS_PROBER=false`

修改后为：

```bash
# GRUB boot loader configuration

GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="Arch"
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=5 nowatchdog"
GRUB_CMDLINE_LINUX=""
GRUB_DISABLE_OS_PROBER=false
...
```

生成 GRUB 所需的配置文件,若引导了 Windows 将会有相应输出

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### 设置root 密码

```bash
passwd root
```

## 2. 基础配置

### 2.1 添加用户

添加用户，比如新增加的用户叫 testuser

```bash
useradd -m -G wheel -s /bin/bash uchin  #wheel附加组可sudo，以root用
```

户执行命令 -m同时创建用户家目录
设置新用户 testuser 的密码

```bash
passwd uchin
```

编辑 sudoers 配置文件

```bash
EDITOR=vim visudo  # 需要以 root 用户运行 visudo 命令
```

找到下面这样的一行，把前面的注释符号 # 去掉，:wq 保存并退出即可。

```bash
# %wheel ALL=(ALL) ALL
```

> 这里稍微解释一下 %wheel 代表是 wheel 组，百分号是前缀 ALL= 代表在所有主机上都生效(如果把同样的sudoers文件下发到了多个主机上) (ALL) 代表可以成为任意目标用户 ALL 代表可以执行任意命令 一个更详细的例子:

```bash
%mailadmin   snow,rain=(root) /usr/sbin/postfix, /usr/sbin/postsuper, /usr/bin/doveadm
nobody       ALL=(root) NOPASSWD: /usr/sbin/rndc reload
```

> 组 mailadmin 可以作为 root 用户，执行一些邮件服务器控制命令。可以在 "snow" 和 "rain"这两台主机上执行 用户 nobody 可以以 root 用户执行rndc reload命令。可以在所有主机上执行。同时可以不输入密码。(正常来说 sudo 都是要求输入调用方的密码的)

### 2.2 archlinuxcn 与 AUR

#### 2.2.1 archlinuxcn
>
> Arch Linux 中文社区仓库是由 Arch Linux 中文社区驱动的非官方软件仓库，包含许多官方仓库未提供的额外的软件包，以及已有软件的 git 版本等变种。一部分软件包的打包脚本来源于 AUR，但也有许多包与 AUR 不一样。

`sudo vim /etc/pacman.conf` 文件末尾添加以下两行

```text
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

更新索引、系统

```bash
sudo pacman -Syyu --noconfirm
```

安装 `archlinuxcn-keyring`

```bash
sudo pacman -S --noconfirm archlinuxcn-keyring
```

#### 2.2.2 AUR

pura 安装

archlinuxcn 中提供了二进制包，可直接安装

```bash
sudo pacman -S paru 
```

## 3. 桌面（x11）

```bash
sudo pacman -S xorg
sudo pacman -S xorg-server
```

### 3.1 DWM

#### 3.1.1 基础安装与启动

没联网先联网，同前文使用 iwd 方式连接，没开服务先开服务

```bash
systemctl enable dhcpcd         # 开机自启 dhcp
systemctl start iwd.service     # 开启 iwd 服务
iwctl                           # 联网
···
```

若没安装 `xorg` 组件，先安装

```bash
sudo pacman -S xorg-server
sudo pacman -S xorg-apps
sudo pacman -S xorg-xinit
```

中文字体未安装先安装

```bash
sudo pacman -S noto-fonts-cjk  #安装中日韩字体，避免不能正常显示
```

下载所需要的包

```bash
git clone https://git.suckless.org/dwm
```

安装

```bash
sudo make clean install
```

修改 `~/.xinitrc`

```bash
# ~/.xinitrc
exec dwm
```

启动

```bash
startx
```

#### 3.1.2 打 patch

todo

### 3.2 Awesome

#### 3.2.1 安装

安装awesome

```bash
pacman -S awesome
```

安装字体

```bash
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei                   #安装几个开源中文字体 一般装上文泉驿就能解决大多wine应用中文方块的问题
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra 
```

#### 3.2.2 配置

todo

<center> ----- ref ----- </center>

---

安装

1. [Arch Linux 安装使用教程](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)
2. [archlinux 简明指南](https://arch.icekylin.online/)

---

DWM

1. [suckless.org](https://suckless.org/)
2. [yaocccc-bilibli](https://www.bilibili.com/video/BV1Ef4y1Z7kA)
3. [Arch Linux 美化 (st + dwm)](https://blog.csdn.net/weixin_44335269/article/details/117930190)
4. [DWM安装及简略配置教程](https://blog.csdn.net/qq_36390239/article/details/112987099)
5. [yaocccc-github](https://github.com/yaocccc/dwm)
6. [在Arch上使用Fcitx5](https://www.programminghunter.com/article/4737964538/)
[TheCW-bilibili](https://space.bilibili.com/13081489/search/video?keyword=arch)
7. [theniceboy的脚本](https://github.com/theniceboy/scripts)

---

输入法

1. [Arch系安装并配置fcitx5输入法](https://www.bilibili.com/read/cv7476615)
2. [wiki](https://wiki.archlinuxcn.org/wiki/Fcitx5#%E9%85%8D%E7%BD%AE%E5%B7%A5%E5%85%B7)
3. [来自blibili的配置](https://www.bilibili.com/read/cv7476615)
4. <https://manateelazycat.github.io/linux/2020/06/19/fcitx5-is-awesome.html>

---

终端

1. [流行终端对比](https://www.v2ex.com/t/900640)

2. [Kitty Terminal 终端](https://josephpei.github.io/2022/04/18/kitty-terminal-%E7%BB%88%E7%AB%AF/)

3. [kitty文档](https://sw.kovidgoyal.net/kitty/conf/#fonts)

---

rofi

1. [Rofi 配置](https://www.cnblogs.com/siyingcheng/p/11706215.html)

---

其他

1. [archlinux 修复uefi引导启动](http://ivo-wang.github.io/2018/05/29/archlinux-%E4%BF%AE%E5%A4%8Duefi%E5%BC%95%E5%AF%BC%E5%90%AF%E5%8A%A8/)

