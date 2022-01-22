# HI3518ev100+ov9712 Camera Module

**16M** Flash - 0x0 0x1000000

![83466850dff7a9c4c70d32bc5a68cce4.png](/_resources/83466850dff7a9c4c70d32bc5a68cce4.png)

## Flash over flasher without solder.

Backup original - hard method. I am using esptool and esp8266 with solder-out flash.

```
python ./esptool.py --port /dev/ttyUSB0  read_flash 0x00000 0x1000000 backup.img
```

![0ee86f277851b558e78ee58ae903b1a0.png](/_resources/0ee86f277851b558e78ee58ae903b1a0.png)

## TFTP

- windows: tftpd - https://pjo2.github.io/tftpd64/
- linux: tftpd-hpa

## Pins

- IR-cut (ICR/IRC): **GPIO0\_1/GPIO0\_0**
- Day/Night: **SAR\_ADC\_CH0**
- Alarm : ?

## UART

![43c308c827e87dd513387b86b219f558.png](/_resources/43c308c827e87dd513387b86b219f558.png)

TFTP server: 192.168.100.249 Backup original - soft method.

```
setenv ipaddr 192.168.100.240
setenv serverip 192.168.100.249
sf probe 0
mw.b 0x82000000 ff 1000000
sf read 0x82000000 0x0 0x1000000
tftp 0x82000000 backup.img 0x1000000

0x00-0x80000 : "boot"
mw.b 0x82000000 ff 1000000
sf read 0x82000000 0x0 0x80000
tftp 0x82000000 boot.img 0x80000

0x80000-0x300000 : "kernel"
mw.b 0x82000000 ff 1000000
sf read 0x82000000 0x80000 0x280000
tftp 0x82000000 kernel.img 0x280000

0x300000-0x1000000 : "rootfs"
mw.b 0x82000000 ff 1000000
sf read 0x82000000 0x300000 0xd00000
tftp 0x82000000 rootfs.img 0xd00000
```

## U-boot

use original u-boot. Others: not works any Ethernet

**MII** \[Media Independent Interface\] standard for MAC level and PHY chip.

That board has:

```#
# mii device 
MII devices: ‘0:0’ ‘0:1’ Current device: ‘0:0’`
```

## Login first and get information.

Turn ON and press Enter to go to the U-boot console.

Change **root** and **admin** password.

```
hisilicon # setenv bootargs mem=43M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=jffs2 mtdparts=hi_sfc:512K(boot),2560K(kernel),13M(rootfs) init=/bin/sh

hisilicon # sf probe 0
16384 KiB hi_sfc at 0:0 is now current device
hisilicon # sf read 0x82000000 0x80000 0x280000

hisilicon # bootm 0x82000000

## Booting kernel from Legacy Image at 82000000 ...
Starting kernel ...

# id
uid=0(root) gid=0(root)
```

root: root and admin: admin.

```
# echo "root:\$1\$tiaLlxGM\$dYNMJJQRKN2buGE0u/R88/:16199:0:99999:7:::" > /etc/shadow
# echo "admin:\$1\$tiaLlxGM\$uIJ4ahSynKJuzEzWdw7sp/:16199:0:99999:7:::" >> /etc/shadow
# sync
# halt
```

Restart and login with a new password.

## Get MAC

```
# ip a
or
# ifconfig

save mac
```

## Get sensor mask:

`# cat /proc/umap/vi`

ComMsk0: **ffc00000**, ComMsk1: **0**

```
# cat /proc/umap/vi
[VIU] Version: [Hi3518_MPP_V1.0.9.0 ], Build Time: [Aug 27 2014, 22:11:58]

-----MODULE PARAM--------------------------------------------------------------
detect_err_frame drop_err_frame stop_int_level  max_cas_gap
               0              0              0        28000

-----VI DEV ATTR---------------------------------------------------------------
 Dev   IntfM  WkM  ComMsk0  ComMsk1 ScanM AD0 AD1 AD2 AD3   Seq   DPath DType DRev
   0      DC 1Mux ffc00000        0     P  -1  -1  -1  -1  YUYV     ISP   RGB    N

-----VI HIGH DEV ATTR---------------------------------------------------------------
 Dev  InputM  WkM  ComMsk0  ComMsk1 ScanM AD0 AD1 AD2 AD3   Seq CombM CompM ClkM  Fix FldP   DPath DType DRev

-----VI PHYCHN ATTR------------------------------------------------------------
 PhyChn CapX CapY  CapW  CapH  DstW  DstH CapSel Mirror Flip IntEn PixFom SrcRat DstRat
      0    0    0  1280   720  1280   720   both      N    N     Y  sp420     25     25

-----VI PHYCHN STATUS 1----------------------------------------------------------
 PhyChn  Dev      IntCnt  VbFail  LosInt  TopLos  BotLos BufCnt  IntT  SendT  Field  Stride
      0    0        2354       0       2       0       2      2   499    350    frm    1280

-----VI PHYCHN STATUS 2---------------------------------------------------------
 PhyChn MaxIntT IntGapT MaxGapT OverCnt LIntCnt  ThrCnt AutoDis CasAutD  TmgErr      ccErrN    IntRat
      0     558   40058   40337    2323       0       1       0       0       0           2        24

-----VI OTHER ATTR------------------------------------------------------------
    LDC   Mode  Ratio  COffX  COffY Enable
     --    All      0      0      0      0

  Flash   Mode StartTime  DuraTime  InterVal CapIdx Enable  FlashedNum
     --   Once         0         0         0      0      0           0

    CSC   Type HueVal  ContrVal   LumaVal  StatuVal
     --    709     50        50        50        50

-----VI EXTCHN ATTR------------------------------------------------------------
 ExtChn BindChn  DstW  DstH PixFom SrcRat DstRat

-----VI CHN STATUS-------------------------------------------------------------
 ViChn   bEnUsrP   FrmTime   FrmRate     SendCnt      SwLost    Rotate
     0         N     40055        25        2352           0      NONE

```

**Reginfo**

```
./ipctool 

```
[reginfo.pdf](/_resources/reginfo.pdf)

** hisi_gpio_scanner**
```
Original
================ Hisilicon GPIO Scaner (2021) OpenIPC.org collective =================
Chip_Id: 0x35180100, CPU: Hi3518Ev100, Hardware: ARM926 @ 440 MHz
Gr: 0, Addr:0x201403FC, Data:0xF0 = 0b11110000, Addr:0x20140400, Dir:0xBF = 0b10111111
Gr: 1, Addr:0x201503FC, Data:0x00 = 0b00000000, Addr:0x20150400, Dir:0x00 = 0b00000000
Gr: 2, Addr:0x201603FC, Data:0x04 = 0b00000100, Addr:0x20160400, Dir:0x04 = 0b00000100
Gr: 3, Addr:0x201703FC, Data:0x00 = 0b00000000, Addr:0x20170400, Dir:0x03 = 0b00000011
Gr: 4, Addr:0x201803FC, Data:0x00 = 0b00000000, Addr:0x20180400, Dir:0x00 = 0b00000000
Gr: 5, Addr:0x201903FC, Data:0x02 = 0b00000010, Addr:0x20190400, Dir:0x00 = 0b00000000
Gr: 6, Addr:0x201A03FC, Data:0x00 = 0b00000000, Addr:0x201A0400, Dir:0x00 = 0b00000000
Gr: 7, Addr:0x201B03FC, Data:0x60 = 0b01100000, Addr:0x201B0400, Dir:0x00 = 0b00000000
Gr: 8, Addr:0x201C03FC, Data:0x03 = 0b00000011, Addr:0x201C0400, Dir:0x00 = 0b00000000
Gr: 9, Addr:0x201D03FC, Data:0x00 = 0b00000000, Addr:0x201D0400, Dir:0x00 = 0b00000000
Gr:10, Addr:0x201E03FC, Data:0x00 = 0b00000000, Addr:0x201E0400, Dir:0x00 = 0b00000000
Gr:11, Addr:0x201F03FC, Data:0x00 = 0b00000000, Addr:0x201F0400, Dir:0x00 = 0b00000000
--------------------------------------------------------------------------------------
OpenIPC
================ Hisilicon GPIO Scaner (2021) OpenIPC.org collective =================
Chip_Id: 0x35180100, CPU: Hi3518Ev100, Hardware: ARM926 @ 440 MHz
Gr: 0, Addr:0x201403FC, Data:0x3C = 0b00111100, Addr:0x20140400, Dir:0x20 = 0b00100000
Gr: 1, Addr:0x201503FC, Data:0x00 = 0b00000000, Addr:0x20150400, Dir:0x00 = 0b00000000
Gr: 2, Addr:0x201603FC, Data:0x00 = 0b00000000, Addr:0x20160400, Dir:0x00 = 0b00000000
Gr: 3, Addr:0x201703FC, Data:0x00 = 0b00000000, Addr:0x20170400, Dir:0x00 = 0b00000000
Gr: 4, Addr:0x201803FC, Data:0x00 = 0b00000000, Addr:0x20180400, Dir:0x00 = 0b00000000
Gr: 5, Addr:0x201903FC, Data:0x00 = 0b00000000, Addr:0x20190400, Dir:0x00 = 0b00000000
Gr: 6, Addr:0x201A03FC, Data:0x00 = 0b00000000, Addr:0x201A0400, Dir:0x00 = 0b00000000
Gr: 7, Addr:0x201B03FC, Data:0x60 = 0b01100000, Addr:0x201B0400, Dir:0x00 = 0b00000000
Gr: 8, Addr:0x201C03FC, Data:0x03 = 0b00000011, Addr:0x201C0400, Dir:0x00 = 0b00000000
Gr: 9, Addr:0x201D03FC, Data:0x00 = 0b00000000, Addr:0x201D0400, Dir:0x00 = 0b00000000
Gr:10, Addr:0x201E03FC, Data:0x00 = 0b00000000, Addr:0x201E0400, Dir:0x00 = 0b00000000
Gr:11, Addr:0x201F03FC, Data:0x00 = 0b00000000, Addr:0x201F0400, Dir:0x00 = 0b00000000
--------------------------------------------------------------------------------------
```

## ipctool

```
ipctool
# ./ipctool
---
chip:
  vendor: HiSilicon
  model: 3518EV100
ethernet:
  mac: "00:e0:f8:02:1a:fd"
  u-mdio-phyaddr: 1
  phy-id: 0xffffffff
  d-mdio-phyaddr: 0
  phy-mode: mii
rom:
  - type: nor
    block: 64K
    partitions:
      - name: boot
        size: 0x80000
        sha1: 29d91feb
        contains:
          - name: uboot-env
            offset: 0x40000
      - name: kernel
        size: 0x280000
        sha1: 2412dcee
      - name: rootfs
        size: 0xd00000
        path: /,jffs2
        sha1: 89ffc35a
    size: 16M
    addr-mode: 3-byte
ram:
  total: 64M
  media: 26M
firmware:
  u-boot: "2010.06 (Mar 18 2014 - 03:42:32)"
  kernel: "3.0.8 (Tue Dec 23 02:56:59 HKT 2014)"
  toolchain: gcc version 4.4.1 (Hisilicon_v100(gcc4.4-290+uclibc_0.9.32.1+eabi+linuxpthread))
  libc: uClibc 0.9.32.1
  sdk: "Hi3518_MPP_V1.0.9.0  (Aug 27 2014, 22:11:58)"
  main-app: /mnt/mtd/ipc/ipc_server
Error: unexpected value for SOI == 0x9711
sensors:
- vendor: OmniVision
  model: OV9712
  control:
    bus: 0
    type: i2c
    addr: 0x60
  data:
    type: DC
  clock: 24MHz

```

## Flash over TFTP

```
Layout from For Original U-boot
0x000000000000-0x000000040000 : "boot" 256k 0x40000   
0x000000040000-0x000000080000 : "env"  256k 0x40000
0x000000080000-0x000000280000 : "kernel" 2048k - 0x200000
0x000000280000-0x000000780000 : "rootfs" 5120k - 0x500000
0x000000780000-0x000001000000 : "rootfs_data" 
```

**Download firmware from OpenIPC github or Telegram Dev. for SOC HI3518ev100. Extract that and put it in the TFTP folder.**

Look size boot,env,kernel,rootfs :  bootargs == Take the size from your Original U-boot.

```
256k(boot),
256k(env),
2048k(kernel),
5120K(rootfs),
-(rootfs_data)'
```

```
# flash
sf probe 0
sf lock 0

# kernel image
mw.b 0x82000000 ff 1000000
tftp 0x82000000 uImage.hi3518ev100
sf erase 0x80000 0x200000
sf write 0x82000000 0x80000 ${filesize}

# file system image
mw.b 0x82000000 ff 1000000
tftp 0x82000000 rootfs.squashfs.hi3518ev100
sf erase 0x280000 0x500000
sf write 0x82000000 0x280000 ${filesize}

```

```

setenv ethaddr SA:VE:MA:CB:EF:OR

setenv bootcmd 'sf probe 0;sf read 0x82000000 0x80000 0x280000;bootm 0x82000000'
setenv bootargs 'totalmem=64M mem=32M console=ttyAMA0,115200 panic=20 root=/dev/mtdblock3 rootfstype=squashfs init=/init mtdparts=hi_sfc:256k(boot),256k(env),2048k(kernel),5120K(rootfs),-(rootfs_data)'

setenv soc hi3518ev100        
setenv osmem 32M
setenv totalmem 64M  

```

## Reboot and check OpenIPC

- Change Sensor MASK : ComMsk0: **ffc00000**, ComMsk1: **0** /etc/sensors/ov9712\_i2c\_dc_720p.ini
- Configure Majestic for PIN night/day CUT 
    - /etc/majestic.yaml

```
nightMode:
  enabled: true
  pinSwitchDelayUs: 150
  irSensorPinInvert: false
  irCutPin1: 1
  irCutPin2: 0
```

- Add script for ADC read and call curl majestic for night/day sensor
  
    [S98rled.txt](/_resources/S98rled.txt)
    
    put that to /etc/init.d/ and run chmod +x
- Reboot. Done now IR sensors and ircut works

## SAR_ADC how to. 

From doc
1. Set ADC_POWERDOWN[adc_pwrdown] to 0 and PERI_CRG32[sar_adc_cken] to 1 to
enable the SAR_ADC to enter the working state.

ADC_POWERDOWN[adc_pwrdown]
```
  # himm 0x200b0008 0x0
  # devmem 0x200b0008 32 0x0
```
PERI_CRG32[sar_adc_cken]
```
  # himm 0x20030080 0x2
  # devmem 0x20030080 32 0x2
```
2. Set ADC_CTRL[adc_chsel] to 0 to select channel 0.

ADC_CTRL[adc_chsel]
```
  # himm 0x200B0004 0x00000000
  # devmem 0x200B0004 32 0x00000000
```
3. Set ADC_INT_MASK[adc_int_mask] to 0 to enable the conversion completion interrupt.
```
  # himm 0x200B0010 0x00000000
  # devmem 0x200B0010 32 0x00000000
```
4. Set ADC_CTRL[adc_start] to 1 to enable an analog-to-digital conversion.
```
  # himm 0x200B0004 0x00000001
  # devmem 0x200B0004 32 0x00000001
```
5. wait interrupt

6. Read ADC_RESULT[adc_result_reg] to query the conversion result.
ADC_RESULT
```
  # himm 0x200B001C
  # devmem 0x200B001C
```
7. Set ADC_INT_CLR[adc_int_clr] to 1 to clear the interrupt.
ADC_INT_CLR
```
  # himm 0x200B0014 0x01
  # devmem 0x200B0014 32 0x01
```

