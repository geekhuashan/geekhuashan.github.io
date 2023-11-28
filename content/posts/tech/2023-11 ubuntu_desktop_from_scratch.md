+++
title = '2023-11 ubuntu 远程桌面从头开始' 
date = 2023-11-27T20:57:43+08:00
lastmod = 2023-11-27T20:57:43+08:00
author = ["geekhuashan"]
draft = false
categories = ["life", "tech"]
tags = ["life", "tech", "ubuntu", "rdp", "guacamole", "remote desktop"]
description = ""
weight = 5
hidemeta = false
+++


# 需求

由于公司电脑的限制，我无法安装很多的软件，无法自由的访问一些我需要的网站，比如chatgpt，gmail等，限制太多，监控太多，而我想到的是拥有一台完全属于自己的主机，完全由我自己掌控的电脑。

1. 通过这台主机，我可以在公司使用微信，完全属于自己的chrome浏览器，装好各种插件，实现自己的需求
   
2. 由于公司的remote desktop conncetion 由于安全方面的policy，无法进行rdp 连接，而且VNC gucamole 等都非常的卡顿，后来我发现了 nomachine 这个软件，说这个可以非常好的解决连接卡顿的问题，后来我试了一下，过来不错。
  
3. 我选择了在闲鱼上购买了一年的上海的4cpu-4RAM的主机，然后通过这个主机来实现我的需求,发现这个主机内存性能完全不够，装了nomachine和chrome之后，基本就废了，后来闲鱼上换了一个4核8G RAM的服务器，这个服务器，再我装了gnome，chrome，vscode,zotero之后，不是一下子全开软件的话，运行的还比较流畅的。




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

## 2. 安装远程控制环境以及分辨率设置

### 2.1 安装 nomachine

下载 nomachine
`wget https://download.nomachine.com/download/8.9/Linux/nomachine_8.9.1_1_amd64.deb`

安装 nomachine
`sudo dpkg -i nomachine_8.9.1_1_amd64.deb`

### 2.2 更改分辨率

#### XROG

在想要远程Ubuntu Server去完成VNC或者NoMachine的时候，由于没有接显示器，会导致无法显示。需要虚拟一个桌面，以供操作。

安装虚拟桌面
`sudo apt-get update`
`sudo apt-get install xserver-xorg-video-dummy`
`sudo apt-get install  xserver-xorg-core-hwe-18.04`

配置 `sudo vi /usr/share/X11/xorg.conf.d/xorg.conf`

```bash
Section "Monitor"
  Identifier "Monitor0"
  HorizSync   5.0 - 1000.0
  VertRefresh 5.0 - 200.0
  # https://arachnoid.com/modelines/
  # 1920x1080 @ 60.00 Hz (GTF) hsync: 67.08 kHz; pclk: 172.80 MHz for most of display
  Modeline "1920x1080" 173.00  1920 2048 2248 2576  1080 1083 1088 1120 
  # 3000x2000 @ 30 Hz (CVT) hsync: 67.08 kHz; pclk: 172.80 MHz for pixel slate
  Modeline "3000x2000" 243.75  3000 3184 3496 3992  2000 2003 2013 2037
  # 2400x1080 @ 60 Hz (CVT) hsync: 67.08 kHz; pclk: 172.80 MHz for Pixel 6
  Modeline "2400x1080"  216.00  2400 2552 2808 3216  1080 1083 1093 1120

EndSection

Section "Device"
  Identifier "Card0"
  Driver "dummy"
  VideoRam 256000
EndSection

Section "Screen"
  DefaultDepth 24
  Identifier "Screen0"
  Device "Card0"
  Monitor "Monitor0"
  SubSection "Display"
    Depth 24
    Modes  "1920x1080" "3000x2000" "2400x1080"
  EndSubSection
EndSection
```


重启系统

#### WYLAND
失败了，参考文章这个，以后再试试
https://askubuntu.com/questions/973499/wayland-how-to-set-a-custom-resolution


## 3. 安装常用软件

### 中文字体包

#### 一般的开源中文字体

输入以下命令以更新你的包列表：

`sudo apt-get update`
输入你的密码后，系统将开始更新你的包列表。

安装中文字体。你可以通过下列命令来安装一种常用的中文字体，如 "WenQuanYi Micro Hei"：

`sudo apt-get install fonts-wqy-microhei xfonts-wqy`

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

如果字体选择器那里显示的是乱码，方框之类的，需要给字体文件设置权限
`sudo chmod 644 /usr/share/fonts/truetype/msyh.ttc`


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

输入下面的内容
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

首先安装 fcitx5
`sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-frontend-gtk4 fcitx5-frontend-gtk3 fcitx5-frontend-gtk2 fcitx5-frontend-qt5`

卸载ibus 
`sudo apt-get remove ibus`

#### 配置
设置为默认输入法
使用 im-config 工具可以配置首选输入法，在任意命令行输入：
`im-config`
根据弹出窗口的提示，将首选输入法设置为 Fcitx 5 即可。

环境变量
需要为桌面会话设置环境变量，即将以下配置项写入某一配置文件中：
`vi /etc/profile` 最后面添加

```bash
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
```

#### 开机自启动
安装 Fcitx 5 后并没有自动添加到开机自启动中，每次开机后需要手动在应用程序中找到并启动，非常繁琐。

解决方案非常简单，在 Tweaks（sudo apt install gnome-tweaks）中将 Fcitx 5 添加到「开机启动程序」列表中即可。

#### 自定义主题
我用的是这个，https://github.com/Reverier-Xu/FluentDark-fcitx5 
下载后，放入到 /usr/share/fcitx5/themes/ 这个文件就下面即可

### Vscode

官网下载vscode的deb 版本
`sudo dpkg -i ` 安装

#### 设置 SOCKS 代理, 

确保插件使用代理，这样就可以非常顺畅的使用copilot了
在 VS Code 的设置中，你可以为 HTTP Proxy 输入 SOCKS 代理地址。例如，如果 Clash 的 SOCKS 代理设置为 127.0.0.1:7890，则你可以输入 socks5://127.0.0.1:7890。

确保插件使用代理
VS Code 的大部分网络活动（包括插件管理和更新）都会遵循你在“HTTP Proxy”设置中定义的代理。但为确保所有网络请求都通过代理，http.proxySupport: 设置为 on，以确保插件也使用代理。

修改设置后，关闭并重新启动 VS Code 以确保设置生效。

这样，你的 VS Code 和其中的插件都应该能够通过 Clash 的 SOCKS 代理进行网络访问。

#### 远程访问 ssh 连接

1. 在本地机器生成密钥对(公钥+私钥)：ssh-keygen
2. 私钥放本机，公钥放远程(~/.ssh路径下)
3. 在远程机器用公钥生成authorized_keys：
- 进入home目录下的.ssh文件夹：cd ~/.ssh
- cat id_rsa.pub >> authorized_keys
4. vscode config文件加入本机私钥路径
- IdentityFile “C:\Users\QUWS\.ssh\id_rsa”

如果没有反应，在remote vps 上执行以下命令
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
原因是SSH 不希望用户目录和~/.ssh目录对组有写权限。

### zotero

在 Ubuntu 上安装 Zotero 的步骤：

1. **下载 tarball**

   首先，从 Zotero 的官方网站下载 tarball。

2. **解压 tarball**

   假设你已经下载了 tarball 并将其保存为 `zotero.tar.bz2`，你可以使用以下命令解压它：

   ```bash
   tar -xvf zotero.tar.bz2
   ```

3. **移动解压后的目录**

   将解压后的目录移动到你选择的位置。在这里，我们选择 `/home/huashan/app` 目录：

   ```bash
   sudo mv zotero /home/huasshan/app
   ```

4. **更新 .desktop 文件**

   .desktop 文件需要图标的绝对路径。运行 `set_launcher_icon` 脚本来更新 .desktop 文件：

   ```bash
   cd /home/huashan/app/zotero/
   ./set_launcher_icon
   ```

5. **创建符号链接**

   最后，创建一个符号链接到 `~/.local/share/applications/` 以将 Zotero 添加到启动器：

   ```bash
   ln -s /home/huashan/app/zotero/zotero.desktop ~/.local/share/applications/zotero.desktop
   ```

6. **启动 Zotero**

   现在，Zotero 应该出现在你的启动器或当你点击网格图标（“显示应用程序”）时的应用程序列表中。你可以从那里将其拖到启动器上。

完成以上步骤后，你应该可以正常使用 Zotero 了。如果在安装过程中遇到任何问题，请告诉我，我会帮助你解决。




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


### 4.4 优化docker 栏点击效果
如图标下只有一个窗口，则显示或最小化窗口。
如图标下有多个窗口则会显示多个窗口的预览，再选择需要的窗口打开。
`gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize-or-previews'`

### 4.5 删除不要的软件
```bash
sudo apt-get remove libreoffice-common

sudo apt-get remove thunderbird totem rhythmbox simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-sudoku

```

# 参考
https://zhuanlan.zhihu.com/p/519648451
https://blog.csdn.net/wu_weijie/article/details/116158271
https://www.cnblogs.com/pipci/p/16377032.html
https://krdesigns.com/articles/how-to-install-guacamole-using-docker-step-by-step
https://devicetests.com/fix-xrdp-login-blank-screen-issues

