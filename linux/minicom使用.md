# minicom使用指南

minicom安装

```
sudo apt-get install minicom
```
安装完成后，请不要着急打开软件。需先进行配置。具体步骤如下：

linux下的所有操作面向用户的都是文件操作，在对串口操作之前，我们应该先确认自己对该文件有没有读写权限。
```
ls -l /dev/ttyUSB*
```
linux下的usb串口命名为ttyUSB，运行上面命令，可以看到有几个设备挂载。
我们这里是：
```
crw-rw---- 1 root dialout 188, 0 Apr 10 17:10 /dev/ttyUSB0
```

输入 minicom -s 开始配置:
```
[root@ ~]# minicom -s
```
弹出如下的选单

```
            +-----[configuration]------+
            | Filenames and paths      |
            | File transfer protocols  |
            | Serial port setup        |
            | Modem and dialing        |
            | Screen and keyboard      |
            | Save setup as dfl        |
            | Save setup as..          |
            | Exit                     |
            | Exit from Minicom        |
            +--------------------------+
```

用方向键「↑↓」选择 Serial port setup, 然后回车进入配置:

每个选项前面都有一字母，输入该字母就可以改变这些选项
我直接接串口, 设备名称为 /dev/ttyUSB0, 所以键盘按字母a将默认的改为/dev/ttyUSB0, 修改完按回车「Enter」就可以了

然后按 F 把 Hardware Flow Control 关闭.

最后修改结果如下, 这样就能正常工作了:
```
    +-----------------------------------------------------------------------+
    | A -    Serial Device      : /dev/ttyUSB0                                |
    | B - Lockfile Location     : /var/lock                                 |
    | C -   Callin Program      :                                           |
    | D -  Callout Program      :                                           |
    | E -    Bps/Par/Bits       : 115200 8N1                                |
    | F - Hardware Flow Control : No                                        |
    | G - Software Flow Control : No                                        |
    |                                                                       |
    |    Change which setting?                                              |
    +-----------------------------------------------------------------------+
```

重启开发板，同时在minicom命令行中狂点空格键，就能够中断开发板的系统引导，进入uboot命令行中。
