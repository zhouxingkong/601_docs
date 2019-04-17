# linux USB 通信


## 方案概述

和Android系统的USB驱动框架进行对比
![](assets/markdown-img-paste-20190329142827697.png)


## 交叉编译libusb


下载libusb源码

执行下列命令进行编译

```
./configure --build=i686-linux --host=arm-linux --prefix=/home/forlinx/libusb
make
make install
```

其中:
+ –build=i686-linux表示该软件在x86平台被编译
+ –host=arm-linux表示该软件编译完成后在arm平台上运行
+ –prefix后面为软件安装目录。

交叉编译方法参考：
https://blog.csdn.net/xfc_1939/article/details/53422071

编译成功后将–prefix指定的输出文件夹拷贝到QT工程路径中。修改pro文件将编译生成的.so文件包含到QT工程中。

```
INCLUDEPATH += "$$PWD/libusb/include/libusb-1.0"
LIBS += -L$$PWD/libusb/lib -lusb-1.0
```

## 应用程序烧写

本节介绍在飞凌MX6UL开发板上烧写QT5应用程序

### step1:挂载SD卡

将编译好的可执行程序和SO库拷贝到SD卡中,插入板子

MX6UL板子装上linux4.1系统后不会自动挂载SD卡，所以需要手动挂载

先查看SD卡对应的设备文件
```
root@imx6ulevk:/dev# fdisk -l /dev/mmcblk0
Disk /dev/mmcblk0: 7.4 GiB, 7948206080 bytes, 15523840 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8b360755

Device         Boot   Start      End  Sectors  Size Id Type
/dev/mmcblk0p1      3278848 15234373 11955526  5.7G 83 Linux
/dev/mmcblk0p2         2048   133119   131072   64M 83 Linux
/dev/mmcblk0p3       133120  3278847  3145728  1.5G 83 Linux
```
意思是/dev/mmcblk0为SD卡设备，对应三个SD卡分区:mmcblk0p1;mmcblk0p2;mmcblk0p3

然后执行挂载命令

```
root@imx6ulevk:/dev# mkdir /sdcard
root@imx6ulevk:/dev# mkdir /sdcard/sdcard0
root@imx6ulevk:/dev# mount -t vfat /dev/mmcblk0p1 /sdcard/sdcard0/
```



#### step2:安装程序

之后就可以访问SD卡了。
将SD卡中的可执行文件拷贝到/forlinx/qtbin/路径下，即可完成安装工作。

将动态链接库拷贝到/lib目录下

运行程序可能报如下错误
```
./app: error while loading shared libraries: libusb-library.so.1: cannot open shared object file: No such file or directory
```

如果拷贝之后运行APP仍然报错找不到SO库，则创建软链接
```
root@imx6ulevk:/lib# ln -s libusb-library.so.1.0.0 libusb-library.so
root@imx6ulevk:/lib# ln -s libusb-library.so.1.0.0 libusb-library.so.1
root@imx6ulevk:/lib# ln -s libusb-library.so.1.0.0 libusb-library.so.1.0
```

ps：装入程序运行前还要输入下面这句话(参考飞凌的软件手册)
```
export DISPLAY=:0.0
```


## 速度测试结果

使用平台：mx6DL

  | 最大  | 最小  |  平均
--|---|---|--
只开USB| 40MB/s  | 20MB/s  |  30MB/s
USB和显示都开| 9MB/s  | 11MB/s  |  12MB/s
虽然速度能满足要求，但是在这个速度下运行CPU占用比较多，所以CPU处理性能是否满足要求还不清楚
