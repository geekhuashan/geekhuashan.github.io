+++
title = 'ubuntu 远程桌面从头开始' 
date = 2023-09-27T20:57:43+08:00
lastmod = 2023-09-27T20:57:43+08:00
author = ["geekhuashan"]
draft = false
categories = ["life", "tech"]
tags = ["life", "tech", "ubuntu", "rdp", "guacamole", "remote desktop"]
description = ""
weight = 3
hidemeta = false
+++


# 需求

由于公司电脑的限制，我无法安装很多的软件，比如说，我无法安装微信，无法安装QQ，无法安装微博，无法安装一些有趣的东西，而我想到的是拥有一台完全属于自己的主机

1. 通过这台主机，我可以在公司使用微信，完全属于自己的chrome浏览器，装好各种插件，实现自己的需求
2. 由于公司的remote desktop conncetion 由于安全方面的policy，无法进行rdp 连接，所以我想到了通过guacamole来实现远程桌面的连接，这是一个基于html5协议的客户端，其也支持ssh，rdp，vnc等协议。
3. 然后我尝试着在免费的digital ocean上部署了一个2cpu 4gRAM的小主机，通过查询文档等最终将guacamole 和 ubuntu 以及 gnome 安装在了一起，也能成功运行，但是发现就是最后延迟不能接受，看视频像PPT一样，一帧一帧的

最后我选择了在闲鱼上购买了一年的上海的4cpu-4RAM的主机，然后通过这个主机来实现我的需求,发现这个主机内存性能完全不够，装了nomachine和chrome之后，基本就废了，后来闲鱼上换了一个4核8G RAM的服务器

# 实现

## 1. 安装ubuntu 系统 以及 gnome 桌面 

### 1.1 安装ubuntu 以及 gnome

```
sudo apt update
sudo apt upgrade
sudo apt install xfce4 #安装轻量级桌面
sudo apt install ubuntu-desktop #安装gnome桌面系统
```
### 1.2 添加新用户

```
sudo -i
adduser huashan
sudo usermod -aG sudo huashan
```



## 2. 安装远程控制环境

### 2.1 安装xrdp

```
sudo apt install xrdp
sudo systemctl status xrdp
sudo nano /etc/xrdp/xrdp.ini
```
修改：
port=tcp://:3389

`sudo systemctl restart xrdp`

### 防火墙开放3389端口

```sudo ufw allow 3389```

### 2.2 安装 nomachine

下载 nomachine
`wget https://download.nomachine.com/download/8.9/Linux/nomachine_8.9.1_1_amd64.deb`

安装 nomachine
`sudo dpkg -i nomachine_8.9.1_1_amd64.deb`


## 3. 安装常用软件

### 中文字体包

输入以下命令以更新你的包列表：

`sudo apt-get update`
输入你的密码后，系统将开始更新你的包列表。

安装中文字体。你可以通过下列命令来安装一种常用的中文字体，如 "WenQuanYi Micro Hei"：

`sudo apt-get install fonts-wqy-microhei`

这将会安装 "WenQuanYi Micro Hei" 字体，这是一种常用的、包含了大量中文字符的字体。

如果你需要安装更多的中文字体，你可以尝试安装 "xfonts-wqy"，这是一个更全面的字体包：

`sudo apt-get install xfonts-wqy``
这将会安装 "WenQuanYi Zen Hei" 字体。

重启你的系统或者你的应用程序。安装完成后，你可能需要重启你的电脑或者关闭并重新打开你的应用程序（如浏览器或文本编辑器）以使新安装的字体生效。

#### 微软雅黑

从 Windows 系统中获取微软雅黑字体文件：
微软雅黑的字体文件名通常为 msyh.ttc 或 msyhbd.ttc（粗体版本）。你可以在 Windows 系统的 C:\Windows\Fonts\ 目录中找到它们。

复制字体文件到 Ubuntu：
在 Ubuntu 中，你需要以 root 用户身份把这些文件复制到 /usr/share/fonts/truetype/ 目录。你可以使用以下命令：

```bash
sudo cp /path/to/msyh.ttc /usr/share/fonts/truetype/
sudo cp /path/to/msyhbd.ttc /usr/share/fonts/truetype/
```
请将 /path/to/ 替换为你的字体文件在 Ubuntu 中的实际路径。

更新字体缓存：
你需要更新 Ubuntu 的字体缓存以使新安装的字体生效。你可以使用以下命令：

`sudo fc-cache -f -v`


### clash 

下载 clash
`wget https://github.com/Fndroid/clash_for_windows_pkg/releases/download/0.20.37/Clash.for.Windows-0.20.37-x64-linux.tar.gz`

安装
```bash
mkdir app
tar -zxvf Clash.for.Windows-0.20.37-x64-linux.tar.gz -C app/
cd app/
mv Clash.for.Windows-0.19.12-x64-linux clash
cd clash
wget https://cdn.jsdelivr.net/gh/Dreamacro/clash@master/docs/logo.png #下载桌面图表
vim clash.desktop
```

#输入下面的内容
```bash
[Desktop Entry]
 Name=clash
 Comment=Clash
 Exec=/home/huashan/app/clash/cfw
 Icon=/home/huashan/app/clash/logo.png
 Type=Application
 Categories=Development;
 StartupNotify=true
 NoDisplay=false

sudo mv clash.desktop /usr/share/applications/
```
然后打开更多应用，里面就有了



### chrome

下载chrome
`wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb`

下载好chrome 之后  安装即可
`sudo dpkg -i google-chrome-stable_current_amd64.deb`

如果出现依赖性问题，你可以使用以下命令来解决：

`sudo apt-get install -f`

### 输入法

下载搜狗输入法 `wget https://ime-sec.gtimg.com/202310100937/a3b737ecf4bdea9fa36fea1295db9d4d/pc/dl/gzindex/1680521603/sogoupinyin_4.2.1.145_amd64.deb`
安装 
`sudo dpkg -i sogoupinyin_4.2.1.145_amd64.deb`

安装依赖
`sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2`

`sudo apt install libgsettings-qt1`

### vscode

官网下载vscode的deb 版本
`sudo dpkg -i ` 安装

### zotero



## 4. 优化措施

### 4.1 authentication is required to create a color profile

```bash
sudo nano /etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla
```

添加以下内容
```
[Allow Colord all Users]
Identity=unix-user:*
Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes
```

### 4.2 优化xrdp连接

#### 调整 Xrdp 配置参数
编辑 `vi /etc/xrdp/xrdp.ini`

```bash
tcp_send_buffer_bytes=4194304
tcp_recv_buffer_bytes=6291456
```

tcp_send_buffer_bytes, tcp_recv_buffer_bytes 两个参数默认被注释了，注释默认值（32768），根据实际情况进行调整。

#### 调整系统参数
临时生效

```bash
sudo sysctl -w net.core.rmem_max=12582912
sudo sysctl -w net.core.wmem_max=8388608
```

重启后保留
将以下内容写入配置文件 `vi /etc/sysctl.conf`
```bash
net.core.rmem_max = 12582912
net.core.wmem_max = 8388608
```

然后执行
`sudo sysctl -p`

重启 xrdp 服务生效
`sudo systemctl restart xrdp`

### 4.3 优化ubuntu原生桌面

install gnome-tweaks

```bash
sudo apt install gnome-tweaks
```
添加配置文件 `vi  ~/.xsessionrc`
    
```bash
export GNOME_SHELL_SESSION_MODE=ubuntu
export XDG_CURRENT_DESKTOP=ubuntu:GNOME
export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
```

重启xrdp后生效
`sudo systemctl restart xrdp.service`

此时再连接，你就将得到和原生桌面一样的效果了








# 参考
https://zhuanlan.zhihu.com/p/519648451
https://blog.csdn.net/wu_weijie/article/details/116158271
https://www.cnblogs.com/pipci/p/16377032.html
https://krdesigns.com/articles/how-to-install-guacamole-using-docker-step-by-step
https://devicetests.com/fix-xrdp-login-blank-screen-issues

