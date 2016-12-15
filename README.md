# isohybrid-bios-uefi


Or how to build from Linux an ISO hybrid image bootable from BIOS or UEFI.


> *An ISO image is an archive file of an optical disc, a type of disk image composed of the data contents from every written sector on an optical disc, including the optical disc file system. ISO image files usually have a file extension of .iso. The name ISO is taken from the ISO 9660 file system used with CD-ROM media, but what is known as an ISO image might also contain a UDF (ISO/IEC 13346) file system (commonly used by DVDs and Blu-ray Discs).* [»](https://en.wikipedia.org/wiki/ISO_image)

> *Universal Disk Format (UDF) is a profile of the specification known as ISO/IEC 13346 and ECMA-167 and is an open vendor-neutral file system for computer data storage for a broad range of media. In practice, it has been most widely used for DVDs and newer optical disc formats, supplanting ISO 9660.* [»](https://en.wikipedia.org/wiki/Universal_Disk_Format)

> *The El Torito Bootable CD specification is an extension to the ISO 9660 CD-ROM specification. It is designed to allow a computer to boot from a CD-ROM.* [»](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))

> *The ISO hybrid feature enhances ISO 9660 file system by a Master Boot Record (MBR) for booting via BIOS from disk storage devices like USB flash drives.* [»](http://www.syslinux.org/wiki/index.php?title=Isohybrid)

 
TCL 7.2 ([Tiny Core Linux](http://tinycorelinux.net/)), a minimal Linux operating system focusing on providing a base system using BusyBox and FLTK, is provided on a small ISO image only bootable from BIOS :

```bash
# modprobe kvm-intel or modprobe kvm-amd before using -enable-kvm option
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core-7.2.iso -snapshot

# TCL displays its start menu :-)


qemu -enable-kvm -m 2048 -machine q35 -hda Core-7.2.iso -snapshot

# no bootable device :-(


# nightly builds UEFI firmware can be found @ https://www.kraxel.org/repos/jenkins/edk2/
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core-7.2.iso -bios OVMF-pure-efi.fd -snapshot

# no bootable device :-(


qemu -enable-kvm -m 2048 -machine q35 -hda Core-7.2.iso -bios OVMF-pure-efi.fd -snapshot

# no bootable device :-(
```

The objective is now to reconstruct an ISO hybrid image bootable from BIOS or UEFI from the TCL ISO image.



## Tree


First, we extract files from TCL ISO image :

```bash
7z x Core-7.2.iso -osources

tree sources/
sources/
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


Next, we remove unnecessry files :

```bash
rm -rf sources/\[BOOT\]/
mv sources/boot/isolinux/ sources/

tree sources/
sources/
├── boot
│   ├── core.gz
│   └── vmlinuz
└── isolinux
    ├── boot.cat
    ├── boot.msg
    ├── f2
    ├── f3
    ├── f4
    ├── isolinux.bin
    └── isolinux.cfg
```



## Isolinux


> *ISOLINUX is a boot loader for Linux/i386 that operates off ISO 9660/El Torito CD-ROMs in "no emulation" mode. This avoids the need to create an "emulation disk image" with limited space (for "floppy emulation") or compatibility problems (for "hard disk emulation").* [»](http://www.syslinux.org/wiki/index.php?title=ISOLINUX)



## Links


- [https://en.wikipedia.org/wiki/ISO_image](https://en.wikipedia.org/wiki/ISO_image)
- [http://www.syslinux.org/wiki/index.php?title=Isohybrid](http://www.syslinux.org/wiki/index.php?title=Isohybrid)
- [https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard)](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))
- [http://www.syslinux.org/wiki/index.php?title=ISOLINUX](http://www.syslinux.org/wiki/index.php?title=ISOLINUX)
- [http://www.rodsbooks.com/gdisk/hybrid.html](http://www.rodsbooks.com/gdisk/hybrid.html)
