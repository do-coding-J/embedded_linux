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

#### 툴체인 구조
- xtools의 경로를 추가해준다.
```
$ PATH=~/x-tools/arm-cortex_a8-linux-gnueabihf/bin/:$PATH 
```
- 간단한 [hello world](./ws/helloworld.c) 파일 만들고 컴파일
```
$ arm-cortex_a8-linux-gnueabihf-gcc helloworld.c -o helloworld
```
- 여기서 생긴 실행파일은 실행 못한다. 비글본 컴파일러로 컴파일 했기 때문에
- 확인하는 방법은 Machine을 통해 알 수 있다.
```
$ arm-cortex_a8-linux-gnueabihf-readelf -a helloworld | grep Machine
  Machine:                           ARM
```

#### 정적 링크와 동적 링크
- 정적 링크를 해보자 정적 링크는 -static을 사용해서 가능하다. 정적 링크를 하게 되면 용량이 엄청 크게 늘어난다.
```
$ arm-cortex_a8-linux-gnueabihf-gcc helloworld.c -static -o helloworld_static
$ ls -l
total 2484
-rwxrwxr-x 1 rb rb   10780 Aug 26 11:38 helloworld
-rw-rw-r-- 1 rb rb     119 Aug 26 11:37 helloworld.c
-rwxrwxr-x 1 rb rb 2525028 Aug 26 13:18 helloworld_static
```
- 정적 링크는 .a 확장자를 사용한다. 또한 rc 옵션을 사용하여 아카이브화 한다.
```
$ touch test1.c
$ touch test2.c
$ arm-cortex_a8-linux-gnueabihf-gcc -c test1.c 
$ arm-cortex_a8-linux-gnueabihf-gcc -c test2.c 
$ arm-cortex_a8-linux-gnueabihf-ar rc libtest.a -o test1.o test2.o
$ ls -l
total 2496
-rwxrwxr-x 1 rb rb   10780 Aug 26 11:38 helloworld
-rw-rw-r-- 1 rb rb     119 Aug 26 11:37 helloworld.c
-rwxrwxr-x 1 rb rb 2525028 Aug 26 13:18 helloworld_static
-rw-rw-r-- 1 rb rb    1752 Aug 26 13:24 libtest.a
-rw-rw-r-- 1 rb rb       0 Aug 26 13:21 test1.c
-rw-rw-r-- 1 rb rb     780 Aug 26 13:22 test1.o
-rw-rw-r-- 1 rb rb       0 Aug 26 13:21 test2.c
-rw-rw-r-- 1 rb rb     780 Aug 26 13:22 test2.o
```
- -l (lib) 옵션을 사용해 링크가 가능하다
```
$ arm-cortex_a8-linux-gnueabihf-gcc helloworld.c -ltest -o helloworld_archive
```

- 공유 라이브러리를 만들 때는 -fPIC 옵션을 추가하여 컴파일 하고 -shared 옵션을 사용하여 링크 해야 한다. 이때 라이브러리의 확장자는 .so다.
```
$ arm-cortex_a8-linux-gnueabihf-gcc -fPIC -c test1.c
$ arm-cortex_a8-linux-gnueabihf-gcc -fPIC -c test2.c
$ arm-cortex_a8-linux-gnueabihf-gcc -shared -o libtest.so test1.o test2.o
```

- 빌드는 동일하게 사용한다.
> 빌드가 안될 경우 LD_LIBRARY_PATH를 확인하라

#### 버전
- 동적 라이브러리는 코드를 공유한다. 그러다보니 A프로그램에서 8버전을 사용하지만 B프로그램에서 9버전을 사용한다면 어떻게 구분할까?
- \${lib_name}.so.${version} 형식으로 관리한다.
- 프로그램이 실행 될 때 그 버전의 링크를 찾아 코드를 실행한다.

#### 기타 크로스 컴파일 툴...이 있다!
- crosstool-ng
- make
- autotool
- cmake
- yocto
- etc.....