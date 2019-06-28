# Windows 下 Docker 与 VMware 共存

[^_^]: # (url:windows-docker-in-vmware)
[^_^]: # (tag:docker,windows,vmware,#tech)
[^_^]: # (excerpt: make docker and vmware both running in windows)

> 本文介绍一种使得 Windows 下 Docker 与 VMware 软件同时可用的方法.

TL;DR

## 问题在哪里

### Hypervisor 以及其分类 Type-1 和 Type-2

#### 何为 Hypervisor

`Hypervisor`又称`Virtual Machine Monitor(VMM)`是用于创建和运行虚拟机(VM)的计算机软件,固件或硬件.承载`Hypervisor`和虚拟机的计算机称为宿主机(`Host Machine`),运行于宿主机上的虚拟机称为客户机(`Guest Machine`). `Hypervisor`为客体操作系统提供虚拟的作业平台并管理客体操作系统,使得不同操作系统的众多实例可以共享虚拟的硬件资源.

可以简单地理解为 Hypervisor 为虚拟机的运行提供了软件层面的基础.

Hypervisor 通常分为两类, Type-1 和 Type-2.

#### 何为 Type-1 Hypervisors

Type-1 Hypervisors, 又称原生(native)hypervisors 或裸机(bare-metal)hypervisors. 这一类型的 hypervisor 直接运行在硬件层面上来管理虚拟机.因此,有时被称为裸机 hypervisor.

像`Xen`,`VMware ESX/ESXi`,`Microsoft Hyper-V`, `Oracle VM Server for x86`都是 Type-1 Hypervisor.

#### 何为 Type-2 Hypervisors

Type-2 Hypervisors, 有时也叫 Hosted Hypervisor. 这类 Hypervisor 像其他应用程序一样运行在常规的操作系统中.一个客户机作为一个进程运行在宿主机上.Type-2 Hypervisor 为客体操作系统提供抽象层.

像`VMWare WorkStation`,`Virtual Box`,`Parallels Desktop for Mac`和`QEMU`都是 Type-2 Hypervisor.

#### Type-1 和 Type-2 界线并不总是清晰的

从前面的概念来看,貌似 Type-1 和 Type-2 很好区分, 但有时候并不是那么简单的.例如, Linux 的 KVM(Kernel-based Virtual Machine)和 FreeBSD 的 bhyve 都是内核模块,它们可以高效地将系统变成 Type-1 Hypervisor; 与此同时,KVM 或 bhyve 都要和其他应用程序竞争资源,从这一角度出发它们又属于 Type-2 Hypervisor.

### 冲突是什么

当前 Docker 官方出品的 Windows 客户端, 叫做`Docker Desktop for Windows`.而其正常运行的条件之一是系统开启了 Hyper-V 虚拟化服务. 由上文知 Hyper-V 是 Type-1 的 Hypervisor, 这将使得像 VMware 等作为 Type-2 Hypervisor 的软件无法运行.

此时矛盾已经出现: 使用 Hyper-V 技术的 Docker 客户端与其他 Type-2 Hypervisor 不能同时运行, 必须重启并关闭 Hyper-V 才能再次运行其他的 Type-2 Hypervisor 软件.

目前笔者见过的折中的解决方法如下:

1. 不在 Windows 环境下使用 Docker, 完全到 Linux 虚拟机里使用 Docker
2. 设置开机策略, 进行 Hyper-V 服务的切换, 达到重启切换的效果(只使用 Hyper-V 和 Docker / 只使用 VMware)

但以上两种方法都有一些很缺憾的地方:

- 可能无法在 Windows 环境的命令行直接使用 Docker 命令
- 可能需要重启来切换, 耗费时间

既然知道了冲突所在,那么解决的思路自然是想办法让两者兼容 ,亦或者是舍弃其中一方.

让两者兼容的可行性太低, 虽然 Hyper-V 也能创建虚拟机, 但易用性不如 VMware 或 VBox 好, 所以选择保留 VMware, 放弃使用 Hyper-V.

让两者兼容 ,亦或者是舍弃其中一方.

那么我们先来了解一下 Docker 的软件架构和官方的 Windows 客户端是如何工作的.

### Docker 的软件架构

Docker 是 CS 架构的软件, 而采用 CS 架构的软件大都有一个优点, C 端的软件可以很轻量.

对 Docker 历史了解的同学都知道, 以前 Docker 的 Windows 客户端并不需要 Hyper-V, 而是靠 VirtualBox 提供 Linux 的环境.而现在的 Docker 客户端是使用 Hyper-V 创建一个 Linuxkit 虚拟机来提供 Linux 和 Docker 服务.其依然是靠虚拟机来支撑 Docker 的服务端.

## 思路

1. 在 Windows 上仅保留 Docker 的客户端
2. 在 VMware 的虚拟机里运行 Docker 服务端
3. 要使用 Docker 时打开虚拟机提供服务, 也可以设置虚拟机开机自启

恰好 docker 和 docker-compose 在 Windows 上都有成熟的客户端
所以,将 docker 服务运行于 VMware 的虚拟机里, 在 Windows 上使用 docker 的 cli 与 vm 里的 docker 服务通信就可以了
方法很简单, 但是单纯这样做会有几点问题:

1. docker 运行在了 vm 里,使用的是 vm 的端口, docker 运行的所有服务,都只能通过 vm 的 IP 去访问
2. Windows 与 Linux 文件系统的不同,导致无法直接卷映射

为了解决上面两点不足, 用到了 wsl 和 hosts 文件:

把 vm 的 ip 绑定自定义的域名到 hosts 文件里, 以后通过域名即可访问 vm

启用 wsl 功能,安装 Ubuntu18.04LTS, 安装 docker-cli, 设置环境变量

把所有盘设置为虚拟机共享, 建立软连接,这样 vm 里的看到的 windows 文件路径与 wsl 里看到的 windows 文件路径是相同的.

## 实施

### 安装虚拟机

在 VMware 里安装一个 Linux 虚拟机, 这里选用 Ubuntu18.04LTS server 版.安装过程无特殊要点,在此不赘述.系统安装完成后,进行下列工作:

1. 更改软件源,建议清华源或者科大源

2. 安装 Docker 与 Docker Compose

   ```bash
   # install docker
   curl -fsSL https://get.docker.com -o get-docker.sh
   sh get-docker.sh
   # install docker-compose
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

3. 修改 Docker 配置,使其能接受远程连接

   将当前用户添加进 docker 组, 并将 docker 设置为服务

   ```bash
   # config group
   sudo groupadd docker
   sudo usermod -aG docker $USER
   # enable service
   sudo systemctl enable docker
   ```

   设置服务选项,使其监听端口

   ```bash
   sudo vi /lib/systemd/system/docker.service
   ```

   ![tcp-2375](..\img-store\post\windows-docker-in-vmware\tcp-2375.png)

   重启服务

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

4. 设置文件共享, 笔者即将整个 D 盘共享给虚拟机(为了后面的 Docker 卷映射功能)

   ![shared-folders](..\img-store\post\windows-docker-in-vmware\shared-folders.png)

5. 为共享文件夹建立软链接

   wsl 是将 windows 挂载到`/mnt`来访问的, 为了使虚拟机视角的 windows 文件路径与 wsl 视角的 windows 文件路径一致, 需要建立一个软链接.

   在虚拟机里运行命令`sudo ln -s /mnt/hgfs/d /mnt/d`

   通过这个软连接, 就统一了虚拟机和 wsl 对待 windows 文件的路径

### 安装 WSL

1. 在 Windows Features 中打开 Windows Subsystem fro Linux

   ![windows-features](..\img-store\post\windows-docker-in-vmware\windows-features.png)

2. 到微软商店里搜索 Ubuntu 并安装 Ubuntu18.04LTS

   ![windows-store-ubuntu18lts](..\img-store\post\windows-docker-in-vmware\windows-store-ubuntu18lts.png)

3. 在开始菜单打开 Ubuntu 18.04 LTS, 等其安装完, 填入用户名和密码,建议设置成与虚拟机相同的用户名和密码

4. 进入 WSL 之后, 更新源, 安装 docker-cli 和 docker-compose

   更新源

   ```bash
   # backup
   sudo cp /etc/apt/sources.list /etc/apt/sources.list.bac
   # change software sources
   sudo vi /etc/apt/sources.list

   sudo apt update
   sudo apt upgrade
   ```

   安装 docker-cli 和 docker-compose

   ```bash
   # pre install step
   sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       gnupg-agent \
       software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
   sudo apt-get update

   # install docker-cli
   sudo apt-get install docker-ce-cli

   # install docker-compose
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

5. 编辑`~/.profile`添加环境变量

   ```bash
   sudo vi ~/.profile
   ```

   在文件末尾添加`export DOCKER_HOST="tcp://local.test:2375"`

   加载配置

   ```bash
   source ~/.profile
   ```

### Windows 设置

1. 下载 Docker 客户端和 docker-compose

   到https://download.docker.com/win/static/stable/x86_64/下载最新版本的二进制应用,将其解压到

   `C:\Program File\Docker`里

   到https://github.com/docker/compose/releases下载最新版本的docker-compose,将其也解压进`C:\Program File\Docker`里

2. 配置 hosts 文件

   修改`C:\Windows\System32\drivers\etc\hosts`, 添加一条记录(把其中 ip 替换为虚拟机的 ip):

   ```bash
   192.168.130.2		local.test
   ```

3. 配置环境变量

   在 Path 中增加一条记录, 指向`C:\Program File\Docker`

   增加一条记录`DOCKER_HOST`, 值为`tcp://local.test:2375`

## 效果

在 cmd 或者 powershell 以及 wsl 环境下, 皆可直接使用 docker 命令和 docker-compose 命令,如图:

![cmd](..\img-store\post\windows-docker-in-vmware\cmd.png)
![powershell](..\img-store\post\windows-docker-in-vmware\powershell.png)
![wsl](..\img-store\post\windows-docker-in-vmware\wsl.png)

### 端口映射

首先开启一个 nginx 映射 80 端口

```bash
docker run -itd -p 80:80 nginx:alpine
```

![port-map](..\img-store\post\windows-docker-in-vmware\port-map.png)

在局域网中用 local.test 域名访问,可以看到 nginx 确实运行成功了

![port-map-browser](..\img-store\post\windows-docker-in-vmware\port-map-browser.png)

### 卷映射

1. 确保虚拟机中的卷分享有效
2. 必须从 wsl 中运行 docker 或 docker-compose 命令

例如, 笔者想要映射`D:\projects\ingsoho\dist`到 nginx 容器中:

首先打开 wsl, 切换到该目录下,运行 docker 命令

```bash
docker run -itd -p 88:80 -v $PWD:/usr/share/nginx/html nginx:alpine
```

![wsl-volume-map](..\img-store\post\windows-docker-in-vmware\wsl-volume-map.png)

在局域网访问`local.test:88`

![volume-map-browser](..\img-store\post\windows-docker-in-vmware\volume-map-browser.png)

### 完善

1. 如果虚拟机重启, 则设置的文件共享会失效,需要重新设置

   不推荐重启虚拟机, 当电脑关机时系统回自动挂起虚拟机, 所以配合下面的自启脚本,基本可以不用管理虚拟机的启动问题.

   但总会遇到要重启虚拟机的情况, 在这里提供一个重新设置文件共享的 bat 脚本(请根据自身情况修改路径信息)

   ```bat
   "C:\Program Files (x86)\VMware\VMware Workstation\vmrun.exe" disableSharedFolders "D:\vmware\dev\dev.vmx"

   "C:\Program Files (x86)\VMware\VMware Workstation\vmrun.exe" enableSharedFolders "D:\vmware\dev\dev.vmx"
   ```

   再顺带提供一个后台启动虚拟机的脚本(将脚本加入到开机自启即可)

   ```bat
   "C:\Program Files (x86)\VMware\VMware Workstation\vmrun.exe" start "D:\vmware\dev\dev.vmx" nogui
   ```

2. 卷映射需要 wsl 环境来操作,才能匹配文件路径

至此, 大部分的问题被完美解决, 小部分问题无伤大雅.

如果您发现有其他的问题, 欢迎留言探讨.
