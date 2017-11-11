---
title: "Ubuntu 安装软件合集"
toc: true
tags:
  - Linux
  - VPS
  - Ubuntu
categories:
  - Linux & VPS
top:
---

记录 Ubuntu 平台下安装的部分软件。
-----

[awesome-linux-software](https://alim0x.gitbooks.io/awesome-linux-software-zh_cn/content/#%E7%9B%AE%E5%BD%95)

# Node.js & 版本管理

- 推荐的安装方式是通过 nvm

```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash

# verify installtion
command -v nvm
# print
nvm
# install nodejs
nvm install node

# special in o-my-zsh
# install as plugin
git clone https://github.com/lukechilds/zsh-nvm~/.oh-my-zsh/custom/plugins/zsh-nvm

# edit ~/.zshrc
plugins+=(zsh-nvm)

```

[nvm](https://github.com/creationix/nvm)

# S-S-Libev

```shell

# 提示
add-apt-repository: command not found

# 安装工具包
apt-get install software-properties-common

# 安装 libev
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev
sudo apt-get update
sudo apt install shadowsocks-libev

# 启动时提示
This system doesn't provide enough entropy to quicklygenerate high-quality random numbers

# 安装 rng-tools 工具

apt install rng-tools


# 4 in 1
wget --no-check-certificate -O shadowsocks-all.shhttps://raw.githubusercontentcom/teddysun/shadowsocks_install/master/shadowsocks-allsh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log

```

# KeePass2

```shell

apt install keepass2

```

# Uget

```shell

sudo add-apt-repository ppa:plushuang-tw/uget-stable
sudo apt update
sudo apt install uget

```

# Oracle Java

```shell

# 清除已经安装的 OpenJDK
sudo apt-get purge openjdk-\*
# 安装 Oracle JDK
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer

```


[how to install java with apt get on ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)

# Chrome

```shell
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb

```

# Typora

```shell
# optional, but recommended
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
# add Typora's repository
sudo add-apt-repository 'deb https://typora.io ./linux/'
sudo apt-get update
# install typora
sudo apt-get install typora
```

# you-get

```shell
# install ffmpeg
sudo add-apt-repository ppa:mc3man/trusty-media
sudo apt update
sudo apt install ffmpeg
# install you-get
pip3 install you-get
#usage
you-get url
```

# Guake 终端

```shell
sudo add-apt-repository ppa:webupd8team/unstable
sudo apt-get update
```
到 Github 页面下载最新版本并解压，之后执行

```
./dev.sh --install
```

[Guake](https://github.com/Guake/guake)

# BBR

```shell
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh 
chmod +x bbr.sh 
./bbr.sh
```

# Coursera-dl

```shell
pip3 install coursera-dl
# sometime you need to restart your shell
exec $SHELL
```

# Firefox beta

```shell
sudo add-apt-repository ppa:mozillateam/firefox-next
sudo apt-get update
sudo apt-fast install firefox
```

# Telegram

```shell
sudo add-apt-repository ppa:atareao/telegram
sudo apt-get update
sudo apt-fast install telegram
```

# Wechat 微信

[electronic-wechat](https://github.com/geeeeeeeeek/electronic-wechat)

# 网易云

- 魔改版

  - [ieaseMusic](https://github.com/trazyn/ieaseMusic)

# 电源管理

```shell
sudo apt-fast install tlp tlp-rdw
sudo tlp start
```

# 系统监视器

```shell
sudo apt-get install python3-psutil curl git gir1.2-appindicator3-0.1
git clone https://github.com/fossfreedom/indicator-sysmonitor.git
cd indicator-sysmonitor
sudo make install
nohup indicator-sysmonitor &
```

[indicator-sysmonitor](https://github.com/fossfreedom/indicator-sysmonitor)