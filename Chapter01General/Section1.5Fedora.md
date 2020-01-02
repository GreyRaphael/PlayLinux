# Freshman

<!-- TOC -->

- [Freshman](#freshman)
    - [System modify](#system-modify)
        - [VMware troubleshooting](#vmware-troubleshooting)
            - [about vmware itself](#about-vmware-itself)
            - [about linux](#about-linux)
        - [grub boot timeout](#grub-boot-timeout)
        - [grub change boot order](#grub-change-boot-order)
        - [grub add menu](#grub-add-menu)
        - [change source](#change-source)
            - [tsinghua Mirror or ustc Mirror](#tsinghua-mirror-or-ustc-mirror)
        - [modify hosts](#modify-hosts)
        - [Remove old kernel](#remove-old-kernel)
    - [Install Software](#install-software)
        - [rpm package](#rpm-package)
        - [install an zip](#install-an-zip)
        - [Remove Software](#remove-software)
        - [About the CentOS](#about-the-centos)
        - [About the HTML5 video](#about-the-html5-video)
        - [About Firefox-dev](#about-firefox-dev)
        - [About remote desktop(need to see the desktop)](#about-remote-desktopneed-to-see-the-desktop)
        - [About the ssh(no need to see the desktop)](#about-the-sshno-need-to-see-the-desktop)
        - [About the GNOME Tweak Tool and extension](#about-the-gnome-tweak-tool-and-extension)
        - [partition tools](#partition-tools)
    - [Development](#development)
        - [Qt development](#qt-development)
        - [Android Development](#android-development)
    - [About mariadb](#about-mariadb)
        - [Other related command](#other-related-command)
    - [PostgreSQL](#postgresql)
        - [install gui](#install-gui)
    - [Android Studio](#android-studio)
    - [Debian](#debian)
    - [Xubuntu](#xubuntu)
    - [in Virtualbox](#in-virtualbox)
    - [in VMWare](#in-vmware)

<!-- /TOC -->

## System modify

### VMware troubleshooting

#### about vmware itself

```bash
the BIOS has corrupted hw-PMU resources(MSR 38d is 3)

#VM/settings/processors/vitualization engine关闭 Virtualze CPU performance counters
```

#### about linux

```bash
piix4_smbus 0000:00:07.3: Host SMBus controller not enabled!

piix4_smbus 0000:00:07.3:SMBus base address uninitialized - upgrade BIOS or use force_addr=0xaddr

###how to fix:
lsmod | grep i2c_piix4 

su

vi  /etc/modprobe.d/blacklist.conf

blacklist i2c_piix4

reboot
```

### grub boot timeout

```bash
#vmware修改启动时间
bios.bootdelay="2000"
```

```bash
#chang grub timeout

#method1:
使用GUI程序，**grub-customizer**必须要用`su`因为wayland不允许terminal打开GUI

#method2:
#in bios
gedit /boot/grub2/grub.cfg
#in uefi
gedit /boot/efi/EFI/fedora/grub.cfg
#然后搜索timeout,修改为
set timeout=1
```

```bash
#recommended
sudo vim /etc/default/grub

GRUB_TIMEOUT="2" #set to 2 seconds
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT="saved"
GRUB_DISABLE_SUBMENU="true"
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rd.lvm.lv=fedora/root rd.lvm.lv=fedora/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"

#then for bios
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
#for efi
sudo grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```



### grub change boot order

```bash
grub2-set-default 1 #第二项来启动
```

### grub add menu

```bash
#for fedora
yum reinstall grub2-efi

grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

实在不行用**grub-customizer**

### change source

阿里云从Ubuntu16.04升级到Ubuntu18.04: `sudo do-release-upgrade -d`

>`cd /etc/yum.repos.d/`

then edit the `fedora.repo` and `fedora-updates.repo`

finally, `dnf makecache`

#### tsinghua Mirror or ustc Mirror

fedora.repo:

```bash
[fedora]
name=Fedora $releasever - $basearch
failovermethod=priority
baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
enabled=1
metadata_expire=28d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```

fedora-updates.repo:

```bash
[updates]
name=Fedora $releasever - $basearch - Updates
failovermethod=priority
baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora/updates/$releasever/$basearch/
enabled=1
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```

### modify hosts

```bash
wget https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts https://raw.githubusercontent.com/googlehosts/hosts/master/hosts-files/hosts

#then
sudo gedit /etc/hosts
```

### Remove old kernel

check current kernel edition: `uname -r` or `uname -a`

```bash
##begin remove old kernel
su

rpm -qa|grep kernel

#choose the package(contains "core")
dnf remove kernel-core-4.13.4-200.fc26.x86_64
```

## Install Software

### rpm package

```bash
#对于已经安装的情况
#查询所有安装(all)
rpm -qa
#查看某一个软件
rpm -qa gedit
#查看某个文件是哪个软件安装的
rpm -qf /lib/xxx
#查看安装的位置
rpm -ql gedit
#查看依赖
rpm -qR gedit
#查看信息(information)
rpm -qi gedit
#查看安装的软件的文档(doc)
rpm -qd gedit
#查看config文件
rpm -qc gedit
```

```bash
#对于未安装的package
#查看一个未安装的信息
rpm -qpi code.x86_64.rpm
#查看包含的文件
rpm -qpl code.x86_64.rpm
rpm -qpd code.x86_64.rpm
rpm -qpc code.x86_64.rpm
rpm -qpR code.x86_64.rpm
```

```bash
#安装一个包
rpm -ivh code.x86_64.rpm
# uninstall a package
rpm -e code.x86-64.rpm
```

### install an zip

```bash
#first step:download the zip and unpack to /home/grey/
mkdir -p /home/grey/phantomjs

#second step:create symlink
ln -s /home/grey/hantomjs/bin/phantomjs /usr/bin/phantomjs

#test helloworld
#new a test.js
console.log('Hello, world!');
phantom.exit();

#test
phantomjs test.js
```

### Remove Software

```python
import os

# uninstall software
## gnome series
os.system("sudo dnf -y remove gnome-boxes")
os.system("sudo dnf -y remove gnome-maps")
os.system("sudo dnf -y remove gnome-weather")
os.system("sudo dnf -y remove gnome-todo")
## multimedia
os.system("sudo dnf -y remove cheese")
os.system("sudo dnf -y remove shotwell")
os.system("sudo dnf -y remove simple-scan")
os.system("sudo dnf -y remove rhythmbox")
os.system("sudo dnf -y remove totem")
## office
os.system("sudo dnf -y remove evolution")
os.system("sudo dnf -y remove libreoffice-core-*")
```

```bash
dnf grouplist

#remove LibreOffice group
dnf remove @LibreOffice
```

### About the CentOS

```bash
#install epel-release to using fedora's repository
sudo yum install epel-release
```

### About the HTML5 video

```bash
#First activate RPMFusion repo
sudo dnf install https://mirrors.ustc.edu.cn/rpmfusion/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.ustc.edu.cn/rpmfusion/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

#then, 或者直接安装vlc,自带有x264
# sudo dnf install x264
sudo dnf install vlc

#last reboot

## remove

sudo dnf remove rpmfusion-free-release
sudo dnf remove rpmfusion-nonfree-release
```

### About Firefox-dev

Download [Firefox-Dev](https://www.mozilla.org/en-US/firefox/developer/all/)

Unpack to `/opt`

Create `FirefoxDev.desktop` file in `/usr/share/applications/`

```bash
[Desktop Entry]

Encoding=UTF-8

Name=FirefoxDev

Name[en]=FirefoxDev

Name[en_US]=FirefoxDev

Comment=Firefox for jason

Exec=/opt/firefox/firefox

Icon=/opt/firefox/browser/icons/mozicon128.png

Terminal=false

Type=Application

Categories=Application;Network;
```

update the FirefoxDev in root account

### About remote desktop(need to see the desktop)

```bash
#install the xrdp
sudo dnf install xrdp

#start the xrdp(when reboot, it's gone)
sudo systemctl start xrdp

#run the xrdp even reboot
sudo systemctl enable xrdp
```

### About the ssh(no need to see the desktop)

```bash
#start the ssh service
sudo systemctl start sshd

#Then in other Linux, Mac or Windows to connet server
#just like the Manjaro
```

```bash
# Fedora的管理员账号(chris是sudoer)一定要设置密码，否者一堆麻烦
# 尤其是在ssh的时候，如何默认的登陆，则是empty password的状态，用ssh就是一个坑
ssh chris@localhost
```



### About the GNOME Tweak Tool and extension

`dnf install chrome-gnome-shell`

### partition tools

[blivet-gui](https://blog.vojtechtrefny.cz/blivet-gui) will replace gparted

`dnf install blivet-gui`

## Development

### Qt development

[Official installation tutorial](http://doc.qt.io/qt-5/linux.html)

step1: install basic requirements

```bash
#gcc,g++,gdb,.....
sudo dnf groupinstall "C Development Tools and Libraries"

#for  building graphical Qt applications
sudo yum install mesa-libGL-devel
```

step2: download & installl

[blog tutorial](https://www.ics.com/blog/qt%20qml)

```bash
# download installation package
wget https://mirrors.tuna.tsinghua.edu.cn/qt/official_releases/qt/5.9/5.9.2/qt-opensource-linux-x64-5.9.2.run

#install
chmod +x qt-opensource-linux-x64-5.9.2.run
./qt-opensource-linux-x64-5.9.2.run
```

or the online one

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/qt/official_releases/online_installers/qt-unified-linux-x64-online.run

#install
chmod +x qt-unified-linux-x64-online.run
./qt-unified-linux-x64-online.run
```

when installed, in qtcreator check that it correctly set up at least one auto-detected kit, compiler, Qt version, and debugger

If you want to update, add or remove any components, you can run the maintenance tool, which can be found under the install directory as MaintenanceTool.

### Android Development

[tutorial](https://www.linode.com/docs/development/install-java-on-centos)

```bash
#first step
sudo dnf install java-1.8.0-openjdk-devel

#second step
sudo yum install zlib.i686 ncurses-libs.i686 bzip2-libs.i686
#reboot
##download "Android Studio" and install
#choose the "SDK config"
#choose the "SDK Tools"
#choose the "NDK"
```

[Linux Install tutorial](https://developer.android.com/studio/install.html)

[Android NDK](https://www.youtube.com/watch?v=KP7XdevA9jE)

## About mariadb

[mariadb](https://downloads.mariadb.org/mariadb/repositories/#mirror=neusoft&distro=Fedora&distro_release=fedora26-amd64--fedora26&version=10.2)

[wiki](https://fedoraproject.org/wiki/MariaDB)

[community](https://developer.fedoraproject.org/tech/database/mariadb/about.html)

```bash
#install mariadb
sudo dnf install mariadb-server

#start mariadb
systemctl start mariadb
```

```bash
#setting mariadb,follow the advice
mysql_secure_installation

#login
mysql -u root -p
```

```bash
#then install mysql-visual-too
```

[sql-workbench](https://dev.mysql.com/downloads/workbench/?os=src)
or for fedora instal `phpmyadmin`

```bash
sudo dnf install phpmyadmin

systemctl restart httpd
```

then in browser, `localhost/phpmyadmin`

### Other related command

```bash
#start a service
systemctl start mariadb
#stop a service
systemctl stop mariadb
#restart a service
systemctl restart mariadb

#show status
systemctl status mariadb

#enable a service on boot
systemctl enable mariadb
#disable a service on boot
systemctl diable mariadb
#check is-enabled on boo
systemctl is-enabled mariadb
#check start-on boot services
systemctl list-unit-files|grep enabled
```

## PostgreSQL

[tuturial](https://developer.fedoraproject.org/tech/database/postgresql/about.html)

[wiki](https://fedoraproject.org/wiki/PostgreSQL)

```bash
sudo dnf install postgresql-server postgresql-contrib

sudo systemctl start postgresql

sudo postgresql-setup --initdb --unit postgresql
```

### install gui

```bash
sudo dnf install https://download.postgresql.org/pub/repos/yum/10/fedora/fedora-27-x86_64/pgdg-fedora10-10-3.noarch.rpm

sudo dnf install pgadmin4-v2
```

## Android Studio

first start the **XX-net**

then in Linux terminal `sudo mount -o remount,size=8G,noatime /tmp`

[Tutorial](https://www.hiroom2.com/2017/12/07/fedora-27-android-studio-3-0-en/)

```bash
sudo dnf install zlib-devel.i686 ncurses-devel.i686 ant

sudo dnf install compat-libstdc++-296.i686 compat-libstdc++-33.i686 compat-libstdc++-33.x86_64 glibc.i686 glibc-devel.i686 libstdc++.i686 libX11-devel.i686 libXrender.i686 libXrandr.i686

sudo dnf install java-1.8.0-openjdk-devel.x86_64
```

then in Android Studio

## Debian

```python
#debian remove & install
import os

#uninstall
#libreoffic
os.system('sudo apt remove -y libreoffice-common libreoffice-core')
#game
os.system('sudo apt remove -y aisleriot four-in-a-row gnome-mines hitori swell-foop quadrapassel lightsoff gnome-chess gnome-klotski gnome-mahjongg gnome-nibbles gnome-robots gnome-sudoku gnome-taquin iagno tali gnome-tetravex five-or-more')
#picture
os.system('sudo apt remove -y gimp inkscape shotwell simple-scan imagemagick-6-common imagemagick')
#media
os.system('sudo apt remove -y rhythmbox totem brasero sound-juicer cheese')
#internet
os.system('sudo apt remove -y thunderbird pidgin hexchat')

#nstall
#gcc etc
os.system('sudo apt install -y build-essential libgl1-mesa-dev')
os.system('sudo apt install -y golang')
#vim
os.system('sudo apt install -y neovim')
#vm
os.system('sudo apt install -y open-vm-tools-desktop')
#pinyin
os.system('sudo apt install -y ibus-libpinyin')
# im-config  #选择ibus
# ibus-setup #添加chinese
#htop
os.system('sudo apt install -y htop')
```

```bash
#debian add sudo
su
vim /etc/sudoers
# add
grey ALL=(ALL) NOPASSWD: ALL 
```

```bash
#debian install deb with dependencies
sudo dpkg -i package_with_unsatisfied_dependencies.deb
sudo apt -f install
```

```bash
#chage grub timeout
sudo vim /etc/default/grub

sudo update-grub
```

```bash
#自动登录
sudo vim /etc/lightdm/lightdm.conf
autologin-user=grey
autologin-user-timeout=0
```

## Xubuntu

```bash
# netinstall

# 1.use root
sudo passwd root

# 2.allow copy in vmware, no need for virtualbox
sudo apt install -y open-vm-tools-desktop

# 3.reboot
reboot

# 4.autologin
#   4.1 "users and groups"修改 "Not asked on login"
#   4.2 autologin.conf
sudo vim /etc/lightdm/lightdm.conf.d/12-autologin.conf
# 修改为
[Seat:*]
autologin-user=<username>
autologin-user-timeout=0

# 5.change keyboard
# WIN key for startMenu: keyboard里面修改xfce4-popup-whiskermenu
```

```bash
# fix ibus
https://bugs.launchpad.net/ubuntu/+source/ibus-libpinyin/+bug/1698629
```

```python
# xubuntu uninstall & install
import os
# uninstall
#libreoffice
os.system('sudo apt remove -y libreoffice-common libreoffice-core xfce4-dict')
#game
os.system('sudo apt remove -y gnome-mines gnome-sudoku sgt-puzzles')
#picture
os.system('sudo apt remove -y simple-scan xfburn')
#internet
os.system('sudo apt remove -y thunderbird pidgin')

# install
#gcc golang jdk
os.system('sudo apt install -y build-essential libgl1-mesa-dev')
os.system('sudo apt install -y golang')
os.system('sudo apt -y install default-jdk')
os.system('sudo apt -y install python3-pip')
#vim
#os.system('sudo apt install -y neovim')
#pinyin
os.system('sudo apt install -y ibus-libpinyin')
# im-config  #选择ibus
# ibus-setup #添加chinese
os.system('sudo apt autoremove --purge')
```

```bash
# 1. install vscode
# 2. install vscode python extension, change 'setting.json'
```

## in Virtualbox

check x11 or wayland `echo $XDG_SESSION_TYPE`

for xubuntu

```bash
# Settings→Softwares&Updates→additional drive→using x86 graphic

# shared folder, 然后进入/media/xxx
sudo usermod -aG vboxsf $(whoami)
```

for Fedora

```bash
# software→enable 3rd repo

# remove virtualbox-guest-addin
https://www.linuxbabe.com/virtualbox/install-virtualbox-guest-additions-fedora-guest-os 
```
for Budgie

```bash
# install virtualbox-guest-addin
sudo apt install linux-headers-$(uname -r) build-essential dkms

# Devices/Install Guest Additions CD image, run
```

```bash
# uninstall in Budgie
import os
# uninstall
#libreoffice
os.system('sudo apt remove -y libreoffice-common libreoffice-core')
#game
os.system('sudo apt remove -y gnome-mines gnome-sudoku aisleriot gnome-mahjongg')
#picture
os.system('sudo apt remove -y simple-scan gnome-maps gnome-weather gnome-mpv cheese rhythmbox')

# install
# open-ssh
os.system('sudo apt install -y openssh-server')
os.system('sudo systemctl enable ssh')
# open in tillix
os.system('sudo apt install -y python-nautilus')
#gcc golang jdk
os.system('sudo apt install -y build-essential libgl1-mesa-dev')
os.system('sudo apt install -y golang')
os.system('sudo apt -y install default-jdk')
os.system('sudo apt -y install python3-pip')
#vim
os.system('sudo apt install -y vim')
#pinyin
os.system('sudo apt install -y ibus-libpinyin')
# im-config  #选择ibus
# ibus-setup #添加chinese
os.system('sudo apt autoremove --purge')
```

## in VMWare

vmware shared with windows

```bash

mkdir ~/MyShare -p

sudo vmhgfs-fuse -o allow_other -o auto_unmount -o uid=1000 -o gid=1000 .host:/VMShared ~/MyShare
```