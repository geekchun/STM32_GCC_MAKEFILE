## 写在前面

​     这是一个为STM32F103ZE处理器写的裸机工程的makefile，具有建立工程，编译工程，下载hex文件到目标板的功能。

​	依赖于gcc-arm-none-eabi交叉编译工具链，以及openocd调试软件。硬件上需要一个jlink。

## 如何使用

这个makefile依赖于STM32F1标准外设库3.5.0版本。

#### 建立工程

​	将从官网下载的外设库解压，新建一个工程文件夹，将解压所得的文件夹与makefile一起放在新建的工程文件夹，敲入make project命令，即可完成工程的建立，自动建立分类的文件夹，并将相关文件拷贝到相应位置。

**注意：这将删除工程目录下的标注外设库**

​	对于不同操作系统需要指定HOST参数才能正常编译，默认为Windows

​	**Linux**

```
make project HOST=Linux
```

​	**Windows**

```
make project HOST=Windows
```

建立好的目录结构如下

```c
.
├── cmsis  //存储内核相关文件，不需要修改
│   ├── inc
│   └── src
├── doc  //存放标准外设库下的chm文件，库函数的说明文档
├── driver  //存放自己编写的外设驱动代码
│   ├── inc
│   └── src
├── libraries  //存放标准外设库
│   ├── inc
│   └── src
├── project   //工程文件夹，存放链接脚本以及输出文件
│   └── output  //存放 bin hex elf 文件
├── startup  //存放启动文件
└── user    //存放用户代码
    ├── inc
    └── src
```

#### 编译工程

​	在编辑完用户代码之后，敲入make来编译工程,编译成功后你将看到如下结果，编译生成的hex文件将保存到project/output下。

```bash
...
arm-none-eabi-objcopy ./project/output/led.elf  ./project/output/led.bin -Obinary 
arm-none-eabi-objcopy ./project/output/led.elf  ./project/output/led.hex -Oihex
```

​	对于不同操作系统需要指定HOST参数才能正常编译，默认为Windows

​	**Linux**

```
make HOST=Linux
```

​	**Windows**

```
make HOST=Windows
```

#### 下载调试

​	在连接jlink并安装好相应驱动之后，敲入make update来将hex文件下载到目标板，下载成功后你将看到如下界面。

​	对于不同操作系统需要指定HOST参数才能正常编译，默认为Windows

​	**Linux**

```
make update HOST=Linux
```

​	**Windows**

```
make update HOST=Windows
```

```bash
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
jtag_ntrst_delay: 100
none separate
cortex_m reset_config sysresetreq
Info : No device selected, using first device.
Info : J-Link V9 compiled May 17 2019 09:50:41
Info : Hardware version: 9.60
Info : VTarget = 3.301 V
Info : clock speed 1000 kHz
Info : JTAG tap: stm32f1x.cpu tap/device found: 0x3ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x3)
Info : JTAG tap: stm32f1x.bs tap/device found: 0x06414041 (mfg: 0x020 (STMicroelectronics), part: 0x6414, ver: 0x0)
Info : stm32f1x.cpu: hardware has 6 breakpoints, 4 watchpoints
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x08001000 msp: 0x2000fff0
auto erase enabled
Info : device id = 0x10036414
Info : flash size = 512kbytes
wrote 6144 bytes from file ./project/output/led.hex in 0.332135s (18.065 KiB/s)
Info : JTAG tap: stm32f1x.cpu tap/device found: 0x3ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x3)
Info : JTAG tap: stm32f1x.bs tap/device found: 0x06414041 (mfg: 0x020 (STMicroelectronics), part: 0x6414, ver: 0x0)
shutdown command invoked
```



## 关于移植

​	这个makefile只适用于STM32F103ZE系列单片机，默认工程名称为 “led” ，如果你想用到其他单片机，或者其他工程，可以通过修改相应位置完成移植。

#### 工程名称

​	这个名字对应目标文件的名字

```makefile
TARGET = led
```

#### 不同的单片机

​	不同的单片机对应不同的启动文件和链接脚本，通过修改拷贝不同单片机的启动文件和链接脚本可以完成对其他stm32f10x单片机的移植。注意在编译的位置也要修改。

对于启动文件，主要有两个位置

```makefile
all:$(obj)
	$(CC)  $(as_flag) ./startup/startup_stm32f10x_hd.o ./startup/startup_stm32f10x_hd.s 
	$(CC) $(obj) ./startup/startup_stm32f10x_hd.o -T ./project/stm32_flash.ld -o ./project/output/$(TARGET).elf  $(target_flag)
```

```makefile
project:
	cp $(lib_path)/...../startup/TrueSTUDIO/startup_stm32f10x_hd.s ./startup
	cp $(lib_path)/...../TrueSTUDIO/STM3210E-EVAL/stm32_flash.ld ./project 
```

若是移植到其他F1系列单片机，需要修改project下的stm32_flash.ld文件

```assembly
/* Highest address of the user mode stack */
_estack = 0x20010000;    /* end of 64K RAM */

/* Generate a link error if heap and stack don't fit into RAM */
_Min_Heap_Size = 0;      /* required amount of heap  */
_Min_Stack_Size = 0x200; /* required amount of stack */

/* Specify the memory areas */
MEMORY
{
  FLASH (rx)      : ORIGIN = 0x08000000, LENGTH = 512K
  RAM (xrw)       : ORIGIN = 0x20000000, LENGTH = 64K
  MEMORY_B1 (rx)  : ORIGIN = 0x60000000, LENGTH = 0K
}
```

#### 示例工程

    https://github.com/geekchun/STM32-GCC-PROJECT
