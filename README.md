# isohybrid-bios-uefi


Or how to build from Linux an ISO hybrid image bootable from BIOS or UEFI.


> *An ISO image is an archive file of an optical disc, a type of disk image composed of the data contents from every written sector on an optical disc, including the optical disc file system. ISO image files usually have a file extension of .iso. The name ISO is taken from the ISO 9660 file system used with CD-ROM media, but what is known as an ISO image might also contain a UDF (ISO/IEC 13346) file system (commonly used by DVDs and Blu-ray Discs).* [»](https://en.wikipedia.org/wiki/ISO_image)

> *Universal Disk Format (UDF) is a profile of the specification known as ISO/IEC 13346 and ECMA-167 and is an open vendor-neutral file system for computer data storage for a broad range of media. In practice, it has been most widely used for DVDs and newer optical disc formats, supplanting ISO 9660.* [»](https://en.wikipedia.org/wiki/Universal_Disk_Format)

> *The El Torito Bootable CD specification is an extension to the ISO 9660 CD-ROM specification. It is designed to allow a computer to boot from a CD-ROM.* [»](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))

> *The ISO hybrid feature enhances ISO 9660 file system by a Master Boot Record (MBR) for booting via BIOS from disk storage devices like USB flash drives.* [»](http://www.syslinux.org/wiki/index.php?title=Isohybrid)

 
[Core Linux](http://tinycorelinux.net/) 7.2, a minimal Linux operating system focusing on providing a base system using BusyBox, is provided on a small ISO image only bootable from BIOS :

```makefile
# Core-7.2.iso is available in this repo
stat -c%s Core-7.2.iso 
11116544

md5sum Core-7.2.iso 
77bf8cceacd2110120451f3f22f85156  Core-7.2.iso

# modprobe kvm-intel or modprobe kvm-amd before using -enable-kvm qemu option
qemu -version 
QEMU emulator version 2.7.0,
Copyright (c) 2003-2016 Fabrice Bellard and the QEMU Project developers

# starting qemu with Core ISO image as cdrom under BIOS firmware
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core-7.2.iso -snapshot

# Core displays its start menu :-)

# starting qemu with Core ISO image as hard disk under BIOS firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core-7.2.iso -snapshot

# no bootable device :-(

# nightly builds UEFI firmware can be found @ https://www.kraxel.org/repos/jenkins/edk2/
# uefi.md (OVMF-pure-efi.fd) is available in this repo

# starting qemu with Core ISO image as cdrom under UEFI firmware
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core-7.2.iso -bios uefi.fd -snapshot

# no bootable device :-(

# starting qemu with Core ISO image as hard disk under UEFI firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core-7.2.iso -bios uefi.fd -snapshot

# no bootable device :-(
```

The objective is now to reconstruct an ISO hybrid image bootable from BIOS or UEFI from the Core ISO image.



## Tree


First, we extract files from Core ISO image :

```make
7z x Core-7.2.iso -oCore/

tree Core/
Core/
├── [BOOT]
│   └── Boot-NoEmul.img
└── boot
    ├── core.gz
    ├── isolinux
    │   ├── boot.cat
    │   ├── boot.msg
    │   ├── f2
    │   ├── f3
    │   ├── f4
    │   ├── isolinux.bin
    │   └── isolinux.cfg
    └── vmlinuz
```


Next, we remove unnecessry files and move `isolinux` folder at the root :

```make
rm -rf Core/\[BOOT\]/
rm -f Core/boot/isolinux/boot.cat

mv Core/boot/isolinux/ Core/

tree Core/
Core/
├── boot
│   ├── core.gz
│   └── vmlinuz
└── isolinux
    ├── boot.msg
    ├── f2
    ├── f3
    ├── f4
    ├── isolinux.bin
    └── isolinux.cfg
```



## Old BIOS side


> *ISOLINUX is a boot loader for Linux/i386 that operates off ISO 9660/El Torito CD-ROMs in "no emulation" mode. This avoids the need to create an "emulation disk image" with limited space (for "floppy emulation") or compatibility problems (for "hard disk emulation").* [»](http://www.syslinux.org/wiki/index.php?title=ISOLINUX)


First, we override the embedded bootloader with our current one and add `ldlinux.c32` :

```make
syslinux --version
syslinux 6.03  Copyright 1994-2014 H. Peter Anvin et al

cp /lib/syslinux/bios/isolinux.bin Core/isolinux/
overwrite 'Core/isolinux/isolinux.bin'? y

cp /lib/syslinux/bios/ldlinux.c32 Core/isolinux/
```


Next, we build our first ISO image bootable from BIOS and test it with qemu :

```make
mkisofs -version 
mkisofs 3.02a06 (x86_64-unknown-linux-gnu)
Copyright (C) 1993-1997 Eric Youngdale (C) 1997-2016 Joerg Schilling

mkisofs -output Core.iso \
  -eltorito-boot \
  isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table \
  -eltorito-catalog isolinux/boot.cat \
  Core/

# starting qemu with new Core ISO image as cdrom under BIOS firmware
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core.iso -snapshot

# Core displays its start menu :-)
```


Now, we enhance the ISO image with the isohybrid feature to be able to boot Core from a USB flash drive :

```make
isohybrid -version
isohybrid version 0.12

isohybrid Core.iso

qemu -enable-kvm -m 2048 -machine q35 -cdrom Core.iso -snapshot

# Core displays its start menu :-)

# starting qemu with new Core ISO image as hard disk under BIOS firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core.iso -snapshot

# Core displays its start menu :-)
```



## New UEFI side


> *The EFI system partition (ESP) is a partition on a data storage device (usually a hard disk drive or solid-state drive) that is used by computers adhering to the Unified Extensible Firmware Interface (UEFI). When a computer is booted, UEFI firmware loads files stored on the ESP to start installed operating systems and various utilities.* [»](https://en.wikipedia.org/wiki/EFI_system_partition)


The « new » UEFI boot is based on the presence of a specific EFI System Partition (ESP) formated with FAT file system.
First, to adjust the size of the ESP image, we prepare the content of this partition in a folder :

```make
mkdir -p Image/{efi/boot/,syslinux}

# because syslinux can't access files outside of the image
# core.gz and vmlinux must be embedded
cp -a Core/boot/ Image/

cp Core/isolinux/{boot.msg,f*} Image/syslinux/
cp Core/isolinux/isolinux.cfg Image/syslinux/syslinux.cfg

cp /lib/syslinux/efi64/ldlinux.e64 Image/syslinux/
cp /lib/syslinux/efi64/syslinux.efi Image/efi/boot/bootx64.efi

tree Image/
Image/
├── boot
│   ├── core.gz
│   └── vmlinuz
├── efi
│   └── boot
│       └── bootx64.efi
└── syslinux
    ├── boot.msg
    ├── f2
    ├── f3
    ├── f4
    ├── ldlinux.e64
    └── syslinux.cfg
```


Next, we build the ESP image from the previous folder :

```make
du -s Image/
10796	Image/

mkdir Core/efi/

# 128Kb for FAT data and 128Kb for syslinux bios files
truncate -s $((10796+128+128))k Core/efi/esp.img

# can also be FAT12 or FAT32 according volumetry
mkfs.msdos -F 16 -f 1 -M 0xF0 -r 112 -R 1 Core/efi/esp.img

# mount for you the ESP image on a ready mount point
sudo mount Core/efi/esp.img mount.point/ -o uid=you

cp -av Image/* mount.point/
sudo umount mount.point/

syslinux --install --directory syslinux/ Core/efi/esp.img

# starting qemu with ESP image under BIOS firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core/efi/esp.img -snapshot

# Core displays its start menu :-)

# starting qemu with ESP image under UEFI firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core/efi/esp.img -bios uefi.fd -snapshot

# Core displays its start menu :-)
```


Now, we insert the ESP image as a second El Torito boot entry :

```make
tree Core/
Core/
├── boot
│   ├── core.gz
│   └── vmlinuz
├── efi
│   └── esp.img
└── isolinux
    ├── boot.msg
    ├── f2
    ├── f3
    ├── f4
    ├── isolinux.bin
    ├── isolinux.cfg
    └── ldlinux.c32

mkisofs -output Core.iso \
  -eltorito-boot \
  isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table \
  -eltorito-alt-boot -eltorito-platform efi -eltorito-boot \
  efi/esp.img -no-emul-boot \
  -eltorito-catalog isolinux/boot.cat \
  Core/

isohybrid Core.iso

qemu -enable-kvm -m 2048 -machine q35 -cdrom Core.iso -snapshot

# Core displays its start menu :-)

qemu -enable-kvm -m 2048 -machine q35 -hda Core.iso -snapshot

# Core displays its start menu :-)

# starting qemu with new Core ISO image as cdrom under UEFI firmware
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core.iso -bios uefi.fd -snapshot

# KVM internal error :-(

# starting qemu with new Core ISO image as hard disk under UEFI firmware
qemu -enable-kvm -m 2048 -machine q35 -hda Core.iso -bios uefi.fd -snapshot

# Core displays its start menu :-)
```




## Links


- [https://en.wikipedia.org/wiki/ISO_image](https://en.wikipedia.org/wiki/ISO_image)
- [http://www.syslinux.org/wiki/index.php?title=Isohybrid](http://www.syslinux.org/wiki/index.php?title=Isohybrid)
- [https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard)](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))
- [http://www.syslinux.org/wiki/index.php?title=ISOLINUX](http://www.syslinux.org/wiki/index.php?title=ISOLINUX)
- [https://en.wikipedia.org/wiki/EFI_system_partition](https://en.wikipedia.org/wiki/EFI_system_partition)
- [http://www.rodsbooks.com/gdisk/hybrid.html](http://www.rodsbooks.com/gdisk/hybrid.html)
