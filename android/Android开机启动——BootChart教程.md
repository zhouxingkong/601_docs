# 使用BootChart工具进行Android开机启动分析

参考：

<https://blog.csdn.net/u010164190/article/details/72833813?utm_medium=distribute.pc_relevant.none-task-blog-title-1&spm=1001.2101.3001.4242> 

<https://blog.csdn.net/dabenxiong666/article/details/52017352>

系统：Android 5.1

## 一、 将bootchart编译进系统镜像

### 1. 修改源码根目录/system/core/init/Android.mk 

将INIT_BOOTCHART:= true加入Android.mk文件，如下图所示：

![](https://github.com/ccxx0914/ccxx0914.github.io/blob/master/601docs/Assets/1602503078.png)

### 2. 修改system/core/init/bootchart.h 

 	\#deinfine BOOTCHART 0 

​	改为

​	\#deinfine BOOTCHART 1

### 3. 修改system/core/init/bootchart.c 

(1)	开机启动时会自动生成bootchart文件夹，修改该文件夹生成的路径，这里必须修改，不能放在/data/文件夹下，否则生成的bootchart会被覆盖掉

​	\#define LOG_ROOT  "/data/bootchart" 

​	改为

​	\#define LOG_ROOT  "/dev/bootchart"

(2) 在bootchart_init()函数加入:

​    在s = strstr(cmdline, KERNEL_OPTION);之前添加：

​    timeout = 120；//120：android系统启动会抓取120秒的log，可以自己随意改动

### 4. 重新编译系统，烧写镜像，并重启

重启后会/dev/bootchart路径下会自动生成以下文件：

header      proc_stat.log      proc_ps.log     proc_diskstats.log     kernel_pacct 

### 5. 重启后打包以上文件

~~~
adb shell
cd /dev/bootchart
busybox tar -czf bootchart.tgz header proc*.log  kernel_pacct 
exit
adb pull /dev/bootchart/bootchart.tgz ./
~~~

## 二、Ubuntu系统上安装bootchart

使用以下命令安装bootchart工具

~~~
sudo apt-get install bootchart
~~~
## 三、生成bootchart.png图表

~~~
bootchart bootchart.tgz
~~~

执行上述命令报错如下：float division by zero，找到报错的地方，以下图为例，打开Ubuntu系统中的/usr/lib/python2.7/dist-packages/pybootchartgui/samples.py，定位到83行，在这之前加入以下语句：

~~~
if interval == 0:
   interval = 1
~~~

![](https://github.com/ccxx0914/ccxx0914.github.io/blob/master/601docs/Assets/1602506101.jpg)

重新执行bootchart bootchart.tgz命令，若其他文件还存在上述报错，则按照上述方法继续进行修改，直至成功执行bootchart命令，最终成功生成bootchart图表，如下图所示：

![](https://github.com/ccxx0914/ccxx0914.github.io/blob/master/601docs/Assets/1602506589.jpg)

## 四、大功告成

最终生成的bootchart图表如下图所示：

![](https://github.com/ccxx0914/ccxx0914.github.io/blob/master/601docs/Assets/ecc5e06e83837857d9a15a913483e50.png)