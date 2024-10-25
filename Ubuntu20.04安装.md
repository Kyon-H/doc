# Ubuntu20.04 lts 桌面版 安装

下载地址：https://releases.ubuntu.com/20.04.6/ubuntu-20.04.6-desktop-amd64.iso
## 一、系统制作与安装
### 1.U盘制作

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89ba4021d.png" alt="image-20240822163714875.png" title="image-20240822163714875.png" style="zoom: 80%;" />

[烧录](https://ghostliner.v6.army:40027/i/2024/10/15/670e89ba4021d.png)

重启电脑按`F2` 选择U盘启动，若打印信息的最后一行报错，重启按`F12` 进入BIOS，`security`>`secure boot` 选择 `disable` ，`F10` 保存并退出。
### 2.硬盘分区

| 挂载    | 格式   | 类型          | 分区大小    |
| ----- | ---- | ----------- | ------- |
| efi   | efi  |             | 500MB   |
| /boot | ext4 | 主分区(gpt可不分) | 1024MB  |
| /tmp  | ext4 | 逻辑          | 5120MB  |
| /     | ext4 | 逻辑或主分区      | 51200MB |
| /home | ext4 | 逻辑          | 剩余空间    |
| swap  | swap |             | 8192MB  |
#### 将EFI启动分区迁移到另一块硬盘

[Ubuntu22.04 将EFI启动分区迁移到另一块硬盘_移动efi分区](https://blog.csdn.net/michaelchain/article/details/130660081) 

<mark>问题：efi分区误装到windows下C盘，Ubuntu硬盘的efi空间未使用</mark>
<mark>现状：Ubuntu硬盘预留有efi空间(exFAT格式)，预留 /boot/efi 分区为 /sdb/sdb1</mark> 

```shell
# 找到 sdb1 的 UUID
sudo blkid | grep /dev/sdb1
# 在 /etc/fstab 中, 将 /boot/efi 的 UUID 修改为刚才获得的 UUID 值
# ctrl+o 写，ctrl+x 退出
sudo nano /etc/fstab
# 从系统中卸载 /boot/efi, 再重新mount
sudo umount /boot/efi && sudo mount /boot/efi
# 确认 mount 的硬盘分区
lsblk | grep /boot/efi
# 在 sdb (硬盘, 不是分区) 上安装 grub
sudo grub-install /dev/sdb
# 生成 initramfs image
sudo update-initramfs -u -k all
sudo reboot
```
## 二、系统配置

### 1.设置中文语言、中文输入法，保留旧的文件名

```shell
sudo apt update
sudo apt upgrade
sudo passwd root
# 安装vim curl nmap sensors hddtemp
sudo apt install vim
```

[VIM常用快捷键 - markleaf - 博客园 (cnblogs.com)](https://www.cnblogs.com/markleaf/p/7808817.html) 
#### 设置命令行界面为英文

```bash
vim ~/.bashrc
```

添加
```bash
export LANGUAGE=en_US 
export LANG=en_US.UTF-8 
```

激活
```bash
source ~/.bashrc
```
### 2.配置源
#### 清华源

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/) 

``vim /etc/apt/sources.list``

```shell
# tsinghua
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# 安全更新软件源
deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
# tsinghua
```
### 3.防火墙

```bash
ufw status
ufw enable|disable

ufw allow|deny <port>/<tcp|udp>
ufw allow from <ipaddr>
ufw allow proto <tcp|udp> from <ip>/24 to any port <port>

ufw delete allow|deny <port|...>
```
### 4.查看运行的服务

```bash
service --status-all
ps -ef | grep xxx
systemctl status xxx
```
### 5.传感器监测软件

**sensors ：**[如何在Linux系统中查看CPU温度|极客教程 (geek-docs.com)](https://geek-docs.com/linux/linux-ask-answer/176_tk_1703986369.html)  
**hddtemp : *`hddtemp /dev/sda /dev/sdb` 
### 6.GUI开启与关闭

**设置系统默认启动目标为多用户模式:** 

```
sudo systemctl set-default multi-user.target
reboot
```

**设置系统默认启动目标为图形模式:** 

```
sudo systemctl set-default graphical.target
reboot
```
### 7.系统代理

创建一个脚本文件 `/etc/profile.d/proxy.sh`，并添加以下内容：

```bash
export http_proxy="http://your_proxy:port"
export https_proxy="http://your_proxy:port"
export ftp_proxy="http://your_proxy:port"
export no_proxy="127.0.0.1,localhost"
```

保存并关闭文件，然后赋予执行权限：

```
sudo chmod +x /etc/profile.d/proxy.sh
```

激活代理设置：

```
source /etc/profile.d/proxy.sh
```
## 三、软件
### 1.v2raya

[Debian / Ubuntu - v2rayA](https://v2raya.org/docs/prologue/installation/debian/) 

[v2raya_installer_debian_x64_2.2.5.8.deb](packages/v2raya_installer_debian_x64_2.2.5.8.deb) 

```
sudo dpkg -i installer_debian_x64_2.2.5.8.deb
```

<mark>检测到 geosite.dat, geoip.dat 文件或 v2ray-core 可能未正确安装</mark> 

[检测到 geosite.dat, geoip.dat 文件或 v2ray-core 可能未正确安装 – 爱思考的人 (aisikao.ren)](https://aisikao.ren/22633/) 
```shell
wget https://github.com/v2fly/v2ray-core/releases/latest/download/v2ray-linux-64.zip

unzip v2ray-linux-64.zip -d ./v2ray

sudo mkdir -p /usr/local/share/v2ray

sudo cp ./v2ray/*dat /usr/local/share/v2ray

sudo install -Dm755 ./v2ray/v2ray /usr/local/bin/v2ray

# 启动v2raya
sudo v2raya
# 自启动
sudo systemctl enable --now v2raya.service # now为现在就启动
```

访问v2raya：http://192.168.0.222:2017

### 2.qBittorrent Enhanced Edition

<mark>qbittorrent EE没有显示器时无法启动，qbittorrent原版可以，两者共享配置文件：~/.config/qBittorrent/</mark> 

[qBittorrent Enhanced Edition : poplite (launchpad.net)](https://launchpad.net/~poplite/+archive/ubuntu/qbittorrent-enhanced) 

[poplite/qBEE-Ubuntu-打包 (github.com)](https://github.com/poplite/qBEE-Ubuntu-Packaging) 

```shell
sudo apt-get update && sudo apt-get install software-properties-common -y
# qBittorrent
sudo add-apt-repository ppa:poplite/qbittorrent-enhanced
sudo apt-get update
sudo apt-get install qbittorrent-enhanced qbittorrent-enhanced-nox
# 启动
sudo service qbittorrent-enhanced-nox start
```

设置启动用户

```
cd /lib/systemd/system/
vim qbittorrent-enhanced-nox.service
```

设置默认保存路径

设置Torrent完成时运行外部脚本
#### BitTorrent

 trackers list:  `https://trackerslist.com/all.txt` 

tracker : [tracker](tracker.md) 
#### rss订阅

间隔30分钟  1000条

```
# 音乐
http://www.kisssub.org/rss-3.xml
# 爱恋动漫
http://www.kisssub.org/rss-1.xml
# Nyaa Pantsu
https://ouo.si/feed?c=_&q=baha
```

自动下载规则（2024.08.23）： [rssconf.json](packages/rssconf.json) 
#### Web UI

设置`0.0.0.0`访问

设置端口`8088`

访问：http://192.168.0.222:8088

第三方UI

[VueTorrent/VueTorrent: The sleekest looking WEBUI for qBittorrent made with Vuejs! (github.com)](https://github.com/VueTorrent/VueTorrent) 
[jesec/flood: A modern web UI for various torrent clients with a Node.js backend and React frontend. (github.com)](https://github.com/jesec/flood) 

下载vuetorrent： [vuetorrent.zip](packages/vuetorrent.zip) 

```
#解压
unzip vuetorrent.zip
#移动vuetorrent文件夹到存放目录
mv vuetorrent /home/load/Templates
#设置文件夹权限
chmod 2775 public/
```

设置文件夹权限如下所示：

```
$ ls -lh
total 8.0K
drwxrwsr-x+ 3 larsluph users 4.0K Jan 20 16:00 public
-rw-rw-r--  1 larsluph users    5 Jan  9 13:45 version.txt
```

<mark>若设置错误导致页面无法显示：</mark>

```
#修改qBittorrent.conf文件
cd /home/load/.config/qBittorrent
vim qBittorrent.conf

#WebUI\AlternativeUIEnabled 改为false
WebUI\AlternativeUIEnabled=false
#WebUI\RootFolder 删去后面
WebUI\RootFolder=

#最后刷新或重启
```

qbittorrent设置备用webui地址：`/home/load/Templates/vuetorrent/`
#### 高级

1.

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e899986cee.png" alt="image-20240820230738951.png" title="image-20240820230738951.png" style="zoom:80%;" />

2.

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89b7efd58.png" alt="image-20240820230937953.png" title="image-20240820230937953.png" style="zoom:80%;" />

3.

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89b59863a.png" alt="image-20240820231006390.png" title="image-20240820231006390.png" style="zoom:80%;" />

4.

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89b8f1f38.png" alt="image-20240820231030176.png" title="image-20240820231030176.png" style="zoom:80%;" />

5.

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89b9a8e6d.png" alt="image-20240820231044716.png" title="image-20240820231044716.png" style="zoom:80%;" />
#### PeerBanHelper

[Linux 手动部署 · PBH-BTN/PeerBanHelper Wiki (github.com)](https://github.com/PBH-BTN/PeerBanHelper/wiki/Linux-手动部署) 

```shell
# 使用 apt 工具安装 OpenJDK 21 或更高版本：
sudo apt-get update
sudo apt-get install openjdk-21-jdk-headless -y
# 验证安装
java -version
```

下载jar包：  [PeerBanHelper.jar](packages/PeerBanHelper_6.2.1.jar) 

使用命令来启动 PBH：

```shell
java -jar -Xmx256M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -jar PeerBanHelper.jar
```

_通常情况下，PBH 会自动探测桌面环境，并在支持的情况下启用 GUI，但在部分设备上，GUI 可能会初始化失败。遇到此类情况，请手动禁用 GUI：_ 

```shell
java -jar -Xmx256M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -jar PeerBanHelper.jar nogui
```

访问：http://192.168.0.222:9898

设置开机自启

`cd /etc/systemd/system && vim peerbanhelper.service`

```
[Unit]
Description=Start PeerBanHelper jar file
After=multi-user.target

[Service]
ExecStart=/usr/bin/java -Xmx386M -XX:+UseG1GC -XX:+UseStringDeduplication -XX:+ShrinkHeapInSteps -Dfile.encoding=UTF-8 -Dstdout.encoding=UTF-8 -Dstderr.encoding=UTF-8 -Dconsole.encoding=UTF-8 -jar PeerBanHelper.jar nogui
Type=simple
# 设置PeerBanHelper.jar存放和工作目录
WorkingDirectory=/home/load/bin/peerbanhelper

[Install]
WantedBy=multi-user.target
```
### 3.SSH

https://developer.aliyun.com/article/1488008

```
sudo apt install openssh-server
sudo apt install openssh-client
# 检查 SSH 服务器的状态
sudo systemctl status ssh
```

配置 SSH 服务器，更改 SSH 服务器的监听端口、允许或禁止密码登录、限制登录用户等。

```
vim /etc/ssh/sshd_config

Port	# 端口
PermitRootLogin yes	# 禁止root登录
PubkeyAuthentication yes	# 密钥认证登录
PasswordAuthentication no	# 禁止密码登录，配置好秘钥登陆后再设置
AllowUsers	# 允许ssh登陆的用户
```

秘钥登录

_远程客户端_ 

```
ssh-keygen	# 生成id_rsa密钥对
cd ~/.ssh;ls
cat id_rsa.pub
```

_复制内容到服务端 ~/.ssh/authorized_keys文件中_，检查权限 

```
chmod 700 .ssh
chmod 644 authorized_keys
```

防火墙设置

```
# 检查防火墙状态
sudo ufw status
# 允许 SSH 通过防火墙
sudo ufw allow OpenSSH
# 启用防火墙
sudo ufw enable
```
### 4.Sunshine

<mark>必须要有独立显卡，安装cuda</mark> 

 [sunshine-ubuntu-20.04-amd64.deb](packages/sunshine-ubuntu-20.04-amd64.deb) 

访问：https://192.168.0.222:47990
#### 安装虚拟显示器

[win远程桌面连接无显示器Ubuntu（22.04.1 LTS）_ubuntu 22.04 虚拟显示器-CSDN博客](https://blog.csdn.net/weixin_43983431/article/details/128793711 ) 

[Ubuntu20.04 虚拟显示器配置(解决无显示器远程黑屏问题)-腾讯云](https://cloud.tencent.com/developer/article/2120950) 

```
sudo apt install xserver-xorg-core-hwe-18.04
sudo apt install xserver-xorg-video-dummy
```

配置文件

`vim /usr/share/X11/xorg.conf.d/xorg.conf` 

```
Section "Monitor"
  Identifier "Monitor0"
  HorizSync 28.0-80.0
  VertRefresh 48.0-75.0
  Modeline "1920x1080_60.00" 172.80 1920 2040 2248 2576 1080 1081 1084 1118 -HSync +Vsync
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
    Modes "1920x1080_60.00"
  EndSubSection
EndSection
```

最后重启
#### 恢复显示器显示

[ubtunu开机黑屏无桌面解决方法_ubtun 待机黑屏-CSDN博客](https://jrhar.blog.csdn.net/article/details/108468903) 

~~开机打印信息时 `Ctrl+Alt+F1` （按住ctrl和alt连续按f1）进入登陆界面（字体格式会发生变化，变细），`Ctrl+Alt+F3` （按住ctrl和alt连续按f3）进入tty界面登录~~

开机打印信息时 `Ctrl+Alt+F3` 进入命令行登陆界面

```
sudo rm /usr/share/X11/xorg.conf.d/xorg.conf
sudo reboot
```
### 5.VNCServer远程桌面

<mark>核显即可</mark> 

安装gnome-session和vnc服务端

```
sudo apt install gnome-session-flashback
sudo apt install tigervnc-standalone-server

# 创建xstartup文件
mkdir ~/.vnc/
vim ~/.vnc/xstartup

# 添加执行权限
chmod +x ~/.vnc/xstartup
```

xstartup内容：

```sh
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
export XKL_XMODMAP_DISABLE=1
export XDG_CURRENT_DESKTOP="GNOME-Flashback:GNOME"
export XDG_MENU_PREFIX="gnome-flashback-"
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
# 窗口参数设置
xsetroot -solid grey
# 运行复制粘贴
vncconfig -iconic &

gnome-terminal &
# 打开文件管理器
nautils &

gnome-session --session=gnome-flashback-metacity --disable-acceleration-check &
```

启动vnc服务端

```
# 启动并获取端口号
vncserver && vncserver -list
# 允许非本地连接
vncserver :1 -localhost no
```
#### <a id="vncserver_start">设置开机自启</a> 

创建vncserver_start启动脚本

```
vim /etc/init.d/vncserver_start
```

```sh
#!/bin/sh
### BEGIN INIT INFO
# Provides:          $vncserver_start
# Required-Start:    $local_fs
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/stop vncserver_start
### END INIT INFO
 
# More details see:
# http://www.penguintutor.com/linux/tightvnc
 
### Customize this entry
# Set the USER variable to the name of the user to start tightvncserver under
export USER='load' #自定义
### End customization required
 
eval cd ~$USER
 
case "$1" in
    start)
         su $USER -c '/usr/bin/vncserver -localhost no' #自定义
         echo "Starting VNC server for $USER "
         ;;
    stop)
 
         su $USER -c '/usr/bin/vncserver -kill :1'
         echo "vncserver stopped"
         ;;
    *)
         echo "Usage: /etc/init.d/vncserver {start|stop}"
         exit 1
         ;;
esac
exit 0
```

给文件加权限

```
chmod 777 /etc/init.d/vncserver_start
update-rc.d vncserver_start defaults
#重载服务
systemctl daemon-reload
systemctl enable vncserver_start.service
```
### 6.Tailscale

https://tailscale.com/download/linux

```curl -fsSL https://tailscale.com/install.sh | sh```

```
# 添加 Tailscale 的包签名密钥和仓库：
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/focal.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/focal.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list

# 安装 Tailscale：
sudo apt-get update
sudo apt-get install tailscale
```

```
# 将您的计算机连接到 Tailscale 网络并在浏览器中进行身份验证：
sudo tailscale up
# 您已连接！您可以通过运行以下命令找到您的 Tailscale IPv4 地址：
tailscale ip -4
# 查看各主机IPv4地址，连接状态，连接方式
tailscale status
```
#### Subnets

[Subnet routers · Tailscale Docs](https://tailscale.com/kb/1019/subnets) 
#### Exit-nodes

[Exit nodes (route all traffic) · Tailscale Docs](https://tailscale.com/kb/1103/exit-nodes) 
### 7.ZeroTier

[Download - ZeroTier](https://www.zerotier.com/download/) 

[ubuntu下zerotier的基本使用教程](https://www.wlplove.com/archives/34/) 

```
curl -s https://install.zerotier.com | sudo bash
```

```
sudo zerotier-cli join 9e1948db63da2436
# 查看当前连接的网络，如果列表中出现网络号说明连接成功
sudo zerotier-cli listnetworks
```

[ZeroTier Central](https://my.zerotier.com/network) 
### 8.Alist

[一键脚本 | AList文档 (nn.ci)](https://alist.nn.ci/zh/guide/install/script.html) 
#### 脚本安装

```shell
# 安装
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install
# 更新
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s update
# 卸载
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s uninstall

systemctl start alist
```
#### 手动安装

 [alist-3.36-linux-amd64.tar.gz](packages/alist-linux-amd64.tar.gz) 

```shell
# 解压下载的文件，得到可执行文件：
tar -zxvf alist-linux-amd64.tar.gz
# 移动文件到相应目录，授予程序执行权限：(默认安装位置 /opt/alist)
chmod +x alist

# 运行程序
./alist server
# 高于v3.25.0版本
# 手动设置一个密码 `NEW_PASSWORD`是指你需要设置的密码
./alist admin set NEW_PASSWORD
```

访问Alist: http://192.168.0.222:5244
#### 全局设置

**前端隐藏文件：**`设置` > `全局` > `隐藏文件` 

```
/\/README.md/i
/.*\.(nfo|ass|srt|svg|ini|config)/
/.*thumb.jpg/
```

**设置Markdown显示本地图片：**在 `设置` > `全局` > `自定义头部` 增加这段代码即可。

```javascript
<script>
  const getCookie = name => (
    document.cookie.match(`(^|;)\\s*${name}\\s*=\\s*([^;]+)`)?.pop() ?? ''
  );

  const fsGet = path => (
    fetch('/api/fs/get', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json;charset=UTF-8',
        'Authorization': localStorage.getItem('token') ?? ''
      },
      body: JSON.stringify({
        path: decodeURIComponent(path),
        password: getCookie('browser-password')
      })
    })
      .then(response => response.json())
  );

  window.addEventListener('error', async event => {
    const { target } = event;
    if (target.tagName.toUpperCase() !== 'IMG' || 'fix' in target.dataset) return;
    const src = target.getAttribute('src');
    if (/^(?:[a-z+]+:)?\/\//i.test(src)) return;
    const pathname = location.pathname.replace(/\/+$/, '');
    const base = await fsGet(pathname);
    const url = new URL(src, `${origin}${pathname}${base.data.is_dir ? '/' : ''}`);
    const file = await fsGet(url.pathname);
    const raw = file.data?.raw_url;
    if (!raw) return;
    target.src = raw;
    target.dataset.fix = '';
  }, true);
</script>
```
#### 索引

[Alist如何将数据库由Sqlite3修改为MySQL？-马春杰杰 (machunjie.com)](https://www.machunjie.com/linux/1396.html) 
### 9.Foobar 2000

<mark>启动foobar2000需要显示器，无显示器可安装vncserver，并 [设置vncserver开机自启](#vncserver_start)  </mark> 
#### 安装wine
#### 安装foobar2000

 [foobar2000_v1.6.11.exe](packages/foobar2000_v1.6.11.exe) 

 [foobox_6.1.6.11.exe](packages/foobox_6.1.6.11.exe) 

插件：[foobar插件](packages/foobar) 

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89aa74953.png" alt="image-20240823151625647.png" title="image-20240823151625647.png" style="zoom:110%;"/>

布局：[2024.08.23type2.fth](packages/foobar/2024.08.23type2.fth) 
#### 设置自启

创建启动脚本

```
vim /home/load/bin/foobar2000_start.sh
```

内容：

```sh
#!/bin/bash
sleep 60 #延时60秒
export DISPLAY=:1
xhost +
wine /home/load/Desktop/foobar2000/foobar2000.exe
```

设置权限

```
chmod +x /home/load/bin/start_foobar2000.sh
```

创建 systemd 服务文件

```
sudo vim /etc/systemd/system/foobar2000.service
```

内容：

 [设置vncserver开机自启](#vncserver_start) 

```
[Unit]
Description=Foobar2000
After=network.target network.service vncserver_start.service

[Service]
Type=simple
User=load
KillMode=process
ExecStart=/home/load/bin/foobar2000_start.sh
Restart=always

[Install]
WantedBy=default.target
```

```
sudo systemctl enable foobar2000.service
sudo systemctl start foobar2000.service
```
### 10.1Panel

一键安装：

```
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```
### 11.Jellyfin

**导入 Jellyfin APT 存储库**

```
# 安装 Jellyfin 媒体服务器的初始软件包
sudo apt install apt-transport-https ca-certificates curl -y

# 导入 Jellyfin Media Server APT 存储库
#入 Jellyfin GPG 密钥：
curl -fsSL https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/jellyfin.gpg > /dev/null

#导入 Jellyfin 存储库：
echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release ) $( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list

sudo apt update
```

**安装**

```
sudo apt install jellyfin
systemctl status jellyfin
sudo systemctl enable jellyfin
```

**卸载** 

```
dpkg -l | grep jelly
sudo dpkg --purge jellyfin-xxx
```
### 12.Python3.11.9

<mark>不推荐直接安装新版本python。应使用系统默认python</mark> 
#### pdb3

启动pdb调试

```
pdb3 xx.py
```

```
help 命令名

l == list : list code
ll == longlist : the function code
n == next : next code
s == step : 
b xx.py:n == break xx.py:n
c == continue : 下一个断点
cl == clear cl n clear n breakpoint clear :clearall
disable|enable n 
until n
p 变量 : 查看变量值
pp 格式化输出
w == where : 查看堆栈
up 切换到上一次上下文
down
```
### 13.Resillo Sync

[保姆级教程:在Linux上安装配置Resilio Sync_resilio sync linux-CSDN博客](https://blog.csdn.net/wy_bk/article/details/112892938) 

[Installing Sync package on Linux – Resilio Sync](https://help.resilio.com/hc/en-us/articles/206178924-Installing-Sync-package-on-Linux) 
### 14.VS Code

[VSCode - 使用VSCode远程连接到Linux并实现免密码登录_vscode连接linux-CSDN博客](https://blog.csdn.net/weixin_42490414/article/details/117750075) 
#### code-erver


### 15.Samba共享

1. [【详细步骤】Ubuntu安装Samba服务及配置共享文件夹_ubuntu samba-CSDN博客](https://blog.csdn.net/qq_44078824/article/details/119847027) 

2. [在 Ubuntu 22.04|20.04|18.04 上安装和配置 Samba 共享 (linux-console.net)](https://cn.linux-console.net/?p=21494) 

```
# 安装
sudo apt install samba samba-common
# 添加用户 设置 SMB 密码
sudo smbpasswd -a load
# 修改配置
sudo vim /etc/samba/smb.conf
```

文件最后添加内容：

```
[share] # 挂载目录名
comment = share folder
browseable = yes
# 实际路径
path = /home/load/bangumi
create mask = 0700
directory mask = 0700
valid users = load
force user = load
force group = load
public = yes
available = yes
writable = no
```

保存后重启服务

```
sudo service smbd restart
```

Windows访问

Win+R 或文件管理器输入 `\\192.168.0.222` 正常显示`share`目录

<mark>若无法访问，查看服务器防火墙、客户端防火墙（出站规则）、文件夹权限等</mark> 
### 16.MySQL

```
sudo apt install mysql-server-8.0
# 设置密码
sudo mysql -uroot -p	#无密码直接回车
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
exit

sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address            = 0.0.0.0

sudo systemctl restart mysql
```
## 四、挂载
### 1.Windows挂载ext4

方式一：安装`Paragon ExtFS for Windows` 软件

方式二：WSL挂载

管理员模式启动PowerShell，查询DeviceID：

```
GET-CimInstance -query "SELECT * from Win32_DiskDrive"
```

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89bad4c33.png" alt="image-20240823124817697.png" title="image-20240823124817697.png" style="zoom:80%;" />

挂载：

```
wsl.exe --mount \\.\PHYSICALDRIVE2 --bare
```

进入WSL系统：

```shell
lsblk # 查询硬盘分区名
mkdir /data # 创建挂载目录
mount -t ext4 /dev/sdc5 /data
ls /data
```

卸载：

```shell
umount /data
```

```powershell
wsl.exe --unmount \\.\PHYSICALDRIVE2
# or
wsl.exe --shutdown
# or
wsl.exe --terminate kali
```

<mark>使用文件管理器移动文件时会产生 *.Identifier </mark> 

```
find . -name "*.Identifier" -exec rm -r "{}" \;
```

<mark>umount: /data: target is busy.</mark>

```
fuser -m /data # 列出正在使用该挂载点的进程
fuser -km /data # 终止这些进程

lsof /data # 显示哪些文件正在被使用

umount -f /data # 强制卸载
umount -l /data # 延迟卸载
```
### 2.Ubuntu挂载ntfs

```
lsblk # 查找到需要挂载的硬盘分区名 nvme0n1p5
sudo fdisk -l | grep nvme0n1p5 # 获取路径
sudo mount -t ntfs /dev/nvme0n1p5 /home/load/ddisk
```
### 3.Ubuntu挂载扩容

[linux挂载硬盘到home目录下（home目录扩容） - 明矾 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mingfan/p/15493270.html) 

```
su root
mkdir /mnt/load
mount /dev/sda2 /mnt/load
cp -a /home/load/* /mnt/load/
mv /home/load /home/load.old
rm -rf /home/load/*
umount /dev/sda2
df -h
mkdir /home/load
```

设置开机挂载

```
vim /etc/fstab
# i 添加
/dev/sda2 /home/load ext4 defaults 1 2
# :wq 保存退出
mount -a ; df -h
```
### 4.根目录扩容

sdb4 /根目录，sdb5 /home目录，sdb5紧挨在sdb4右侧可无损扩容

`sudo fdisk -l /dev/sdb` 查看分区情况，记住 **设备**、**起点**、**末尾** 

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89ae608d2.png" alt="image-20240930212900132.png" title="image-20240930212900132.png" />

<mark>以root身份执行</mark> 

* **备份sdb5数据** 

关闭相关服务、进程，卸载/home下的其他分区

```
#卸载sda3
lsof /dev/sda3 #查看占用分区的进程
kill -9 PID
umount /dev/sda2
#卸载sda2
umount /dev/sda2
#卸载sdb5分区
umount /dev/sdb5

mount /dev/sdb5 /mnt
mv /mnt/* /home
unmount /dev/sdb5

#修改fstab文件，删去/dev/sdb5挂载信息
vim /etc/fstab
```

* **删除sdb5分区** 

<mark>MBR使用fdisk，GPT使用gdisk</mark>

```
gdisk /dev/sdb
:? #显示帮助
:p #打印分区信息
:d #删除指定分区
:5 #分区号
:w #保存
 ```

* **扩容是sdb4分区** 

```
gdisk /dev/sdb
:d #删除sdb4分区（未w保存，数据不会消失、程序仍正常运行）
:4
:n #添加sdb4分区
:4
:直接回车 #设置起点扇区，与原sdb4起点相同
:直接回车、输入扇区值、+10G指定空间大小 #末尾扇区，小于sdb6起点
:w #保存
```

* **刷新** 

```
partprobe -s
```

重启：<mark>修改根目录时必须</mark> 

使用resize2fs重新定义文件系统大小（这步一定要做，要不然容量不会有变化）

```
resize2fs /dev/sdb4
fdisk -l /dev/sdb
```

<img src="https://ghostliner.v6.army:40027/i/2024/10/15/670e89af3781d.png" alt="image-20240930220907160.png" title="image-20240930220907160.png" />
## 五、公网访问
### 1.ipv6申请域名

https://dynv6.com 邮箱注册，使用梯子验证邮箱，注册zone主域名 `ghostliner.v6.army`
点击 账户 -> keys ,生成`HTTP Tokens` (用于更新ip地址) 和 `TSIG Keys` (用于生成证书，使用https) 
### 2.DDNS-GO更新ip

docker安装ddns-go，初次启动设置用户名、密码。
修改配置：
**DDNS服务商：** CallBack
URL：https://dynv6.com/api/update?hostname=ghostliner.v6.army&token=\<http token>&ipv6=auto
RequestBody：ghostliner.v6.army
TTL：自动

**IPv4不启用**

**IPv6启用**
获取 IP 方式：通过网卡获取
匹配正则表达式：@1
Domains：ghostliner.v6.army <mark>无法更新子域名，需要使用ssh更新</mark>
### 3.启用HTTPS
#### swag申请证书

[dynv6 域名启用SSL with LET'S ENCRYPT - 知乎](https://zhuanlan.zhihu.com/p/435603828)
安装swag docker拉取镜像 `linuxserver/swag` 
绑定80和443端口
挂载/config目录
设置环境变量
```
PUID=1000
PGID=1000
TZ=Asia/Shanghai
URL=ghostliner.v6.army
SUBDOMAINS=wildcard
VALIDATION=dns
DNSPLUGIN=rfc2136
```

**修改rfc2136.ini文件** 
```bash
# Target DNS server
dns_rfc2136_server = ns1.dynv6.com
# TSIG key name
dns_rfc2136_name = ghostliner.v6.army
# TSIG key secret
dns_rfc2136_secret = <TSIG key>
# TSIG key algorithm
dns_rfc2136_algorithm = HMAC-SHA512
```
重启docker后自动申请证书，查看日志，keys在`/config/keys`目录下

#### acme申请证书

### 4.雷池WAF
[安装雷池 | 雷池 SafeLine](https://docs.waf-ce.chaitin.cn/zh/%E4%B8%8A%E6%89%8B%E6%8C%87%E5%8D%97/%E5%AE%89%E8%A3%85%E9%9B%B7%E6%B1%A0) 

**反向代理** 

### 5.子域名更新
 进入zone -> records，添加records `bgm.ghostliner.v6.army` 
 type: AAAA
 name: bgm
 data: \<ipv6addr>
 
