#2020-08-08 
##修改默认信道为5G .开启802.11n  802.11ac 。


#### 推荐安装方法， 其他安装方法请看下文未安装make,请执行，apt install make，否则请忽略
    安装后无法运行请安装下面的依赖包
    apt-get install util-linux procps hostapd iproute2 iw haveged dnsmasq
    安装方法
    git clone https://github.com/kaozb/Create_AP_wifi.git
    cd create_ap
    make install
    
    启动
    service create_ap start
    
    设置开机启动
    systemctl enable create_ap.service
    
    默认wifi名称=PangPang
    默认账户密码=abigkiss
## 功能
* 创建WIFI，在任意信道.
* 支持以下加密方式: WPA, WPA2, WPA/WPA2, Open (no encryption).
* 隐藏WIFI名称.
* 客户端之间互相隔离；禁止互相访问
* IEEE支持的模式 802.11n & 802.11ac support
* 上网共享的模式，默认Bridged: NATed or Bridged or None (no Internet sharing).
*	修改默认为桥接模式，便于有线无线设备无障碍协同工作。如果无线设备密码认证通过但是无法获取IP，可以修改 /etc/create_ap.conf 中的bridge为nat(nat模式下默认网段为10.95.32.0/24)
* 选择AP网关IP（仅适用于“ NATed”和“ None” Internet共享方法）。
* 您可以使用与Internet连接相同的接口来创建AP。
* 您可以通过管道或参数传递SSID和密码（请参见示例）。


## 依存关系
### General
* bash (to run this script)
* util-linux (for getopt)
* procps or procps-ng
* hostapd
* iproute2
* iw
* iwconfig (you only need this if 'iw' can not recognize your adapter)
* haveged (optional)

### For 'NATed' or 'None' Internet sharing method
* dnsmasq
* iptables


## 安装方法
### Generic
    git clone https://github.com/kaozb/Create_AP_wifi.git
    cd create_ap
    make install

### ArchLinux
    pacman -S create_ap

### Gentoo
    emerge layman
    layman -f -a jorgicio
    emerge net-wireless/create_ap

## 创建AP方法
### 无密码AP:
    create_ap wlan0 eth0 PangPang

### WPA + WPA2 passphrase:
    create_ap wlan0 eth0 PangPang abigkiss

### AP without Internet sharing:
    create_ap -n wlan0 PangPang abigkiss

### 桥接模式 Internet sharing:
    create_ap -m bridge wlan0 eth0 PangPang abigkiss

### Bridged Internet sharing (pre-configured bridge interface):
    create_ap -m bridge wlan0 br0 PangPang abigkiss

### Internet sharing from the same WiFi interface:
    create_ap wlan0 wlan0 PangPang abigkiss

### 更换WIFI驱动
    create_ap --driver rtl871xdrv wlan0 eth0 PangPang abigkiss

### 无密码 (open network) using pipe:
    echo -e "PangPang" | create_ap wlan0 eth0

### 更换加密方式WPA + WPA2 passphrase using pipe:
    echo -e "PangPang\nabigkiss" | create_ap wlan0 eth0

### Enable IEEE 802.11n
    create_ap --ieee80211n --ht_capab '[HT40+]' wlan0 eth0 PangPang abigkiss

### Client Isolation:
    create_ap --isolate-clients wlan0 eth0 PangPang abigkiss

## Systemd service
Using the persistent [systemd](https://wiki.archlinux.org/index.php/systemd#Basic_systemctl_usage) service

### Start service immediately:
    systemctl start create_ap
### Start on boot:
    systemctl enable create_ap


## License
FreeBSD
## Try this first

如果您在使用Realtek适配器时遇到任何问题（例如Edimax EW-7811Un）
首先尝试使用`-w 2`运行create_ap（即仅使用WPA2）或使用它
没有密码。如果您仍然遇到任何问题或想要
也使用WPA1，然后按照以下说明进行操作。
NOTE: 以下说明仅对带有8192芯片组的Realtek适配器有效。

## Before installation

If you're using ArchLinux, run:

```
pacman -S base-devel linux-headers dkms git
pacman -R hostapd
```

If you're using Debian, Ubuntu, or any Debian-based distribution, run:

```
apt-get install build-essential linux-headers-generic dkms git
apt-get remove hostapd
apt-get build-dep hostapd
```

## Install driver

Linux内核主线中的驱动程序不适用于8192适配器。因此，您需要安装Realtek提供的驱动程序。其无法使用较新的内核编译驱动程序，但是由于它是开源的
根据GPL许可发布的某些人能够对其进行修复并使其编译。

使用以下命令，您可以安装Realtek驱动程序的固定版本：

```
git clone https://github.com/pvaret/rtl8192cu-fixes.git
dkms add rtl8192cu-fixes
dkms install 8192cu/1.9
cp rtl8192cu-fixes/blacklist-native-rtl8192.conf /etc/modprobe.d
cp rtl8192cu-fixes/8192cu-disable-power-management.conf /etc/modprobe.d
```

安装后，请卸载以前的驱动程序并加载新的驱动程序，或者只是重新启动。

## Install hostapd

Realtek的驱动程序使用的是旧的子系统，称为“无线扩展”。（或`wext`）。 Hostapd仅与新子系统（称为nl80211）一起使用。因此，Realtek为hostapd编写了一个补丁。您可以使用
以下命令:

如果您有ArchLinux，请安装[hostapd-rtl871xdrv]（https://aur.archlinux.org/packages/hostapd-rtl871xdrv）
from AUR or just run:

```
yaourt -S hostapd-rtl871xdrv
```

如果您使用任何其他发行版, run:

```
git clone https://github.com/pritambaral/hostapd-rtl871xdrv.git
wget http://w1.fi/releases/hostapd-2.2.tar.gz
tar zxvf hostapd-2.2.tar.gz
cd hostapd-2.2
patch -p1 -i ../hostapd-rtl871xdrv/rtlxdrv.patch
cp ../hostapd-rtl871xdrv/driver_* src/drivers
cd hostapd
cp defconfig .config
echo CONFIG_DRIVER_RTW=y >> .config
make
make install
```
