## 커널 구성과 빌드

#### 커널은 무엇을 하는가
1. 자원을 관리한다.
2. 하드웨어와 인터페이스한다.
3. 사용자 공간 프로그램에게 추상화를 제공하는 API를 제공한다.

```
    사용자 공간 (어플리케이션, C 라이브러리) 
            |
커널 공간 (시스템 호출 처리기, 일반 서비스, 장치 드라이버)
            | (인터럽트)
        하드웨어
```

- 어플리케이션은 커널을 통해 이뤄진다.

#### 커널 선택하기
- 리눅스는 홀수(개발자용), 짝수(릴리즈용)으로 나뉜다.
- 사용 할 때는 lts (long term support)를 사용하면 된다.
```
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
```

#### 커널 구성
##### Kconfig
- Kconfig는 커널의 구성을 담당한다.  
- 다양한 장치에서 돌아가는 만큼 작업에 맞도록 커널을 구성해야 하는 필요가 있다.  
- 구성은 이름이Kconfig인 파일들의 계층 구조로 선언되며 문법은 Documentation/kbuild/kconfig-language.rst에 있다.

Kconfig
```
mainmenu "Linux/$(ARCH) $(KERNELVERSION) Kernel Configuration"

source "scripts/Kconfig.include"
```
- 빌드하기 전에 ARCH를 명시해야하는 이유이다. 설정하지 않으면 로컬 아키텍처로 사용하게 된다.
- Kconfig의 항목들을 사용해 빌드하면 .config 파일이 나온다. (흔히 사용하는 menuconfig와 같음)
- 이후 make ARCH= 를 사용하여 아키텍처 별로 빌드가 가능해진다 

Kbuild는 Kconfig에 통합된 빌드 시스템이다.

#### 커널 식별
```
$ make ARCH=arm kernelversion
5.4.50
$ make ARCH=arm kernelrelease
5.4.50
```
커널의 버전을 변경하고 싶을 경우 CONFIG_LOCALVERSION을 설정 해준다. local version을 설정 해두면 릴리즈에 포함 되어 식별하기 편하다.

#### 모듈  
언제 모듈을 사용하는가?  
- 라이선스의 문제로 인해 비공개 모듈이 있는 경우
- 비필수 드라이버의 로딩을 연기해 부트시간을 줄이기 위해
- 정적링크를 하게 되면 너무 많은 메모리가 소요 될 때

#### 커널 빌드
> 참고 http://www.wearedev.net/298?PHPSESSID=6130f2208f35379a121a8f83cd827e6d
