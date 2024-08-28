## 라즈베리파이 u-boot 빌드하기
> http://www.wearedev.net/284?PHPSESSID=0be74ec2d642c13c2099d2da4a3ff974

### cross-ng로 컴파일러 빌드하기
#### 1. 작업 환경 만들기
```
$ ./bootstrap
$ ./configure --prefix=${PWD}
$ make
$ make install
```
#### 2. 타겟 컴파일러 확인하기 ->  rpi3
```
$ bin/ct-ng list-samples | grep rpi3
[L...]   aarch64-rpi3-linux-gnu
[L...]   armv8-rpi3-linux-gnueabihf
```
64-bit를 사용하기 위해 aarch64-rpi3-linux-gnu로 결정  
```
$ bin/ct-ng show-aarch64-rpi3-linux-gnu
[L...]   aarch64-rpi3-linux-gnu
    Languages       : C,C++
    OS              : linux-6.10
    Binutils        : binutils-2.42
    Compiler        : gcc-14.2.0
    Linkers         :
    C library       : glibc-2.40
    Debug tools     : gdb-15.1
    Companion libs  : expat-2.5.0 gettext-0.22.5 gmp-6.2.1 isl-0.26 libiconv-1.16 mpc-1.3.1 mpfr-4.2.1 ncurses-6.4 zlib-1.3 zstd-1.5.6
    Companion tools :

```
```
$ bin/ct-ng aarch64-rpi3-linux-gnu
  CONF  aarch64-rpi3-linux-gnu
#
# configuration written to .config
#

***********************************************************

Initially reported by: Bryan Hundven
URL: 

Comment:
Raspberry PI 3 aarch64

***********************************************************

Now configured for "aarch64-rpi3-linux-gnu"
```
#### 3. menuconfig
menuconfig에서 설정한다.
```
$ bin/ct-ng menuconfig
```
현재는 Paths and misc options -> Maximum log level to see -> DEBUG 만 설정  
#### 4. build
```
$ bin/ct-ng build
```
> LD_LIBRARY_PATH 에서 에러 나올 수 있음 $unset LD_LIBRARY_PATH 실행하기
#### 5. 컴파일러 경로 설정
```
$ PATH=$PATH:$HOME/x-tools/aarch64-rpi3-linux-gnu/bin/
```
#### 6. 컴파일러 테스트  
dummy.c
```
int main(void)
{
    retun 0;
}
```
```
$ aarch64-rpi3-linux-gnu-gcc dummy.c -o dummy
$ file dummy
```
expected result
```
dummy: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 6.10.0, with debug_info, not stripped
```
#### 7. 컴파일러 export
```
$ export ARCH=arm64
$ export CROSS_COMPILE=$HOME/x-tools/aarch64-rpi3-linux-gnu/bin/aarch64-rpi3-linux-gnu-
```
#### 8. uboot 빌드하기
빌드 가능한 목록 u-boot/configs 에서 확인 가능 (라즈베리파이 3B+는 rpi_3_b_plus_defconfig)
```
$ make rpi_3_b_plus_defconfig
$ make
```

#### 9. 라즈베리파이 펌웨어 받기 (디바이스 트리 기타 등등)
```
$ git clone https://github.com/raspberrypi/firmware.git
$ cd firmware
$ cp boot/bootcode.bin {sd boot}/.
$ cp boot/start.elf {sd boot}/.
$ cp boot/bcm2710-rpi-3-b-plus.dtb {sd boot}/.
```

#### 10. u-boot 복사
```
$ cp u-boot.bin {sd boot}/.
```

#### 11. config 파일 생성
```
cat << EOF > {sd boot}/config.txt
enable_uart=1
arm_64bit=1
kernel=u-boot.bin
EOF
```

#### 성공
```
U-Boot 2021.01 (Aug 28 2024 - 16:35:34 +0900)

DRAM:  128 MiB
RPI 3 Model B+ (0xa020d3)
MMC:   mmc@7e202000: 0, sdhci@7e300000: 1
Loading Environment from FAT... *** Warning - bad CRC, using default environment

In:    serial
Out:   vidconsole
Err:   vidconsole
Net:   No ethernet found.
starting USB...
Bus usb@7e980000: USB DWC2
scanning bus usb@7e980000 for devices... 4 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  0 
U-Boot>  
```