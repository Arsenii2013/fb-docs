# Компиляция U-boot

Для начала надо сделать Device Tree Blob(DTB). Он компилируется из Device Tree(DTS) который генерируется из Hardware Rlatform(XSA), из Vivado. 
Делал по [инструкции от Xilinx](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842279/Build+Device+Tree+Blob).

## Генерируем DTS
Для этого нужен Device Tree Generator, с гитхаба Xilinx. Это набор скриптов которые будут использоваться через Xilinx-овскую командную строку(xsct)

```
git clone https://github.com/Xilinx/device-tree-xlnx
cd device-tree-xlnx
git checkout <xilinx_rel_v20XX.X>
```

Идем к XSA файлу и выполняем по очереди

```
xsct
hsi open_hw_design <design_name>.xsa
hsi set_repo_path <path to device-tree-xlnx repository>
set procs [hsi get_cells -hier -filter {IP_TYPE==PROCESSOR}]
puts "List of processors found in XSA is $procs"
hsi create_sw_design device-tree -os device_tree -proc ps7_cortexa9_0
hsi generate_target -dir <dts output dir>
hsi close_hw_design [hsi current_hw_design]
exit
```

На данном этапе получаются DTS файлы, но их получается несколько, инклюдящих друг друга. Надо слиньковать.

```
gcc -E -nostdinc -undef -D__DTS__ -x assembler-with-cpp -o system.dts system-top.dts
```

## Компилируем в DTB

Нужен Device Tree Compiler.
```
git clone https://git.kernel.org/pub/scm/utils/dtc/dtc.git
cd dtc
make
export PATH=$PATH:/<path-to-dtc>/dtc
``` 

Теперь можно скомпилировать
```
dtc -O dtb -o system.dtb system.dts
```
После этого получится блоб, system.dtb, который нужно использовать при компиляции U-Boot.

## Компиляция U-Boot

[Инструкция от Xilinx](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841973/Build+U-Boot).

Скачиваем исходники
```
git clone https://github.com/Xilinx/u-boot-xlnx.git
cd u-boot-xlnx
```

Настраиваем компилятор. Тут Xilinx использует arm-linux-gnueabihf-, но я собрал с помощью arm-none-eabi-, так как загрузка линукс не предполагется.

```
export CROSS_COMPILE=arm-none-eabi-
export ARCH=arm
```

```
make distclean
make xilinx_zynq_virt_defconfig
export DEVICE_TREE="system"
```

Теперь надо положить system.dtb в arch/arm/dts/.   
И можно собирать.

```
make
```

В результате получится u-boot.elf.

## Проблема с чтением env из QSPI флешки
Наблюдалась проблема c QSPI флешкой.  
При включении не могли прочитаться значения переменных.
```
Loading Environment from SPIFlash... zynq_qspi spi@e000d000: Invalid chip select 0:0 (err=-19)
*** Warning - spi_flash_probe_bus_cs() failed, using default environment
```
Также `sf probe 0` приводило к ошибке 
```
zynq_qspi spi@e000d000: Invalid chip select 0:0 (err=-19)
Failed to initialize SPI flash at 0:0 (error -19)
```
И только явное указание chip select = 0 помогало
```
Zynq> sf probe 0 0 0
SF: Detected s25fl128s with page size 256 Bytes, erase size 64 KiB, total 16 MiB
```
Как я понял это какой то косяк с определением chip select в u-boot. 
Нашел [китайский форум](https://whycan.com/t_8322.html) с такой же проблемой. 
Там показано что поведение u-boota 22 года и 23 отличаются.
Попробовал версию 22 года и все заработало.
```
git clone --depth 1 --branch xilinx-v2022.2  https://github.com/Xilinx/u-boot-xlnx.git
```
Все остальны шаги сборки остаются такими же.

## Загрузка U-Boot по JTAG

[Инструкция с сайта Xilinx](https://docs.xilinx.com/r/en-US/ug1400-vitis-embedded/Loading-U-Boot-over-JTAG).

Помимо U-Boot нужен FSBL. Его можно достать из standalone application проекта в Vitis, например из hello world-а. Нужен файл zynq_fsbl.elf.  

Теперь идем в Vitis, находим справа сниху окно XSCT Console, исполняем команды в нем.
```
connect

targets -set -nocase -filter {name =~ "ARM*#0"}
dow zynq_fsbl.elf 
con
after 3000
stop

dow uboot.elf
con
```

Также если скомпилировали U-Boot без блоба. то можно залить блоб перед запусок U-Boot
```
dow -data system.dtb 0x100000
```

###
Если 
```
Memory write error at 0x4000000. Cannot access DDR: the controller is held in reset
```
то надо проверить точно ли на плате выбран режим загрузки по jtag.

# Создание и загрузка образа

Используем bootgen, идущий с Vivado. 
Для создания образа нужны: fsbl, bitstream, u-boot, и файл с описанием образа - BIF. 
BIF имеет такую структуру:

```
the_ROM_image:
{
	[bootloader]~/xilinx/vitis/bootimg/fsbl.elf
	~/xilinx/vitis/bootimg/top.bit
	~/xilinx/vitis/bootimg/u-boot.elf
}
```

Запускаем bootgen
```
bootgen -image output.bif -arch zynq -o ~/xilinx/vitis/bootimg/BOOT.bin 
```

Также можно генерировать через GUI в Vitis, Vitis - Create Boot Image.

Теперь полученный BOOT.bin можно залить по JTAG в оперативную память, со стартовым адресом, например, 256 Мб. Пишем в Vitis в XSCT console.
```
stop
dow -data ~/xilinx/vitis/bootimg/BOOT.bin 0x10000000
con
```
Теперь надо залить это в флешку. Пишем в U-Boot.
```
sf update 0x10000000 0x0 0x600000
```
последнее число - размер, который у меня получился 6 Мб.
