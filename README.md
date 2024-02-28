在S905X3机顶盒 A95XF3编译通过，Armbian内核5.10，安装的O大的https://github.com/ophub/amlogic-s9xxx-armbian/releases


WIFI可用，蓝牙未测试。

参考：
https://github.com/ophub/amlogic-s9xxx-armbian/issues/1043
https://github.com/ophub/amlogic-s9xxx-armbian/issues/1701

1.需要下载arm版本编译的内核，因为交叉编译的内核linux-headers是x86的，在编译时有类似错误：
/bin/sh: 1: scripts/basic/fixdep: Exec format error

下载arm环境下编译的内核，在ophub kernel项目的releases下载的kernel_dev下面有，如：
wget https://github.com/ophub/kernel/releases/download/kernel_dev/5.10.209.tar.gz
如果你已经是当前最新的，可以下载上一个版本的，如208。

armbian-kernel -u
tar xzvf 5.10.209.tar.gz
cd 5.10.209
armbian-update

验证是arm编译的内核
执行/usr/src/linux-headers-5.10.209-ophub/scripts/basic/fixdep


2.对齐gcc
查看gcc版本
root@armbian:~/mt7668-ce/MT7668-WiFi# cat /proc/version
Linux version 5.10.209-ophub (root@armbian) (aarch64-none-elf-gcc (Arm GNU Toolchain 13.2.rel1 (Build arm-13.7)) 13.2.1 20231009, GNU ld (Arm GNU Toolchain 13.2.rel1 (Build arm-13.7)) 2.41.0.20231009) #1 SMP PREEMPT Fri Jan 26 11:31:50 CST 2024

下载对应版本的gcc
wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-aarch64-aarch64-none-elf.tar.xz
mkdir /opt/gcc-aarch64-none-elf
sudo tar xf arm-gnu-toolchain-12.2.rel1-aarch64-aarch64-none-elf.tar.xz --strip-components=1 -C /opt/gcc-aarch64-none-elf
echo 'export PATH=$PATH:/opt/gcc-aarch64-none-elf/bin' | sudo tee -a /etc/profile.d/gcc-aarch64-none-elf.sh
source /etc/profile
ln -sf /opt/gcc-aarch64-none-elf/bin/aarch64-none-elf-gcc /usr/local/bin/gcc



3.编译mt7688
git clone -b 5.15 https://github.com/fujianzz/mt7668-ce.git
cd MT7668
nano Makefile.x86
#第3行 ,  第28行的x86改成arm64
修改src路径为linux-header src路径
![image](https://github.com/fujianzz/mt7668-armbian/assets/25293511/91626455-36c0-4337-9b05-dcbc550d2d99)


make  EXTRA_CFLAGS="-w" CROSS_COMPILE= -f Makefile.x86 -j4


4.挂载内核
cp /root/mt7668-ce/MT7668-WiFi/7668_firmware/* /usr/lib/firmware/

cd /root/mt7668-ce/MT7668-WiFi/drv_wlan/MT6632/wlan

modprobe cfg80211
insmod wlan_mt76x8_sdio.ko

mkdir /lib/modules/5.10.209-ophub/kernel/drivers/wifi
install -m 644 wlan_mt76x8_sdio.ko /lib/modules/5.10.209-ophub/kernel/drivers/wifi/

添加到/etc/modules
root@armbian:~# cat /etc/modules
cfg80211
wlan_mt76x8_sdio

depmod -a


5.加载这个驱动后，会导致有线掉IP，需要配置
nano /etc/NetworkManager/system-connections/Wired\ connection\ 1.nmconnection
[ethernet]下添加
duplex=full
speed=100
![image](https://github.com/fujianzz/mt7668-armbian/assets/25293511/6ff3311a-2e37-495c-a377-6a4e03d137fc)
