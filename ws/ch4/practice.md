## 라즈베리파이 리눅스 돌리기
> https://www.raspberrypi.com/documentation/computers/linux_kernel.html

### u-boot 빌드
- [ch3 practice 8. uboot 빌드하기](../ch3/ch3.md)참고

### 커널 소스 가져오기
```
git clone --depth=1 https://github.com/raspberrypi/linux
git clone --depth=1 https://github.com/raspberrypi/firmware.git
```

### 설정 변수
```
cd linux
KERNEL=kernel8
make bcm2711_defconfig
```

### 커널 빌드
```
make -j6 Image.gz modules dtbs
```

### 커널 모듈 설치
\* INSTALL_MOD_PATH 위치 확인
```
sudo env PATH=$PATH make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/root modules_install
```

### 커널 파일 복사
/* boot 위치 확인
```
sudo cp /boot/firmware/$KERNEL.img /boot/firmware/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image.gz /boot/firmware/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/overlays/
```


### U BOOT env 설정
> https://www.thegoodpenguin.co.uk/blog/build-boot-linux-on-raspberry-pi-3-model-b/
```
U-Boot> setenv bootcmd 'fatload mmc 0 ${kernel_addr_r} Image ; fatload mmc 0 ${fdt_addr_r} bcm2837-rpi-'
U-Boot> setenv bootargs 'console=ttyS1,115200n8 rootwait root=/dev/mmcblk0p2'
U-Boot> saveenv
U-Boot> reset
```


### WHY....
```
U-Boot 2024.10-rc3-00012-gee2af844ba1b (Sep 01 2024 - 00:02:03 +0900)

DRAM:  128 MiB
RPI 3 Model B+ (0xa020d3)
Core:  87 devices, 13 uclasses, devicetree: board
MMC:   mmc@7e202000: 0, mmcnr@7e300000: 1
Loading Environment from FAT... OK
In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
Net:   No ethernet found.

starting USB...
Bus usb@7e980000: USB DWC2
scanning bus usb@7e980000 for devices... 4 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  0 
Failed to load 'Image'
21443 bytes read in 3 ms (6.8 MiB/s)
Bad Linux ARM64 Image magic!

```

