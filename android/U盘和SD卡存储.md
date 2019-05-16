# U盘和SD卡存储

周星宇

## 方案1：OWC法
方案程序参考：
https://www.cnblogs.com/xiaoli3007/p/4309356.html

Open- write-close。使用这种方法时程序会产生异常，并提示没有访问权限。
在APP获取root用户权限后也复制失败

网上找到一个方法：
Android源码在这个路径：\frameworks\base\data\etc\platform.xml
Android系统在这个路径：\system\etc\permissions  下面的 platform.xml 文件
``` xml
<permission name="android.permission.WRITE_EXTERNAL_STORAGE" >
	   <group gid="sdcard_r" />
	   <group gid="sdcard_rw" />
</permission>
```

修改为
``` xml
  <permission name="android.permission.WRITE_EXTERNAL_STORAGE" >
   <group gid="sdcard_r" />
       <group gid="sdcard_rw" />
			<group gid="media_rw" />
  </permission>
```
参考：https://www.cnblogs.com/l2rf/p/5997264.html

使/system文件夹可写方案：
``` shell
mount|grep /system
mount -o remount  /dev/block/platform/dw_mmc.2/by-num/p2 /system
```


## 方案2: shell命令法

三种文件拷贝的命令：

``` shell
Cp a.pdf /storage/usbdisk1/PA/
Cat a.pdf > /storage/usbdisk1/PA/
Dd input=a.pdf  output=/storage/usbdisk1/PA/ a.pdf
```
使用三种命令进行文件的复制在拔出U盘或SD卡后，windows上都显示拷贝出来的PDF文件已损坏且大小为0kb

查找原因：为了优化外部存储IO，对linux外部存储设备的IO操作不能马上执行，而是采用延迟写入的方法。
先将数据缓存在内存中，当IO总线复旦小的时候再写入U盘或SD卡。

为了验证是否是这个原因，安装了ES文件管理器，使用ES进行复制，复制完后拔出U盘。文件大小仍然是0kb


尝试1：写完U盘后读取该文件

尝试2：使用unmount命令弹出U盘
1.	[root@localhost /]# mkdir /mnt/usbdisk/
2.	[root@localhost /]# mount -t vfat /dev/sda1 /mnt/usbdisk
3.	[root@localhost /]# unmount /mnt/usbdisk/

https://blog.csdn.net/e421083458/article/details/13506659

U盘和SD卡文件挂载点如下：
```
/dev/fuse /storage/sdcard1 fuse rw,nosuid,nodev,noexec,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
/dev/fuse /storage/usbdisk1 fuse rw,nosuid,nodev,noexec,relatime,user_id=1023,group_id=1023,default_permissions,allow_other 0 0
```
Sync命令

方案3：使用开源库
https://github.com/magnusja/libaums

据说可以实现U盘读写



________________________________________

问题：

在上位机使用USB进行数据交互时插入或拔出U盘会对通信产生干扰，导致USB连接断掉。
建议使用SD卡
