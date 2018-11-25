

安装编译环境以及执行编译命令参考友善之臂文档


________________________________________
## 错误1

JDK版本不正确的报错：
应该用JDK1.7
```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-7-jdk  // OpenJdk 7安装
```

如果系统中存在两个版本的openJDK，可以使用如下命令切换JDK

________________________________________

## 错误2

编译程序时多处报错：

unsupported reloc 43

修改HOST_x86_common.mk
``` makefile
cd SourcePath/build/core/clang/
sudo vim ./HOST_x86_common.mk
# 在如下文档中添加 -B$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/x86_64-linux/bin \
ifeq ($(HOST_OS),darwin)

# nothing required here yet

endif

ifeq ($(HOST_OS),linux)
CLANG_CONFIG_x86_LINUX_HOST_EXTRA_ASFLAGS := \
--gcc-toolchain=$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG) \
--sysroot=$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/sysroot \
-B$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/x86_64-linux/bin \
-no-integrated-as

CLANG_CONFIG_x86_LINUX_HOST_EXTRA_CFLAGS := \
--gcc-toolchain=$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG) \
-no-integrated-as
```
对于Android7.0以下的，需要保留-no-integrated-as这句指令。

________________________________________
## 错误3

error while loading shared libraries: libz.so.1: cannot open shared object file: No such file or dir


ubuntu14.04-64位编译linux内核时出现问题：error while loading shared libraries: libz.so.1: cannot open shared object file: No such file or directory.
解决方法：很明显就是安装这个库所在包咯，可是我们怎么这知道 libz.so.1在哪个包呢？
我们使用命令： apt-file search 来查找，首先安装，按如下命令：
1	sudo apt-get install apt-file
安装完以后系统会提示你update，如果没有提示，在终端输入如下命令：
1	sudo apt-file update
apt-file 是用来查找某个命令或者某一个库所在的包的，具体用法如下：
```
01	:~$ apt-file search libz.so.1
02	lib32z1: /usr/lib32/libz.so.1
03	lib32z1: /usr/lib32/libz.so.1.2.3.3
04	lib32z1-dev: /usr/lib32/libz.so
05	lsb-build-base3: /usr/lib/lsb3/libz.so
06	zlib1g: /lib/libz.so.1
07	zlib1g: /lib/libz.so.1.2.3.3
08	zlib1g-dbg: /usr/lib/debug/lib/libz.so.1.2.3.3
09	zlib1g-dbg: /usr/lib/debug/usr/lib32/libz.so.1.2.3.3
10	zlib1g-dev: /usr/lib/libz.so
```
使用apt-file search 查找你的库所在包就行了，右边的是匹配你的库，左边的是你查的库所在的包，所以最后安装对应的包就行了。

## 错误4


出现错误：

Checking API: checkpublicapi-current
out/target/common/obj/PACKAGING/public_api.txt:20: error 5: Added public field android.Manifest.permission.BACKUP
out/target/common/obj/PACKAGING/public_api.txt:82: error 5: Added public field android.Manifest.permission.INVOKE_CARRIER_SETUP
out/target/common/obj/PACKAGING/public_api.txt:106: error 5: Added public field android.Manifest.permission.READ_PRIVILEGED_PHONE_STATE
out/target/common/obj/PACKAGING/public_api.txt:116: error 5: Added public field android.Manifest.permission.RECEIVE_EMERGENCY_BROADCAST

******************************
You have tried to change the API from what has been previously approved.

To make these errors go away, you have two choices:
   1) You can add "@hide" javadoc comments to the methods, etc. listed in the
      errors above.

   2) You can update current.txt by executing the following command:
         make update-api

      To submit the revised current.txt to the main Android repository,
      you will need approval.

输入
make update-api
等跑完之后从新
make -k
继续编译输入
make update-api
等跑完之后从新
make -k
继续编译

## 错误5
```
build/core/shared_library_internal.mk:68 recipe for target 'out/target/product/generic/obj/SHARED_LIBRARIES/libwebviewchromium_intermediates/LINKED/libwebviewchromium.so' failed
make: ** [out/target/product/generic/obj/SHARED_LIBRARIES/libwebviewchromium_intermediates/LINKED/libwebviewchromium.so] Error 1
make: ** Deleting file 'out/target/product/generic/obj/SHARED_LIBRARIES/libwebviewchromium_intermediates/LINKED/libwebviewchromium.so'
make: Leaving directory 'home/username/android'
#### make failed to build some target (07:53:05 (hh:mm:ss)) ####
```
是因为没有设置Linux的swap原因

```
1.#dd if=/dev/zero of=/opt/swap1 bs=3.0G count=2
  (count的值等于1024 x 你想要的文件大小, 4096000是4G，1024000是1G)
2.#mkswap /opt/swap1
 (把这个文件变成swap文件)
3.#swapon /opt/swap1
 (启用这个swap文件)
4.为了使每次开机时都能正常使用swap文件，所以这里需要把swap文件增加到fstab文件中
 #sudo vi /etc/fstab
 在最后一行增加如下内容
 /opt/swap1 swap swap defaults 0 0
```
做完以后用free -m查看swap分区是否成功开启
