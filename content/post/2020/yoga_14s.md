---
title: "New laptop - Yoga 14s AMD version"
date: 2020-08-02T19:05:34+08:00
lastmod: 2020-08-02T19:05:34+08:00
draft: false
tags: ["laptop", "arch"]
---

As my old laptop ages, the condition of it keeps degrading (broken chassis/hinge, wear-out battery, nonfunctional numpad, etc.). Recently I switched to a new laptop - yoga 14s AMD version.

<!--more-->

## AMD YES

AMD Ryzen CPUs are much more powerful than Intel's. With this low voltage mobile CPU  Ryzen 4800U, it can even beat Intel's high-end desktop CPUs.

{{% center %}}
![](/image/4800u_ladder.webp)
{{% /center %}}

Not to mention AMD's CPUs are affected by less vulnerabilities as well. Finally I can drop `mitigations=off` from the kernel command line.

```bash
$ grep . /sys/devices/system/cpu/vulnerabilities/*
/sys/devices/system/cpu/vulnerabilities/itlb_multihit:Not affected
/sys/devices/system/cpu/vulnerabilities/l1tf:Not affected
/sys/devices/system/cpu/vulnerabilities/mds:Not affected
/sys/devices/system/cpu/vulnerabilities/meltdown:Not affected
/sys/devices/system/cpu/vulnerabilities/spec_store_bypass:Mitigation: Speculative Store Bypass disabled via prctl and seccomp
/sys/devices/system/cpu/vulnerabilities/spectre_v1:Mitigation: usercopy/swapgs barriers and __user pointer sanitization
/sys/devices/system/cpu/vulnerabilities/spectre_v2:Mitigation: Full AMD retpoline, IBPB: conditional, IBRS_FW, STIBP: conditional, RSB filling
/sys/devices/system/cpu/vulnerabilities/srbds:Not affected
/sys/devices/system/cpu/vulnerabilities/tsx_async_abort:Not affected
```

## Update BIOS firmware from Linux

> **<span style="color:black; font-size:1.2em;"> :warning:  <span style="color:red;">WARNING</span>: If there is an error during the BIOS update process, it may cause unrecoverable consequences. If you are worried about these consequences, stop reading and consult the technicians in Lenovo's service center. If you understand and accept the risks, and want to do it yourself (DIY), continue the read.</span>**

Updating BIOS firmware from Linux could be tricky. It's better to update the BIOS firmware from Windows first, then erase Windows and install Linux.

Yoga 14s AMD Ryzen doesn't have firmware on [Linux Vendor Firmware Service](https://fwupd.org/), and the updater is released as an executable instead of a bios update iso image ([link to Lenovo's site](https://newsupport.lenovo.com.cn/driveList.html?fromsource=driveList&selname=LR0CBAQV)). I have to follow <https://wiki.archlinux.org/index.php/Flashing_BIOS_from_Linux#Using_a_windows_PE>. The tricky parts are:
- The laptop only boots from UEFI. (So the FreeDOS based methods doesn't work.)
- The BIOS updater has secure update enabled, and can not mount the ESP partition.

To fix these problems, I have to disable the secure update function for the BIOS updater, and make an UEFI bootable Windows PE image.

Firstly, install the needed tools
```bash
$ sudo pacman -S innoextract wimlib cdrtools
```

Download the BIOS updater executable from Lenovo's site. Use `innoextract` to extract the BIOS updater.
```bash
$ innoextract BIOS-DMCN32WW.exe
Extracting "Lenovo BIOS Update Utility" - setup data version 5.5.7 (unicode)
 - "codeGetExtractPath/Win64.bat"
 - "codeGetExtractPath/WIN64.exe"
Done.
```

The extracted files are in `codeGetExtractPath` folder. Inside it there is an executable `WIN64.exe`. Extract it.
```bash
$ cd codeGetExtractPath
$ 7z x WIN64.exe -oupdater

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,16 CPUs AMD Ryzen 7 4800U with Radeon Graphics          (860F01),ASM,AES-NI)

Scanning the drive for archives:
1 file, 6429646 bytes (6279 KiB)

Extracting archive: WIN64.exe
WARNING:
WIN64.exe
Can not open the file as [PE] archive
The file is open as [7z] archive

--
Path = WIN64.exe
Open WARNING: Can not open the file as [PE] archive
Type = 7z
Offset = 590965
Physical Size = 5838681
Headers Size = 508
Method = LZMA:24 BCJ
Solid = +
Blocks = 2

Everything is Ok

Archives with Warnings: 1
Files: 15
Size:       23024732
Compressed: 6429646
```
Edit the `updater/platform.ini` file, find and change `viaESP=1` to `viaESP=0`.
```
[SecureUpdate]
viaESP=0
```

When creating the Windows PE image using `mkwinpeimg`, use the `--overlay` option to include contents in `updater` folder to the image.
```bash
$ mkwinpeimg --iso --windows-dir=/media/winimg --start-script=start.cmd winpe.iso --overlay codeGetExtractPath/updater

```

Then make the image UEFI bootable by following <https://wiki.archlinux.org/index.php/Windows_PE#Prepare_a_bootable_Windows_PE_USB_key_for_UEFI_systems>.

Reboot, hold F12 key, choose to boot from the Windows PE. The Windows command line will pop up. Go to the root folder of Windows PE, and run the actual updater `H2OFFT-Wx64.exe`, the BIOS updater will start and do it's job.
```
$ cd X:\
$ H2OFFT-Wx64.exe
```

## Enabling S3 suspend-to-RAM

For some laptop models, there is an option in BIOS to switch the sleep mode between Windows and Linux.

To check whether S3 is recognized and usable by Linux, run
```bash
$ dmesg | grep -i "acpi: (supports"
[    0.358340] ACPI: (supports S0 S4 S5)
```

For BIOS version DMCN32WW, S3 doesn't appear in the output, and there is no option to switch the sleep mode, which means I have to modify the DSDT table to enable S3 suspend.

Firstly, install the needed tools.
```bash
$ sudo pacman -S acpica cpio
```

Dump the DSDT table.
```bash
$ cat /sys/firmware/acpi/tables/DSDT > dsdt.aml
```

De-compile the dumped table.
```bash
$ iasl -d dsdt.aml
```

Get and apply the patch.
```bash
$ curl -O https://gist.githubusercontent.com/zurohki/4b859668c901e6ba13e8187a0d5d734c/raw/a04e217f273630cfae8ab3aa82002e99b9b039d5/dsdt.patch
$ patch -p1 < dsdt.patch
```

Recompile the patched DSDT source.
```bash
$ iasl -ve -tc dsdt.dsl
```

Create an CPIO archive.
```bash
$ mkdir -p kernel/firmware/acpi
$ cp dsdt.aml kernel/firmware/acpi
$ find kernel | cpio -H newc --create > acpi_override
```

Copy the CPIO archive to `/boot`.
```bash
$ sudo cp acpi_override /boot
```

Edit the boot loader's configuration file, add the initrd for CPIO archive `initrd /acpi_override` and add `mem_sleep_default=deep` to kernel's command line arguments.
```bash
$ cat /boot/loader/entries/arch_zen.conf
title Arch Linux - zen kernel
linux /vmlinuz-linux-zen
initrd /amd-ucode.img
initrd /acpi_override
initrd /initramfs-linux-zen.img
options root=PARTUUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw quiet mem_sleep_default=deep
```

Reboot, and check whether S3 state again.
```bash
$ dmesg | grep -i "acpi: (supports"
[    0.358340] ACPI: (supports S0 S3 S4 S5)
$ cat /sys/power/mem_sleep
s2idle [deep]
```

The S3 suspend is recognized and used.

## References
- <https://wiki.archlinux.org/index.php/Flashing_BIOS_from_Linux>
- <https://wiki.archlinux.org/index.php/Windows_PE>
- <https://www.reddit.com/r/linuxhardware/comments/i28nm5/ideapad_14are05_s3_sleep_fix/>
- <https://wiki.archlinux.org/index.php/Lenovo_ThinkPad_X1_Yoga_(Gen_3)#Verifying_S3>
- <https://gist.github.com/zurohki/4b859668c901e6ba13e8187a0d5d734c>
