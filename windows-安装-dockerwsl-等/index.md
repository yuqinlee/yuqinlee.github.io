# Windows 安装 Docker、wsl 等

<!-- # Windows 安装 docker、wsl 等 -->

## 安装 WSL——Archlinux

控制面板找到 `启用或关闭windows功能` 勾选 `Hyper-V` 和 `适用于 Linux 的 Windows 子系统`（只用勾选后者实际就能WSL安装，如果需要使用原来的 Hyper-V 方式安装 Docker 才需要开启前者，这里直接都打开了），重启系统，执行 `wsl --update` 更新

安装 Archlinux 作为子系统

1. 从 `https://github.com/yuk7/ArchWSL/releases` 下载压缩包
2. 解压到指定目录
3. 双击解压好的 Arch.exe 进行安装，这个 .exe 的名字 就是要创建的 WSL 实例的名字，改不同的名字就能创建多个 Arch WSL。

常用命令

```bash
wsl --list # 查看当前 wsl 中有的子系统
wsl --update # 下载或者更新
wsl --shutdown # 重新启动
wsl -d  Arch # 进入名为 Arch 的子系统
```

官网文档：<https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands>

配置用户

```bash
passwd root

useradd -m -G wheel -s /bin/bash uchin
passwd uchin
```

退出ArchLinux，进入刚刚安装ArchLinux的目录（例如D:\vm\archlinux），将默认用户改为非root用户：

```bash
exit # 退出archlinux，之后你会回到Windows
cd D:\WSL\Arch
.\Arch.exe config --default-user uchin
```

配置hostname

参考：[WSL设置hostname，不修改Windows主机名](https://www.cnblogs.com/stellae/p/14969599.html)

默认的 `/etc/ws.conf` 如下

```bash
[uchin@DESKTOP-UCHIN ~]$ cat /etc/wsl.conf
[boot]
systemd=true

[automount]
enabled = true
options = "metadata"
mountFsTab = true
```

该文件详细配置参考文档：<https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config>

```bash
[uchin@DESKTOP-UCHIN ~]$ cat /etc/wsl.conf
[boot]
systemd=true

[automount]
enabled = true
options = "metadata"
mountFsTab = true

# 设置 wsl 自己的 hostname
[network]
generateResolvConf = true
hostname = uchin-arch

# 禁用 windows 环境变量
[interop]
enabled = false
appendWindowsPath = false

```

## 安装 docker

### 安装

这里使用 WSL2 的方式安装 Windows Docker

WSL 对比 Hyper-V 体验更好，可以不考虑原来 Hyper-V 的方式了

运行 `wsl --update`，如果之前没更新会更新WSL，然后下载 Windows Docker

<https://www.docker.com/products/docker-desktop/>

安装时勾选 WSL2 安装即可安装完成

## 修改镜像位置

默认安装 docker 后镜像存放路径为 `C:\Users\uchin\AppData\Local\Docker\wsl` 当镜像过多后会大量占用 C 盘

> Docker Desktop WSL2 默认会安装2个子系统，使用命令wsl -l -v --all查看
>
> ```bash
> C:\Users\uchin>wsl -l -v --all
>   NAME                   STATE           VERSION
> * Arch                   Running         2
>   docker-desktop-data    Running         2
>   docker-desktop         Running         2
> ```
>
> 两个子系统分别为 `docker-desktop-data` 与 `docker-desktop`

`docker-desktop-data` 与 `docker-desktop` 前者存放数据，后者存放程序

```bash
# 1. 退出Docker Desktop

# 2. 关闭 WSL
wsl --shutdown

# 3. 将子系统导出为 tar 文件
wsl --export docker-desktop D:\WSL\docker\docker-desktop.tar
wsl --export docker-desktop-data D:\WSL\docker\docker-desktop-data.tar

# 4. 注销子系统
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data

# 5. 使用新路径导入子系统
wsl --import docker-desktop D:\WSL\docker\docker-desktop D:\WSL\docker\docker-desktop.tar --version 2
wsl --import docker-desktop-data D:\WSL\docker\docker-desktop-data D:\WSL\docker\docker-desktop-data.tar --version 2
```

---

*参考：*

- [√ Arch WSL 安装 1](https://twor.me/posts/wsl_archlinux_gui/)
- [√ Arch WSL 安装 2](https://zhuanlan.zhihu.com/p/613454594)
- [× Arch WSL appx 安装方式](https://moe23333.vercel.app/posts/install-arch-wsl2)
- [√ windows docker 安装](https://cloud.tencent.com/developer/article/2208269)
- [√ Docker Desktop(WSL2)修改镜像存储位置](https://blog.csdn.net/daihaoxin/article/details/116104599)
- [√ ls 命令输出的颜色](https://linux.cn/article-16108-1.html)
- [√ 系统配置bash shell字体颜色](https://www.cnblogs.com/JavicxhloWong/p/14541469.html)
- [Linux PS1 的常用配置](https://www.cnblogs.com/enwaiax/p/15767346.html)
- [√ 通过 VcXsrv 在 WSL2 上使用图形化界面](https://www.cnblogs.com/KylinBlog/p/16588037.html)
- [√ 使用 WSL2 + X11 转发 - 在 Windows10 中打造 GNU/Linux 学习生产环境](https://steinslab.io/archives/2082)

