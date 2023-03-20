![Microchip](docs/microchip_logo.png)

# Microchip EVSE Level 2/3 Type 1/2 Buildroot External

This [buildroot external][1] includes Microchip packages, patches, setup, and
configuration to work with Microchip provided software that is not included in
mainline buildroot.  This includes creating demo root filesystems. This project
provides an extension to buildroot to support these customizations outside of
the standard buildroot tree.


## Install System Dependencies

The external is tested on Ubuntu 20.04 LTS.  The following system build
dependencies are required.

    sudo apt-get install subversion build-essential bison flex gettext \
    libncurses5-dev texinfo autoconf automake libtool mercurial git-core \
    gperf gawk expat curl cvs libexpat-dev bzr unzip bc python-dev \
    wget cpio rsync xxd bmap-tools

In some cases, buildroot will notify that additional host dependencies are
required.  It will let you know what those are.


## Buildroot Dependencies

For AT91, this buildroot external works only with the specific buildroot-at91
version 2022.02-at91.


## Build

Clone, configure, and build.  When building, use the appropriate defconfig in
the `buildroot-external-microchip/configs` directory for your board.
Here, for EVSE L1/2 Referance design, we use `evse-l2-ocpp-test-egt_defconfig`.

    $ git clone https://github.com/linux4sam/buildroot-at91.git
    Cloning into 'buildroot-at91'...
    remote: Counting objects: 271126, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 271126 (delta 0), reused 2 (delta 0), pack-reused 271124
    Receiving objects: 100% (271126/271126), 61.00 MiB | 2.36 MiB/s, done.
    Resolving deltas: 100% (186357/186357), done.

    $ ls buildroot-at91/
    arch  board  boot  CHANGES  Config.in  Config.in.legacy  configs  COPYING  DEVELOPERS  docs  fs  linux  Makefile  Makefile.legacy  package  README  support  system  toolchain  utils



    git clone https://bitbucket.microchip.com/projects/SALESAPPS/repos/ref_public_ev_charger/

    cd buildroot
    git checkout linux4sam-2022.10 -b buildroot-at91-linux4sam-2022.10

    BR2_EXTERNAL=../ref_public_ev_charger-dev/Firmware/buildroot-external-evse-l2-charger/ make evse-l2-ocpp-test-egt_defconfig
    make

The resulting bootloader, kernel, and root filesystem will be put in the
'output/images' directory.  There is also a complete `sdcard.img`.

#### Optionally Configure Packages and Kernel

Userspace packages and the Linux kernel, for example, can be optionally selected
and configured using buildroot.

To configure userspace packages and build:

    make menuconfig
    make


To configure the kernel and build:

    make linux-menuconfig
    make


Create a list of software licenses used:

    make legal-info

## Supported Families

This buildroot external repository supports WLSOM1-EK board only right now 
Microchip Technology, Inc.


#### Create an SD Card

A SD card image is generated in the file `sdcard.img`.  The first partition of
this image contains a FAT filesystem with at91bootstrap, u-boot, a u-boot env,
ITB file, which contains kernel and device tree. The second partition contains
the root filesystem. This image can be written directly to an SD card.

You need at least a 1GB SD card. All the data on the SD card will be
lost. Find the device node name for your card.  To copy the image on the SD
card:

    cd output/images
    sudo dd if=sdcard.img of=/dev/sdX bs=1M

Another method, which is cross platform, to write the SD card image is to use
[Etcher][5].

#### Run Demo
Use the same SD card, and insert into board. Power on the board, you will see Kernel
log on serial port. 
To findout serail port you can use bellow command on you machine
    dmesg | grep tty
This will give you avaliable serial ports. 

You can use PICOCOM to see board logs 
    picocom -b 115200 /dev/ttyACM0

For exit from PICOCOM use bellow key sequence
    CTRL+aq

#### Configuring the LCD Display

Right now UBOOT is not detecting LCD automatically.

So during UBOOT, pleas press enter to enter into UBOOT menu

#### Example 
    U-Boot 2018.07-linux4sam_6.0 (Oct 03 2018 - 16:03:04 +0000)

    CPU: SAMA5D27-CU
    Crystal frequency:       12 MHz
    CPU clock        :      498 MHz
    Master clock     :      166 MHz
    DRAM:  512 MiB
    MMC:   sdio-host@a0000000: 0, sdio-host@b0000000: 1
    Loading Environment from SPI Flash... SF: Detected at25df321a with page size 256 Bytes, erase size 4 KiB, total 4 MiB
    OK
    In:    serial@f8020000
    Out:   serial@f8020000
    Err:   serial@f8020000
    PDA TM5000 detected
    Net:   eth0: ethernet@f8008000
    Hit any key to stop autoboot:  0

Then use folloing command 
    => print
    at91_pda_detect=run pda4300test; run pda7000test; run pda7000btest; run pda5000test; run hdmi_test;
    at91_prepare_bootargs=test -n $display_var && setenv bootargs ${bootargs} ${at91_video_bootargs}
    at91_prepare_overlays_config=test -n $display_var && setenv at91_overlays_config '#'${display_var}
    at91_prepare_video_bootargs=test -n $display_var && setenv at91_video_bootargs video=${video_mode}
    at91_set_display=test -n $pda && setenv display $pda
    baudrate=115200
    bootargs=console=ttyS0,115200 root=/dev/mmcblk0p1 rw rootfstype=ext4 rootwait atmel.pm_modes=standby,ulp1
    bootcmd=run at91_set_display; run at91_pda_detect; run at91_prepare_video_bootargs; run at91_prepare_bootargs; run at91_prepare_overlays_config; run bootcmd_boot;
    bootcmd_boot=ext4load mmc 0 0x24000000 boot/sama5d2_xplained.itb; bootm 0x24000000#kernel_dtb${at91_overlays_config}
    bootdelay=1
    ethaddr=fc:c2:3d:0d:1f:4b
    fdtcontroladdr=3fb773c8
    hdmi_test=test -n $display && test $display = hdmi && setenv display_var 'hdmi' && setenv video_mode ${video_mode_hdmi}
    pda4300test=test -n $display && test $display = 4300 && setenv display_var 'pda4' && setenv video_mode ${video_mode_pda4}
    pda5000test=test -n $display && test $display = 5000 && setenv display_var 'pda5' && setenv video_mode ${video_mode_pda5}
    pda7000btest=test -n $display && test $display = 7000B && setenv display_var 'pda7b' && setenv video_mode ${video_mode_pda7b}
    pda7000test=test -n $display && test $display = 7000 && setenv display_var 'pda7' && setenv video_mode ${video_mode_pda7}
    stderr=serial@f8020000
    stdin=serial@f8020000
    stdout=serial@f8020000
    video_mode_hdmi=HDMI-A-1:1152x768-16
    video_mode_pda4=Unknown-1:480x272-16
    video_mode_pda5=Unknown-1:800x480-16
    video_mode_pda7=Unknown-1:800x480-16
    video_mode_pda7b=Unknown-1:800x480-16 


    serenv display_var 'pda5'

    saveenv 

    boot

After this unless you change SD card, display will work automatically 

more information 
    https://www.linux4sam.org/bin/view/Linux4SAM/UsingFITwithOverlays
    https://www.linux4sam.org/bin/view/Linux4SAM/SelectingPDAatBoot


#### Kernel and Device Tree Blob packaging

Linux Kernel and the Device Tree Blob will be included in a single file
named FIT Image (*.itb files). U-boot needs to boot a FIT Image, unlike before,
when it was loading two separate files (zImage and dtb).
For more information, check the information on the [at91Wiki][7].

#### Documentation

For more information on using and updating buildroot-at91, see the [buildroot
documentation][3].



## License

This project is licensed under the [GPLv2][2] or later with exceptions.  See the
`COPYING` file for more information.  Buildroot is licensed under the [GPLv2][2]
or later with exceptions. See the `COPYING` file in that project for more
information.


[1]: https://buildroot.org/downloads/manual/manual.html#outside-br-custom
[2]: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
[3]: https://buildroot.org/docs.html
[4]: https://www.linux4sam.org/bin/view/Linux4SAM/SDCardBootNotice
[5]: https://etcher.io/
[6]: https://www.linux4sam.org/bin/view/Linux4SAM/PDADetectionAtBoot
[7]: https://www.linux4sam.org/bin/view/Linux4SAM/UsingFITwithOverlays
[8]: https://github.com/polarfire-soc/polarfire-soc-documentation
[9]: https://mi-v-ecosystem.github.io/redirects/asymmetric-multiprocessing_amp
[10]: https://bztsrc.gitlab.io/usbimager/
[11]: https://mi-v-ecosystem.github.io/redirects/booting-from-qspi_booting-from-qspi
[12]: https://mi-v-ecosystem.github.io/redirects/boards-mpfs-generic-updating-mpfs-kit
[13]: https://github.com/polarfire-soc/icicle-kit-reference-design/releases
[14]: https://github.com/linux4microchip/buildroot-external-microchip/releases/tag/linux4microchip%2Bfpga-2022.11
