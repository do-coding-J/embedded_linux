## 부트로더

> 트리, U-Boot, 비글본 블랙
---
#### 부트로더란 무엇이고 무엇을 하는가
- 부트로더는
    - 시스템을 기본 수준으로 초기화 한다.
    - 커널을 로드한다.

- 전원을 키거나 리셋을 행하였을 때 시스템은 아주 최소의 상태에 있다
    - DRAM 제어 시작 안되었음으로 주메모리 접근 불가
    - 인터페이스가 시작 되지 않았음으로 주변기기 접근 불가
    - CPU, Boot Rom 사용 가능

- 과거에는 부트로더를 플래시에 맵핑하였다
- 메모리 최상단에 위치한 부트로더는 코드의 시작을 가리키는 주소로 되어있다.

#### 부트로더 동작 방법
1. 롬코드
    - 모든 것이 초기화 되지 않은 상태에서 코드의 첫 줄을 실행하게 되면 롬코드를 불러온다.
    - DRAM이 초기화 되지 않았기 때문에 SRAM을 사용한다.
    - SRAM이 부트로더를 다 로드하지 못 하는 크기라면 SPL이라는 중간 로더를 호출한다.
2. SPL
    - SPL은 DRAM을 초기화한다.
    - 또한 SPL은 TPL을 DRAM에 옮기는 역할을 한다.
    - 비공개 바이너리로 제공 된다.

3. TPL (Tertiary Program Loader)
    - 부트로더가 실행 되는 단계이다.
    - 필요한 커널 및 인터페이스를 DRAM으로 가져온다.
    - 이후 제어를 커널로 넘긴다.

4. 부트로더에서 커널로 제어를 넘길 때 필요한 정보
    - SoC의 종류를 파악하기 위한 머신 번호
    - 하드웨어의 기본적인 세부사항 (램 크기와 위치, CPU 클럭 등이 포함)
    - 커널 명령 줄
    - 장치 트리 바이너리 위치와 크기
    - 초기 램 디스크의 위치와 크기, 초기 파일 시스템

#### 장치 트리
- 컴퓨터 시스템의 하드웨어 요소를 정의하는 방법
- 정적 데이터 (실행 코드 X)
- 부트로더가 로드해서 커널로 넘기지만 커널 이미지 자체에도 포함 가능
> www.devicetree.org 에 자세한 정보가 있다.

1. 기초
    - 리눅스 커널은 arch/$ARCH/boot/dts에 위치하고 있다.
    - 시작은 / 로 시작한다.
    - 루트 노드는 cpus노드와 memory노드를 담고있다.
```
/dts-v1/;
/{
    model = "TI AM335x BeagleBone";
    compatible = "ti,am33xx";
    #address-cells = <1>;
    #size-cells = <1>;
    cpus {
        #address-cells = <1>;
        #size-cells = <0>;
        cpu@0 {
            compatible = "arm,cortex-a8";
            device_type = "cpu";
            reg = <0>;
        };
    };
    memofy@0x8000000 {
        device_type = "memory";
        reg = <0x8000000 0x2000000>; /*512*/
    };
};
```

2. reg 프로퍼티
    - 메모리와 cpu에는 reg 프로퍼티가 있다. 이는 레지스터 공간 구성 단위의 범위를 나타낸다.
    - 1. 기초 의 예시를 들면 0x8000000에서 0x2000000까지의 공간을 가진 메모리이다.
    - 위 예시는 32bit 에서 이고 64bit일 경우에는 <0x00000000 0x8000000 0x8000000>과 같이 표기 된다.


#### 레이블과 인터럽트
- 트리에는 다양한 계층구조가 존재한다.
    - 인터럽트제어기
    - 클럭 공급원
    - 전압 조정기
    - etc
- 구조들을 연결하는 방법은 노드에 레이블을 추가하고 다른 노드에서 레이블을 참조한다. (phandle)
- 아래는 LCD 제어기와 인터럽트 제어기를 갖고 있는 시스템을 예로 든다.
```
/dts-v1/;
{
    intc: interrupt-controller@48200000{
        compatible = "ti,am33xx-intc";
        interrupt-controller;
        #interrupt-cells = <1>;
        reg = <0x48200000 0x1000>;
    };
    lcdc: lcdc@4830e000{
        compatible= "ti,am33xx-tilcdc";
        reg = <0x4830e000 0x1000>;
        interrupt-parent=<&intc>;
        interrupts = <36>;
        ti,hwmods = "lcdc";
        status = "disabled";
    };
};
```
- 인터럽트 컨트롤러인 intc가 정의 되고, #interrupt-cells를 통해 몇개의 인터럽트가 있는지 파악한다.
- LCD 제어기인 lcdc는 interrupt-parent로 <&intc>를 가진다. interrupt<36>을 가지고 동작한다.

#### 장치트리 컴파일
- 장치트리는 dtc를 통해 컴파일 된다. dts파일을 dtb 파일로 바꿔주는것

#### U-Boot을 활용한 빌드
1. 소스를 받는다
```
$ git clone git://git.denx.de/u-boot.git
$ cd u-boot
$ git checkout v2021.01
```

2. 이전에 빌드한 컴파일러로 빌드한다.
```
{add arm-cortex-a8-linux-gnnueabihf to PATH}
$ make am335x_evm_defconfig
$ make
```

3. 빌드 결과물
    - u-boot : elf 결과물 디버깅 용
    - u-boot.map : 심볼 테이블
    - u-boot.bin : 가공되지 않은 바이너리 장치에서 실행하기 알맞다
    - u-boot.img : u-boot 헤더가 추가된 파이너리 실행 중인 u-boot에 업로드 한다.
    - u-boot.srec : 시리얼 연결을 통해 전송하기 알맞다  

4. SD 카드 포맷
    - 파티션은 두개로 만든다.
        1. FAT32, 64 MiB, boot flag (/boot)
        2. Linux, 1024 MiB (/root)

5. boot 업로드
    - u-boot 이미지를 /boot 에 복사한다
```
$ cp MLO u-boot.img /media/{USER}/boot
```

6. 부팅 후 확인
    - 전원을 인가 한 후에 시리얼 선을 연결한다.
    - /dev/ttyUSB0를 터미널로 열어보면 부팅이 된 것을 확인 할 수 있다.

7. 부트 이미지 생성
    - u-boot-tools를 활용한다 
    - u-boot 디렉토리에서 mkimage를 사용하여 이미지 생성
```
mkimage -A arm -O linux -T kernel -C gzip -a 0x80008000 -e 0x80008000 -n 'Linux' -d zImage uImage
```
뜻:   
    - 아키텍처 : arm  
    - 운영체네 : linux  
    - 이미지 유형 : kernel  
    - 압축 방식 : gzip  
    - 로드 주소 : 0x80008000  
    - 진입 점 : 0x80008000  
    - 이미지 데이터 파일 : zImage  
    - 생성되는 이미지 이름 : uImage  

8. 이미지 인식
방법 1
```
=> mmc rescan
=> fatload mmc 0:1 82000000 uimage
=> iminfo 82000000
=> tftpboot 82000000 uimage
=> nandecc hw
=> nand erase 280000 400000
=> nand write 82000000 280000 400000
=> nand read 82000000 280000 400000
=> bootm 82000000 - 83000000
```
방법 2
```
setenv bootcmd nand read 82000000 400000 200000\;bootm 82000000
```

## [여기까지 한 것을 라즈베리파이로 해보자](./practice.md)
> http://www.wearedev.net/284?PHPSESSID=0be74ec2d642c13c2099d2da4a3ff974

