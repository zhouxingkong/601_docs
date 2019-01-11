# LCD驱动移植

2018.4.3
周星宇、闫隆鑫（ylx601@gmail.com）

## 1.LCD触摸屏浅析
一个Linux的触摸屏驱动其实包含三个部分：
1.	LCD设备驱动，负责图像显示的部分；
2.	触摸驱动，负责触摸屏下触摸板的响应；
3.	背光驱动，负责显示屏亮度的调节。
本文的余下部分将会分别从这三个部分介绍电阻触摸屏在Android系统上的移植细节。
+	电阻触摸屏型号：AM-800600P4TMQW-TB0H
+	Android源码版本：Android 5.1
+	Android开发板型号：NanoPI T2(友善之臂提供)

### 2.1 LCD设备描述
NanoPi2支持的所有LCD型号使用一个结构体来存储nanopi2_lcd_config
arch\arm\plat-s5p4418\nanopi2\lcds.c

``` c
1.	/* Try to guess LCD panel by kernel command line, or
2.	 * using *HD101* as default  
3.	 * \linux-3.4.y-nanopi2-lollipop-mr1\arch\arm\plat-s5p4418\nanopi2\lcds.c
4.	*/  
5.	static struct {  
6.	    char *name;  
7.	    struct nxp_lcd *lcd;  
8.	    int ctp;  
9.	} nanopi2_lcd_config[] = {  
10.	    { "HD101",  &wxga_hd101, 1 },  
11.	    { "HD101B", &wxga_hd101, CTP_GOODIX  },  
12.	    { "HD700",  &wxga_hd700, 1 },  
13.	    { "HD702",  &wxga_hd700, CTP_GOODIX  },  
14.	    { "S70",    &wvga_s70,   1 },  
15.	    { "S702",   &wvga_s702,  1 },  
16.	    { "S70D",   &wvga_s70d,  0 },  
17.	    { "X710",   &wsvga_x710, CTP_ITE7260 },   
18.	    { "S430",   &wvga_s430,  CTP_HIMAX   },  
19.	#ifndef CONFIG_ANDROID  
20.	    { "H43",    &hvga_h43,   0 },  
21.	    { "P43",    &hvga_p43,   0 },  
22.	    { "W35",    &qvga_w35,   0 },  
23.	#endif  
24.	    /* TODO: Testing */  
25.	    { "W50",    &wvga_w50,   0 },  
26.	    { "W101",   &wsvga_w101, 1 },  
27.	    { "A97",    &xga_a97,    0 },  
28.	    { "LQ150",  &xga_lq150,  1 },  
29.	    { "L80",    &vga_l80,    1 },  
30.	    { "BP101",  &wxga_bp101, 1 },  
31.	    { "HDMI",   &hdmi_def,   0 },   /* Pls keep it at last */  
32.	};  

```

其中存储了每种LCD的名称与驱动参数。
第一列：LCD名字
第二列：LCD参数结构体
第三列：使用触摸芯片型号
下面用wsvga_x710型号的LCD举例。

``` c
1.	/*  
2.	 * \linux-3.4.y-nanopi2-lollipop-mr1\arch\arm\plat-s5p4418\nanopi2\lcds.c
3.	 */  
4.	static struct nxp_lcd wsvga_x710 = {  
5.	    .width= 1024,  
6.	    .height = 600,  
7.	    .p_width = 154,  
8.	    .p_height = 90,  
9.	    .bpp = 24,  
10.	    .freq = 61,  
11.	    .timing = {  
12.	        .h_fp = 84,  
13.	        .h_bp = 84,  
14.	        .h_sw = 88,  
15.	        .v_fp = 10,  
16.	        .v_fpe = 1,  
17.	        .v_bp = 10,  
18.	        .v_bpe = 1,  
19.	        .v_sw = 20,  
20.	    },  
21.	    .polarity = {  
22.	        .rise_vclk = 0,  
23.	        .inv_hsync = 1,  
24.	        .inv_vsync = 1,  
25.	        .inv_vden = 0,  
26.	    },  
27.	    .gpio_init = hd101_gpio_init,  
28.	};  

```


### 2.2 驱动移植
1)	根据数据手册《附件3 AM-800600P4TMQW-TB0H（TTL接口 4线电阻触摸 400流明 带背光驱动）》中的相关数据定义我们所用型号的LCD描述结构体：

``` c
1.	static struct nxp_lcd wsvga_l601 = {  
2.	 .width= 800,  
3.	 .height = 600,  
4.	 .p_width = 163,  
5.	 .p_height = 122,  
6.	 .bpp = 24,  
7.	 .freq = 60,  
8.	   
9.	 .timing = {  
10.	  .h_fp = 112,  
11.	  .h_bp = 40,  
12.	  .h_sw = 48,  
13.	  .v_fp = 21,  
14.	  .v_fpe = 1,  
15.	  .v_bp = 36,  
16.	  .v_bpe = 1,  
17.	  .v_sw = 3,  
18.	 },  
19.	 .polarity = {  
20.	  .rise_vclk = 1,  
21.	  .inv_hsync = 1,  
22.	  .inv_vsync = 1,  
23.	  .inv_vden = 0,  
24.	 },  
25.	 .gpio_init = s70_gpio_init,  
26.	 //.gpio_init = hd101_gpio_init,  
27.	};  
28.	   

```

2)	把我们刚刚定义的结构体添加进nanopi2_lcd_config中，我们把它添加到了下标为8的位置；

``` c
1.	static struct {  
2.	 char *name;  
3.	 struct nxp_lcd *lcd;  
4.	 int ctp;  
5.	} nanopi2_lcd_config[] = {  
6.	 { "HD101", &wxga_hd101, 1 },  
7.	 { "HD101B", &wxga_hd101, CTP_GOODIX  },  
8.	 { "HD700", &wxga_hd700, 1 },  
9.	 { "HD702", &wxga_hd700, CTP_GOODIX  },  
10.	 { "S70", &wvga_s70,   1 },  
11.	 { "S702", &wvga_s702,  1 },  
12.	 { "S70D", &wvga_s70d,  0 },  
13.	 { "X710", &wsvga_x710, CTP_ITE7260 },  
14.	 {"L601", &wsvga_l601, 0},  
15.	 { "S430", &wvga_s430,  CTP_HIMAX   },

```

3)	修改nanopi2_setup_lcd函数，这个函数的原理是根据1-wire协议读取到的lcd屏的名称，将名称与nanopi2_lcd_config中的字符串对比，选择名称一样的结构体进行加载。但是我们使用的屏幕不支持1-wire协议，所以我们在函数中将加载的lcd描述结构体标号强制赋值为8；
``` c
1.	static int __init nanopi2_setup_lcd(char *str)  
2.	{  
3.	 char *delim;  
4.	 int i;  
5.	   
6.	 delim = strchr(str, ',');  
7.	 if (delim)  
8.	  *delim++ = '\0';  
9.	   
10.	 if (!strncasecmp("HDMI", str, 4)) {  
11.	  goto __ret;//ylx add  
12.	  struct hdmi_config *cfg = &nanopi2_hdmi_config[0];  
13.	  struct nxp_lcd *lcd;  
14.	   
15.	  lcd_idx = ARRAY_SIZE(nanopi2_lcd_config) - 1;  
16.	  lcd = nanopi2_lcd_config[lcd_idx].lcd;  
17.	   
18.	  for (i = 0; i < ARRAY_SIZE(nanopi2_hdmi_config); i++, cfg++) {  
19.	   if (!strcasecmp(cfg->name, str)) {  
20.	    lcd->width = cfg->width;  
21.	    lcd->height = cfg->height;  
22.	    goto __ret;  
23.	   }  
24.	  }  
25.	 }  
26.	   
27.	 for (i = 0; i < ARRAY_SIZE(nanopi2_lcd_config); i++) {  
28.	  goto __ret;//ylx add  
29.	  if (!strcasecmp(nanopi2_lcd_config[i].name, str)) {  
30.	   lcd_idx = i;  
31.	   break;  
32.	  }  
33.	 }  
```

## 3.触摸
### 3.1 编译驱动
修改.config文件，添加触摸屏驱动支持；更方便的方法是在make menuconfig 中修改Linux内核编译要加载的驱动模块；具体路径为：Device DriversInput device supportTouchscreens
注意：为了避免触摸屏使用的I2C地址发生冲突，安全起见除了要保留的触摸屏驱动tsc2007其他驱动全部取消。
```
1.	# CONFIG_TOUCHSCREEN_TOUCHIT213 is not set  
2.	# CONFIG_TOUCHSCREEN_TSC_SERIO is not set  
3.	# CONFIG_TOUCHSCREEN_TSC2005 is not set  
4.	CONFIG_TOUCHSCREEN_TSC2007=y
5.	# CONFIG_TOUCHSCREEN_ST1232 is not set  
6.	# CONFIG_TOUCHSCREEN_TPS6507X is not set  
```

### 3.2 定义I2c设备
#### 1)	定义I2C设备驱动的相关结构体
触摸屏相关的I2C设备在
linux-3.4.y-nanopi2-lollipop-mr1\arch\arm\plat-s5p4418\nanopi2\device.c
中定义。

``` c
1.	//ylx add   
2.	#if defined(CONFIG_TOUCHSCREEN_TSC2007)   
3.	#define  TSC2007_I2C_BUS    (2)   
4.	static int tsc2007_get_pendown_state(void){   
5.	  return !gpio_get_value(CFG_IO_TOUCH_IRQ);   
6.	}   
7.	   
8.	struct tsc2007_platform_data tsc2007_data ={   
9.	  .model = 2007,   
10.	  .x_plate_ohms = 180,   
11.	  .get_pendown_state = tsc2007_get_pendown_state,   
12.	};   
13.	   
14.	static struct i2c_board_info __initdata tsc2007_i2c_bdi = {   
15.	  I2C_BOARD_INFO("tsc2007", (0x90>>1)),   
16.	  .irq = PB_PIO_IRQ(CFG_IO_TOUCH_IRQ),   
17.	  .platform_data = &tsc2007_data,   
18.	};   
19.	#endif

```

其中需要注意：
1.	I2C设备的地址需要改成触摸芯片手册中的(例子中tsc2007为0x90>>1)
1.	"tsc2007"为I2C设备名称，需要和驱动tsc2007设备驱动程序中的驱动名称一致,具体的驱动设备名称可以在\drivers\input\touchscreen\tsc2007.c 中的tsc2007_probe函数中查看。input_dev->name = "tsc2007";  
1.	不同触摸芯片型号需要填充I2c_board_info结构体中的不同成员，(如tsc2007需要填充device成员)。
1.	tsc2007_platform_data的定义包含在在头文件中，记得在程序头部加入#include <linux/i2c/tsc2007.h>

#### 2)	注册I2C设备
在nxp_board_devices_register函数中添加

``` c
1.	//ylx add   
2.	#if defined(CONFIG_TOUCHSCREEN_TSC2007)   
3.	  printk("plat: add touch(tsc2007) device\n");   
4.	  i2c_register_board_info(TSC2007_I2C_BUS, &tsc2007_i2c_bdi, 1);   
5.	#endif   
6.	   
```

### 3.3 修改驱动程序
驱动程序所在路径为： \drivers\input\touchscreen\tsc2007.c
设备驱动程序一般情况下linux内核的drivers目录中会自带，也很少需要我们去修改。只要修改.config文件将它们编译进内核就可以了。但是在本例中还是要针对实际情况略作修改：
1)	修改屏幕分辨率，根据实际使用的LCD分辨率修改相应的宏定义：（本例中的LCD分辨率为800x600）

```
1.	#define LCD_SCREEN_X_PIXEL  800 //1024    
2.	#define LCD_SCREEN_Y_PIXEL  600 //600  

```

2)	修改CFG_IO_TOUCH_PENDOWN_DETECT，这个宏定义对应的是电阻触摸芯片pendown脚所连接的CPU的引脚，作用是感应触摸操作并发起相应的中断请求；我们可以通过查阅板子的原理图，找到这个pendown实际对应的CPU的引脚。在本例中实际 对应的引脚与宏定义CFG_IO_TOUCH_IRQ所定义的引脚一致。

```
1.	#define CFG_IO_TOUCH_PENDOWN_DETECT CFG_IO_TOUCH_IRQ   
```

3)	修改软中断处理函数tsc2007_soft_irq。先把驱动烧写如板子里，试验一下。看实际操作中，x,y坐标的移动方向是否与实际操作相反。根据实际情况决定x\y是否要反向。

``` c
1.	tc.x = LCD_SCREEN_X_PIXEL*(x)/(ADCVAL_X_MAX -ADCVAL_X_MIN);    
2.	 tc.y = LCD_SCREEN_Y_PIXEL*(y)/(ADCVAL_Y_MAX-ADCVAL_Y_MIN);   
3.	tc.x = LCD_SCREEN_X_PIXEL - tc.x; //x反向  
4.	tc.y = LCD_SCREEN_Y_PIXEL - tc.y; //y反向

```

4)	校准x、y坐标的最大最小值

``` c
1.	#define ADCVAL_X_MIN    120 //150    
2.	#define ADCVAL_X_MAX    3970 //3950    
3.	#define ADCVAL_Y_MIN    190 //280    
4.	#define ADCVAL_Y_MAX    3930 //3850   

```

校准方法：
a)	在tsc2007_soft_irq中在对tc.x、tc.y做处理前，添加以下代码：

``` c
1.	//ylx add   
2.	      printk(    
3.	        "DOWN point(%4d,%4d), pressure (%4u)\n",   
4.	        tc.x, tc.y, rt);   

```

b)	使用adb shell进入Android系统，输入dmesg查看内核信息。
c)	分别点击屏幕的上下左右边缘，使用dmesg查看log中打印的坐标，分别选出x、y的最大最小值，把它写入tsc2007.c中的宏定义里。
下面再介绍一个移植设备驱动的诀窍
诀窍：如何编写设备注册程序可参考linux内核源码中未编译进内核的其他平台的实现(如linux-3.4.y-nanopi2-lollipop-mr1\arch\arm\mach-imx\mach-cpuimx51sd.c中就有tsc2007设备的注册，如下图所示)

``` c
1.	//linux-3.4.y-nanopi2-lollipop-mr1\arch\arm\mach-imx\mach-cpuimx51sd.c  
2.	static struct tsc2007_platform_data tsc2007_info = {  
3.	    .model          = 2007,  
4.	    .x_plate_ohms       = 180,  
5.	    .get_pendown_state  = ts_get_pendown_state,  
6.	    .init_platform_hw   = ts_init,  
7.	};  
8.	  
9.	static struct i2c_board_info tsc_device = {  
10.	    I2C_BOARD_INFO("tsc2007", 0x48),  
11.	    .type       = "tsc2007",  
12.	    .platform_data  = &tsc2007_info,  
13.	    /*.irq is selected on ap4evb_init */  
14.	};  

```

这样一来器件I2C地址也不需要去芯片手册找了。

### 3.3 修改IDC文件

如果是linux系统，上面的工作做完之后已经ok。
但是android系统上还需要修改.idc文件，如果不修改的话会变成触摸鼠标
编写tsc2007.idc如下，将其adb push入Android系统的system/usr/idc目录下，也可放在out/target/product/nanopi2/system/usr/idc下重新编译镜像，重新烧写板子。

``` c
1.	touch.deviceType = touchScreen  
2.	touch.orientationAware = 1  
3.	  
4.	# Size  
5.	touch.size.calibration = diameter  
6.	touch.size.scale = 10  
7.	touch.size.bias = 0  
8.	touch.size.isSummed = 0  
9.	  
10.	# Pressure  
11.	touch.pressure.calibration = amplitude  
12.	touch.pressure.scale = 0.005  
13.	  
14.	# Orientation  
15.	touch.orientation.calibration = none  

```

参考：
http://blog.chinaunix.net/uid-149881-id-4347448.html

## 4.背光驱动
Linux提供了专门的backlight驱动框架，根据实际情况更改里面的PWM接口即可。
由于我们所使用的LCD屏本身亮度较低，在实际使用中要一直将亮度置为最高，所以直接将背光引脚接至5V直流上（相当于100%PWM）
这部分驱动的移植，读者可以自行查阅相关资料开展。


## 5.参考资料

+	关于怎么编译Linux内核 和Android镜像可以参考：
+	http://wiki.friendlyarm.com/wiki/index.php/NanoPC-T2/zh
+	ref/ Android镜像编译补充说明.pdf
+	随本文档的文件夹ref/中的所有文档
+	https://github.com/ylxseu/linux3.4y-lab601
+ 最终版的Linux内核源码
