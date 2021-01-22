#project name
TARGET = led

HOST = Windows
CROSS = arm-none-eabi-

export CC         = $(CROSS)gcc           
export AS         = $(CROSS)as
export LD         = $(CROSS)ld
export OBJCOPY    = $(CROSS)objcopy

ifeq ($(HOST),Linux)
top = $(shell pwd)
else ifeq ($(HOST),Windows)
top = $(shell echo %cd%)
endif

ifeq ($(HOST),Linux)
lib_path= ./STM32F0xx_StdPeriph_Lib_V1.5.0
else ifeq ($(HOST),Windows)
lib_path= $(top)\STM32F0xx_StdPeriph_Lib_V1.5.0
endif

inc = -I $(top)/cmsis/inc \
      -I $(top)/libraries/inc \
      -I $(top)/user/inc \
      -I $(top)/driver/inc 

obj_flag    = -Os -W -Wall -g -mcpu=cortex-m0 -mthumb -D STM32F030 -D USE_STDPERIPH_DRIVER $(inc) -std=gnu11 
target_flag = -Os -mthumb -mcpu=cortex-m0 -Wl,--start-group -lc -lm -Wl,--end-group \
	      -specs=nano.specs -specs=nosys.specs -static -Wl,-cref,-u,Reset_Handler \
	      -Wl,-Map=./project/output/$(TARGET).map -Wl,--gc-sections -Wl,--defsym=malloc_getpagesize_P=0x80
as_flag     = -Os -c -mthumb -mcpu=cortex-m0 -g -Wa,--warn -o


ifeq ($(HOST),Linux)
src = $(shell find ./ -name '*.c')
else ifeq ($(HOST),Windows)
src = $(shell for /r ./ %%i in (*.c) do @echo %%i)
endif

obj = $(src:%.c=%.o)


all:$(obj)
	$(CC)  $(as_flag) ./startup/startup_stm32f030.o ./startup/startup_stm32f030.s 
	$(CC) $(obj) ./startup/startup_stm32f030.o -T ./project/stm32_flash.ld -o ./project/output/$(TARGET).elf  $(target_flag)
	$(OBJCOPY) ./project/output/$(TARGET).elf  ./project/output/$(TARGET).bin -Obinary 
	$(OBJCOPY) ./project/output/$(TARGET).elf  ./project/output/$(TARGET).hex -Oihex

$(obj):%.o:%.c
	$(CC) -c $(obj_flag) -o $@ $<

project:
ifeq ($(HOST),Linux)
	mkdir cmsis cmsis/inc cmsis/src driver driver/inc driver/src project project/output user user/inc user/src doc startup
	cp $(lib_path)/stm32f0xx_stdperiph_lib_um.chm ./doc
	cp -r $(lib_path)/Libraries/STM32F0xx_StdPeriph_Driver ./
	mv STM32F0xx_StdPeriph_Driver libraries
	cp $(lib_path)/Libraries/CMSIS/Include/* ./cmsis/inc
	cp $(lib_path)/Libraries/CMSIS/Device/ST/STM32F0xx/Include/* ./cmsis/inc
	cp $(lib_path)/Libraries/CMSIS/Device/ST/STM32F0xx/Source/Templates/TrueSTUDIO/startup_stm32f030.s ./startup
	cp $(lib_path)/Libraries/CMSIS/Device/ST/STM32F0xx/Source/Templates/system_stm32f0xx.c ./cmsis/src 
	cp $(lib_path)/Projects/STM32F0xx_StdPeriph_Templates/main.h ./user/inc
	cp $(lib_path)/Projects/STM32F0xx_StdPeriph_Templates/stm32f0xx_it.c ./user/src 
	cp $(lib_path)/Projects/STM32F0xx_StdPeriph_Templates/stm32f0xx_it.h ./user/inc
	cp $(lib_path)/Projects/STM32F0xx_StdPeriph_Templates/stm32f0xx_conf.h ./user/inc
	cp $(lib_path)/Projects/STM32F0xx_StdPeriph_Templates/TrueSTUDIO/STM32F030/STM32F030R8_FLASH.ld ./project/stm32_flash.ld
	touch ./user/src/main.c
	rm -rf $(lib_path)

else ifeq ($(HOST),Windows)
	md cmsis cmsis\inc cmsis\src driver driver\inc driver\src project project\output user user\inc user\src doc startup libraries
	copy $(lib_path)\stm32f0xx_stdperiph_lib_um.chm $(top)\doc
	xcopy /e $(lib_path)\Libraries\STM32F0xx_StdPeriph_Driver $(top)\libraries
	xcopy $(lib_path)\Libraries\CMSIS\Include\core*.h $(top)\cmsis\inc
	xcopy $(lib_path)\Libraries\CMSIS\Include\arm*.h $(top)\cmsis\inc
	xcopy $(lib_path)\Libraries\CMSIS\Device\ST\STM32F0xx\Include\stm32f0xx.h $(top)\cmsis\inc
	xcopy $(lib_path)\Libraries\CMSIS\Device\ST\STM32F0xx\Include\system_stm32f0xx.h $(top)\cmsis\inc
	xcopy $(lib_path)\Libraries\CMSIS\Device\ST\STM32F0xx\Source\Templates\TrueSTUDIO\startup_stm32f030.s $(top)\startup
	xcopy $(lib_path)\Libraries\CMSIS\Device\ST\STM32F0xx\Source\Templates\system_stm32f0xx.c $(top)\cmsis\src 
	xcopy $(lib_path)\Projects\STM32F0xx_StdPeriph_Templates\main.h $(top)\user\inc
	xcopy $(lib_path)\Projects\STM32F0xx_StdPeriph_Templates\stm32f0xx_it.c $(top)\user\src 
	xcopy $(lib_path)\Projects\STM32F0xx_StdPeriph_Templates\stm32f0xx_it.h $(top)\user\inc
	xcopy $(lib_path)\Projects\STM32F0xx_StdPeriph_Templates\stm32f0xx_conf.h $(top)\user\inc
	xcopy $(lib_path)\Projects\STM32F0xx_StdPeriph_Templates\TrueSTUDIO\STM32F030\STM32F030R8_FLASH.ld $(top)\project
	rename $(top)\project\STM32F030R8_FLASH.ld stm32_flash.ld
	echo #include "stm32f0xx.h" int main() {}> $(top)\user\src\main.c
	rd /s /q $(lib_path)
endif

update:
ifeq ($(HOST),Linux)
	openocd -f /usr/local/share/openocd/scripts/interface/jlink.cfg \
	-f /usr/local/share/openocd/scripts/target/stm32f1x.cfg \
	-c init -c halt -c "flash write_image erase ./project/output/$(TARGET).hex" -c reset -c shutdown
else ifeq ($(HOST),Windows)
	openocd -f $(top)/project/jlink.cfg \
	-f $(top)/project/stm32f1x.cfg \
	-c init -c halt -c "flash write_image erase ./project/output/$(TARGET).hex" -c reset -c shutdown
endif

clean:
ifeq ($(HOST),Linux)
	rm -f $(shell find ./ -name '*.o')
	rm -f $(shell find ./ -name '*.map')
	rm -f $(shell find ./ -name '*.elf')
	rm -f $(shell `find ./ -name '*.bin')
	rm -f $(shell find ./ -name '*.hex')
else ifeq ($(HOST),Windows)
	del /q  $(shell for /r ./ %%i in (*.o,*.map,*.hex,*.elf,*.bin) do @echo %%i)
endif
