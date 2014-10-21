# Manufacturing tool

## Introduction

* The Manufacturing tool is a tool developed by Freescale which helps uses to flash a i.MX6 
based-board.

* Flash commands are indicated on a `XML` file

* Several `flashing profiles` have already been constructed but these can extended to
  specific needs

* This basic tutorial is based on **Android KK 4.4.2** version. Flashing procedure for future
release may change.

## Android Download

1. Go to the [i.MX6Q Design tools site][imx6q_design]
1. Expand the section titled 'Run-time Software' and download the tarball
'IMX6_SABRE_AI_KK442_100_ANDROID_DEMO_BSP' under 'Operating System
Software-Board Support Packages' subsection
1. Untar the downloaded file

## MFGtool download 

1. Go to the [i.MX6Q Design tools site][imx6q_design]
1. Expand the section titled `Hardware Development Tools` and download the
tarball 'IMX_6_MFG_kk4.4.2_1.0.0_TOOL' under the `Programmers (Flash, etc)`
1. Untar the downloaded file

## Android documentation

1. Go to the [i.MX6Q Documentation site][imx6q_doc]
1. Download the tarball titled 'IMX6_KK442_100_ANDROID_DOCS'
1. Untar the downloaded file

## MFGtool configuration

1. We are flashing an Android System into the `eMMC` flash memory. On the MFGtool
folder, open the file `Profiles\MX6Q Linux Update\OS Firmware\ucl2.xml` and
look for this profile. The best set of commands that match our needs is named
`Android-SabreSD-eMMC`. You could rename it if you want.

~~~~{.xml}
<LIST name="Android-SabreSD-eMMC" desc="Choose eMMC android as media">
  <!-- Android Profile: -->
	<CMD state="BootStrap" type="boot" body="BootStrap" file ="u-boot-mx6q-sabresd.bin" >Loading U-boot</CMD>
	<CMD state="BootStrap" type="load" file="uImage" address="0x10800000"
        loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Loading Kernel.</CMD>
    <CMD state="BootStrap" type="load" file="initramfs.cpio.gz.uboot" address="0x10C00000"
        loadSection="OTH" setSection="OTH" HasFlashHeader="FALSE" >Loading Initramfs.</CMD>
    <CMD state="BootStrap" type="jump" > Jumping to OS image. </CMD>
	<CMD state="Updater" type="push" body="$ dd if=/dev/zero of=/dev/mmcblk0 bs=512 seek=1536 count=16">clean up u-boot parameter</CMD>
	<CMD state="Updater" type="push" body="$ echo 1 > /sys/devices/platform/sdhci-esdhc-imx.3/mmc_host/mmc0/mmc0:0001/boot_config">access boot partition 1</CMD>
	<CMD state="Updater" type="push" body="send" file="files/android/u-boot-6q.bin">Sending U-Boot</CMD>
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk0 bs=512 seek=2 skip=2">write U-Boot to sd card</CMD>

	<CMD state="Updater" type="push" body="$ echo 8 > /sys/devices/platform/sdhci-esdhc-imx.3/mmc_host/mmc0/mmc0:0001/boot_config">access user partition and enable boot partion 1 to boot</CMD>
	<CMD state="Updater" type="push" body="send" file="mksdcard-android.sh.tar">Sending partition shell</CMD>
	<CMD state="Updater" type="push" body="$ tar xf $FILE "> Partitioning...</CMD>
	<CMD state="Updater" type="push" body="$ sh mksdcard-android.sh /dev/mmcblk0"> Partitioning...</CMD>

	<CMD state="Updater" type="push" body="$ ls -l /dev/mmc* ">Formatting sd partition</CMD>

	<CMD state="Updater" type="push" body="send" file="files/android/boot.img">Sending kernel uImage</CMD>
	<CMD state="Updater" type="push" body="$ dd if=$FILE of=/dev/mmcblk0p1">write boot.img</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>

	<CMD state="Updater" type="push" body="$ mkfs.ext4 -b 4096 -m 0 /dev/mmcblk0p4">Formatting data partition</CMD>
	<CMD state="Updater" type="push" body="send" file="mk-encryptable-data-android.sh.tar">Sending data partition shell</CMD>
	<CMD state="Updater" type="push" body="$ tar xf $FILE ">Extracting data partition shell</CMD>
	<CMD state="Updater" type="push" body="$ sh mk-encryptable-data-android.sh /dev/mmcblk0 /dev/mmcblk0p4">Making data encryptable</CMD>

	<CMD state="Updater" type="push" body="$ mkfs.ext4 /dev/mmcblk0p5">Formatting system partition</CMD>
	<CMD state="Updater" type="push" body="$ mkfs.ext4 /dev/mmcblk0p6">Formatting cache partition</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>
	<CMD state="Updater" type="push" body="$ mkfs.ext4 /dev/mmcblk0p7">Formatting device partition</CMD>
	<CMD state="Updater" type="push" body="pipe dd of=/dev/mmcblk0p5 bs=512" file="files/android/system.img">Sending and writting system.img</CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>
	<!-- Write userdata.img is optional, for some customer this is needed, but it's optional. -->
	<!-- Also, userdata.img will have android unit test, you can use this to do some auto test. -->
<!--	<CMD state="Updater" type="push" onError="ignore" body="pipe dd of=/dev/mmcblk0p7" file="file/android/userdate.img"> Sending userdata.img(optional) </CMD>
	<CMD state="Updater" type="push" body="frf">flush the memory.</CMD>  -->
	<CMD state="Updater" type="push" body="pipe dd of=/dev/mmcblk0p2 bs=512" file="files/android/recovery.img">Sending and writting recovery.img</CMD>

	<CMD state="Updater" type="push" body="frf">Finishing rootfs write</CMD>

	<CMD state="Updater" type="push" body="$ echo Update Complete!">Done</CMD>
</LIST>
~~~~

1.Edit the file `cfg.ini`

~~~~
[profiles]
chip = MX6Q Linux Update

[platform]
board = SabreSD

[LIST]
name = Android-SabreSD-eMMC
~~~~

## Android KK images into MFGtool

1. Place the Android KK images (`u-boot-6q.bin` and `eMMC\*.img`), into the
 `Mfgtools-Rel-13.01.00_ER_MX6Q_UPDATER\Profiles\MX6Q Linux Update\OS
 Firmware\files\android`

## MFGtool Flashing

1. No dedicated boot dips are reserved for serial download mode on SABRE-SD board. Therefore, a tricky method is used
 to enter serial download mode. Change the SABRE-SD SW6 (boot) to 00001100 (from 1-8 bit) to enter download
 mode

1. Connect the board to the PC using an USB cable, this time connected to the
USB OTG port (right next to the HDMI port)

1. Start the Tool (Double click to `MfgTool2`)

1. Click start

1. When *done* (green), click stop and exit the application

## Board power-on

1. Change `Boot Switch(SW6)` to `11010110` (from 1-8 bit) to switch the board back to eMMC 8-bit boot mode.

1. Power-on the board 

1. After a few seconds, the Android banner should be on screen

[imx6q_design]: http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q&fpsp=1&tab=Design_Tools_Tab
[imx6q_doc]: http://www.freescale.com/webapp/sps/site/prod_summary.jsp?code=i.MX6Q&fpsp=1&tab=Documentation_Tab
