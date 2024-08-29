```
sudo apt install -y \
    autoconf automake bison bzip2 cmake flex g++ gawk gcc gettext git gperf help2man libncurses5-dev libstdc++6 libtool libtool-bin make patch python3-dev rsync texinfo unzip wget xz-utils \
    device-tree-compiler u-boot-tools
```

```
[submodule "crosstool-ng"]
	path = crosstool-ng
	url = https://github.com/crosstool-ng/crosstool-ng.git
[submodule "Mastering-Embedded-Linux-Programming-Third-Edition"]
	path = Mastering-Embedded-Linux-Programming-Third-Edition
	url = https://github.com/PacktPublishing/Mastering-Embedded-Linux-Programming-Third-Edition.git
[submodule "u-boot"]
	path = u-boot
	url = git://git.denx.de/u-boot.git
[submodule "rpi/linux"]
	path = rpi/linux
	url = https://github.com/raspberrypi/linux.git
[submodule "rpi/firmware"]
	path = rpi/firmware
	url = https://github.com/raspberrypi/firmware.git
```