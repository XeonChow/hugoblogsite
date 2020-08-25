---
title: "树莓派入门记录"
date: 2020-05-10T14:51:03+08:00
slug: raspberry-pi-101
tags: ["diy","pi"]
align: justify
indent: false
dropCap: false
toc: true
mathjax: false
---

本文是疫情期间在家折腾树莓派时记下的笔记，从树莓派如何安装系统到树莓派的一些简单应用，记录了各个步骤的命令、配置文件修改等等内容。本文不是全面的树莓派教程，只是个人学习的记录，内容比较松散、浅显，也可能存在错误，仅供参考。

<!--more-->

## 树莓派散热改装

原始的树莓派仅仅是一个卡片大小的单板电脑，没有外壳、散热等部件。尽管树莓派价格低廉，功耗也不高，但在夏天长时间运行时，也可能出现高温过热从而降频甚至自动关机的可能，因此有必要对树莓派加装一些散热措施。一般在淘宝店家购买树莓派时都会附送两块金属的散热片，可以粘贴在树莓派的 CPU 和 内存芯片上，也有店家会多送一个粘贴在 USB 管理芯片上。散热片的粘贴位置如下图所示：

<img src="/raspberry-pi-101/thermal.jpg" width="500" >

如果担心散热片被动散热效果不够好，可以使用风扇主动散热。淘宝上售卖的树莓派风扇大多是 5V 电压的，风扇的两根线中，红色的是 5V 线，接树莓派 GPIO 接口的 4 号针脚，黑色的是地线，接树莓派的 6 号针脚。树莓派 GPIO 各引脚定义如下图：

<img src="/raspberry-pi-101/gpio.jpg" width="500" >

这种两线的接法无法用软件控制风扇的开关或调速，只要树莓派通电，无论出于开机或关机状态，风扇都会运行。如果嫌树莓派风扇太吵，想要控制树莓派随 CPU 温度高低来自动启停，需要加接一个三极管，额外引出一根线接树莓派的 GPIO 口，并编写脚本进行控制。这部分内容我没有尝试，有需要的可以参考[这篇文章](http://www.raspigeek.com/index.php?c=read&id=126&page=1)。

## 树莓派系统安装

原始的树莓派不附带存储卡，需要自行准备一张 micro SD 卡进行系统烧录。树莓派目前可安装的系统不少，推荐新手安装树莓派官方制作的 Raspbian 系统，尤其是带有图形界面的版本，可以帮助初学者更方便地进行初始设置。

按照以下步骤烧录树莓派系统：

* 在这个页面下载 [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)，下载后解压提取出 `.img` 镜像文件；

* 下载并运行 [DiskGenius](http://www.diskgenius.cn/download.php)，删除原 SD 卡分区，新建分区，格式化为 FAT32；

* 下载安装 [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)，选择下载的 *Raspbian* 系统的镜像文件，写入格式化后的 SD 卡；

树莓派的系统即烧录成功。

Raspbian 默认未开启 SSH，需手动开启，打开写入镜像后的 SD 卡盘符（此时在 Windows 系统中显示为两百兆左右大小的 `boot` 盘符），新建一个文件名为 `ssh` 的..无后缀名..文件即可。

## 无显示器设置 WiFi 连接

### 修改 SD 中的网络配置文件

如果选择了无图形界面版本的 Raspbian，或者没有条件让树莓派连接有线网络，需要提前配置好树莓派的 WiFi 连接信息，以让树莓派能自己找到并连上无线网络。

将刷好 Raspbian 系统的 SD 卡用电脑读取。在 `boot` 分区、也就是树莓派的 `/boot` 目录下新建 `wpa_supplicant.conf` 文件，按照下面的参考格式填入内容并保存 `wpa_supplicant.conf` 文件：

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="WiFi-A"
psk="12345678"
key_mgmt=WPA-PSK
priority=1
}

network={
ssid="WiFi-B"
psk="12345678"
key_mgmt=WPA-PSK
priority=2
scan_ssid=1
}
```

其中

- ssid: WiFi 网络的名称；
- psk: WiFi 密码；
- priority: 网络优先级，数字越小优先级越高；
- scan_ssid: 连接隐藏WiFi时需要指定该值为1。

### 通过 Terminal 输入命令连接 WiFi

无图形界面的树莓派，如果可以连接显示器，或已经连接有线网络并进行 SSH 连接的情况下，可以通过 Terminal 用命令连接 WiFi。

查看可用的 WiFi 热点

```bash
iwlist scan
```

配置连接到某个热点，编辑 WiFi 文件

```bash
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
```

在最后加入

```bash
network={
  ssid="WIFINAME"
  psk="password"
}
```

保存文件后几秒应该会自动连接，查看 WiFi 连接状态

```bash
ifconfig wlan0
```

如果未连接，可手动重启网络

```bash
sudo /etc/init.d/networking restart
```

或直接重启系统。

## 树莓派开机

树莓派网线连接到路由器（或按照上文配置好 WiFi 信息），连接电源线，树莓派的电源指示灯和网线接口灯亮起，开机成功。

## SSH 连接

使用 [Advanced IP Scanner](https://www.advanced-ip-scanner.com/cn/) 或登陆路由器后台，查看树莓派被分配的内网 IP 地址，用 SSH 登陆。SSH 的端口为默认端口 22，用户名 pi，默认密码 Raspberry。

## 替换系统源为国内清华源

Raspbian 系统的默认软件源在国外，国内访问速度较慢，可以将默认软件源换为国内清华的镜像源。

首先备份系统默认源文件

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.bak
```

以上 `cp` 命令为复制文件命令。

接着用 nano 编辑器打开并编辑 `/etc/apt/sources.list` 文件

```bash
sudo nano /etc/apt/sources.list
```

将初始的源使用 `#` 注释掉，添加如下两行清华的镜像源

```text
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

`Ctrl`+`O` 保存，`Enter` 键确认，`Ctrl`+`X ` 退出 nano 编辑器。

编辑`/etc/apt/sources.list.d/raspi.list`文件

```bash
sudo nano /etc/apt/sources.list.d/raspi.list
```

将初始的源使用 # 注释掉，添加如下两行清华的镜像源

```text
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```

`Ctrl`+`O` 保存，`Enter` 键确认，`Ctrl`+`X ` 退出 nano 编辑器。

最后更新软件包索引

```bash
sudo apt-get update
sudo apt-get upgrade
```

## Windows 远程访问

安装了带图形界面 Raspberry 系统的树莓派通过 HDMI 线连接显示器即可显示图形界面，若不想连接显示器、又想要直接操作图形界面，可以通过远程桌面来访问。

首先树莓派上安装 xrdp

```bash
sudo apt-get install xrdp
```

接着使用 Windows 自带远程桌面软件连接树莓派，`Win`+`R` 搜索 `mstsc` 打开 Windows 自带远程桌面软件，在选项中可以设置显示分辨率、是否同步剪贴板等选项，建议打开同步剪贴板功能，可以方便地在 Windows 和树莓派之间复制文件。输入树莓派的局域网 IP 访问树莓派，登录界面输入树莓派的用户名和密码登陆。

## 给树莓派分配局域网固定 IP

一般路由器给设备分配的 IP 都不会变，但为了好记并以防重启路由器后树莓派的 IP 变动造成 SSH 连接不便，可以手动给树莓派分配固定的局域网 IP。

### GUI 内设置

在树莓派 GUI 桌面的右上角状态栏有 WiFi 或网络连接标志，右键进入 Wireless & Wired Network Settings，将 Configure: Interface-eth0（对应有线网络设置）和 Interface-wlan0（对应无线网络设置）的 IPv4 Address 修改为固定 IP 地址，勾选 Automatically configure empty options 自动配置其余选项。从而树莓派在该局域网内拥有一个固定 IP 地址。

### 无 GUI 设置

如果没有图形界面，首先输入以下命令查看当前使用的网卡

```bash
ifconfig -a
```

可以查看当前是有线网卡（eth0）或无线网卡（wlan0）连接和相关信息。

修改`/etc/dhcpcd.conf`文件，在末尾添加以下内容

```bash
interface wlan0  #网卡名
inform xxx.xxx.xxx.xxx/24    #分配给树莓派的 IP
static routers=xxx.xxx.xxx.xxx  #网关，即路由器 IP
static domain_name_servers=xxx.xxx.xxx.xxx  #DNS，一般也填路由器IP
```

如果是有线网卡

```bash
interface eth0
static ip_address=xxx.xxx.xxx.xxx/24
static routers=xxx.xxx.xxx.xxx
static domain_name_servers=xxx.xxx.xxx.xxx
```

注意分配给树莓派的 IP 需在和路由器在同一网段，即 IP 地址的前三段内容相同。本地网卡 + 无线网卡 IP 不可设置一样，否则会冲突。

然后 reboot 重启系统。

## 树莓派挂载 U 盘

Linux 的文件系统与 Windows 不同，无法直接挂在一些格式的外接存储设备，需安装相关支持。

### 安装 exfat-fuse

```bash
sudo apt-get install exfat-fuse
```

### 安装 nfts 支持

```bash
sudo apt-get install -y ntfs-3g
```

加载内核支持

```bash
modprobe fuse
```

### 手动挂载

查看 U 盘序号

```bash
sudo fdisk -l
```

得到 U 盘序号，一般为 `/dev/sda1` 这样的路径

新建想要挂载位置的路径，如

```bash
sudo mkdir /mnt/1GB_USB_flash
```

挂载 U 盘

```bash
sudo mount -o uid=pi,gid=pi /dev/sda1 /mnt/1GB_USB_flash
```

卸载 U 盘

```bash
sudo umount /mnt/1GB_USB_flash
```

### 开机自动挂载

每次重启树莓派需要重新挂载 U 盘，可以设置开机自动挂载。

#### rc.local 脚本方式

将挂载命令写入开机自启的脚本，来实现开机自动挂载U盘的命令，命令行如下

```bash
sudo nano /etc/rc.local
```

在`exit 0`前一行，写上挂载命令，即：

```bash
mount -o uid=pi,gid=pi /dev/sda1 /mnt/udisk/
```

#### fstab 方式

查看要指定加载的储存设备的 UUID

```bash
sudo blkid
```

结果中找到外接存储设备信息，如

```
/dev/mmcblk0p1: LABEL_FATBOOT="boot" LABEL="boot" UUID="9969-E3D2" TYPE="vfat" PARTUUID="97709164-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="8f2a74a4-809c-471e-b4ad-a91bfd51d7c3" TYPE="ext4" PARTUUID="97709164-02"
/dev/sda1: LABEL="Seagate Backup Plus Drive" UUID="A4E83FF7E83FC5F8" TYPE="ntfs" PTTYPE="atari" PARTUUID="1bd1a60b-01"
/dev/mmcblk0: PTUUID="97709164" PTTYPE="dos"
```

其中 `/sda1` 外接存储设备的 UUID 为 `A4E83FF7E83FC5F8`

编辑设备管理

```bash
sudo nano /etc/fstab
```

在最后一行添加你要挂载的设备

针对非 ntfs 格式的移动硬盘

```text
UUID=A4E83FF7E83FC5F8 /mnt/disk1 auto defaults,noexec,umask=0000 0 0
```

针对 ntfs 格式的移动硬盘

```text
UUID=A4E83FF7E83FC5F8 /mnt/disk1 ntfs-3g defaults,noexec,umask=0000 0 0
```

## 树莓派安装 Samba

Samba 可以用于局域网内共享文件，方便在同一局域网内访问树莓派上的内容。

### 安装 Samba

```bash
sudo apt-get install samba samba-common-bin
```

修改 Samba 配置文件

```bash
sudo nano /etc/samba/smb.conf
```

打开后在最底部添加配置（示例）

```
[pi_udisk]						#samba 服务名称，随意
comment = Public Storage		#备注，随意
path = /media/pi/udisk	#路径，想要分享的文件位置，示例为挂载的 U 盘
read only = no					#是否只读
create mask = 0777				#路径使用限制配置
directory mask = 0777			#路径使用限制配置
guest ok = yes					#是否支持 guest 用户
browseable = yes				#是否支持浏览文件
```

`Ctrl`+`O` 保存，`Enter` 键确认，`Ctrl`+`X ` 退出 nano 编辑器。

重启 Samba 服务

```bash
sudo systemctl restart smbd
```

### 完全卸载 Samba

```bash
sudo apt-get remove --purge samba samba-*
sudo apt-get autoremove
```

## FRP 内网穿透

如果想要在外网访问树莓派，可以使用 FRP 和 VPS 进行内网穿透。

查看 VPS 和树莓派的处理器架构

```bash
sudo uname -a
```

一般 VPS 是 amd64 架构，树莓派是 arm64 架构。根据架构在 FRP 的 [release](https://github.com/fatedier/frp/releases) 页面找到对应的下载链接。目前 v0.31.2 版本的 release 分别为：

VPS: [frp_0.31.2_linux_amd64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.31.2/frp_0.31.2_linux_amd64.tar.gz) 

树莓派: [frp_0.31.2_linux_arm64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.31.2/frp_0.31.2_linux_arm.tar.gz)

### VPS 服务端配置

以下均在 root 权限下操作。

下载对应的 release

``` bash
wget https://github.com/fatedier/frp/releases/download/v0.31.2/frp_0.31.2_linux_amd64.tar.gz 
```

默认的下载路径为当前用户目录，即 `/root`

解压

```bash
tar -zxvf frp_0.31.2_linux_amd64.tar.gz
```

转到对应目录

```bash
cd frp_0.31.2_linux_amd64
```

打开修改配置文件

```bash
nano frps.ini
```

配置文件内容如下

```text
[common]
bind_port = 7000
vhost_http_port = 88
dashboard_port = 7500
dashboard_user = username
dashboard_pwd = yourpassword
privilege_token = yourtoken
```

参数说明

- bind_port：绑定的端口，需要与客户端中 server_port 参数保持一致
- vhost_http_port：虚拟主机运行在本机的端口，如果 vps 有服务占用了端口，应当更换
- dashboard_port：frp 后台服务页面的端口，如果设置 8000，便可通过 [http://yourip:8000](https://link.zhihu.com/?target=http%3A//yourip%3A8000/) 来访问 frps 的后台页面
- dashboard_user：frp 后台服务页面的管理员用户名
- dashboard_pwd：frp 后台服务页面的管理员密码
- privilege_token：自定义值，必须与客户端中的 privilege_token 保持一致

在 VPS 上启动服务端 frps

```bash
./frps -c ./frps.ini
```

开机自动启动 FRP

新建一个 Service

```bash
nano /lib/systemd/system/frps.service
```

内容为

```text
[Unit]
Description=frps service
After=network.target network-online.target syslog.target
Wants=network.target network-online.target

[Service]
Type=simple
ExecStart=/root/frp/frps -c /root/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

启动 FRP

```bash
systemctl start frps
```

设置自动启动

```bash
systemctl enable frps
```

### 树莓派客户端设置

下载对应的 release

```bash
wget https://github.com/fatedier/frp/releases/download/v0.31.2/frp_0.31.2_linux_arm.tar.gz
```

解压

```bash
tar -zxvf frp_0.31.2_linux_arm.tar.gz
```

修改文件夹名称为`frp`

转到对应目录

```bash
cd frp
```

打开修改配置文件

```bash
nano frpc.ini
```

配置文件内容如下

```text
[common]
server_addr = your-vps-ip
server_port = 7000
privilege_token = yourtoken
login_fail_exit = false

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

参数说明

- server_addr：服务器端的 ip
- server_port：服务器端的端口，即 bind_port
- privilege_token：同服务器端的 privilege_token 保持一致
- login_fail_exit：失败时自动重连
- remote_port：远程端口，即 ssh 连接树莓派时的端口

启动树莓派客户端 frpc

```bash
sudo ./frpc -c ./frpc.ini
```

设置树莓派开机自启

新建一个 Service

```bash
nano /lib/systemd/system/frpc.service
```

内容为

```text
[Unit]
Description=frpc service
After=network.target network-online.target syslog.target
Wants=network.target network-online.target

[Service]
Type=simple
ExecStart=/home/pi/frp/frpc -c /home/pi/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

启动 frpc

```bash
systemctl start frpc
```

设置开机自动启动

```bash
systemctl enable frpc
```

### 外网 SSH 登陆树莓派

VPS IP@remote_port 端口

## 树莓派 Aria 2

Aria 2 是一款小巧且支持协议全面的下载应用。

### 安装

```bash
sudo apt-get update
sudo apt-get install aria2
```

### 配置

创建配置文件

```bash
mkdir -p ~/.config/aria2/
touch ~/.config/aria2/aria2.session
vi ~/.config/aria2/aria2.config
```

配置文件设置如下

```text
# set your own path
dir=[yourpath]
disk-cache=32M
file-allocation=trunc
continue=true

max-concurrent-downloads=10

max-connection-per-server=16
min-split-size=10M
split=5
max-overall-download-limit=0
#max-download-limit=0
#max-overall-upload-limit=0
#max-upload-limit=0
disable-ipv6=false

save-session=/home/pi/.config/aria2/aria2.session
input-file=/home/pi/.config/aria2/aria2.session
save-session-interval=60


enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
#rpc-secret=secret
#event-poll=select
#rpc-listen-port=6800


# for PT user please set to false
enable-dht=true
enable-dht6=true
enable-peer-exchange=true

# for increasing BT speed
listen-port=51413
#follow-torrent=true
#bt-max-peers=55
#dht-listen-port=6881-6999
#bt-enable-lpd=false
#bt-request-peer-speed-limit=50K
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
seed-ratio=0
#force-save=false
#bt-hash-check-seed=true
bt-seed-unverified=true
bt-save-metadata=true
bt-tracker=udp://62.138.0.158:6969/announce,udp://87.233.192.220:6969/announce,udp://111.6.78.96:6969/announce,udp://90.179.64.91:1337/announce,udp://51.15.4.13:1337/announce,udp://151.80.120.113:2710/announce,udp://191.96.249.23:6969/announce,udp://35.187.36.248:1337/announce,udp://123.249.16.65:2710/announce,udp://210.244.71.25:6969/announce,udp://78.142.19.42:1337/announce,udp://173.254.219.72:6969/announce,udp://51.15.76.199:6969/announce,udp://51.15.40.114:80/announce,udp://91.212.150.191:3418/announce,udp://103.224.212.222:6969/announce,udp://5.79.83.194:6969/announce,udp://92.241.171.245:6969/announce,udp://5.79.209.57:6969/announce,udp://82.118.242.198:1337/announce
```

测试aria2是否成功启动

```bash
aria2c --conf-path=/home/pi/.config/aria2/aria2.config
ps aux | grep aria2
```

若看到如下内容，则启动成功

``` bash
pi       21128  0.0  0.0   7348   536 pts/0    S+   17:10   0:00 grep --color=auto aria2
```

### 开机启动

创建systemctl service文件

```bash
sudo nano /lib/systemd/system/aria2.service
```

User, conf-path 下换成自己的 username

```
[Unit]
Description=Aria2 Service
After=network.target

[Service]
User=ubuntu
ExecStart=/usr/bin/aria2c --conf-path=/home/ubuntu/.config/aria2/aria2.config

[Install]
WantedBy=default.target
```

重载服务并设置开机启动

```
sudo systemctl daemon-reload
sudo systemctl enable aria2
sudo systemctl start aria2
sudo systemctl status aria2
```

看到如下文字证明启动成功（记住TCP port，AiraNg配置以及公网端口映射需要）

```
● aria2.service - Aria2 Service
   Loaded: loaded (/lib/systemd/system/aria2.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-12-29 08:24:20 CST; 4 days ago
 Main PID: 21855 (aria2c)
   CGroup: /system.slice/aria2.service
           └─21855 /usr/bin/aria2c --conf-path=/home/ubuntu/.config/aria2/aria2.config

Dec 29 08:24:20 ubuntu systemd[1]: Started Aria2 Service.
Dec 29 08:24:20 ubuntu aria2c[21855]: 12/29 08:24:20 [NOTICE] IPv4 RPC: listening on TCP port 6800
Dec 29 08:24:20 ubuntu aria2c[21855]: 12/29 08:24:20 [NOTICE] IPv6 RPC: listening on TCP port 6800
Dec 29 08:25:21 ubuntu aria2c[21855]: 12/29 08:25:21 [NOTICE] Serialized session to '/home/ubuntu/.config/aria2/aria
```

### aria2 的 web 管理界面（AriaNg）

安装 Git 和 Nginx

```bash
sudo apt install -y git nginx
```

下载 aira-ng

```bash
wget https://github.com/mayswind/AriaNg/releases/download/1.1.4/AriaNg-1.1.4.zip -O aria-ng.zip
```

解压

```bash
unzip aria-ng.zip -d aria-ng
```

将 aria-ng 放到 nginx 的 `/var/www/html/` 目录下，然后设置开机启动 nginx

```bash
sudo mv aria-ng /var/www/html/
sudo systemctl enable nginx
```

用浏览器访问树莓派 IP 下的 aira-ng，即：`http://yourraspberrypi-ip/aira-ng`
然后在 `系统设置` 点击 `AriaNg设置` –> `全局` –> 设置语言为中文 –> 点击`RPC`–>修改为 rpc 密钥：`secret`

### 设置端口映射实现外网访问 AriaNg 页面

在树莓派本地的 frpc 配置中增加以下内容

```bash 
nano ~/frp/frpc.ini
```

```text
[aria2]
local_ip = 127.0.0.1
local_port = 6800
remote_port = 6800

[http]
type = tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 8800
```

以上配置即增加 Frp 对树莓派 80 端口的映射，从而在外网通过地址 `remote_ip:remote_port/aria-ng` 即可访问树莓派 AriaNg 页面，如 `your-vps-ip:8800/aria-ng`

## 搭建 Git 服务器

### 安装 Git

```bash
sudo apt-get install git-core
```

### 创建用户

为树莓派创建一个 git 用户，方便局域网内其他用户使用同时将 git 服务器文件与 pi 用户数据隔离开来。命令行如下

```bash
sudo adduser --system --bash /bin/bash --gecos 'git version control by pi' --group --home /home/git git
```

这个命令在 `/home/git` 目录下创建一个 git 用户

更改 git 密码

```bash
sudo passwd git
```

切换到 git 用户

```bash
su git
```

### 初始化仓库

git 用户负责 git 项目的管理，这里将所有仓库存放在 /home/git 中，初始化一个空仓库（在切换到 git 用户之后）：

```bash
cd /home/git
mkdir test.git
cd test.git
git --bare init
```

### 使用仓库

以上树莓派服务端配置完成后，就可以在本地客户端使用仓库了。

在客户端中，可以直接使用相对应的仓库，例如上述的 test.git

```bash
git clone git@your_raspi_ip:/home/git/test.git
```

其中，your_raspi_ip 是你的树莓派 IP 地址。

## 安装 Pi-Board

Pi-Board 是现实树莓派信息的网络面板，将其安装到树莓派的网页文件夹 `/var/www/html` 中

```bash
cd /var/www/html
```

使用 Git 获取

```bash
sudo git clone https://github.com/spoonysonny/pi-dashboard.git
```

为 Pi-Dashboard 赋予权限

```bash
sudo chown -R www-data pi-dashboard
```

浏览器访问 `localhost/pi-dashboard` 即可看到树莓派信息面板。

## 查看树莓派 CPU 温度

```bash
/opt/vc/bin/vcgencmd measure_temp
```

## 最后

以上是目前我在树莓派上安装过的服务和用过的命令等等，主要只是实现了局域网内文件共享的功能。作为一台拥有丰富接口和（相对）强大性能的单板电脑，基于 Linux 系统，树莓派能做的事情远不止于此，因此对于树莓派，我依然在持续折腾与学习中，后续会持续记录这个过程。