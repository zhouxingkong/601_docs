# android 5.1 系统root

2018-06-10 闫隆鑫

## Android5.1系统root思路

+ 在Android系统中运行一个APP时,Android会根据系统编译时对APK的标签,以相应的用户身份启动这个进程.
+ 如果一个APK在Android系统编译时被标记为"系统应用",那么这个APP在运行时就会以ROOT用户启用,相应的它就获得了改变系统任何配置\获取系统任何资源的权利.
+ 但是大部分APP不会直接被编译进源码并标记为"系统应用,那么他们在启动时通常会以一个临时普通用户的身份启动,只能获取少量的不会危及系统本身安全的权限.尽管有时可以通过更改APK的签名使其获取更高的权限,但通常达不到root用户的至高权力.
+ 在LINUX系统中,将用户身份切换为root用户的命令是su,因此倘若能在普通APP里执行这个命令,那么就能改变普通APP的权限,即所谓的"获得root权限"
+ 在Android5.1系统中,google设置了两个障碍物阻止普通用户执行su命令:
 - Linux系统及su程序本身的权限限制;
 - Android5.1系统新加入新安全子系统的SELINUX,在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。
因此,解决Android5.1系统root问题的可以分为两步:
1. 使能su
2. 解除SELINUX

## 使能su

如果在APP中调用su命令时出现以下异常,则说明该APP没有获取root用户的权限:

``` java
System.err: java.io.IOException: Error running exec(). Command: [su] Working Directory: null Environment: null
```

如何在Android应用中执行命令行可以参考:Android 在Android代码中执行命令行,

在普通Android应用中使能su命令涉及到以下四层阻碍:
1. su执行文件本身的权限限制
2. su命令源码中添加了uid检验，只允许shell/root用户进行调用　
3. Zygote源码中添加了添加DropCapabilitiesBoundingSet屏蔽APP了setuid的功能
4. adb源码中添加了添加should_drop_privileges屏蔽adb了setuid的功能(对于userdebug/eng版本该函数未被调用)

针对这四层阻碍,需要依次对源码进行以下的更改:

改变su的默认访问权限,更改frameworks/base/include/private/android_filesystem_config.h

``` c
diff --git a/include/private/android_filesystem_config.h b/include/private/android_filesystem_config.h
index 2f528b9..1223b45 100644
--- a/include/private/android_filesystem_config.h
+++ b/include/private/android_filesystem_config.h
@@ -244,7 +244,7 @@ static const struct fs_path_config android_files[] = {
 /* the following five files are INTENTIONALLY set-uid, but they
  * are NOT included on user builds. */
-    { 04750, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
+    { 06755, AID_ROOT,      AID_SHELL,     0, "system/xbin/su" },
 { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/librank" },
 { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procrank" },
 { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procmem" },
@@ -255,6 +255,7 @@ static const struct fs_path_config android_files[] = {
 { 00750, AID_ROOT,      AID_ROOT,      0, "system/bin/uncrypt" },
 { 00750, AID_ROOT,      AID_ROOT,      0, "system/bin/install-recovery.sh" },
+    { 06755, AID_ROOT,      AID_ROOT,      0, "system/bin/su" },
 { 00755, AID_ROOT,      AID_SHELL,     0, "system/bin/*" },
 { 00755, AID_ROOT,      AID_ROOT,      0, "system/lib/valgrind/*" },
 { 00755, AID_ROOT,      AID_ROOT,      0, "system/lib64/valgrind/*" },
```

可见,我们将su的默认权限改成了06755,这涉及到Linux/Android的权限制度,可以参考安卓权限数字的含义 这篇文章.

更改su源码,解除对用户ID的访问限制,更改源码system/extras/su/su.c

``` c
  project system/extras/
diff --git a/su/su.c b/su/su.c
index 8365379..826acfc 100644
--- a/su/su.c
+++ b/su/su.c
@@ -107,11 +107,12 @@ int main(int argc, char **argv)
     /* Until we have something better, only root and the shell can use su. */
     myuid = getuid();
+    /*
     if (myuid != AID_ROOT && myuid != AID_SHELL) {
         fprintf(stderr,"su: uid %d not allowed to su\n", myuid);
         return 1;
     }
-
+    */
     if(argc < 2) {
         uid = gid = 0;
     } else {
```

可见,我们解除了UID只能是ROOT或者SHELL的访问限制.因为在su后续的程序中使用到了setuid()这个函数,所以还需解除系统中其他地方对这个函数的访问限制.

解除Zygote对setuid()的访问限制

更改源码frameworks/base/cmds/app_process/app_main.cpp

``` c
diff --git a/cmds/app_process/app_main.cpp b/cmds/app_process/app_main.cpp
index 1bb28c3..3e69750 100644
--- a/cmds/app_process/app_main.cpp
+++ b/cmds/app_process/app_main.cpp
@@ -185,6 +185,7 @@ static const char ZYGOTE_NICE_NAME[] = "zygote";
int main(int argc, char* const argv[])
{
+/*
 if (prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0) < 0) {
     // Older kernels don't understand PR_SET_NO_NEW_PRIVS and return
     // EINVAL. Don't die on such kernels.
@@ -193,6 +194,7 @@ int main(int argc, char* const argv[])
         return 12;
     }
 }
+*/
 AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
 // Process command line arguments
```

更改源码frameworks/base/core/jni/com_android_internal_os_Zygote.cpp

``` c
    diff --git a/core/jni/com_android_internal_os_Zygote.cpp b/core/jni/com_android_internal_os_Zygote.cpp
index 4f5e08b..8b136bd 100644
--- a/core/jni/com_android_internal_os_Zygote.cpp
+++ b/core/jni/com_android_internal_os_Zygote.cpp
@@ -208,6 +208,7 @@ static void EnableKeepCapabilities(JNIEnv* env) {
 }
 static void DropCapabilitiesBoundingSet(JNIEnv* env) {
+/*
   for (int i = 0; prctl(PR_CAPBSET_READ, i, 0, 0, 0) >= 0; i++) {
     int rc = prctl(PR_CAPBSET_DROP, i, 0, 0, 0);
     if (rc == -1) {
@@ -220,6 +221,7 @@ static void DropCapabilitiesBoundingSet(JNIEnv* env) {
       }
     }
   }
+*/
 }
 static void SetCapabilities(JNIEnv* env, int64_t permitted, int64_t effective) {
project system/core/
diff --git a/adb/adb.c b/adb/adb.c
index 10a1e0d..2cd4f97 100644
--- a/adb/adb.c
+++ b/adb/adb.c
@@ -1261,6 +1261,7 @@ static void drop_capabilities_bounding_set_if_needed() {
 }
 static int should_drop_privileges() {
+    return 0;
 #ifndef ALLOW_ADBD_ROOT
     return 1;
 #else /* ALLOW_ADBD_ROOT */
```

解除adb源码中对setuid()的限制,更改源码system/core/adb/adb.c

``` c
diff --git a/adb/adb.c b/adb/adb.c
index 10a1e0d..2cd4f97 100644
--- a/adb/adb.c
+++ b/adb/adb.c
@@ -1261,6 +1261,7 @@ static void drop_capabilities_bounding_set_if_needed() {
}
static int should_drop_privileges() {
+    return 0;
#ifndef ALLOW_ADBD_ROOT
 return 1;
#else /* ALLOW_ADBD_ROOT */
```
经过以上对Android源码的修正,现在在adb shell里以普通app的用户身份执行su命令应该可以了.   
怎么获取app的用户名,可以参考调用su命令时候会遇到 的问题这篇文章.   
但是,在adb shell里可以切换成root用户,并不代表在应用程序中可以执行su切换身份,还需要解决SELINUX的阻碍

## 解除SELINUX

Android 5.0 Lollipop 如何像4.2.2一样通过su命令获取root权限一文中介绍了怎么使SELINUX的防护失能的方法,但是这种方法不是永久的而且需要在adb shell里才能实现,所以不适合嵌入式应用开发.  
如何设置确认selinux 模式,  
永久设置Selinux成permissive（宽容模式），防止集成应用自动关闭,  
快速解决Android中的selinux权限问题等文章介绍了三种永久解除SELINUX的方法,但是经笔者实验测试效果不佳或者不实用.
SELINUX有三个模式:
- enforcing 强制执行模式
- permissive 宽容模式
- disable 失能模式
网络上的方法,或许是为了给Android系统的安全性留一点尊重,大多是将SELINUX设置为permissive模式.这里介绍笔者测试有效的一个简单粗暴的方式,直接永久关闭SELINUX,即让它永久失能的方法.  
更改Android源码 system\core\init\init.c

``` c
static bool selinux_is_disable(void)
{
return true;//让这个函数不管青红皂白用于返回true,永久使SELINUX失能
#ifdef ALLOW_DISABLE_SELINUX
...
}
```

## 重新编译Android

因为root系统涉及到更改Android的/system文件系统和修改启动参数配置,所以必须重新编译boot.img和system.img这两个镜像文件,如果时间不紧张的话重新编译整个Android源码也是可以的.  
如何快速的部分编译单个镜像文件,可以参考笔者之前整理的文档Android 镜像编译补充说明

## ROOT结果验证

将新编译好的镜像烧写到板子上,在adb shell里执行getenforce,如果返回disable,则证明SELINUX被成功解除了
在APP内调用命令行su, process = Runtime.getRuntime().exec("su");如果可以成功获取su的Process,则证明系统root成功
