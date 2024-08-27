## 라즈베리파이 u-boot 빌드하기

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
현재는 Paths and misc options -> Maximum log level -> DEBUG 만 설정  
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
빌드 가능한 목록
```
valid arguments are: cortex-a34 cortex-a35 cortex-a53 cortex-a57 cortex-a72 cortex-a73 thunderx thunderxt88p1 thunderxt88 octeontx octeontx81 octeontx83 thunderxt81 thunderxt83 ampere1 ampere1a ampere1b emag xgene1 falkor qdf24xx exynos-m1 phecda thunderx2t99p1 vulcan thunderx2t99 cortex-a55 cortex-a75 cortex-a76 cortex-a76ae cortex-a77 cortex-a78 cortex-a78ae cortex-a78c cortex-a65 cortex-a65ae cortex-x1 cortex-x1c neoverse-n1 ares neoverse-e1 octeontx2 octeontx2t98 octeontx2t96 octeontx2t93 octeontx2f95 octeontx2f95n octeontx2f95mm a64fx tsv110 thunderx3t110 neoverse-v1 zeus neoverse-512tvb saphira cortex-a57.cortex-a53 cortex-a72.cortex-a53 cortex-a73.cortex-a35 cortex-a73.cortex-a53 cortex-a75.cortex-a55 cortex-a76.cortex-a55 cortex-r82 cortex-a510 cortex-a520 cortex-a710 cortex-a715 cortex-a720 cortex-x2 cortex-x3 cortex-x4 neoverse-n2 cobalt-100 neoverse-v2 grace demeter generic generic-armv8-a generic-armv9-a
```