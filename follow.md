## crosstool-NG를 사용해 툴체인 빌드하기

#### 1. 설치
```
$ git clone https://github.com/crosstool-ng/crosstool-ng.git
$ cd crosstool-ng
$ ./bootstrap
$ ./configure --prefix=${PWD}
$ make
$ make install
```

#### 비글본 블랙용 툴체인 빌드
- 아래 커맨드 사용하여 사용 가능한 c-lib 찾기
```
$ bin/ct-ng list-samples
```
- 비글본은 arm-cortex a8코어와 vfpv3라는 부동소수점장치가 있는 TI AM335x Soc를 사용한다.
- 따라서 이번에는 arm-cortex_a8-linux-gunueabi를 사용한다.
```
$ bin/ct-ng show-arm-cortex_a8-linux-gnueabi
```
- show-를 사용하면 디폴트 구성을 볼 수 있다.
```
$ bin/ct-ng show-arm-cortex_a8-linux-gnueabi
[L...]   arm-cortex_a8-linux-gnueabi
    Languages       : C,C++
    OS              : linux-6.10
    Binutils        : binutils-2.42
    Compiler        : gcc-14.2.0
    Linkers         :
    C library       : glibc-2.40
    Debug tools     : duma-2_5_21 gdb-15.1 ltrace-0.7.3 strace-6.10
    Companion libs  : expat-2.5.0 gettext-0.22.5 gmp-6.2.1 isl-0.26 libelf-0.8.13 libiconv-1.16 mpc-1.3.1 mpfr-4.2.1 ncurses-6.4 zlib-1.3 zstd-1.5.6
    Companion tools :

```
- 선택은 show- 접두사를 뺀 커맨드로 선택한다.
```
$ bin/ct-ng arm-cortex_a8-linux-gnueabi
```
- 선택 후 결과는 아래와 같이 나온다.
```
$ bin/ct-ng arm-cortex_a8-linux-gnueabi
  CONF  arm-cortex_a8-linux-gnueabi
#
# configuration written to .config
#

***********************************************************

Initially reported by: Yann E. MORIN
URL: http://ymorin.is-a-geek.org/

***********************************************************

Now configured for "arm-cortex_a8-linux-gnueabi"

```
- 이후부터는 menuconfig를 사용하여 구성한다.
```
$ bin/ct-ng menuconfig
```

- 대충 구성이후 빌드를 한다.
```
$ bin/ct-ng build
```
> LD_LIBRARY_PATH를 unset 해야 할 경우도 있음, ROS2 사용시 LD_LIBRARY_PATH가 정해져 있는 경우가 있다.

결과물은 ~/x-tools/arm-cortex_a8-linux-gnueabihf에서 확인 가능하다.

#### QEMU용 툴체인 빌드

- 우선 디렉토리를 clean 해주고 arm-unknown-linux-gnueabi로 타겟을 바꿔준 후에 빌드.
```
$ bin/ct-ng distclean
$ bin/ct-ng arm-unknown-linux-gnueabi
  CONF  arm-unknown-linux-gnueabi
#
# configuration written to .config
#

***********************************************************

Initially reported by: Alexander BIGGA
URL: http://sourceware.org/ml/crossgcc/2008-06/msg00031.html

***********************************************************

Now configured for "arm-unknown-linux-gnueabi"

$ bin/ct-ng build
```
- 빌드 된 툴체인은 ~/x-tools/arm-unknown-linux-gnueabi 에서 확인 가능하다


