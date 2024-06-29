---
layout: post
title: Linux 个人初始化配置
---

因为最近频繁折腾Linux，故记录一些常用的配置，避免忘记。以下配置在Debian12.5上保证有效

## 初始化

### 配置 sudo

su root
ll /etc/sudoers
chmod u+w /etc/sudoers
vim /etc/sudoers ...
chmod 440 /etc/sudoers

或者直接 /sbin/addgroup 用户名 sudo

### 安装各种包

apt update && apt upgrade
apt install -y net-tools? ssh vim git gcc curl zsh neofetch build-essential libncurses-dev flex bison libssl-dev libelf-dev dwarves initramfs-tools? qemu-system-x86 gdb

### 配置 zsh

`chsh -s /bin/zsh`

<!-- sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" -->
`sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"`

`git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k`

`vim ~/.zshrc`

`ZSH_THEME="powerlevel10k/powerlevel10k"`

`git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`

`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`

`vim ~/.zshrc ...+z`

### 配置免密登录

主机 ssh-keygen 后手动复制 ~/.ssh/id_rsa.pub 到虚拟机 ~/.ssh/authorized_keys

### 添加 /sbin 环境变量

sudo vim ~/.zshrc
+export PATH=/sbin:$PATH
+source /etc/profile.d/proxy.sh

### 配置全局代理和 git 代理

https://www.zhihu.com/question/495148700

sudo vim /etc/profile.d/proxy.sh

```sh
# set proxy config via profie.d - should apply for all users
# http/https/ftp/no_proxy
export http_proxy="http://172.19.88.1:7890/"
export https_proxy="http://172.19.88.1:7890/"
export ftp_proxy="http://172.19.88.1:7890/"
export no_proxy="127.0.0.1,localhost"

# For curl
export HTTP_PROXY="http://172.19.88.1:7890/"
export HTTPS_PROXY="http://172.19.88.1:7890/"
export FTP_PROXY="http://172.19.88.1:7890/"
export NO_PROXY="127.0.0.1,localhost"
```

sudo chmod +x /etc/profile.d/proxy.sh
logout
source /etc/profile.d/proxy.sh
env | grep -i proxy

git config --global http.proxy http://172.19.88.1:7890
git config --global https.proxy https://172.19.88.1:7890
或 vim ~/.gitconfig

### 其他

~~配置 linux 内核开发环境~~
配置其他插件比如折腾 clangd
md 格式化调整
kernel 各种姿势 make config

怎样修复zsh历史记录错误：zsh: corrupt history file /home/me/.zsh_history？
cd ~
mv .zsh_history .zsh_history_bad
strings -eS .zsh_history_bad > .zsh_history
fc -R .zsh_history

sftp

> 查看ssh服务状态: /etc/init.d/ssh status
> 手动开启：/etc/init.d/ssh start

![1](/assets/images/nett.png)


make menuconfig
make bzImage -j 4 > log
cd arch/x86_64/boot
mkinitramfs -o initrd.img
