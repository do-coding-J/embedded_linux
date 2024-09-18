## buildroot과 yocto

### buildroot qemu
```
$ git clone https://github.com/buildroot/buildroot.git
$ cd buildroot
$ make qemu_arm_versatile_defconfig
$ make
$ qemu-system-arm -M versatilepb -m 256 -kernel output/images/zImage -dtb output/images/versatile-pb.dtb -drive file=output/images/rootfs.ext2,if=scsi,format=raw -append "root=/dev/sda console=ttyAMA0, 115200" -serial stdio -net nic,model=rtl8139 -net user 
```

#### buildroot rpi
```
$ make clean
$ make raspberrypi3_64_defconfig
$ make -j$(nproc)
```

balena-etcher 설치 후 sdcard.img 선택하여 flash  
uart로 동작 확인 가능  

### Yocto
```
$ git clone -b dunfell https://github.com/yoctoproject/poky.git
$ source poky/oe-init-build-env ws/build-qemuarm
$ bitbake core-image-minimal
```
```
$ runqemu qemuarm
```

### Yocto meta-nova
#### create layer
```
$ source poky/oe-init-build-env ws/build-nova
$ bitbake-layers create-layer nova
$ mv nova ../meta-nova
```

#### add layer
```
$ bitbake-layers add-layer ../meta-nova
$ bitbake-layers show-layers
NOTE: Starting bitbake server...
layer                 path                                      priority
==========================================================================
meta                  /home/jj/Desktop/embedded_linux/poky/meta  5
meta-poky             /home/jj/Desktop/embedded_linux/poky/meta-poky  5
meta-yocto-bsp        /home/jj/Desktop/embedded_linux/poky/meta-yocto-bsp  5
meta-nova             /home/jj/Desktop/embedded_linux/ws/meta-nova  6
```

#### build
> 책에서는 BeagleBone Black으로 빌드하지만 qemu로 빌드해보자   

**conf/local.conf**
```
MACHINE ?= "qemuarm64" # 주석 해제
```

#### 레시피 수정
- recipes :  *.bb로 끝나는 파일들 소스코드의 사본과 의존성 파일을 얻는 방법, 빌드와 설치법을 포함한다.
- append : *.bbappend로 끝내는 파일들 레시피 세부 항목을 덮어씌우는데 사용된다.
- include : *.inc 여러 레시피를 위한 공통의 정보를 갖고 있다. include나 require키워드를 사용한다.
- classes : *.bbclass로 끝난다. 공통 빌드 정보를 포함하는데 사용된다. inherite 키워드를 통해 상속 및 확장 된다.
- configuration : *.conf로 사용된다. 빌드 프로세스의 통제를 담당한다.
> 확인 하는법
```
$ bitbake -c listtasks core-image-minimal
Loading cache: 100% |############################################| Time: 0:00:00
Loaded 1333 entries from dependency cache.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.46.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "universal"
TARGET_SYS           = "aarch64-poky-linux"
MACHINE              = "qemuarm64"
DISTRO               = "poky"
DISTRO_VERSION       = "3.1.33"
TUNE_FEATURES        = "aarch64 armv8a crc"
TARGET_FPU           = ""
meta                 
meta-poky            
meta-yocto-bsp       = "dunfell:63d05fc061006bf1a88630d6d91cdc76ea33fbf2"
meta-nova            = "main:0a827fb25b86e19524630959c7b6b1f85366ff08"

Initialising tasks: 100% |#######################################| Time: 0:00:00
Sstate summary: Wanted 0 Found 0 Missed 0 Current 0 (0% match, 0% complete)
NOTE: No setscene tasks
NOTE: Executing Tasks
do_build                              Default task for a recipe - depends on all other normal tasks required to 'build' a recipe
do_checkuri                           Validates the SRC_URI value
do_clean                              Removes all output files for a target
do_cleanall                           Removes all output files, shared state cache, and downloaded source files for a target
do_cleansstate                        Removes all output files and shared state cache for a target
do_compile                            Compiles the source in the compilation directory
do_configure                          Configures the source by enabling and disabling any build-time and configuration options for the software being built
do_deploy_source_date_epoch           
do_deploy_source_date_epoch_setscene   (setscene version)
do_devpyshell                         Starts an interactive Python shell for development/debugging
do_devshell                           Starts a shell with the environment set up for development/debugging
do_fetch                              Fetches the source code
do_flush_pseudodb                     
do_image                              
do_image_complete                     
do_image_complete_setscene             (setscene version)
do_image_ext4                         
do_image_qa                           
do_image_qa_setscene                   (setscene version)
do_image_tar                          
do_install                            Copies files from the compilation directory to a holding area
do_listtasks                          Lists all defined tasks for a target
do_package                            Analyzes the content of the holding area and splits it into subsets based on available packages and files
do_package_qa_setscene                Runs QA checks on packaged files (setscene version)
do_package_setscene                   Analyzes the content of the holding area and splits it into subsets based on available packages and files (setscene version)
do_package_write_rpm_setscene         Creates the actual RPM packages and places them in the Package Feed area (setscene version)
do_packagedata                        Creates package metadata used by the build system to generate the final packages
do_packagedata_setscene               Creates package metadata used by the build system to generate the final packages (setscene version)
do_patch                              Locates patch files and applies them to the source code
do_populate_lic_deploy                
do_populate_lic_setscene              Writes license information for the recipe that is collected later when the image is constructed (setscene version)
do_populate_sdk                       Creates the file and directory structure for an installable SDK
do_populate_sdk_ext                   
do_populate_sysroot_setscene          Copies a subset of files installed by do_install into the sysroot in order to make them available to other recipes (setscene version)
do_prepare_recipe_sysroot             
do_rootfs                             Creates the root filesystem (file and directory structure) for an image
do_rootfs_wicenv                      
do_sdk_depends                        
do_unpack                             Unpacks the source code into a working directory
do_write_qemuboot_conf                
NOTE: Tasks Summary: Attempted 1 tasks of which 0 didn't need to be rerun and all succeeded.

```

#### helloworld 레시피를 만들어보자
1. 디렉토리 및 파일 생성
```
$ tree recipes-local/
recipes-local/
└── helloworld
    ├── files
    │   └── helloworld.c
    └── helloworld_1.0.bb
```

2. helloworld.c 작성
```
#include <stdio.h>

int main(int argc, char* argv[])
{
    printf("hello world\n");
    return 0;
}
```

3. helloworld_1.0.bb 작성  
    - 라이선스 파일은 poky/meta/files/common-licenses에 있다.
    - md5sum {file}을 통해 체크섬 흭득 가능
    - 소스 추가해주고 컴파일과 설치 방법 설명
```
DESCRIPTION = "prints Hello World"
PRIORITY = "optional"
SECTION = "examples"

LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/GPL-2.0;md5=801f80980d171dd6425610833a22dbe6"

SRC_URI = "file://helloworld.c"

S = "${WORKDIR}"

do_compile(){
    ${CC} ${CFLAGS} ${LDFLAGS} helloworld.c -o helloworld
}

do_install(){
    install -d ${D}${bindir}
    install -m 0755 helloworld ${D}${bindir}
}

```

4. 확인
```
$ bitbake core-image-minimal
$ runqemu qemuarm64
---
$ root
$ helloworld
hello world
```
#### 이미지 작성
1. meta-nova/recipes 아래에 images 폴더 생성
2. images 안에 nova-image.bb 생성
```
require recipes-core/images/core-image-minimal.bb
IMAGE_INSTALL += " helloworld strace"
```
3. 빌드
```
$ bitbake nova-image
```
