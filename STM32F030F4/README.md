## 写在前面

​     这是一个为STM32F030F4处理器写的裸机工程的makefile，具有建立工程，编译工程，下载hex文件到目标板的功能。

​	依赖于gcc-arm-none-eabi交叉编译工具链，以及openocd调试软件。硬件上需要一个jlink。

## 如何使用

这个makefile依赖于STM32F0标准外设库1.5.0版本。

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

## 关于移植

​	这个makefile只适用于STM32F030F4系列单片机，默认工程名称为 “led” ，如果你想用到其他单片机，或者其他工程，可以通过修改相应位置完成移植。

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
	$(CC)  $(as_flag) ./startup/startup_stm32f030.o ./startup/startup_stm32f030.s 
	$(CC) $(obj) ./startup/startup_stm32f030.o -T ./project/stm32_flash.ld -o ./project/output/$(TARGET).elf  $(target_flag)
```

```makefile
project:
	cp $(lib_path)/Libraries/CMSIS/Device/ST/STM32F0xx/Source/Templates/TrueSTUDIO/startup_stm32f030.s ./startup
```

若是移植到其他F0系列单片机，需要修改project下的stm32_flash.ld文件

```assembly
/* Highest address of the user mode stack */
_estack = 0x20000FFF;    /* end of RAM */

/* Generate a link error if heap and stack don't fit into RAM */
_Min_Heap_Size = 0x200;      /* required amount of heap  */
_Min_Stack_Size = 0x400; /* required amount of stack */

/* Specify the memory areas */
MEMORY
{
FLASH (rx)      : ORIGIN = 0x8000000, LENGTH = 16K
RAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 4K
}
```

#### 示例工程

    https://github.com/geekchun/STM32-GCC-PROJECT