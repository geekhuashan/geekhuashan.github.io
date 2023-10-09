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

最后我选择了在闲鱼上购买了一年的上海的4cpu-4RAM的主机，然后通过这个主机来实现我的需求

# 实现

## 1. 安装ubuntu 系统 以及 gnome 桌面 

### 1.1 安装ubuntu 以及 gnome

```
sudo apt update
sudo apt upgrade
sudo apt install ubuntu-desktop
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
` sudo dpkg -i nomachine_8.9.1_1_amd64.deb`

## 3. 安装常用软件

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
#输入下面的内容
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
下载好chrome 之后 `sudo dpkg -i xxxx.deb` 安装即可

### 输入法

### vscode

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

