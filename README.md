# isohybrid-bios-uefi


Or how to build from Linux an ISO hybrid image bootable from BIOS or UEFI.


> *An ISO image is an archive file of an optical disc, a type of disk image composed of the data contents from every written sector on an optical disc, including the optical disc file system. ISO image files usually have a file extension of .iso. The name ISO is taken from the ISO 9660 file system used with CD-ROM media, but what is known as an ISO image might also contain a UDF (ISO/IEC 13346) file system (commonly used by DVDs and Blu-ray Discs).* [»](https://en.wikipedia.org/wiki/ISO_image)

> *Universal Disk Format (UDF) is a profile of the specification known as ISO/IEC 13346 and ECMA-167 and is an open vendor-neutral file system for computer data storage for a broad range of media. In practice, it has been most widely used for DVDs and newer optical disc formats, supplanting ISO 9660.* [»](https://en.wikipedia.org/wiki/Universal_Disk_Format)

> *The El Torito Bootable CD specification is an extension to the ISO 9660 CD-ROM specification. It is designed to allow a computer to boot from a CD-ROM.* [»](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))

> *The ISO hybrid feature enhances ISO 9660 file system by a Master Boot Record (MBR) for booting via BIOS from disk storage devices like USB flash drives.* [»](http://www.syslinux.org/wiki/index.php?title=Isohybrid)


[Tiny Core Linux](http://tinycorelinux.net/) (TCL) is a minimal Linux operating system focusing on providing a base system using BusyBox and FLTK.

TCL 7.2 is provided on an ISO image only bootable from BIOS :

```bash
# modprobe kvm-intel or modprobe kvm-amd
qemu -enable-kvm -m 2048 -machine q35 -cdrom Core-7.2.iso
```



## Links


- [https://en.wikipedia.org/wiki/ISO_image](https://en.wikipedia.org/wiki/ISO_image)
- [http://www.syslinux.org/wiki/index.php?title=Isohybrid](http://www.syslinux.org/wiki/index.php?title=Isohybrid)
- [https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard)](https://en.wikipedia.org/wiki/El_Torito_(CD-ROM_standard))
- [http://www.rodsbooks.com/gdisk/hybrid.html](http://www.rodsbooks.com/gdisk/hybrid.html)
