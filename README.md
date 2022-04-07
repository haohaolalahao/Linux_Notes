# 准备与安装工作

## Ubuntu installer

**lts 长期支持版本：**

- [ubuntu 20.04](https://ubuntu.com/download/desktop)
- [ubuntu 22.04](https://releases.ubuntu.com/jammy/) Comming Soon

**boot:**

- [ventoy 启动盘](https://www.ventoy.net/cn/index.html)

## 分区建议

以 2t 系统盘为例：

|      盘       |      大小      |      detail       |
| :-----------: | :------------: | :---------------: |
| efi 系统分区  |   1gb/1024mb   |   空间起始位置    |
|   grub 分区   |   2gb/2048mb   |   空间起始位置    |
| boot 系统分区 |   2gb/2048mb   |   空间起始位置    |
| swap 交换分区 |  16gb/16384mb  |   空间起始位置    |
|       /       | 512gb/524288mb | 空间起始位置/ext4 |
|     /home     |     other      | 空间起始位置/ext4 |

_GPT 分区方式不需要细分主分区或是逻辑分区_

## 磁盘挂载与分区

1. 查看硬盘信息

   ```bash
   # 硬盘信息
   sudo fdisk -l
   # 挂载信息
   sudo df -hT
   # 查看系统可用块设备以及依赖关系
   sudo lsblk
   # 分区uuid
   sudo blkid
   ```

2. 使用 fdisk 分区

   ```bash
   # 参考 https://www.myfreax.com/fdisk-command-in-linux/#menu
   # 进入 fdisk
   sudo fdisk /dev/sdb
   # 列出命令列表
   m
   # 创建全新空GPT分区表
   g
   # 创建新分区
   n
   # 分区大小使用+，大小可以为K/M/G/T/P
   +100G
   # 显示分区表
   p
   # 删除分区表
   d
   # 保存分区表
   w
   ```

3. 使用 parted 分区

   ```bash
   # 进入 parted
   sudo parted /dev/sda

   # 设置为GPT格式
   mklabel gpt
   # mkpart [分区类型：primary/logical/extend] 起始点 结束点
   # 划分成一个主分区, 默认起始位置2048
   mkpart primary 2 -1
   # 创建一个10G主分区
   mkpart primary 2 10GB

   # 格式化
   sudo mkfs.ext4 -F /dev/sda1
   sudo mkfs.xfs -F /dev/sda1
   ```

4. 临时挂载

   ```bash
   sudo mount <device> <target>
   sudo unmount <device>
   ```

5. /etc/fstab 开机自动挂载分区

   ```bash
   # 写入信息或直接修改
   sudo echo "<device> <target> ext4 defaults,x-gvfs-show 0 0" >> /etc/fstab
   # <device> 第一种
   /dev/sda1
   # <device> 第二种
   UUID=UUID_ID
   # <device> 第三种
   /dev/disk/by-uuid/UUID_ID
   # 修改后自动挂载
   mount -a
   ```

   ```bash
   /etc/fstab文件主要包括6段，依次是：
   <file system>　　<dir>　　<type>　　<options>　　<dump>　　<pass>
   <file system> 要挂载的分区或存储设备
   <dir>  挂载的目录位置
   <type> 挂载分区的文件系统类型，比如：ext3、ext4、xfs、swap
   <options> 挂载使用的参数有哪些。举例如下：
   	auto - 在启动时或键入了 mount -a 命令时自动挂载。
   	noauto - 只在你的命令下被挂载。
   	exec - 允许执行此分区的二进制文件。
   	noexec - 不允许执行此文件系统上的二进制文件。
   	ro - 以只读模式挂载文件系统。
   	rw - 以读写模式挂载文件系统。
   	user - 允许任意用户挂载此文件系统，若无显示定义，隐含启用 noexec, nosuid, nodev 参数。
   	users - 允许所有 users 组中的用户挂载文件系统.
   	nouser - 只能被 root 挂载。
   	owner - 允许设备所有者挂载.
   	sync - I/O 同步进行。
   	async - I/O 异步进行。
   	dev - 解析文件系统上的块特殊设备。
   	nodev - 不解析文件系统上的块特殊设备。
   	suid - 允许 suid 操作和设定 sgid 位。这一参数通常用于一些特殊任务，使一般用户运行程序时临时提升权限。
   	nosuid - 禁止 suid 操作和设定 sgid 位。
   	noatime - 不更新文件系统上 inode 访问记录，可以提升性能。
   	nodiratime - 不更新文件系统上的目录 inode 访问记录，可以提升性能(参见 atime 参数)。
   	relatime - 实时更新 inode access 记录。只有在记录中的访问时间早于当前访问才会被更新。（与 noatime 相似，但不会打断如 	mutt 或其它程序探测文件在上次访问后是否被修改的进程。），可以提升性能。
   	flush - vfat 的选项，更频繁的刷新数据，复制对话框或进度条在全部数据都写入后才消失。
   	defaults - 使用文件系统的默认挂载参数，例如 ext4 的默认参数为:rw, suid, dev, exec, auto, nouser, async.
   	x-gvfs-show - 与挂载后GUI显示有关
   <dump>  dump 工具通过它决定何时作备份. dump 会检查其内容，并用数字来决定是否对这个文件系统进行备份。 允许的数字是 0 和 1 。0 表示忽略， 1 则进行备份。大部分的用户是没有安装 dump 的 ，对他们而言 <dump> 应设为 0。

   <pass> fsck 读取 <pass> 的数值来决定需要检查的文件系统的检查顺序。允许的数字是0, 1, 和2。 根目录应当获得最高的优先权 1, 其它所有需要被检查的设备设置为 2. 0 表示设备不会被 fsck 所检查。
   ```

## NVIDIA Driver + CUDA + CUDNN

1. NVIDIA 驱动

   [参考](https://www.linuxcapable.com/zh-CN/how-to-install-nvidia-drivers-on-ubuntu-22-04-lts/)

   ```bash
   # 显示驱动信息
   ubuntu-drivers devices
   # 安装指定版本
   sudo apt install nvidia-driver-510
   # 重启
   sudo reboot
   # 查看GPU状态
   nvidia-smi
   ```

2. NVIDIA CUDA

   [CUDA_Ubuntu_x86_64_20.04](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_network)

   ```bash
   # deb network 下载
   wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
   sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
   sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
   sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
   sudo apt-get update
   sudo apt-get -y install cuda

   # path 路径
   export $path='/usr/local/cuda':$path
   ```

3. NVIDIA CUDNN

   - [CUDNN Archive](https://developer.nvidia.com/rdp/cudnn-archive)

   - [Documentation](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)

   tar 安装方法

   ```bash
   tar -xvf cudnn-linux-x86_64-8.x.x.x_cudaX.Y-archive.tar.xz
   sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda/include
   sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda/lib64
   sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
   ```

   network 安装方法

   ```bash
   wget https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/cuda-${OS}.pin

   sudo mv cuda-${OS}.pin /etc/apt/preferences.d/cuda-repository-pin-600
   sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/7fa2af80.pub
   sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/${OS}/x86_64/ /"
   sudo apt-get update

   sudo apt-get install libcudnn8=${cudnn_version}-1+${cuda_version}
   sudo apt-get install libcudnn8-dev=${cudnn_version}-1+${cuda_version}

   # ${cudnn_version} is 8.4.0.*
   # ${cuda_version} is cuda10.2 or cuda11.6\
   # network 的方法可能需要等到8.4.0出来，还没有验证
   ```

# Package

## visudo

```bash
sudo visudo
<username> ALL=(ALL) NOPASSWD:ALL
```

## apt 换源

```bash
# for ubuntu20.04
sudo mv /etc/apt/sources.list /etc/apt/source.list.backup
sudo vim /etc/apt/sources.list

# 华为源
deb http://repo.huaweicloud.com/ubuntu/ focal main restricted universe multiverse
deb-src http://repo.huaweicloud.com/ubuntu/ focal main restricted universe multiverse
deb http://repo.huaweicloud.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://repo.huaweicloud.com/ubuntu/ focal-security main restricted universe multiverse
deb http://repo.huaweicloud.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://repo.huaweicloud.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://repo.huaweicloud.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://repo.huaweicloud.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://repo.huaweicloud.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://repo.huaweicloud.com/ubuntu/ focal-backports main restricted universe multiverse

# 阿里云源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

# update&&upgrade&&clean
sudo apt update && sudo apt upgrade -y && sudo apt autoremove $$ sudo apt autoclean
```

## Useful Package & Tools

### Shell

```bash
sudo apt-get install zsh
```

### Git

```bash
sudo apt-get install git
```

### Compress

```bash
sudo apt-get install tar gzip
```

### Net

```bash
sudo apt-get install net-tools
# find ip
ifconfig
```

### file/http/https

```bash
sudo apt-get install curl wget httpie rsync
```

### Searching

```bash
sudo apt-get install fzf gawk ripgrep autojump
```

### Terminal multiplexer

```bash
sudo apt-get install tmux
```

### Query

```bash
sudo apt-get install tldr thefuck
```

### Development tools

```bash
sudo apt isntall build-essential cmake libboost-all-dev
```

### Top

```bash
# htop
sudo apt install htop
# btop
sudo snap install btop --edge
# nvitop
sudo pip install nvitop
# gpustat
sudo apt install gpustat
```

### SSH

```bash
sudo apt-get install openssh-server -y
sudo systemctl enable --now ssh
# change this for security
sudo vim /etc/ssh/sshd_config
# 禁用root登录
PermitRootLogin no
# 空闲时间间隔
ClientAliveInterval 300
# 禁用x11
X11Forwarding no
```

### fail2ban

[reference](https://linuxhandbook.com/fail2ban-basic/)

```bash
sudo apt isntall fail2ban
sudo systemctl enable --now fail2ban
# use
sudo fail2ban-client status
sudo fail2ban-client stautus sshd
```

### smartmontools

```bash
# nvme状态
sudo apt install smartmontools
# /dev/* 具体名称请运行 sudo fdisk -l 查看
sudo smartctl --all /dev/nvme0n1

```

### New Tools

#### fd -> find

```bash
# https://github.com/sharkdp/fd/releases
# 下载前查阅新版本
wget https://github.com/sharkdp/fd/releases/download/v8.3.2/fd-musl_8.3.2_amd64.deb
sudo dpkg -i fd-musl_8.3.2_amd64.deb
```

#### bat -> cat

```bash
# https://github.com/sharkdp/bat/releases
# 下载前查阅新版本
wget https://github.com/sharkdp/bat/releases/download/v0.20.0/bat-musl_0.20.0_amd64.deb
sudo dpkg -i bat-musl_0.20.0_amd64.deb
```

#### exa -> ls

```bash
# https://github.com/ogham/exa/releases
# 下载前查阅新版本
wget https://github.com/ogham/exa/releases/download/v0.10.1/exa-linux-x86_64-musl-v0.10.1.zip
sudo unzip -q exa-linux-x86_64-musl-v0.10.1.zip bin/exa -d /usr/local
```

#### gdu -> du

```bash
sudo add-apt-repository ppa:daniel-milde/gdu
sudo apt-get update
sudo apt-get install gdu
```

#### duf -> du

```bash
# https://github.com/muesli/duf/releases
# 下载前查阅新版本
wget https://github.com/muesli/duf/releases/download/v0.8.1/duf_0.8.1_linux_amd64.deb
sudo dpkg -i duf_0.8.1_linux_amd64.deb
```

## Neovim

```bash
# for ubuntu neovim
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt-get update
sudo apt-get install neovim
```

## Proxy

1. [原版 Clash](https://github.com/Dreamacro/clash)

2. [ShellClash](https://github.com/juewuy/ShellClash)

   - curl

     ```bash
     #ghproxy.com加速
     export url='https://ghproxy.com/https://raw.githubusercontent.com/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     #github-CDN源
     export url='https://raw.githubusercontent.com/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     #jsdelivrCDN源
     export url='https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
     ```

   - wget

     ```bash
     #jsdelivrCDN源
     export url='https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master' && wget -q --no-check-certificate -O /tmp/install.sh $url/install.sh  && sh /tmp/install.sh && source /etc/profile &> /dev/null
     ```

## frp

1. server

   [install script](https://github.com/MvsCode/frps-onekey)

   ```bash
   #Aliyun
   wget https://code.aliyun.com/MvsCode/frps-onekey/raw/master/install-frps.sh -O ./install-frps.sh
   chmod 700 ./install-frps.sh
   sudo ./install-frps.sh install

   # Github
   wget https://raw.githubusercontent.com/MvsCode/frps-onekey/master/install-frps.sh -O ./install-frps.sh
   chmod 700 ./install-frps.sh
   sudo ./install-frps.sh install

   # uninstall or update
   ./install-frps.sh uninstall
   ./install-frps.sh update
   # Server management
   Usage: /etc/init.d/frps {start|stop|restart|status|config|version}
   ```

2. client

   [install script](https://github.com/stilleshan/frpc)

   ```bash
   wget https://raw.githubusercontent.com/stilleshan/frpc/master/frpc_linux_install.sh && chmod +x frpc_linux_install.sh && sudo ./frpc_linux_install.sh
   sudo vim /usr/local/frp/frpc.ini
   sudo systemctl restart frpc
   ```

## BITlogin

1. [BIT-srun-login-script](https://github.com/Mmx233/BitSrunLoginGo)

   ```bash
   git clone https://github.com/coffeehat/BIT-srun-login-script.git ~/
   sudo cp ~/BIT-srun-login-script /usr/local/bin/BIT-login
   # change always_online.py
   sudo vim alyways_online.py

   # systemctl
   sudo vim /etc/systemd/system/online.service
   # copy
   [Unit]
   Description=login service
   After=network.target

   [Service]
   Type=simple
   User=root
   ExecStart=/usr/bin/python3 /usr/local/bin/BIT-login/always_online.py
   Restart=always # or always, on-abort, etc

   [Install]
   WantedBy=multi-user.target
   ```

2. [BITSrunLoginGo](https://github.com/Mmx233/BitSrunLoginGo): [releases](https://github.com/Mmx233/BitSrunLoginGo/releases/)

   ```bash
   wget https://github.com/Mmx233/BitSrunLoginGo/releases/download/v3.1/autoLogin_linux_amd64.zip
   unzip autoLogin_linux_amd64.zip
   sudo mkdir /usr/local/bin/BIT-login
   sudo mv autoLogin /usr/local/bin/BIT-login/
   sudo chmod u+x /usr/local/bin/BIT-login/autoLogin
   # 第一次运行创建config.yaml
   sudo sh /usr/local/bin/BIT-login/autoLogin
   # 修改config.yaml
   sudo vim /usr/local/bin/BIT-login/config.yaml

   # systemctl
   sudo vim /etc/systemd/system/online.service
   # copy
   [Unit]
   Description=login service
   After=network.target

   [Service]
   Type=simple
   User=root
   ExecStart=/usr/local/bin/BIT-login/autoLogin
   Restart=always # or always, on-abort, etc

   [Install]
   WantedBy=multi-user.target
   ```

## xrdp

**参考**：[xrdp_installer.sh](https://c-nergy.be/blog/?p=17175): [release](https://c-nergy.be/products.html)

```bash
wget https://www.c-nergy.be/downloads/xRDP/xrdp-installer-1.3.zip
unzip xrdp-installer-1.3.zip
chmod +x xrdp-installer-1.3.sh
./xrdp-installer-1.3.sh -l
```

# Dev Env & Tools

## Python

### Anaconda3

下载地址：[Anaconda3](https://www.anaconda.com/products/distribution#Downloads)

下载与安装：

```bash
wget https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh
sudo sh Anaconda3-2021.11-Linux-x86_64.sh
# 安装路径推荐选择 /usr/local/anaconda3/
```

多用户配置，共享 pkg 缓存，重新设置用户和组权限：

```bash
# 切换root用户，进入目录
sudo su -
cd /usr/local/
# 创建anaconda组
groupadd anaconda
# 添加用户进anaconda组
usermod -a -G anaconda <username>
# 另一种添加方法
adduser <username> anaconda
# 更改组, 路径以anaconda3为例
chgrp -R anaconda /usr/local/anaconda3
# 设置读写权限与SGID权限，SGID权限作用于目录时，可使此目录下新创建的文件或目录所属组与此目录所属组保持一致
# SUID -> 4 SGID 2 SBIT -> 1
chmod 2770 -R /usr/local/anaconda3
# 另一种写法
chmod -R g+s /usr/local/anaconda3
# 关闭共享环境的写入权限
chmod g-w /usr/local/anaconda3/envs
# 组生效
newgrp anaconda
# 第一次需要使用root用户创建一个新环境
source /usr/local/anaconda3/bin/activate
conda create -n first_env python=3.9
conda activate first_env
# install numpy for test
conda install numpy
```

## C/C++

## Rust

## ZSH

[reference](https://github.com/haohaolalahao/dotfile)

1. 插件管理：[zplug](https://github.com/zplug/zplug)

   ```bash
   $ curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
   ```

2. 提示符：[Starship](https://starship.rs/)

   ```bash
   # for linux
   curl -sS https://starship.rs/install.sh | sh
   ```

3. Change .zshrc & starship.toml

   ```bash
   git clone https://github.com/haohaolalahao/dotfile .
   # cp or mv
   cp dotfile/.zshrc ~/
   cp dotfile/starship.toml ~/.config/
   chsh -s zsh
   zsh
   ```

## Tmux

[reference](https://github.com/haohaolalahao/dotfile)

1. Change .tmux.conf

   ```bash
   git clone https://github.com/haohaolalahao/dotfile .
   # cp or mv
   cp dotfile/.tmux.conf ~/
   tmux
   ```

## NeoVim

[reference](https://github.com/haohaolalahao/dotfile)

1. 插件管理：packer

   ```
   # for linux or unix
   git clone --depth 1 https://github.com/wbthomason/packer.nvim\
    ~/.local/share/nvim/site/pack/packer/start/packer.nvim
   ```

2. 共享系统剪切版

   ```
   sudo apt install xsel
   ```

3. cp config

   ```
   git clone https://github.com/haohaolalahao/dotfile .
   # cp or mv
   cp -r dotfile/nvim ~/.config/nvim
   nvim
   :PackerSync
   ```

4. neoformat

   ```
   python: sudo apt install python3-autopep8
   lua: npm install -g lua-fmt
   html css vue js ts json: npm install -g prettier
   ```

5. dap

   ```
   python: python3 -m pip install debugpy

   -- 指定 Python 解释器路径
   -- 修改lua/core/settings.lua
   vim.g.python_path = "/usr/bin/python3.8"
   ```

# Other

## motd

```bash
sudo apt install screenfetch neofetch
# modify /etc/update-motd.d/01-custom
# add code below
/usr/bin/screenfetch
```

## interesting

### toipe

typing tester [home page](https://github.com/Samyak2/toipe)

# Tips and Tricks

## SSH

### 免密登录

1.  生成密钥

```
ssh-keygen -t rsa -b 4096
```

2. windows 连接 linux

   ```powershell
   $USER_AT_HOST="your-user-name-on-host@hostname"
   $PUBKEYPATH="$HOME\.ssh\id_rsa.pub"

   $pubKey=(Get-Content "$PUBKEYPATH" | Out-String); ssh "$USER_AT_HOST" "mkdir -p ~/.ssh && chmod 700 ~/.ssh && echo '${pubKey}' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
   ```

3. linux 连接 ssh

   ```bash
   export USER_AT_HOST="your-user-name-on-host@hostname"
   export PUBKEYPATH="$HOME/.ssh/id_rsa.pub"

   ssh-copy-id -i "$PUBKEYPATH" "$USER_AT_HOST"
   ```

### config

## File

### size

```bash
# 显示目录所占文件大小
du -sh <target>
```

### 同步

```bash
rsync -av <source> <target>
rsync -rlptzv --progress --delete --exclude=.git "user@hostname:/remote/source/code/path" .
```

## Log

### journalctl

- [reference](https://linuxhandbook.com/journalctl-command/)

- [reference](https://linuxhandbook.com/clear-systemd-journal-logs/)

```bash
# 查看日志以less方式
sudo journalctl
# 倒序
sudo journalctl -r
# 实时
sudo journalctl -f
# 仅显示内核消息
sudo journalctl -k
# Filter specifi service log like ssh
sudo journalctl -u <service name>
# List boot sessions
sudo journalctl --list-boots
# Choose sessions
sudo journalctl -b 0/-1/-2
# 特定时间
sudo journalctl --since=yesterday --until=now
sudo journalctl --since "2020-07-10"
sudo journalctl --since "2020-07-10 15:10:00" --until "2020-07-12"
# UID GID PID
sudo journalctl _PID=1234
# 查看最后几条日志
sudo journalctl -xe
# 查看日志所占空间
sudo journalctl --disk-usage
# 轮换日志
sudo journalctl --rotate
# 清楚超过2天的日志
sudo journalctl --vacuum-time=2d
# 限制大小
sudo journalctl --vacuum-size=100M
```

## Network

### 查看端口号

[reference](https://linuxhandbook.com/check-open-ports-linux/)

```bash
sudo lsof -i -P -n
sudo lsof -i -P -n | grep LISTEN
# 参考
-i：如果没有指定IP地址，这个选项选择所有网络文件的列表
-P：禁止将端口号转换为网络文件的端口名
-n：禁止将网络号转换为网络文件的主机名
```

## Bugs

### WD SSD nvme bug

[reference](https://cubox.pro/web/reader/ff8080817e0f2bf9017e388efe61435f)

```bash
sudo vim /etc/default/grub
# append below into the GRUB_CMDLINE_LINUX=" "
nvme_core.default_ps_max_latency_us=0
sudo update-grub
sudo reboot
cat /sys/module/nvme_core/parameters/default_ps_max_latency_us
```

# Shell Scripts
