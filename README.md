# n1-setup
斐讯n1降级、刷官改系统，安装armbain、CoreELEC系统
## 降级前置准备
+ USB双头线一根
+ HDMI 数据线一根
+ U盘一个
+ USB鼠标一个
+ adb工具。
+ 降级固件：[微云链接](https://share.weiyun.com/5vAkZ7p) 密码：nzwy53. 注意下载N1版本的！注意下载N1版本的！注意下载N1版本的！[帖子](http://www.right.com.cn/forum/thread-322736-1-1.html)
+ webpad官改系统 [微云链接](https://share.weiyun.com/5wenYgZ) 密码：faewbc N1_mod_by_webpad_v2.0_20180601-sf-gms-xposed-2.img -> 沙发桌面 和 Lighthome桌面，沙发桌面v2.0_20180601 修正版无需打补丁patch1，但是首次启动后xposed框架处于未激活状态，请手动激活；
+ armbain 镜像： [下载地址](https://yadi.sk/d/pHxaRAs-tZiei)
+ CoreELEC 镜像：[下载地址](https://pan.baidu.com/s/1YLYrauq3gtJS3cs0T9TnKg)  密码：3jur [帖子](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=331363&extra=page%3D1%26filter%3Dtypeid%26typeid%3D21)

## 一、 降级
斐讯官方固件比较新(>V2.22)的版本bootloader有问题，如果不降级就无法刷机。所以先用HDMI连接线将N1盒子插上显示器。启动后首屏即可看到版本号，版本号大于V2.22的都需要降级。

1.如果需要降级，需要先插入USB鼠标，利用鼠标连上无线网络,或者直接插入有线(最好是不能上网的网络,以防自动升级)，获取到盒子的IP地址。

2.打开远程调试开关adb：鼠标点击四次版本号的位置即可开关远程调试。

3.电脑打开上cmd
```
# adb 连接盒子
adb connect 盒子IP地址
# 启动盒子到fastboot
adb shell reboot fastboot
# 刷入bootloader
fastboot flash bootloader bootloader.img
# 刷入boot
fastboot flash boot boot.img 
# 刷入recovery
fastboot flash recovery recovery.img
# 重启
fastboot reboot
```    
## 刷webpad官改系统
安装好Usb burning tools 工具
加载n1wepad官方系统镜像，点击开始按钮**取消默认的擦除flash和解除bootloader**
连接双头USB线，n1尽量接在靠近hdmi口的usb口上
打开adb
```
# adb 连接盒子
adb connect 盒子IP地址
# 启动盒子到fastboot
adb shell reboot update
```
等待固件烧录完成，点击停止，拔掉电源后重新上电开机。
## 刷armbian系统

下载好自己喜欢的系统文件，server为没有桌面的服务器版，其他的都是带了桌面的，桌面占用内存对比： mate > xfce > icewm
这里选择Armbian_5.44_S9xxx_Ubuntu_xenial_3.14.29_server_20180515.img.xz
制作启动U盘，在linux下
```
xzcat Armbian_5.44_S9xxx_Ubuntu_xenial_3.14.29_server_20180515.img.xz | dd of=/dev/sdd
```
## 刷coreelec系统

制作启动U盘，在linux下
```
gzip -dc CoreELEC-PhiComm_N1.arm-8.95.0.img.gz | dd of=/dev/sdd
```
## 进入系统

使用adb命令重启N1
```
adb connect N1的IP
adb shell reboot update
```
使用Android内置终端重启N1：
```
su
reboot update
```
进入recovery模式后再插入制作好的U盘，点击reboot to system。
###这里有个坑：U盘制作好后千万别插在正在运行的Android系统下，要不会被更改权限，导致后面armbian出现问题
如果U盘制作没问题，重启后就可以armbian系统了，默认armbian的root密码是1234，第一次需要重新设置armbian密码之后才能登入系统。

Coreelec系统的ssh用户名：root、密码：coreelec

这样是通过U盘启动的，速度不够快也比较麻烦，我们还需要将armbian、coreelec刷到自带的eMMC里面去。

## 将系统写入eMMC存储
### armbian系统

仍需u盘引导：
```
nand-sata-install
```
脱离U盘：
到这里下载脚本：[链接](http://www.right.com.cn/forum/thread-327496-1-1.html) 替换系统中的/root/install.sh
```
mv install.sh install.shorg
gunzip install.sh.gz
chmod +x install.sh
nand-sata-install
```
重新切换到Android系统：
```
mv /boot/s905_autoscript /boot/s905_autoscriptbak
reboot
```
重启后就会自动进入Android系统
### coreelec系统:
若要写入eMMC，开启ssh，用户名、密码：root、coreelec，登录后执行installtointernal，按屏幕提示操作完成。

如果没有报错的运行完成就可以拔掉U盘重启系统了

### 错误重刷
如果内置的armbian被破坏了，可以根据上面的步骤重刷启动U盘，
用U盘启动之后，运行命令：（注意:这样 data 分区上的数据会全部丢失，但是Android系统还是在的）
```
mke2fs -F -q -t ext4 -m 0 -O ^64bit,^metadata_csum /dev/data
```
完成后就可以全新安装armbian到emmc了
