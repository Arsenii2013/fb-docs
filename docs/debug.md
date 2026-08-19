# Отладка и запуск CPU1

Для удобства отладки была сделана система загрузки битстримов и bare metal.  
По включении fsbl загружает из флешки: в плис битстрим, в память uboot и bare metal. Затем отдает управление u-boot. В bootcmd прописано скачивание и загрузка в ПЛИС битстрима по tftp, скачивание и запуск bare metal. U-boot не скачивается и используется тот, который лежит на флешке. 

## На ПК
На ПК надо поднять tftp сервер.  
Важно знать какие файлы нужны для загрузки. Просто .bit или .elf не подойдут, нужны бинарные.  
Для битсрима:  

- Создаем файл `fpga.bif` и пишем в него 
```
fpga:{
    <path-to-bitstream>.bit
}
```
- Теперь собираем через bootgen, получаем на выходе `<path-to-bitstream>.bit.bin`
```
bootgen -image fpga.bif -arch zynq  -w on -process_bitstream bin
```


Для bare metal собраем с помощью arm-none-eabi-objcopy.
```
arm-none-eabi-objcopy -O binary <path-to-bare-meatal>.elf <path-to-bare-meatal>.bin
```

## В U-Boot
```
setenv bootcmd run fpga_fetch; run fpga_load; run cpu1_fetch; run cpu1_start
setenv bootdelay 0
setenv cpu1_addr 0x10000000
setenv cpu1_bare_metal /var/lib/tftpboot/xilinx/bare_metal.bin
setenv cpu1_fetch dhcp $cpu1_addr $tftpip:$cpu1_bare_metal
setenv cpu1_start mw.l 0xFFFFFFF0 $cpu1_addr 1; sev
setenv fpga_addr 0x4000000
setenv fpga_bitstream /var/lib/tftpboot/xilinx/top.bit.bin
setenv fpga_fetch dhcp $fpga_addr $tftpip:$fpga_bitstream
setenv fpga_load fpga load 0 $fpga_addr $filesize
setenv tftpip 192.168.0.32
saveenv
```
Также некотрые переменные будут выставляться по ходу работы различными командами.
```
bootfile=:/var/lib/tftpboot/xilinx/bare_metal.bin
serverip=192.168.0.32
fileaddr=10000000
filesize=8008
```
 