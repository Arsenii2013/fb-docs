# Настройки U-Boot

```
setenv bootcmd ';'
setenv bootdelay -1

env delete bootcmd_mmc0
env delete bootcmd_mmc1
env delete bootcmd_dhcp
env delete bootcmd_jtag
env delete bootcmd_nand
env delete bootcmd_nor
env delete bootcmd_pxe
env delete bootcmd_qspi
env delete bootcmd_usb0
env delete bootcmd_usb1
env delete bootcmd_usb_dfu0
env delete bootcmd_usb_dfu1
env delete bootcmd_usb_thor0
env delete bootcmd_usb_thor1
env delete distro_bootcmd
env delete boot_targets
env delete usb_boot
env delete ubifs_boot
env delete mmc_boot
env delete ubifs_boot
env delete scan_dev_for_boot
env delete scan_dev_for_boot_part
env delete scan_dev_for_efi
env delete scan_dev_for_extlinux
env delete scan_dev_for_scripts
env delete boot_a_script
env delete boot_efi_binary
env delete boot_efi_bootmgr
env delete boot_extlinux
env delete boot_net_usb_start
env delete boot_prefixes
env delete boot_script_dhcp
env delete boot_scripts
env delete efi_dtb_prefixes
env delete load_efi_dtb
env delete boot_syslinux_conf

setenv cpu1_addr 0x3E000000
setenv cpu1_bare_metal xilinx/bare_metal.bin
setenv cpu1_fetch dhcp \$cpu1_addr \$tftpip:\$cpu1_bare_metal
setenv cpu1_start dcache flush\; mw.l 0xFFFFFFF0 \${cpu1_addr} 1\; sev
setenv fpga_addr 0x3C000000
setenv fpga_bitstream xilinx/top.bit.bin
setenv fpga_fetch dhcp \$fpga_addr \$tftpip:\$fpga_bitstream
setenv fpga_load fpga load 0 \$fpga_addr \$filesize
setenv tftpip 192.168.0.32

setenv cpu1_fetch dhcp \${cpu1_addr} \${tftpip}:\$cpu1_bare_metal 
setenv image /xilinx/BOOT.bin
setenv image_addr 0x3D000000 
setenv image_fetch dhcp \$image_addr \$tftpip:\$image
setenv image_load sf update \$image_addr 0x0 \$filesize

setenv reload if run image_fetch\; then sf probe\; run image_load\; reset\; fi
setenv afe_bitstream_addr 0x39000000
setenv bootcmd afe pwr on\; sleep 1\; afe load \$afe_bitstream_addr 400000\; run cpu1_start
setenv bootdelay 0
saveenv
```
