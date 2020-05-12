# android USB打印

### 安装交叉编译器

下载
注意，一定要用arm-linux-gcc 5.0以前的版本，否则无法编译cups

``` shell
export PATH=/usr/local/arm/5.4.0(文件夹版本号)/bin:$PATH
```

然后运行profile文件

``` shell
source  /etc/profile
```

 参考: https://blog.csdn.net/weixin_42108484/article/details/84295214

### 交叉编译CUPS

首先在官网上下载源码 https://github.com/apple/cups/releases

配置

``` shell
./configure --build=i686-linux\
 CC=arm-linux-gcc\
 CXX=arm-linux-g++\
 CCLD=arm-linux-gcc\
 CCAUX=gcc\
 RANLIB=arm-linux-ranlib\
 AR=arm-linux-ar\
 --host=arm-linux\
 --target=arm-linux\
 --build=i686-linux\
 --prefix=/home/forlinx/output/cups\
 --disable-gnutls\
 --disable-network-build\
 --disable-gssapi\
 --disable-dbus
```

编译

``` shell
make && make install DSTROOT=<安装的绝对路径>
```
编译通过后会在安装路径找到生成的库文件，将库文件拷贝到开发板的文件系统中即可使用

参考:
+ https://blog.csdn.net/qq_27681837/article/details/79866906

#### 问题解决
如果编译报错
```
undefined reference to `clock_gettime'
```
说明系统找不到`clock_gettime`库，需要在链接时加入依赖

修改Makedefs文件，找到LDFLAGS变量加入`-lrt`参数

``` Makefile
LDFLAGS		=	-lrt
```

### 交叉编译GhostScript


### 交叉编译libusb

下载libusb源码:https://github.com/libusb/libusb/releases

执行下列命令进行编译

``` shell
./configure --build=i686-linux\
 CC=arm-linux-gcc\
 CCLD=arm-linux-gcc\
 CCAUX=gcc\
 --host=arm-linux\
 --target=arm-linux\
 --prefix=/home/forlinx/output/libusb\
 --disable-udev
```

``` shell
make && make install
```

其中:
+ –build=i686-linux表示该软件在x86平台被编译
+ –host=arm-linux表示该软件编译完成后在arm平台上运行
+ –prefix后面为软件安装目录。

ps: libusb-1.0.23以上的版本编译还会报网络的错误，使用1.0.22版本就可以编译通过

交叉编译方法参考：
https://blog.csdn.net/xfc_1939/article/details/53422071

### 交叉编译python

因为编译arm平台的python需要用到Pc平台的python。所以需要首先编译PC平台的python。将下载的压缩包解压两份，分别将文件夹命名为`Python-2.7.13-pc`和`Python-2.7.13-arm`

``` shell
./configure
make python Parser/pgen
```
pc端的编译好后，给为arm平台的python源码打上补丁、

``` shell
patch -p1 < ../Python-2.7.13-xcompile.patch
```

然后进行编译

``` shell
./configure --build=i686-linux\
 CC=arm-linux-gcc\
 CCLD=arm-linux-gcc\
 CCAUX=gcc\
 --host=arm-linux\
 --target=arm-linux\
 --prefix=/home/forlinx/output/python\
 --enable-ipv6\
 --enable-shared\
 ac_cv_file__dev_ptmx="yes"\
 ac_cv_file__dev_ptc="no"
```

``` shell
make HOSTPYTHON=../Python-2.7.13-pc/python HOSTPGEN=../Python-2.7.13-pc/pgen \
BLDSHARED="arm-linux-gcc -shared" \
CROSS_COMPLIE=arm-linux- \
CROSS_COMPLIE_TARGET=yes \
HOSTARCH=arm-linux \
BUILDARCH=i686-linux \
-j2
```

``` shell
make install HOSTPYTHON=../Python-2.7.13-pc/python \
BLDSHARED="arm-linux-gcc -shared" \
CROSS_COMPILE=arm-linux- \
CROSS_COMPILE_TARGET=yes \
prefix=/home/forlinx/output/python
```

#### 异常处理
如果在install的过程中出现如下错误

```
ImportError: No module named _struct
```

在Makefile文件中删除PYTHONPATH变量的赋值即可

参考:
+ https://blog.csdn.net/tangweiguo0305/article/details/81223472

### 交叉编译hplip

下载hplip: https://developers.hp.com/hp-linux-imaging-and-printing/gethplip

``` shell
./configure --prefix=/home/forlinx/output/hplip \
CC=arm-linux-gcc \
CCLD=arm-linux-gcc \
CCAUX=gcc \
--host=arm-linux \
--target=arm-linux \
--build=i686-linux \
LDFLAGS="-L/home/forlinx/output/cups/usr/lib \
-L/home/forlinx/output/libusb/lib \
-L/home/forlinx/output/python/lib \
-lrt" \
CFLAGS="-I/home/forlinx/output/cups/usr/include \
-I/home/forlinx/output/libusb/include \
-I/home/forlinx/output/python/include" \
CPPFLAGS="-I/home/forlinx/output/cups/usr/include \
-I/home/forlinx/output/libusb/include \
-I/home/forlinx/output/python/include" \
PYTHON="/usr/local/bin/python" \
PYTHONINCLUDEDIR="/home/forlinx/output/python/include/python2.7" \
--with-cupsbackenddir=/home/forlinx/output/cups/usr/lib/cups/backend \
--with-cupsfilterdir=/home/forlinx/output/cups/usr/lib/cups/filter \
--disable-doc-build \
--disable-network-build \
--disable-scan-build \
--enable-cups-drv-install \
--disable-hpcups-install \
--disable-gui-build \
--disable-qt4 \
--disable-libusb01_build \
--enable-cups-ppd-install \
--disable-foomatic-drv-install \
--disable-foomatic-ppd-install \
--disable-hpijs-install \
--disable-udev_sysfs_rules \
--disable-policykit \
--disable-dbus-build \
--disable-fax-build \
--disable-scan-build \
--without-htmldir
```

如果configue出错的话通过查看config.log来定位问题的原因。

执行make命令前，首先要检查Makefile文件里的变量地址定义是否有误。一般情况下是库文件路径出错，例如在下面几行的定义中，头文件包含的路径和我们指定的路径不相符。

``` Makefile
#python头文件路径，换成正确的路径
PYTHONINCLUDEDIR = /usr/local/include/python2.7

#libusb头文件路径，换成正确的路径
am__append_8 = -I/usr/include/libusb-1.0
#......
am__append_20 = -I/usr/include/libusb-1.0
```

改完后执行`make`命令。成功后执行`make install`

#### 异常处理

如果指定了libusb库文件路径，编译时仍然报找不到`libusb-1.0.so.0`文件的错误。则将LDFLAGS变量里用`-L`指定的路径后面再用`-Wl,-rpath=`指定一次，如下所示:
``` Makefile
LDFLAGS = -L/home/forlinx/output/libusb/lib -Wl,-rpath=/home/forlinx/output/libusb/lib
```

其他的错误见参考文献

参考:
+ https://github.com/jianglei12138/hplip
+ 论文:《基于ARM平台的Linux打印系统设计与实现》
