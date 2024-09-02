## 루트 파일 시스템

### 루트 파일 시스템에는 무엇이 있는가?
- 커널은 부트로더부터 포인터로 전달 된 initramfss나 root= 파라미터를 통해 커널 명령에 지정된 블록 장치를 마운트 함으로써 루트파일시스템을 구할 것이다.  
- 부트로더는 커널을 불러오면 끝이다. 그 이후 시스템을 가동 하는 것은 init이 한다.
- 최소한의 루트파일시스템에는 다음과 같은 항목이 있다.
    - init
    - shell
    - 데몬
    - 공유 라이브러리
    - 구성 파일
    - proc과 sys
    - 커널 모듈

### 레이아웃 FHS
- FHS는 리눅스 시스템의 기본 디렉토리이다.
    - /bin : 모든 사용자에게 필수적인 요소
    - /dev : 장치노드화 기타 특수파일들
    - /etc : 시스템 구성
    - /lib : 필수 공유 라이브러리
    - /proc : 가상파일로 표현되는 프로세스에 대한 정보 
    - /sbin : 시스템 관리자에게 필수적인 프로그램들
    - /sys : 가상파일로 ㅠㅛ시되는 장치와 드라이버에 대한 정보
    - /tmp : 임시파일이나 휘발성 파일을 담아두는 곳
    - /usr : 사용자 프로그램, 라이브러리, 시스템 관리 유틸리티 (/usr/bin, /usr/lib, /usr/sbin)
    - /var : 실행 중 변경 될 수도 있는 파일과 디렉토리

### 루트파일시스템용 프로그램
- init 프로그램
    - 실행되는 첫번째 프로그램 루트파일시스템의 필수 요소이다.
- shell
    - bash, ash, hush와 같은 명령 프롬프트이다.


### BusyBox!
- 리눅스 유틸리티의 필수기능을 수행하도록 처음부터 작성됐다.  
- BusyBox는 모든 도구를 하나의 바이너리로 묶어서 사용한다. applet의 모음이다.
- 예를 들어 cat은 coreutils/cat.c에 구현이 되어있다.
```
$ busybox cat file.txt
```

#### BusyBox를 사용해보자.
```
$ git clone git://git.busybox.net/busybox
$ cd busybox
$ make distclean
$ make defconfig
$ make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu
```