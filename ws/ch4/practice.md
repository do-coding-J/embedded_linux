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
make -j6 Image modules dtbs
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
Loading Environment from FAT... Unable to read "uboot.env" from mmc0:1...       
In:    serial,usbkbd                                                            
Out:   serial,vidconsole                                                        
Err:   serial,vidconsole                                                        
Net:   No ethernet found.                                                       
                                                                                
starting USB...                                                                 
Bus usb@7e980000: USB DWC2                                                      
scanning bus usb@7e980000 for devices... 4 USB Device(s) found                  
       scanning usb for storage devices... 0 Storage Device(s) found            
Hit any key to stop autoboot:  0                                                
Card did not respond to voltage select! : -110                                  
No EFI system partition                                                         
No EFI system partition                                                         
Failed to persist EFI variables                                                 
No EFI system partition                                                         
Failed to persist EFI variables                                                 
No EFI system partition                                                         
Failed to persist EFI variables                                                 
** Booting bootflow '<NULL>' with efi_mgr                                       
Loading Boot0000 'mmc 0' failed                                                 
EFI boot manager: Cannot load any image                                         
Boot failed (err=-14)                                                           
Card did not respond to voltage select! : -110                                  
lan78xx_eth Waiting for PHY auto negotiation to complete......... TIMEOUT !     
phy_startup failed                                                              
lan78xx_eth Waiting for PHY auto negotiation to complete......... TIMEOUT !     
phy_startup failed  
```

### 컴파일러 변경
```
$ wget https://developer.arm.com/-/media/Files/downloads/gnu/13.3.rel1/binrel/arm-gnu-toolchain-13.3.rel1-x86_64-aarch64_be-none-linux-gnu.tar.xz

$ tar xf arm-gnu-toolchain-13.3.rel1-x86_64-aarch64_be-none-linux-gnu.tar.xz
$ mv arm-gnu-toolchain-13.3.rel1-x86_64-aarch64_be-none-linux-gnu gcc-arm-aarch64-none-linux-gnu
$ cp gcc-arm-aarch64-none-linux-gnu ~/x-tools/. -r
$ PATH=$PATH:~/x-tools/gcc-arm-aarch64-none-linux-gnu/bin
$ make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu bcm2711_defconfig
$ make -j 8 ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu
```

### 파일 수정
해당 설정은 u-boot을 사용하는 것이 아닌 라즈베리파이의 부팅 시스템을 사용한 것.
```
$ cp firmware/boot rpi/boot -r
$ rm rpi/boot/kernel*
$ rm rpi/boot/*dtb
$ rm rpi/boot/overlays/*.dtbo
$ cp linux/arch/arm64/boot/Image rpi/boot/kernel8.img
$ cp linux/arch/arm64/boot/*.dtb rpi/boot/.
$ cp linux/arch/arm64/boot/overlays/*.dtbo rpi/boot/overlays/.
```

### 부팅
```
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd034]
[    0.000000] Linux version 6.6.47-v8+ (jj@dev) (aarch64-none-linux-gnu-gcc (Arm GNU Toolchain 13.3.Rel1 (Build arm-13.24)) 13.3.1 20240614, GNU ld (Arm GNU Toolchain 13.3.Rel1 (Build arm-13.24)) 2.42.0.2024064
[    0.000000] KASLR enabled
[    0.000000] random: crng init done
[    0.000000] Machine model: Raspberry Pi 3 Model B Plus Rev 1.3
[    0.000000] efi: UEFI not found.
[    0.000000] Reserved memory: created CMA memory pool at 0x0000000037400000, size 64 MiB
[    0.000000] OF: reserved mem: initialized node linux,cma, compatible id shared-dma-pool
[    0.000000] OF: reserved mem: 0x0000000037400000..0x000000003b3fffff (65536 KiB) map reusable linux,cma
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x0000000000000000-0x000000003b3fffff]
[    0.000000]   DMA32    empty
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000000000-0x000000003b3fffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000000000-0x000000003b3fffff]
[    0.000000] On node 0, zone DMA: 19456 pages in unavailable ranges
[    0.000000] percpu: Embedded 30 pages/cpu s85672 r8192 d29016 u122880
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: kernel page table isolation forced ON by KASLR
[    0.000000] CPU features: detected: Kernel page table isolation (KPTI)
[    0.000000] CPU features: detected: ARM erratum 843419
[    0.000000] CPU features: detected: ARM erratum 845719
[    0.000000] alternatives: applying boot alternatives
[    0.000000] Kernel command line: coherent_pool=1M 8250.nr_uarts=1 snd_bcm2835.enable_headphones=0 bcm2708_fb.fbwidth=656 bcm2708_fb.fbheight=416 bcm2708_fb.fbswap=1 vc_mem.mem_base=0x3ec00000 vc_mem.mem_sizet[    0.000000] Dentry cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
[    0.000000] Inode-cache hash table entries: 65536 (order: 7, 524288 bytes, linear)
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 238896
[    0.000000] mem auto-init: stack:all(zero), heap alloc:off, heap free:off
[    0.000000] Memory: 858644K/970752K available (13440K kernel code, 2212K rwdata, 4268K rodata, 4864K init, 1083K bss, 46572K reserved, 65536K cma-reserved)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] ftrace: allocating 43238 entries in 169 pages
[    0.000000] ftrace: allocated 169 pages with 4 groups
[    0.000000] trace event string verifier disabled
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=256 to nr_cpu_ids=4.
[    0.000000]  Trampoline variant of Tasks RCU enabled.
[    0.000000]  Rude variant of Tasks RCU enabled.
[    0.000000]  Tracing variant of Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] Root IRQ handler: bcm2836_arm_irqchip_handle_irq
[    0.000000] rcu: srcu_init: Setting srcu_struct sizes based on contention.
[    0.000000] arch_timer: cp15 timer(s) running at 19.20MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x46d987e47, max_idle_ns: 440795202767 ns
[    0.000001] sched_clock: 56 bits at 19MHz, resolution 52ns, wraps every 4398046511078ns
[    0.000378] Console: colour dummy device 80x25
[    0.000395] printk: console [tty1] enabled
[    0.001275] Calibrating delay loop (skipped), value calculated using timer frequency.. 38.40 BogoMIPS (lpj=76800)
[    0.001320] pid_max: default: 32768 minimum: 301
[    0.001431] LSM: initializing lsm=capability,integrity
[    0.001683] Mount-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.001734] Mountpoint-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.002717] cgroup: Disabling memory control group subsystem
[    0.004635] RCU Tasks: Setting shift to 2 and lim to 1 rcu_task_cb_adjust=1.
[    0.004782] RCU Tasks Rude: Setting shift to 2 and lim to 1 rcu_task_cb_adjust=1.
[    0.004955] RCU Tasks Trace: Setting shift to 2 and lim to 1 rcu_task_cb_adjust=1.
[    0.005276] rcu: Hierarchical SRCU implementation.
[    0.005302] rcu:     Max phase no-delay instances is 1000.
[    0.007320] EFI services will not be available.
[    0.007681] smp: Bringing up secondary CPUs ...
[    0.008497] Detected VIPT I-cache on CPU1
[    0.008640] CPU1: Booted secondary processor 0x0000000001 [0x410fd034]
[    0.009449] Detected VIPT I-cache on CPU2
[    0.009552] CPU2: Booted secondary processor 0x0000000002 [0x410fd034]
[    0.010328] Detected VIPT I-cache on CPU3
[    0.010425] CPU3: Booted secondary processor 0x0000000003 [0x410fd034]
[    0.010531] smp: Brought up 1 node, 4 CPUs
[    0.010658] SMP: Total of 4 processors activated.
[    0.010681] CPU features: detected: 32-bit EL0 Support
[    0.010702] CPU features: detected: 32-bit EL1 Support
[    0.010725] CPU features: detected: CRC32 instructions
[    0.010857] CPU: All CPU(s) started at EL2
[    0.010890] alternatives: applying system-wide alternatives
[    0.013749] devtmpfs: initialized
[    0.026988] Enabled cp15_barrier support
[    0.027048] Enabled setend support
[    0.027297] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.027349] futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
[    0.030319] pinctrl core: initialized pinctrl subsystem
[    0.031066] DMI not present or invalid.
[    0.031700] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.038824] DMA: preallocated 1024 KiB GFP_KERNEL pool for atomic allocations
[    0.039238] DMA: preallocated 1024 KiB GFP_KERNEL|GFP_DMA pool for atomic allocations
[    0.039864] DMA: preallocated 1024 KiB GFP_KERNEL|GFP_DMA32 pool for atomic allocations
[    0.039983] audit: initializing netlink subsys (disabled)
[    0.040293] audit: type=2000 audit(0.040:1): state=initialized audit_enabled=0 res=1
[    0.041087] thermal_sys: Registered thermal governor 'step_wise'
[    0.041150] cpuidle: using governor menu
[    0.041436] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.041598] ASID allocator initialised with 32768 entries
[    0.042654] Serial: AMBA PL011 UART driver
[    0.047970] bcm2835-mbox 3f00b880.mailbox: mailbox enabled
[    0.056257] raspberrypi-firmware soc:firmware: Attached to firmware from 2024-08-30T19:19:11, variant start
[    0.060277] raspberrypi-firmware soc:firmware: Firmware hash is 2808975b80149bbfe86844655fe45c7de66fc078
[    0.068437] Modules: 2G module region forced by RANDOMIZE_MODULE_REGION_FULL
[    0.068474] Modules: 0 pages in range for non-PLT usage
[    0.068482] Modules: 517776 pages in range for PLT usage
[    0.074149] bcm2835-dma 3f007000.dma-controller: DMA legacy API manager, dmachans=0x1
[    0.076442] iommu: Default domain type: Translated
[    0.076478] iommu: DMA domain TLB invalidation policy: strict mode
[    0.076991] SCSI subsystem initialized
[    0.077287] usbcore: registered new interface driver usbfs
[    0.077358] usbcore: registered new interface driver hub
[    0.077438] usbcore: registered new device driver usb
[    0.077918] pps_core: LinuxPPS API ver. 1 registered
[    0.077945] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.077998] PTP clock support registered
[    0.079467] vgaarb: loaded
[    0.080008] clocksource: Switched to clocksource arch_sys_counter
[    0.080606] VFS: Disk quotas dquot_6.6.0
[    0.080672] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.080836] FS-Cache: Loaded
[    0.084101] CacheFiles: Loaded
[    0.094462] NET: Registered PF_INET protocol family
[    0.094745] IP idents hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.096865] tcp_listen_portaddr_hash hash table entries: 512 (order: 1, 8192 bytes, linear)
[    0.096929] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.096971] TCP established hash table entries: 8192 (order: 4, 65536 bytes, linear)
[    0.097107] TCP bind hash table entries: 8192 (order: 6, 262144 bytes, linear)
[    0.097483] TCP: Hash tables configured (established 8192 bind 8192)
[    0.097832] MPTCP token hash table entries: 1024 (order: 2, 24576 bytes, linear)
[    0.097936] UDP hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.098007] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.098251] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.098934] RPC: Registered named UNIX socket transport module.
[    0.098965] RPC: Registered udp transport module.
[    0.098986] RPC: Registered tcp transport module.
[    0.099006] RPC: Registered tcp-with-tls transport module.
[    0.099027] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.099062] PCI: CLS 0 bytes, default 64
[    0.104155] kvm [1]: IPA Size Limit: 40 bits
[    0.105949] kvm [1]: Hyp mode initialized successfully
[    1.722259] Initialise system trusted keyrings
[    1.722622] workingset: timestamp_bits=46 max_order=18 bucket_order=0
[    1.722718] zbud: loaded
[    1.723585] NFS: Registering the id_resolver key type
[    1.723631] Key type id_resolver registered
[    1.723654] Key type id_legacy registered
[    1.723697] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    1.723726] nfs4flexfilelayout_init: NFSv4 Flexfile Layout Driver Registering...
[    1.724572] Key type asymmetric registered
[    1.724600] Asymmetric key parser 'x509' registered
[    1.724682] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    1.724919] io scheduler mq-deadline registered
[    1.724948] io scheduler kyber registered
[    1.725003] io scheduler bfq registered
[    1.727358] pinctrl-bcm2835 3f200000.gpio: GPIO_OUT persistence: yes
[    1.729704] bcm2708_fb soc:fb: FB found 1 display(s)
[    1.738531] Console: switching to colour frame buffer device 82x26
[    1.741393] bcm2708_fb soc:fb: Registered framebuffer for display 0, size 656x416
[    1.747401] Serial: 8250/16550 driver, 1 ports, IRQ sharing enabled
[    1.751050] bcm2835-rng 3f104000.rng: hwrng registered
[    1.752890] vc-mem: phys_addr:0x00000000 mem_base=0x3ec00000 mem_size:0x40000000(1024 MiB)
[    1.770714] brd: module loaded
[    1.780694] loop: module loaded
[    1.782817] Loading iSCSI transport class v2.0-870.
[    1.789416] usbcore: registered new device driver r8152-cfgselector
[    1.790930] usbcore: registered new interface driver r8152
[    1.792399] usbcore: registered new interface driver lan78xx
[    1.793815] usbcore: registered new interface driver smsc95xx
[    1.795357] dwc_otg: version 3.00a 10-AUG-2012 (platform bus)
[    2.525805] Core Release: 2.80a
[    2.527079] Setting default values for core params
[    2.528408] Finished setting default values for core params
[    2.730085] Using Buffer DMA mode
[    2.731399] Periodic Transfer Interrupt Enhancement - disabled
[    2.732749] Multiprocessor Interrupt Enhancement - disabled
[    2.734077] OTG VER PARAM: 0, OTG VER FLAG: 0
[    2.735374] Dedicated Tx FIFOs mode
[    2.737085] 
[    2.737093] WARN::dwc_otg_hcd_init:1070: FIQ DMA bounce buffers: virt = ffffffc080489000 dma = 0x00000000f7810000 len=9024
[    2.740743] FIQ FSM acceleration enabled for :
[    2.740743] Non-periodic Split Transactions
[    2.740743] Periodic Split Transactions
[    2.740743] High-Speed Isochronous Endpoints
[    2.740743] Interrupt/Control Split Transaction hack enabled
[    2.746631] 
[    2.746636] WARN::hcd_init_fiq:496: MPHI regs_base at ffffffc080065000
[    2.749101] dwc_otg 3f980000.usb: DWC OTG Controller
[    2.750432] dwc_otg 3f980000.usb: new USB bus registered, assigned bus number 1
[    2.751801] dwc_otg 3f980000.usb: irq 74, io mem 0x00000000
[    2.753164] Init: Port Power? op_state=1
[    2.754454] Init: Power Port (0)
[    2.755955] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 6.06
[    2.758587] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    2.759978] usb usb1: Product: DWC OTG Controller
[    2.761348] usb usb1: Manufacturer: Linux 6.6.47-v8+ dwc_otg_hcd
[    2.762729] usb usb1: SerialNumber: 3f980000.usb
[    2.764846] hub 1-0:1.0: USB hub found
[    2.766239] hub 1-0:1.0: 1 port detected
[    2.769021] usbcore: registered new interface driver uas
[    2.770403] usbcore: registered new interface driver usb-storage
[    2.772386] mousedev: PS/2 mouse device common for all mice
[    2.776620] sdhci: Secure Digital Host Controller Interface driver
[    2.777987] sdhci: Copyright(c) Pierre Ossman
[    2.779500] sdhci-pltfm: SDHCI platform and OF driver helper
[    2.783094] ledtrig-cpu: registered to indicate activity on CPUs
[    2.784940] hid: raw HID events driver (C) Jiri Kosina
[    2.786396] usbcore: registered new interface driver usbhid
[    2.787731] usbhid: USB HID core driver
[    2.794131] hw perfevents: enabled with armv8_cortex_a53 PMU driver, 7 counters available
[    2.798190] NET: Registered PF_PACKET protocol family
[    2.799740] Key type dns_resolver registered
[    2.838139] registered taskstats version 1
[    2.839734] Loading compiled-in X.509 certificates
[    2.856286] Key type .fscrypt registered
[    2.857602] Key type fscrypt-provisioning registered
[    2.868110] uart-pl011 3f201000.serial: cts_event_workaround enabled
[    2.869948] 3f201000.serial: ttyAMA1 at MMIO 0x3f201000 (irq = 99, base_baud = 0) is a PL011 rev2
[    2.872984] serial serial0: tty port ttyAMA1 registered
[    2.876168] Indeed it is in host mode hprt0 = 00021501
[    2.880111] bcm2835-aux-uart 3f215040.serial: there is not valid maps for state default
[    2.884151] printk: console [ttyS0] disabled
[    2.886209] 3f215040.serial: ttyS0 at MMIO 0x3f215040 (irq = 71, base_baud = 50000000) is a 16550
[    2.889248] printk: console [ttyS0] enabled
[    3.068022] usb 1-1: new high-speed USB device number 2 using dwc_otg
[    3.069555] bcm2835-wdt bcm2835-wdt: Broadcom BCM2835 watchdog timer
[    3.079330] Indeed it is in host mode hprt0 = 00001101
[    3.087351] bcm2835-power bcm2835-power: Broadcom BCM2835 power domains driver
[    3.325851] usb 1-1: New USB device found, idVendor=0424, idProduct=2514, bcdDevice= b.b3
[    3.334073] mmc-bcm2835 3f300000.mmcnr: mmc_debug:0 mmc_debug2:0
[    3.335513] usb 1-1: New USB device strings: Mfr=0, Product=0, SerialNumber=0
[    3.337175] mmc-bcm2835 3f300000.mmcnr: DMA channel allocated
[    3.344803] hub 1-1:1.0: USB hub found
[    3.377406] sdhost: log_buf @ 000000007770a1de (c2bce000)
[    3.384706] hub 1-1:1.0: 4 ports detected
[    4.288221] mmc0: sdhost-bcm2835 loaded - DMA enabled (>1)
[    4.298051] mmc1: new high speed SDIO card at address 0001
[    4.299066] of_cfs_init
[    4.309467] of_cfs_init: OK
[    4.314105] clk: Disabling unused clocks
[    4.320313] Waiting for root device /dev/mmcblk0p2...
[    4.359640] mmc0: host does not support reading read-only switch, assuming write-enable
[    4.376430] mmc0: Host Software Queue enabled
[    4.382350] mmc0: new high speed SDHC card at address aaaa
[    4.390510] mmcblk0: mmc0:aaaa SB16G 14.8 GiB
[    4.402315]  mmcblk0: p1 p2
[    4.407236] mmcblk0: mmc0:aaaa SB16G 14.8 GiB (quirks 0x00004000)
[    4.429884] EXT4-fs (mmcblk0p2): mounted filesystem bde09605-d318-4a7d-90f6-16d73c372533 ro with ordered data mode. Quota mode: none.
[    4.445022] VFS: Mounted root (ext4 filesystem) readonly on device 179:2.
[    4.456297] devtmpfs: error mounting -2
[    4.471968] Freeing unused kernel memory: 4864K
[    4.478484] Run /sbin/init as init process
[    4.484381] Run /etc/init as init process
[    4.489972] Run /bin/init as init process
[    4.495489] Run /bin/sh as init process
[    4.501062] Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
[    4.518153] CPU: 2 PID: 1 Comm: swapper/0 Not tainted 6.6.47-v8+ #3
[    4.525941] Hardware name: Raspberry Pi 3 Model B Plus Rev 1.3 (DT)
[    4.533739] Call trace:
[    4.537593]  dump_backtrace+0x9c/0x100
[    4.542749]  show_stack+0x20/0x38
[    4.547428]  dump_stack_lvl+0x48/0x60
[    4.552446]  dump_stack+0x18/0x28
[    4.557077]  panic+0x330/0x398
[    4.560032] usb 1-1.1: new high-speed USB device number 3 using dwc_otg
[    4.569414]  kernel_init+0x1ac/0x1f8
[    4.574292]  ret_from_fork+0x10/0x20
[    4.579119] SMP: stopping secondary CPUs
[    4.584245] Kernel Offset: 0x2406c00000 from 0xffffffc080000000
[    4.591414] PHYS_OFFSET: 0x0
[    4.595472] CPU features: 0x0,0000000d,00020000,0000421b
[    4.602039] Memory Limit: none
[    4.606315] ---[ end Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance. ]---

```
- Linux version 6.6.47-v8+ (jj@dev) 을 보면 내가 빌드한 리눅스 임을 알 수 있다.
- end kernel panic으로 끝나는 이유는 파일 시스템이 존재하지 않기 때문이다.
