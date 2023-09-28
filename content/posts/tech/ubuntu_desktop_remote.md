+++
title = 'ubuntu 远程桌面 实现 记录' 
date = 2023-09-27T20:57:43+08:00
lastmod = 2023-09-27T20:57:43+08:00
author = ["geekhuashan"]
draft = true
categories = ["life", "tech"]
tags = ["life", "tech"，"ubuntu","rdp","guacamole","remote desktop"]
description = ""
weight = 
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
sudo usermod -a G  huashan
```

### 1.3 安装xrdp

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


## 2. 安装guacamole

### 2.1 安装docker & docker-compose

安装 Docker 和 Docker Compose 在 Ubuntu 上的过程相对简单。这里是安装步骤：

1. 更新软件库**
    ```bash
    sudo apt update
    ```

2. **安装依赖**
    ```bash
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ```

3. **添加 Docker 官方密钥**
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

4. **添加 Docker 软件源**
    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

5. **更新软件库**
    ```bash
    sudo apt update
    ```

6. **安装 Docker**
    ```bash
    sudo apt install docker-ce
    ```

7. 安装 Docker Compose

    
    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```
    或者
    ```bash
     sudo curl -L "https://get.daocloud.io/docker/compose/releases/download//latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```


     **修改权限**
    ```bash
    sudo chmod +x /usr/local/bin/docker-compose
    ```

8. 验证安装

   - 验证 Docker：
    ```bash
    sudo docker --version
    ```

   - 验证 Docker Compose：
    ```bash
    docker-compose --version
    ```

### 2.2 安装guacamole

docker image pull
I will not include on how to install docker and docker-compose because its beyond the scope of this guide. Please also remember since Apache Guacamole did not include other container other then AMD64, ao this guide will not work for ARM (sorry RPI user)

```bash
docker pull guacamole/guacamole:1.4.0
docker pull guacamole/guacd:1.4.0
docker pull mariadb:10.9.5
```

First of all lets download the image from docker-hub, by the time this guide is written the latest tag are 1.4.0 and I will be using MariaDB instead of Postgres.
Grab the latest sql info from the latest images
Run this command to retrive the db.sql

```bash
docker run --rm guacamole/guacamole:1.4.0 /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

Making initial DB docker-compose.yml

using any Linux editor - in my case using nano create a new docker-compose.yml.

```yaml
version: '3'
services:
  guacdb:
    container_name: guacamoledb
    image: mariadb:10.9.5
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'MariaDBRootPass'
      MYSQL_DATABASE: 'guacamole_db'
      MYSQL_USER: 'guacamole_user'
      MYSQL_PASSWORD: 'MariaDBUserPass'
    volumes:
      - './db-data:/var/lib/mysql'
volumes:
  db-data:
```

after saving the files, please run `docker-compose up -d`

Next you need to copy the SQL file into the docker container

`docker cp initdb.sql guacamoledb:/initdb.sql`
Last but not least begin to input it to the DB by running this:

```bash
docker exec -it guacamoledb bash
cat /initdb.sql | mysql -u root -p guacamole_db
exit
```

and now time for your to turn off the DB by running `docker-compose down`

**Password is MariaDBRootPass**

Complete the docker-compose.yml with all the necessary image
Now your best practice is to backup your docker-compose files by typing `cp docker-compose.yml docker-compose.yml.bak` Next you will edit and add more detail

Now you will be able to run docker-compose up -d and you should have your Guacamole up and running on your server.

```yaml
version: '3'
services:
  guacdb:
    container_name: guacamoledb
    image: mariadb:10.9.5
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'MariaDBRootPass'
      MYSQL_DATABASE: 'guacamole_db'
      MYSQL_USER: 'guacamole_user'
      MYSQL_PASSWORD: 'MariaDBUserPass'
    volumes:
      - './db-data:/var/lib/mysql'
  guacd:
    container_name: guacd
    image: guacamole/guacd:1.4.0
    restart: unless-stopped
  guacamole:
    container_name: guacamole
    image: guacamole/guacamole:1.4.0
    restart: unless-stopped
    ports:
      - 8080:8080
    environment:
      GUACD_HOSTNAME: "guacd"
      MYSQL_HOSTNAME: "guacdb"
      MYSQL_DATABASE: "guacamole_db"
      MYSQL_USER: "guacamole_user"
      MYSQL_PASSWORD: "MariaDBUserPass"
    depends_on:
      - guacdb
      - guacd
volumes:
  db-data:
```

Now you will be able to run `docker-compose up -d` and you should have your Guacamole up and running on your server.

How do I access Guacamole?
Very simple just open your browser and put in your Guacamole IP with port 8080

For example: [http://Ipadress:8080/guacamole](http://mylocalip.home:8080/guacamole)  guacamole can't be access via root directory, so you will have to add /guacamole.

The original username/password are guacadmin/guacadmin

For security reason I include TOTP in my installation guide, so be sure to have your google authenticator ready for scanning. Else make sure you remove TOTP_ENABLED: "true" line from your docker-compose.yml file.


## 3. 优化措施

### 3.1 优化xrdp连接

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

### 3.2 优化ubuntu原生桌面

install gnome-tweaks

```bash
sudo apt install gnome-tweaks
```
添加配置文件 `vi vim ~/.xsessionrc`
    
```bash
export GNOME_SHELL_SESSION_MODE=ubuntu
export XDG_CURRENT_DESKTOP=ubuntu:GNOME
export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
```

重启xrdp后生效
`sudo systemctl restart xrdp.service`

此时再连接，你就将得到和原生桌面一样的效果了
