# RTMP推流延时解决办法


首先需要确定是哪一端造成的延时:
+ 推流端:使用obs studio代替自己写的推流端，对比延时。如果延时差别大则推流端存在延时
+ 服务端:使用局域网代替服务器，对比延时。一般会有较大差别
+ 拉流端:使用播放器代替拉流程序，对比延时。。。。

## 推流端延时


### 整定时序参数

pts dts duration time_base


### 一些其他设置

``` c
av_dict_set(&param, "preset", "superfast", 0);
av_dict_set(&param, "tune", "zerolatency", 0);

av_dict_set(&param, "rtbufsize", "10", 0);
av_dict_set(&param, "start_time_realtime", 0, 0);
```

参考:
https://blog.csdn.net/zhuweigangzwg/article/details/82223011

## 服务端延时

感觉只能提升服务端网速

或者使用CDN等技术来缩短链路


## 拉流端延时
