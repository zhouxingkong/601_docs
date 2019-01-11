
## 1.底层系统服务裁剪

\system\core\rootdir\init.rc

>init.rc文件语法参考
https://www.jianshu.com/p/cb73a88b0eed

### 1.1 Action

动作的一般格式如下：
```
on  <trigger>           ## 触发条件
      <command>     ##执行命令
      <command1>   ##可以执行多个命令
```
从上面，我们可以知道，一个Action其实就是响应某事件的过程。即当<trigger>所描述的触发事件产生时，依次执行各种command(同一事件可以对应多个命令)。

### 1.2 service

"服务"(service)的关键字 service后面跟着的是服务名称。我们可以使用"start"命令加"服务名称"来启动一个服务。关键字以下的行称为"选项"，没一个选项占用一行，选项也有多种，常见的"class"表示服务所属的类别。

services 其实是可执行程序，它们在特定选项的约束下是被init程序运行或者重启(service可以在配置中指定是否需要退出重启，这样当service出现异常crash时就可以有机会复原)
```
service <name><pathname> [ <argument> ]*
    <option>
    <option>
```
我们简单的解释上面的参数

+ \<name>：表示此service的名称
+ \<pathname>：此service所在路径。因为是可执行文件，所以一定有存储路径
+ \<argument>：启动service所带的参数
+ \<option>：对此service的约束选项

### 1.3 Android所有系统服务

本节列出了android5.1系统init.rc中所有使用service关键字启动的系统服务。

服务名称|功能|操作
--|--|--
ueventd|创建或删除设备节点的守护进程|不能删
logd|打印调试日志|看使用
healthd|获取电池电量数据|看使用
adbd|adb守护进程|别删吧
lmkd  |lowmemorykiller驱动程序,终止不必要的程来缓解内存压力 https://www.jianshu.com/p/1ef6e8a1c773|不知道
servicemanager|服务管理进程,binder通信框架组成部分|不能删
vold|外部存储设备管理守护进程|不能删
netd|网络守护进程|不能删
debuggerd|应用程序崩溃诊断守护进程|可删
rild|Android中负责通信功能的Telephony中间层 https://blog.csdn.net/jason_wzn/article/details/53232022 |应该可以删
surfaceflinger|Android界面显示系统|不能删
drmserver|数字版权管理服务|可删
mediaserver|媒体服务|不能删
vdc|加密库crypto|可删
bootanimation|开机动画|不能删
installd|应用程序安装守护进程|不能删
racoon|和网络有关|不知道
mtpd  |媒体传输协议守护进程|不知道
keystore|证书服务|可删
dumpstate|一种性能测试工具|可删
mdnsd  |多播DNS守护进程|可删  
uncrypt|也和加密有关|可删

## 2.上层系统服务裁剪

### 2.1 上层服务启动流程

SystemServer功能为启动各种上层服务。源码位于
\frameworks\base\services\java\com\android\server\SystemServer.java

不同Android版本有不同的实现。至少Android5.1版本和4.2.2版本已经不一样了。本文介绍5.1版本裁剪

启动上层服务的代码在run()方法中。

``` java
private void run() {
//.......
        // Initialize native services.
        System.loadLibrary("android_servers");
        nativeInit();
//......................
        // Start services.
        try {
            startBootstrapServices();   //这里面启动的服务不能删
            startCoreServices();  //这里也不能删
            startOtherServices(); //主要删这里的
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        }
//.............
}
```

所以上层服务裁剪就是在startOtherServices()中裁剪服务的启动代码。


### 2.2 上层服务裁剪

服务名称|功能|操作
--|--|--
SchedulingPolicyService|调度策略|不能删


## 3.应用程序裁剪
