MKISOFS
=======

[NAME](#NAME)
[SYNOPSIS](#SYNOPSIS)
[DESCRIPTION](#DESCRIPTION)
[OPTIONS](#OPTIONS)
[HFS OPTIONS](#HFS%20OPTIONS)
[CHARACTER SETS](#CHARACTER%20SETS)
[HFS CREATOR/TYPE](#HFS%20CREATOR/TYPE)
[HFS MACINTOSH FILE FORMATS](#HFS%20MACINTOSH%20FILE%20FORMATS)
[HFS MACINTOSH FILE NAMES](#HFS%20MACINTOSH%20FILE%20NAMES)
[HFS CUSTOM VOLUME/FOLDER ICONS](#HFS%20CUSTOM%20VOLUME/FOLDER%20ICONS)
[HFS BOOT DRIVER](#HFS%20BOOT%20DRIVER)
[EL TORITO BOOT INFORMATION TABLE](#EL%20TORITO%20BOOT%20INFORMATION%20TABLE)
[CONFIGURATION](#CONFIGURATION)
[EXAMPLES](#EXAMPLES)
[AUTHOR](#AUTHOR)
[NOTES](#NOTES)
[BUGS](#BUGS)
[HFS PROBLEMS/LIMITATIONS](#HFS%20PROBLEMS/LIMITATIONS)
[SEE ALSO](#SEE%20ALSO)
[FUTURE IMPROVEMENTS](#FUTURE%20IMPROVEMENTS)
[AVAILABILITY](#AVAILABILITY)
[MAILING LISTS](#MAILING%20LISTS)
[MAINTAINER](#MAINTAINER)
[HFS MKHYBRID MAINTAINER](#HFS%20MKHYBRID%20MAINTAINER)
[INTERFACE STABILITY](#INTERFACE%20STABILITY)

------------------------------------------------------------------------

NAME []()
---------

mkisofs − create an hybrid ISO-9660/JOLIET/HFS/UDF filesystem-image with optional Rock Ridge attributes.

SYNOPSIS []()
-------------

**mkisofs** \[ *options* \] \[ **−o** *filename* \] *pathspec \[pathspec ...\]* **
mkisofs** \[ *options* \] \[ **−o** *filename* \] **−find** *\[find expression\]*

DESCRIPTION []()
----------------

***mkisofs*** is effectively a pre-mastering program to generate an ISO-9660/JOLIET/HFS/UDF hybrid filesystem.

ISO-9660/JOLIET/UDF filesystems are limited to a maximum size of 8 TB. The maximum size of a single file is 8 TB (single files in UDF are currently limited to aprox. 200 GB). If you like to have files larger than 2 GB, you need to specify **−iso−level 3** or above. If a HFS hybrid is created, the maximum file size for files in the HFS hybrid is 2 GB in any case.

**Hybrid filesystem support
mkisofs** is capable of generating the **System Use Sharing Protocol records (SUSP)** specified by the **Rock Ridge Interchange Protocol.** This is used to further describe the files in the ISO-9660 filesystem to a UNIX host, and provides information such as longer filenames, uid/gid, posix permissions, symbolic links, hard links, block and character devices.

If Joliet, HFS or UDF hybrid command line options are specified, **mkisofs** will create additional separate filesystem meta data for Joliet, HFS or UDF. The file content in this case refers to the same data blocks on the media. It will generate a pure ISO-9660 filesystem unless the Joliet, HFS or UDF hybrid command line options are given.

**mkisofs** can generate a *true* (or *shared*) HFS hybrid filesystem. The same files are seen as HFS files when accessed from a Macintosh and as ISO-9660 files when accessed from other machines. HFS stands for *Hierarchical File System* and is the native file system used on Macintosh computers up to Mac OS 9.

As an alternative, **mkisofs** can generate the *Apple Extensions to ISO-9660* or *UDF* for each file. These extensions provide each file with CREATOR, TYPE and certain Finder Flags when accessed from a Macintosh. See the **HFS MACINTOSH FILE FORMATS** section below.

**Functional description
mkisofs** takes a snapshot of a given directory tree, and generates a binary image which will correspond to an ISO-9660 or Joliet/HFS/UDF filesystem when written to a block device.

Each file written to the ISO-9660 filesystem must have a filename in the 8.3 format (8 characters, period, 3 characters, all upper case), even if Rock Ridge attributes are in use. This filename is used on systems that are not able to make use of the Rock Ridge extensions (such as MS-DOS), and each filename in each directory must be different from the other filenames in the same directory. **mkisofs** generally tries to form correct names by forcing the UNIX filename to upper case and truncating as required, but often times this yields unsatisfactory results when there are cases where the truncated names are not all unique. **mkisofs** assigns weightings to each filename, and if two names that are otherwise the same are found the name with the lower priority is renamed to have a 3 digit number as an extension (where the number is guaranteed to be unique). An example of this would be the files foo.bar and foo.bar.~1~ - the file foo.bar.~1~ would be written as FOO000.BAR;1 and the file foo.bar would be written as FOO.BAR;1

When used with various HFS or UDF options, **mkisofs** will attempt to recognise files stored in a number of Apple/Unix file formats and will copy the data and resource forks as well as any relevant finder information. See the **HFS MACINTOSH FILE FORMATS** section below for more about formats **mkisofs** supports.

Note that **mkisofs** is not designed to communicate with writers for optical media directly. Most writers have proprietary command sets which vary from one manufacturer to another, and you need a specialized tool like **cdrecord** to actually burn the disk.

The **cdrecord** utility is a utility capable of burning an actual disc. The latest version of **cdrecord** is available from **https://sourceforge.net/projects/cdrtools/files/** or **https://sourceforge.net/projects/cdrtools/files/alpha/**

Also you should know that most CD writers are very particular about timing. Once you start to burn a disc, you cannot let their buffer empty before you are done, or you will end up with a corrupt disc. Thus it is critical that you be able to maintain an uninterrupted data stream to the writer for the entire time that the disc is being written.

**Dealing with path names
pathspec** is the path of the directory tree to be copied into the ISO-9660 filesystem. Multiple paths can be specified, and **mkisofs** will merge the files found in all of the specified path components to form the cdrom image.

If the option **−graft−points** has been specified, it is possible to graft the paths at points other than the root directory, and it is possible to graft files or directories onto the cdrom image with names different than what they have in the source filesystem. This is easiest to illustrate with a couple of examples. Let’s start by assuming that a local file ../old.lis exists, and you wish to include it in the cdrom image.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td></td>
<td><p>foo/bar/=../old.lis</p></td>
</tr>
</tbody>
</table>

will include the file old.lis in the cdrom image at /foo/bar/old.lis, while

<table>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p>foo/bar/xxx=../old.lis</p></td>
</tr>
</tbody>
</table>

will include the file old.lis in the cdrom image at /foo/bar/xxx. The same sort of syntax can be used with directories as well. **mkisofs** will create any directories required such that the graft points exist on the cdrom image - the directories do not need to appear in one of the paths. By default, any directories that are created on the fly like this will have permissions 0555 and appear to be owned by the person running mkisofs. If you wish other permissions or owners of the intermediate directories, see **−uid**, **−gid**, **−dir−mode**, **−file−mode** and **−new−dir−mode**.

**mkisofs** will also run on Win9*x*/NT*x* machines when compiled with Cygnus’ cygwin (available from http://sourceware.cygnus.com/cygwin/). Therefore most references in this man page to *Unix* also apply to *Win32* or *Win64*.

OPTIONS []()
------------

**−abstract** *FILE*

Specifies the abstract file name in the primary volume descriptor. There is space on the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. This parameter can also be set in the file **.mkisofsrc** with ABST=filename. If specified in both places, the command line version is used.

It is up to the user of **mkisofs** to include a file with the apropriate name in the created filesystem tree.

**−A** *application\_id* **
−appid** *application\_id*

Specifies a text string that will be written into the volume header. This should describe the application that will be on the disc. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. This parameter can also be set in the file **.mkisofsrc** with APPI=id. If specified in both places, the command line version is used.

**−allow−leading−dots**

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−ldots</strong></p></td>
<td></td>
<td><p>Allow ISO-9660 filenames to begin with a period. Usually, a leading dot is replaced with an underscore in order to maintain MS-DOS compatibility.</p></td>
</tr>
</tbody>
</table>

This violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

**−allow−lowercase**

This options allows lower case characters to appear in ISO-9660 filenames.
This violates the ISO-9660 standard, but it happens to work on some systems. Use with caution.

**−allow−multidot**

This options allows more than one dot to appear in ISO-9660 filenames. A leading dot is not affected by this option, it may be allowed separately using the **−allow−leading−dots** option.
This violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

**−biblio** *FILE*

Specifies the bibliographic file name in the primary volume descriptor. There is space on the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. This parameter can also be set in the file **.mkisofsrc** with BIBLO=filename. If specified in both places, the command line version is used.

It is up to the user of **mkisofs** to include a file with the apropriate name in the created filesystem tree.

**−cache−inodes**

Cache inode and device numbers to find hard links to files. If **mkisofs** finds a hard link (a file with multiple names), then the file will only appear once on the CD. This helps to save space on the CD. The option **−cache−inodes** is default on UNIX like operating systems. Be careful when using this option on a filesystem without unique inode numbers as it may result in files containing the wrong content on CD.

If inodes are not cached, **mkisofs** will revert to the old Rrip Version-1.10 (see **−rrip110**) and **mkisofs** will not be able to create **correct inode numbers** for zero sized files.

**−no−cache−inodes**

Do not cache inode and device numbers. This option is needed whenever a filesystem does not have unique inode numbers. It is the default on old **Cygwin** versions. As the Microsoft operating system that runs below **Cygwin** uses 64 bit inode numbers for NTFS, it does not have unique inode numbers in the 32 bit range. Old Cygwin versions create fake 32-bit inode numbers from a hash algorithm and thus create non-unique numbers. If **mkisofs** would cache inodes on old Cygwin versions, it would believe that some files are identical although they are not. The result in this case are files that contain the wrong content if a significant amount of different files (&gt; ~5000) is in inside the tree that is to be archived. This does not happen when the **−no−cache−inodes** is used, but the disadvantage is that **mkisofs** cannot detect hardlinks anymore and the resulting CD image may be larger than expected.

If inodes are not cached, **mkisofs** will revert to the old Rrip Version-1.10 (see **−rrip110**) and **mkisofs** will not be able to create **correct inode numbers** for zero sized files.

**−b** *eltorito\_boot\_image* **
−eltorito−boot** *eltorito\_boot\_image*

Specifies the path and filename of the boot image to be used when making an "El Torito" bootable CD. The pathname must be relative to the source path and inside the source tree specified to **mkisofs.** This option is required to make an "El Torito" bootable CD. The boot image must be exactly the size of either a 1200, 1440, or a 2880 kB floppy, and **mkisofs** will use this size when creating the output ISO-9660 filesystem. It is assumed that the first 512 byte sector should be read from the boot image (it is essentially emulating a normal floppy drive). This will work, for example, if the boot image is a boot floppy.

If the boot image is not an image of a floppy, you need to add one of the options: **−hard−disk−boot** or **−no−emul−boot**. If the system should not boot off the emulated disk, use **−no−boot**.

More than one boot entry may be specified, see **−eltorito−platform** and **−eltorito−alt−boot** on how to specify more boot entries. The first boot entry is the **default boot entry**. Additional boot entries are members for a multi boot configuration.

If the **−sort** option has not been specified, the boot images are sorted with low priority (+2) to the beginning of the medium. If you don’t like this, you need to specify a sort weight of 0 for the boot images.

**−eltorito−alt−boot**

Start with a new set of "El Torito" boot parameters. This allows to have more than one El Torito boot entry on a CD. A maximum of 63 El Torito boot entries may be put on a single CD.

The **−eltorito−alt−boot** option starts a new boot entry with the same *platform id* but no new boot section except when it appears past the first boot entry which is the default boot entry.

**−eltorito−platform** *id*

Set the "El Torito" platform id for a boot record or a section of boot records. The. *id* parameter may be either:

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>x86</strong></p></td>
<td></td>
<td><p>This is the default <em>platform id</em> value and specifies entries for the PC platform. If no <strong>−eltorito−platform</strong> option appears before the first <strong>−eltorito−boot</strong> option, the default boot entry becomes an entry for the x86 PC platform.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>PPC</strong></p></td>
<td></td>
<td><p>Boot entries for the Power PC platform.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>Mac</strong></p></td>
<td></td>
<td><p>Boot entries for the Apple Mac platform.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>efi</strong></p></td>
<td></td>
<td><p>Boot entries for EFI based PCs.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>#</strong></p></td>
<td></td>
<td><p>A numeric value specifying any platform id.</p></td>
</tr>
</tbody>
</table>

If the option **−eltorito−platform** appears before the first **−eltorito−boot** option, it sets the *platform id* for the default boot entry.

If the option **−eltorito−platform** appears after an **−eltorito−boot** option and sets the *platform id* to a value different from the previous value, it starts a new set of boot entries.

The second boot entry and any new *platform id* creates a new section header and reduces the number of boot entries per CD by one.

**errctl=** *name* **
errctl=** *error control spec*

Add the content from file *name* to the error control definitions or add *error control spec* to the error control definitions. More than one error control file and more than one *error control spec* as well as a mixture of both forms is possible.

The reason for using error control is to make **mkisofs** quiet about error conditions that are known to be irrelevant on the quality of the created filesystem or to tell **mkisofs** to abort on certain error conditions instead of trying to continue with the filesystem.

A typical reason to use error control is to suppress warnings about growing log files while doing a backup on a live file system. Another typical reason to use error control is to tell **mkisofs** to abort if e.g. a file could not be archived instead of continuing to archive other files from a list.

The error control file contains a set of lines, each starting with a list of error conditions to be ignored followed by white space followed by a file name pattern (see **match**(1) or **patmatch**(3) for more information). The *error control spec* uses the same syntax as a single line from the error control file. If the file name pattern needs to start with white space, use a backslash to escape the start of the file name. It is not possible to have new line characters in the file name pattern. Whenever an error situation is encountered, **mkisofs** checks the lines in the error control file starting from the top. If the current error condition is listed on a line in the error control file, then **mkisofs** checks whether the pattern on the rest of the line matches the current file name. If this is the case, **mkisofs** uses the current error control specification to control the current error condition.

The list of error conditions to be handled may use one or more (in this case separated by a ’|’ character) identifiers from the list below:

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>ABORT</strong></p></td>
<td></td>
<td><p>If this meta condition is included in an error condition, <strong>mkisofs</strong> aborts (exits) as soon as possible after this error condition has been seen instead of making <strong>mkisofs</strong> quiet about the condition. This error condition flag may only be used together with at another error condition or a list of error conditions (separated by a ’|’ character).</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>WARN</strong></p></td>
<td></td>
<td><p>If this meta condition is included in an error condition, <strong>mkisofs</strong> prints the warning about the error condition but the error condition does not affect the exit code of <strong>mkisofs</strong> and the error statistics (which is printed to the end) does not include the related errors. This error condition flag may only be used together with at another error condition or a list of error conditions (separated by a ’|’ character). The <strong>WARN</strong> meta condition has a lower precedence than <strong>ABORT</strong>.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>ALL</strong></p></td>
<td></td>
<td><p>This is a shortcut for all error conditions below.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>STAT</strong></p></td>
<td></td>
<td><p>Suppress warnings that <strong>mkisofs</strong> could not <strong>stat</strong>(2) a file.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>GETACL</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on which <strong>mkisofs</strong> had problems to retrieve the ACL information.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>OPEN</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that could not be opened.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>READ</strong></p></td>
<td></td>
<td><p>Suppress warnings read errors on files.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>WRITE</strong></p></td>
<td></td>
<td><p>Suppress warnings write errors on files.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>READLINK</strong></p></td>
<td></td>
<td><p>Suppress warnings <strong>readlink</strong>(2) errors on symbolic links.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>GROW</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that did grow while they have been archived.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SHRINK</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that did shrink while they have been archived.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>MISSLINK</strong></p></td>
<td></td>
<td><p>Suppress warnings about files for which <strong>mkisofs</strong> was unable to archive all hard links.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>NAMETOOLONG</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that could not be archived because the name of the file is too long for the archive format.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>FILETOOBIG</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that could not be archived because the size of the file is too big for the archive format.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SPECIALFILE</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that could not be archived because the file type is not supported by the archive format.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>GETXATTR</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on that <strong>mkisofs</strong> could not retrieve the extended file attribute information.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SETTIME</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on that <strong>mkisofs</strong> could not set the time information during extraction.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>SETMODE</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on that <strong>mkisofs</strong> could not set the access modes during extraction.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SECURITY</strong></p></td>
<td></td>
<td><p>Suppress warnings about files that have been skipped on extraction because they have been considered to be a security risk. This currently applies to all files that have a ’/../’ sequence inside when <strong>−..</strong> has not been specified.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>LSECURITY</strong></p></td>
<td></td>
<td><p>Suppress warnings about links that have been skipped on extraction because they have been considered to be a security risk. This currently applies to all link names that start with ’/’ or have a ’/../’ sequence inside when <strong>−secure−links</strong> has been specified. In this case, <strong>mkisofs</strong> tries to match the link name against the pattern in the error control file.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SAMEFILE</strong></p></td>
<td></td>
<td><p>Suppress warnings about links that have been skipped on extraction because source and target of the link are pointing to the same file. If <strong>mkisofs</strong> would not skip these files, it would end up with removing the file completely. In this case, <strong>mkisofs</strong> tries to match the link name against the pattern in the error control file.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>BADACL</strong></p></td>
<td></td>
<td><p>Suppress warnings access control list conversion problems.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SETACL</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on that <strong>mkisofs</strong> could not set the ACL information during extraction.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>SETXATTR</strong></p></td>
<td></td>
<td><p>Suppress warnings about files on that <strong>mkisofs</strong> could not set the extended file attribute information during extraction.</p></td>
</tr>
</tbody>
</table>

If a specific error condition is ignored, then the error condition is not only handled in a silent way but also excluded from the error statistics that are printed at the end of the **mkisofs** run.

Be very careful when using error control as you may ignore any error condition. If you ignore the wrong error conditions, you may not be able to see real problems anymore.

Note that currently only the tags **OPEN**, **READ**, **GROW**, **SHRINK**, are checked from **mkisofs**. **
−B** *img\_sun4,img\_sun4c,img\_sun4m,img\_sun4d,img\_sun4e* **
−sparc−boot** *img\_sun4,img\_sun4c,img\_sun4m,img\_sun4d,img\_sun4e*

Specifies a comma separated list of boot images that are needed to make a bootable CD for sparc systems. Partition 0 is used for the ISO-9660 image, the first image file is mapped to partition 1. There may be empty fields in the comma separated list. The maximum number of possible partitions is 8 so it is impossible to specify more than 7 partition images. This option is required to make a bootable CD for Sun sparc systems. If the **−B** or **−sparc−boot** option has been specified, the first sector of the resulting image will contain a Sun disk label. This disk label specifies slice 0 for the ISO-9660 image and slice 1 ... slice 7 for the boot images that have been specified with this option. Byte offset 512 ... 8191 within each of the additional boot images must contain a primary boot that works for the appropriate sparc architecture. The rest of each of the images usually contains an ufs filesystem that is used primary kernel boot stage.

The implemented boot method is the boot method found with SunOS 4.x and SunOS 5.x. However, it does not depend on SunOS internals but only on properties of the Open Boot prom. For this reason, it should be usable for any OS that boots off a sparc system.

For more information also see the **NOTES** section below.

If the special filename **...** is used, the actual and all following boot partitions are mapped to the previous partition. If **mkisofs** is called with **−G** *image* **−B** *...* all boot partitions are mapped to the partition that contains the ISO-9660 filesystem image and the generic boot image that is located in the first 16 sectors of the disk is used for all architectures.

**−G** *generic\_boot\_image*

Specifies the path and filename of the generic boot image to be used when making a generic bootable CD. The **generic\_boot\_image** will be placed on the first 16 sectors of the CD. The first 16 sectors are the sectors that are located before the ISO-9660 primary volume descriptor. If this option is used together with the **−sparc−boot** option, the Sun disk label will overlay the first 512 bytes of the generic boot image.

**−hard−disk−boot**

Specifies that the boot image used to create "El Torito" bootable CDs is a hard disk image. The hard disk image must begin with a master boot record that contains a single partition.

**−ignore−error**

Ignore errors. **mkisofs** by default aborts on several errors, such as read errors. With this option in effect, **mkisofs** tries to continue. Use with care.

**−no−emul−boot**

Specifies that the boot image used to create "El Torito" bootable CDs is a ’no emulation’ image. The system will load and execute this image without performing any disk emulation.

**−no−boot**

Specifies that the created "El Torito" CD should be marked as not bootable. The system will provide an emulated drive for the image, but will boot off a standard boot device.

**−boot−load−seg** *segment\_address*

Specifies the load segment address of the boot image for no-emulation "El Torito" CDs.

**−boot−load−size** *load\_sectors*

Specifies the number of "virtual" (512-byte) sectors to load in no-emulation mode. The default is to load the entire boot file. Some BIOSes may have problems if this is not a multiple of 4.

**−boot−info−table**

Specifies that a 56-byte table with information of the CD-ROM layout will be patched in at offset 8 in the boot file. If this option is given, the boot file is modified in the source filesystem, so make sure to make a copy if this file cannot be easily regenerated! See the **EL TORITO BOOT INFO TABLE** section for a description of this table.

**−C** *last\_sess\_start,next\_sess\_start* **
−cdrecord−params** *last\_sess\_start,next\_sess\_start*

This option is needed when **mkisofs** is used to create a CDextra or the image of a second session or a higher level session for a multi session disk. The option **−C** takes a pair of two numbers separated by a comma. The first number is the sector number of the first sector in the last session of the disk that should be appended to. The second number is the starting sector number of the new session. The expected pair of numbers may be retrieved by calling **cdrecord −msinfo ...** If the **−C** option is used in conjunction with the **−M** option, **mkisofs** will create a filesystem image that is intended to be a continuation of the previous session. If the **−C** option is used without the **−M** option, **mkisofs** will create a filesystem image that is intended to be used for a second session on a CDextra. This is a multi session CD that holds audio data in the first session and a ISO-9660 filesystem in the second session.

**−c** *boot\_catalog* **
−eltorito−catalog** *boot\_catalog*

Specifies the path and filename of the boot catalog to be used when making an "El Torito" bootable CD. The pathname must be relative to the source path specified to **mkisofs.** This option is required to make a bootable CD. This file will be inserted into the output tree and not created in the source filesystem, so be sure the specified filename does not conflict with an existing file, as it will be excluded. Usually a name like "boot.catalog" is chosen.

If the **−sort** option has not been specified, the boot catalog sorted with low priority (+1) to the beginning of the medium. If you don’t like this, you need to specify a sort weight of 0 for the boot catalog.

**−check−oldnames**

Check all filenames imported from old session for compliance with actual **mkisofs** ISO-9660 file naming rules. It his option is not present, only names with a length &gt; 31 are checked as these files are a hard violation of the ISO-9660 standard.

**−check−session** *FILE*

Check all old sessions for compliance with actual **mkisofs** ISO-9660 file naming rules. This is a high level option that is a combination of the options: **−M** *FILE* **−C 0,0 −check−oldnames** For the parameter *FILE* see description of **−M** option.

**−copyright** *FILE*

Specifies the Copyright file name in the primary volume descriptor. There is space on the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. This parameter can also be set in the file **.mkisofsrc** with COPY=filename. If specified in both places, the command line version is used.

It is up to the user of **mkisofs** to include a file with the apropriate name in the created filesystem tree.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−d</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−omit−period**

Omit trailing period from files that do not have a period.
This violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−D</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−disable−deep−relocation**

Do not use deep directory relocation, and instead just pack them in the way we see them.
If ISO-9660:1999 has not been selected, this violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

**−data−change−warn**

If the size of a file changes while the file is being archived, treat this condition as a warning only that does not cause **mkisofs** to abort. A warning message is still written if the condition is not otherwise ignored by another rule from an **errctl=** option. The **−data−change−warn** option works as if the last error control option was

**errctl=***"WARN|GROW|SHRINK \*"*

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−debug</strong></p></td>
<td></td>
<td><p>Increment debug value by one.</p></td>
<td></td>
</tr>
</tbody>
</table>

**−dir−mode** *mode*

Overrides the mode of directories used to create the image to *mode*. See **−new−dir−mode** on how to specify a different *mode* that is used for directories that do not exist in the tree specified by the source-path. Specifying the **−dir−mode** option automatically enables Rock Ridge extensions.

**−dvd−audio**

Generate DVD-Audio compliant UDF file system. This is done by sorting the order of the content of the appropriate files. Sorting only works if the DVD-Audio filenames include upper case characters only.

Note that in order to get a DVD-Audio compliant filesystem image, you need to prepare a DVD-Audio compliant directory tree. This means you need to have a directory AUDIO\_TS (all caps) in the root directory of the resulting DVD and you should have a directory VIDEO\_TS. The directory AUDIO\_TS needs to include all needed files (file names must be all caps) for a compliant DVD-Audio filesystem.

**−dvd−hybrid**

Equivalent to selecting both -dvd-audio and -dvd-video

**−dvd−video**

Generate DVD-Video compliant UDF file system. This is done by sorting the order of the content of the appropriate files and by adding padding between the files if needed. Sorting only works if the DVD-Video filenames include upper case characters only.

Note that in order to get a DVD-Video compliant filesystem image, you need to prepare a DVD-Video compliant directory tree. This means you need to have a directory VIDEO\_TS (all caps) in the root directory of the resulting DVD and you should have a directory AUDIO\_TS. The directory VIDEO\_TS needs to include all needed files (file names must be all caps) for a compliant DVD-Video filesystem.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−f</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−follow−links**

Follow all symbolic links when generating the filesystem. When this option is not in use, symbolic links will be entered using Rock Ridge if enabled, otherwise the file will be ignored.

See also **−posix−L** option.

**−file−mode** *mode*

Overrides the mode of regular files used to create the image to *mode*. Specifying this option automatically enables Rock Ridge extensions.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−find</strong></p></td>
<td></td>
<td><p>This option acts a separator. If it is used, all <strong>mkisofs</strong> options must be to the left of the <strong>−find</strong> option. To the right of the <strong>−find</strong> option, <strong>mkisofs</strong> accepts the <strong>find</strong> command line syntax only.</p></td>
</tr>
</tbody>
</table>

The **find** expression acts as a filter between the source of file names and the consumer, which is archiving engine. If the **find** expression evaluated as TRUE, then the related file is selected for processing, otherwise it is omited.

In order to make the evaluation of the **find** expression more convenient, **mkisofs** implements additional **find primaries** that have side effects on the file meta data. **Mkisofs** implements the following additional **find** primaries:

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−help</strong></p></td>
<td></td>
<td><p>Lists the available <strong>find</strong>(1) syntax.</p></td>
<td></td>
</tr>
</tbody>
</table>

**−chgrp** *gname*

The primary always evaluates as true; it sets the group of the file to *gname*.

**−chmod** *mode*

The primary always evaluates as true; it sets the permissions of the file to *mode*. Octal and symbolic permissions are accepted for *mode* as with **chmod**(1).

**−chown** *uname*

The primary always evaluates as true; it sets the owner of the file to *uname*.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−false</strong></p></td>
<td></td>
<td><p>The primary always evaluates as false; it allows to make the result of the full expression different from the result of a part of the expression.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−true</strong></p></td>
<td></td>
<td><p>The primary always evaluates as true; it allows to make the result of the full expression different from the result of a part of the expression.</p></td>
</tr>
</tbody>
</table>

The command line:

**mkisofs -o o.iso -find . ( -type d -ls -o false ) -o ! -type d**

lists all directories and puts all non-directories to the image **o.iso**.

The command line:

**mkisofs -o o.iso -find . ( -type d -chown root -o true )**

archives all directories so they appear to be owned by root in the archive, all non-directories are archived as they are in the file system.

Note that the **−ls**, **−exec** and the **−ok** primary cannot be used if **stdin** or stdout has not been redirected.

**−gid** *gid*

Overrides the gid read from the source files to the value of *gid*. Specifying this option automatically enables Rock Ridge extensions.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−gui</strong></p></td>
<td></td>
<td><p>Switch the behaviour for a GUI. This currently makes the output more verbose but may have other effects in future.</p></td>
</tr>
</tbody>
</table>

**−graft−points**

Allow to use graft points for filenames. If this option is used, all filenames are checked for graft points. The filename is divided at the first unescaped equal sign. All occurrences of ’\\\\’ and ’=’ characters must be escaped with ’\\\\’ if *−graft−points* has been specified.

**−hide** *glob*

Hide *glob* from being seen on the ISO-9660 or Rock Ridge directory. *glob* is a shell wild-card-style pattern that must match any part of the filename or path. Multiple globs may be hidden. If *glob* matches a directory, then the contents of that directory will be hidden. In order to match a directory name, make sure the pathname does not include a trailing ’/’ character. All the hidden files will still be written to the output CD image file. Should be used with the **−hide−joliet** option. See README.hide for more details.

**−hide−list** *file*

A file containing a list of *globs* to be hidden as above.

**−hidden** *glob*

Add the hidden (existence) ISO-9660 directory attribute for *glob*. This attribute will prevent *glob* from being listed on DOS based systems if the /A flag is not used for the listing. *glob* is a shell wild-card-style pattern that must match any part of the filename or path. In order to match a directory name, make sure the pathname does not include a trailing ’/’ character. Multiple globs may be hidden.

**−hidden−list** *file*

A file containing a list of *globs* to get the hidden attribute as above.

**−hide−joliet** *glob*

Hide *glob* from being seen on the Joliet directory. *glob* is a shell wild-card-style pattern that must match any part of the filename or path. Multiple globs may be hidden. If *glob* matches a directory, then the contents of that directory will be hidden. In order to match a directory name, make sure the pathname does not include a trailing ’/’ character. All the hidden files will still be written to the output CD image file. Should be used with the **−hide** option. See README.hide for more details.

**−hide−joliet−list** *file*

A file containing a list of *globs* to be hidden as above.

**−hide−joliet−trans−tbl**

Hide the **TRANS.TBL** files from the Joliet tree. These files usually don’t make sense in the Joliet World as they list the real name and the ISO-9660 name which may both be different from the Joliet name.

**−hide−rr−moved**

Rename the directory **RR\_MOVED** to **.rr\_moved** in the Rock Ridge tree. This option has been introduced when **mkisofs** was not able to hide the directory in the Rock Ridge tree. This version of **mkisofs** always automatically hides the **RR\_MOVED** directory in the Rock Ridge tree. If you need to have no **RR\_MOVED** directory at all (even in the ISO-9660 tree), you should use the **−D** option. Note that in case that the **−D** option has been specified, the resulting filesystem is not ISO-9660 level-1 compliant and will not be readable on MS-DOS. See also **NOTES** section for more information on the **RR\_MOVED** directory.

**−hide−udf** *glob*

Hide *glob* from being seen on the UDF directory. *glob* is a shell wild-card-style pattern that must match any part of the filename or path. Multiple globs may be hidden. If *glob* matches a directory, then the contents of that directory will be hidden. In order to match a directory name, make sure the pathname does not include a trailing ’/’ character. All the hidden files will still be written to the output CD image file. Should be used with the **−hide** option. See README.hide for more details.

**−hide−udf−list** *file*

A file containing a list of *globs* to be hidden as above.

**−input−charset** *charset*

Set up the input charset that defines the characters used in local file names. To get a list of valid charset names, call **mkisofs −input−charset help.** To get a 1:1 mapping, you may use **default** as charset name. If the input charset has not been set up from the locale in the environment, the default initial values are *cp437* on DOS based systems and *iso8859-1* on all other systems. See **CHARACTER SETS** section below for more details.

If **−input−charset** has not been specified, it will be set up from the locale in the environment. If you like to disable this automatic setup, use the empty string as locale name.

**−output−charset** *charset*

Set up the output charset that defines the characters that will be used in Rock Ridge file names. Defaults to the input charset. See **CHARACTER SETS** section below for more details.

**−iso−level** *level*

Set the ISO-9660 conformance level. Valid numbers are 1..3 and 4.

With level 1, files may only consist of one section and filenames are restricted to 8.3 characters.

With level 2, files may only consist of one section.

With level 3, no restrictions (other than ISO-9660:1988) do apply. Starting with this level, mkisofs also allows files to be larger than 4 GB by implementing ISO-9660 multi-extent files.

With all ISO-9660 levels from 1..3, all filenames are restricted to upper case letters, numbers and the underscore (\_). The maximum filename length is restricted to 31 characters, the directory nesting level is restricted to 8 and the maximum path length is limited to 255 characters.

Level 4 officially does not exists but **mkisofs** maps it to ISO-9660:1999 which is ISO-9660 version 2.

With level 4, an enhanced volume descriptor with version number and file structure version number set to 2 is emitted. There may be more than 8 levels of directory nesting, there is no need for a file to contain a dot and the dot has no more special meaning, file names do not have version numbers, the maximum length for files and directory is raised to 207. If Rock Ridge is used, the maximum ISO-9660 name length is reduced to 197.

When creating Version 2 images, **mkisofs** emits an enhanced volume descriptor which looks similar to a primary volume descriptor but is slightly different. Be careful not to use broken software to make ISO-9660 images bootable by assuming a second PVD copy and patching this putative PVD copy into an El Torito VD.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−J</strong></p></td>
<td></td>
<td><p>Generate Joliet directory records in addition to regular ISO-9660 file names. This is primarily useful when the discs are to be used on Windows-NT or Windows-95 machines. The Joliet filenames are specified in Unicode and each path component can be up to 64 Unicode characters long. Note that Joliet is no standard - CD’s that use only Joliet extensions but no standard Rock Ridge extensions may usually only be used on Microsoft Win32 systems. Furthermore, the fact that the filenames are limited to 64 characters and the fact that Joliet uses the UTF-16 coding for Unicode characters causes interoperability problems.</p></td>
</tr>
</tbody>
</table>

**−joliet−long**

Allow Joliet filenames to be up to 103 Unicode characters. This breaks the Joliet specification - but appears to work. Use with caution. The number 103 is derived from: the maximum Directory Record Length (254), minus the length of Directory Record (33), minus CD-ROM XA System Use Extension Information (14), divided by the UTF-16 character size (2).

**−jcharset** *charset*

Same as using **−input−charset** *charset* and **−J** options. See **CHARACTER SETS** section below for more details.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−l</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−full−iso9660−filenames**

Allow full 31 character filenames. Normally the ISO-9660 filename will be in an 8.3 format which is compatible with MS-DOS, even though the ISO-9660 standard allows filenames of up to 31 characters. If you use this option, the disc may be difficult to use on a MS-DOS system, but this comes in handy on some other systems (such as the Amiga). Use with caution.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−L</strong></p></td>
<td></td>
<td><p>Outdated option reserved by POSIX.1-2001, use <strong>−allow−leading−dots</strong> instead. This option will get POSIX.1-2001 semantics with mkisofs-3.02.</p></td>
</tr>
</tbody>
</table>

**−log−file** *log\_file*

Redirect all error, warning and informational messages to *log\_file* instead of the standard error.

**−long−rr−time**

Use the long ISO-9660 time format for the file time stamps used in Rock Ridge. This time format allows to represent year 0 .. year 9999 with a granularity of 10ms.

The short ISO-9660 time format only allows to represent year 1900 .. year 2155 with a granularity of 1s.

**−m** *glob*

Exclude *glob* from being written to CDROM. *glob* is a shell wild-card-style pattern that must match part of the filename (not the path as with option **−x**). Technically *glob* is matched against the *d-&gt;d\_name* part of the directory entry. Multiple globs may be excluded. Example:

mkisofs −o rom −m ’\*.o’ −m core −m foobar

would exclude all files ending in ".o", called "core" or "foobar" to be copied to CDROM. Note that if you had a directory called "foobar" it too (and of course all its descendants) would be excluded.

NOTE: The **−m** and **−x** option description should both be updated, they are wrong. Both now work identical and use filename globbing. A file is excluded if either the last component matches or the whole path matches.

**−exclude−list** *file*

A file containing a list of *globs* to be excluded as above.

**−max−iso9660−filenames**

Allow 37 chars in ISO-9660 filenames. This option forces the **−N** option as the extra name space is taken from the space reserved for ISO-9660 version numbers.
This violates the ISO-9660 standard, but it happens to work on many systems. Although a conforming application needs to provide a buffer space of at least 37 characters, disks created with this option may cause a buffer overflow in the reading operating system. Use with extreme care.

**−M** *path*

or

**−M** *device*

or

**−dev** *device*

Specifies path to existing ISO-9660 image to be merged. The alternate form takes a SCSI device specifier that uses the same syntax as the **dev=** parameter of **cdrecord.** The output of **mkisofs** will be a new session which should get written to the end of the image specified in −M. Typically this requires multi-session capability for the recorder and cdrom drive that you are attempting to write this image to. This option may only be used in conjunction with the **−C** option.

**−modification−date** *date-spec*

Set the **modification date** in the primary volume descriptor (PVD) to a value different from the current time. This allows e.g. to set up an intentional UUID for **grub**.

The format of *date-spec* is:

*yyyy*\[*mm*\[*dd*\[*hh*\[*mm*\[*ss*\]\]\]\]\]\[.*hh*\]\[+-*ghgm*\]

The fields are **year**, **month**, **day of month**, **hour**, **minute**, **second**, **hundreds of a second**, **GMT offset in hours and minutes**. The time is interpreted as local time.

Year and the GMT offset are four digit fields, all other fields take two digits. The GMT offset may be between -12 and +13 hours in 15 minute steps. Locations east to Greenwich have positive values. The value is the sum of the time zone offset and the effects from daylight saving time. Omited values are replaced by the minimal possible values. If the GMT offset is omited, it is computed from the local time value that has been supplied.

Between year and month as well as between month and day of month, a separator chosen from ’/’ and ’-’ may appear. In this case, the year may be a two digit number with values 69..99 representing 1969..1999 and values 00..68 representing 2000..2068. Between date and time spec, an optional space is permitted. Between hours and minutes as well as between minutes and seconds, an optional ’:’ separator is permitted. This allows **mkisofs** to parse the popular POSIX date format created by:

**date "+%Y-%m-%d %H:%M:%S %z"**

Note that the possible range for *date-spec* for 32 bit programs is limited to values up to 2038 Jan 19 04:14:07 GMT.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−N</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−omit−version−number**

Omit version numbers from ISO-9660 file names.
This violates the ISO-9660 standard, but no one really uses the version numbers anyway. Use with caution.

**−new−dir−mode** *mode*

Mode to use when creating new directories in the iso fs image. The default mode in the absence of a **−dir−mode** option is 0555.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−nobak</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−no−bak**

Do not include backup files files on the ISO-9660 filesystem. If the **−no−bak** option is specified, files that contain the characters ’~’ or ’\#’ or end in ’.bak’ will not be included (these are typically backup files for editors under UNIX).

**−no−limit−pathtables**

A ISO-9660 filesystem contains path tables that contain a list of directories. This list may contain many directories but only 65535 of them may be parent directories. When **−no−limit−pathtables** is in use, further parent directories will be folded to the root directory and the resulting filesystem will no longer be usable on **DOS**.

**−no−long−rr−time**

Use the short ISO-9660 time format for the file time stamps used in Rock Ridge. This time format allows to represent year 1990 .. year 2155 with a granularity of one second.

**−force−rr**

Do not use the automatic Rock Ridge attributes recognition for previous sessions. This helps to show rotten ISO-9660 extension records as e.g. created by NERO burning ROM.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−no−rr</strong></p></td>
<td></td>
<td><p>Do not use the Rock Ridge attributes from previous sessions. This may help to avoid getting into trouble when <strong>mkisofs</strong> finds illegal Rock Ridge signatures on an old session.</p></td>
</tr>
</tbody>
</table>

**−no−split−symlink−components**

Don’t split the SL components, but begin a new Continuation Area (CE) instead. This may waste some space, but the SunOS 4.1.4 cdrom driver has a bug in reading split SL components (link\_size = component\_size instead of link\_size += component\_size).

Note that this option has been introduced by Eric Youngdale in 1997. It is questionable whether it makes sense at all. When it has been introduced, **mkisofs** did have a serious bug that did create defective CE signatures if a symlink contained ‘/../’. This CE signature bug in **mkisofs** has been fixed in May 2003.

**−no−split−symlink−fields**

Don’t split the SL fields, but begin a new Continuation Area (CE) instead. This may waste some space, but the SunOS 4.1.4 and Solaris 2.5.1 cdrom driver have a bug in reading split SL fields (a ‘/’ can be dropped).

Note that this option has been introduced by Eric Youngdale in 1997. It is questionable whether it makes sense at all. When it has been introduced, **mkisofs** did have a serious bug that did create defective CE signatures if a symlink contained ‘/../’. This CE signature bug in **mkisofs** has been fixed in May 2003.

**−o** *filename*

is the name of the file to which the ISO-9660 filesystem image should be written. This can be a disk file, a tape drive, or it can correspond directly to the device name of the optical disc writer. If not specified, stdout is used. Note that the output can also be a block special device for a regular disk drive, in which case the disk partition can be mounted and examined to ensure that the premastering was done correctly.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−pad</strong></p></td>
<td></td>
<td><p>Pad the end of the whole image by 150 sectors (300 kB). If the option <strong>−B</strong> is used, then there is a padding at the end of the ISO-9660 partition and before the beginning of the boot partitions. The size of this padding is chosen to make the first boot partition start on a sector number that is a multiple of 16.</p></td>
</tr>
</tbody>
</table>

The padding is needed as many operating systems (e.g. Linux) implement read ahead bugs in their filesystem I/O. These bugs result in read errors on one or more files that are located at the end of a track. They are usually present when the CD is written in Track at Once mode or when the disk is written as mixed mode CD where an audio track follows the data track.

To avoid problems with I/O error on the last file on the filesystem, the **−pad** option has been made the default.

**−no−pad**

Do not Pad the end by 150 sectors (300 kB) and do not make the the boot partitions start on a multiple of 16 sectors.

**−path−list** *file*

A file containing a list of *pathspec* directories and filenames to be added to the ISO-9660 filesystem. This list of pathspecs are processed after any that appear on the command line. If the argument is *−*, then the list is read from the standard input.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−P</strong></p></td>
<td></td>
<td><p>Outdated option reserved by POSIX.1-2001, use <strong>−publisher</strong> instead. This option will get POSIX.1-2001 semantics with mkisofs-3.02.</p></td>
</tr>
</tbody>
</table>

**−publisher** *publisher\_id*

Specifies a text string that will be written into the volume header. This should describe the publisher of the CDROM, usually with a mailing address and phone number. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. This parameter can also be set in the file **.mkisofsrc** with PUBL=. If specified in both places, the command line version is used.

**−p** *preparer\_id* **
−preparer** *preparer\_id*

Specifies a text string that will be written into the volume header. This should describe the preparer of the CDROM, usually with a mailing address and phone number. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. This parameter can also be set in the file **.mkisofsrc** with PREP=. If specified in both places, the command line version is used.

**−posix−H**

Follow all symbolic links encountered on command line when generating the filesystem.

**−posix−L**

Follow all symbolic links when generating the filesystem. When this option is not in use, symbolic links will be entered using Rock Ridge if enabled, otherwise the file will be ignored.

**−posix−P**

Do not follow symbolic links when generating the filesystem (this is the default). If **−posix−P** is specified after **−posix−H** or **−posix−L**, the effect of these options will be reset.

**−print−size**

Print estimated filesystem size in multiples of the sector size (2048 bytes) and exit. This option is needed for Disk At Once mode and with some CD-R drives when piping directly into **cdrecord.** In this case it is needed to know the size of the filesystem before the actual CD-creation is done. The option −print−size allows to get this size from a "dry-run" before the CD is actually written. Old versions of **mkisofs** did write this information (among other information) to *stderr*. As this turns out to be hard to parse, the number without any other information is now printed on **stdout** too. If you like to write a simple shell script, redirect **stderr** and catch the number from **stdout**. This may be done with:

**cdblocks=‘ mkisofs −print−size −quiet ... ‘**

**mkisofs ... | cdrecord ... tsize=${cdblocks}s -**

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−quiet</strong></p></td>
<td></td>
<td><p>This makes <strong>mkisofs</strong> even less verbose. No progress output will be provided.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−R</strong></p></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>−rock</strong></p></td>
<td></td>
<td><p>Generate SUSP and RR records using the Rock Ridge protocol to further describe the files on the ISO-9660 filesystem. The Rock Ridge protocol is needed in order to add POSIX like file meta data like permissions, extended time stamps, user/group is’d, link counts, inode numbers and symbolic links. The Rock Ridge protocol allows to archive hierarchy trees with unlimited depth.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−r</strong></p></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

**−rational−rock**

This is like the −R option, but file ownership and modes are set to more useful values. The uid and gid are set to zero, because they are usually only useful on the author’s system, and not useful to the client. All the file read bits are set true, so that files and directories are globally readable on the client. If any execute bit is set for a file, set all of the execute bits, so that executables are globally executable on the client. If any search bit is set for a directory, set all of the search bits, so that directories are globally searchable on the client. All write bits are cleared, because the CD-Rom will be mounted read-only in any case. If any of the special mode bits are set, clear them, because file locks are not useful on a read-only file system, and set-id bits are not desirable for uid 0 or gid 0. When used on Win32, the execute bit is set on *all* files. This is a result of the lack of file permissions on Win32 and the Cygwin POSIX emulation layer. See also −uid −gid, −dir−mode, −file−mode and −new−dir−mode.

**−relaxed−filenames**

The option **−relaxed−filenames** allows ISO-9660 filenames to include digits, upper case characters and all other 7 bit ASCII characters (resp. anything except lowercase characters).
This violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

**−root** *dir*

Moves all files and directories into *dir* in the image. This is essentially the same as using **-graft-points** and adding *dir* in front of every pathspec, but is easier to use.

*dir* may actually be several levels deep. It is created with the same permissions as other graft points.

**−rrip110**

Create ISO-9660 file system images that follow the old Rrip Version-1.10 standard from 1993. This option may be needed if you know of systems that do not implement the Rrip protocol correctly and like the file system to be read by such a system. Currently no such system is known.

If a file system has been created with **−rrip110**, the Rock Ridge attributes do not include inode number information.

**−rrip112**

Create ISO-9660 file system images that follow the new Rrip Version-1.12 standard from 1994, this is the default.

**−old-root** *dir*

This option is necessary when writing a multisession image and the previous (or even older) session was written with **-root** *dir.* Using a directory name not found in the previous session causes **mkisofs** to abort with an error.

Without this option, **mkisofs** would not be able to find unmodified files and would be forced to write their data into the image once more.

**−root** and **−old-root** are meant to be used together to do incremental backups. The initial session would e.g. use: **mkisofs −root backup\_1** *dirs.* The next incremental backup with **mkisofs −root backup\_2 −old-root backup\_1** *dirs.* would take another snapshot of these directories. The first snapshot would be found in **backup\_1**, the second one in **backup\_2**, but only modified or new files need to be written into the second session.

Without these options, new files would be added and old ones would be preserved. But old ones would be overwritten if the file was modified. Recovering the files by copying the whole directory back from CD would also restore files that were deleted intentionally. Accessing several older versions of a file requires support by the operating system to choose which sessions are to be mounted.

**−short−rr−time**

Use the short ISO-9660 time format for the file time stamps used in Rock Ridge. This time format allows to represent year 1990 .. year 2155 with a granularity of one second.

**−s** *sector type* **
−sectype** *sector type*

Set the *sector type* to be used for the output file with the ISO-9660 filesystem. The *sector type* may be one of:

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>data</strong></p></td>
<td></td>
<td><p>This is the default. It results in standard CD-ROM data sectors with 2048 bytes per sector.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>xa1</strong></p></td>
<td></td>
<td><p>This sets the sector type to CD-ROM XA mode 1 with 2056 bytes per sector. This sector type is the official sector type for multi-session CDs, it should be used together with the <strong>−XA</strong> option of mkisofs. It is required to write Kodak Photo CDs and Kodak Picture CDs. Use the <strong>−xa1</strong> option from <strong>cdrecord</strong> to tell <strong>cdrecord</strong> to write CD-ROM XA mode 1 sectors. Do not use for DVD or BluRay media.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>raw</strong></p></td>
<td></td>
<td><p>This sets the sector type to raw audio sectors with 2352 bytes per sector. This is reserved for future enhancements. Do not use for DVD or BluRay media.</p></td>
</tr>
</tbody>
</table>

**−sort** *sort file*

Sort file locations on the media. Sorting is controlled by a file that contains pairs of filenames and sorting offset weighting. If the weighting is higher, the file will be located closer to the beginning of the media, if the weighting is lower, the file will be located closer to the end of the media. There must be only one space or tabs character between the filename and the weight and the weight must be the last characters on a line. The filename is taken to include all the characters up to, but not including the last space or tab character on a line. This is to allow for space characters to be in, or at the end of a filename. This option does **not** sort the order of the file names that appear in the ISO-9660 directory. It sorts the order in which the file data is written to the CD image - which may be useful in order to optimize the data layout on a CD. See README.sort for more details.

**−sparc−boot** *img\_sun4,img\_sun4c,img\_sun4m,img\_sun4d,img\_sun4e*

See **−B** option above.

**−sparc−label** *label*

Set the Sun disk label name for the Sun disk label that is created with the **−sparc-boot** option.

**−split−output**

Split the output image into several files of approximately 1 GB. This helps to create DVD sized ISO-9660 images on operating systems without large file support. Cdrecord will concatenate more than one file into a single track if writing to a DVD. To make **−split−output** work, the **−o** *filename* option must be specified. The resulting output images will be named: *filename\_00*,*filename\_01,*filename\_02*...*

**−stream−media−size** *\#*

Select streaming operation and set the media size to \# sectors. This allows you to pipe the output of the tar program into mkisofs and to create a ISO-9660 filesystem without the need of an intermediate tar archive file. If this option has been specified, **mkisofs** reads from **stdin** and creates a file with the name **STREAM.IMG**. The maximum size of the file (with padding) is 200 sectors less than the specified media size. If **−no−pad** has been specified, the file size is 50 sectors less than the specified media size. If the file is smaller, then mkisofs will write padding. This may take a while.

The option **−stream−media−size** creates simple ISO-9660 filesystems only and may not used together with multi-session or hybrid filesystem options.

**−stream−file−name** *name*

Set the file name used with **−stream−media−size** *\#* to a value different from **STREAM.IMG**. If this option is used, the filesystem is created as if **−iso−level** 4 has been specified.

**−sunx86−boot** *UFS-img,,,AUX1-img*

Specifies a comma separated list of filesystem images that are needed to make a bootable CD for Solaris x86 systems.

Note that partition 1 is used for the ISO-9660 image and that partition 2 is the whole disk, so partition 1 and 2 may not be used by external partition data. The first image file is mapped to partition 0. There may be empty fields in the comma separated list, and list entries for partition 1 and 2 must be empty. The maximum number of supported partitions is 8 (although the Solaris x86 partition table could support up to 16 partitions), so it is impossible to specify more than 6 partition images. This option is required to make a bootable CD for Solaris x86 systems.

If the **−sunx86−boot** option has been specified, the first sector of the resulting image will contain a PC fdisk label with a Solaris type 0x82 fdisk partition that starts at offset 512 and spans the whole CD. In addition, for the Solaris type 0x82 fdisk partition, there is a SVr4 disk label at offset 1024 in the first sector of the CD. This disk label specifies slice 0 for the first (usually UFS type) filesystem image that is used to boot the PC and slice 1 for the ISO-9660 image. Slice 2 spans the whole CD slice 3 ... slice 7 may be used for additional filesystem images that have been specified with this option.

A Solaris x86 boot CD uses a 1024 byte sized primary boot that uses the **El-Torito no-emulation** boot mode and a secondary generic boot that is in CD sectors 1..15. For this reason, both **-b** *bootimage* **-no−emul−boot** and **−G** *genboot* must be specified.

**−sunx86−label** *label*

Set the SVr4 disk label name for the SVr4 disk label that is created with the **−sunx86-boot** option.

**−sysid** *ID*

Specifies the system ID. There is space on the disc for 32 characters of information. This parameter can also be set in the file **.mkisofsrc** with SYSI=system\_id. If specified in both places, the command line version is used.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−T</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−translation−table**

Generate a file TRANS.TBL in each directory on the CDROM, which can be used on non-Rock Ridge capable systems to help establish the correct file names. There is also information present in the file that indicates the major and minor numbers for block and character devices, and each symlink has the name of the link file given.

**−table−name** *TABLE\_NAME*

Alternative translation table file name (see above). Implies the **−T** option. If you are creating a multi-session image you must use the same name as in the previous session.

**−ucs−level** *level*

Set Unicode conformance level in the Joliet SVD. The default level is 3. It may be set to 1..3 using this option.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−UDF</strong></p></td>
<td></td>
<td><p>Include a <strong>UDF</strong> hybrid in the generated filesystem image. As <strong>mkisofs</strong> always creates a ISO-9660 filesystem, it is not possible to create UDF only images. Note that <strong>UDF</strong> wastes the space from sector ~20 to sector 256 at the beginning of the disk in addition to the space needed for real <strong>UDF</strong> data structures.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−udf</strong></p></td>
<td></td>
<td><p>Rationalized UDF with user and group set to 0 and with simplified permissions. See <strong>−r</strong> option for more information.</p></td>
</tr>
</tbody>
</table>

**−udf−symlinks**

Support symlinks in **UDF** filesystems. This is the default.

**−no−udf−symlinks**

Do not support symlinks in **UDF** filesystems.

**−uid** *uid*

Overrides the uid read from the source files to the value of *uid*. Specifying this option automatically enables Rock Ridge extensions.

**−use−fileversion**

The option **−use−fileversion** allows mkisofs to use file version numbers from the filesystem. If the option is not specified, **mkisofs** creates a version number of 1 for all files. File versions are strings in the range *;1* to *;32767* This option is the default on VMS.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−U</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−untranslated−filenames**

Allows "Untranslated" filenames, completely violating the ISO-9660 standards described above. Forces on the −d, −l, −N, −allow−leading−dots, −relaxed−filenames, −allow−lowercase, −allow−multidot and −no−iso−translate flags. It allows more than one ’.’ character in the filename, as well as mixed case filenames. This is useful on HP-UX system, where the built-in CDFS filesystem does not recognize ANY extensions. Use with extreme caution.

**−no−iso−translate**

Do not translate the characters ’\#’ and ’~’ which are invalid for ISO-9660 filenames. These characters are though invalid often used by Microsoft systems.
This violates the ISO-9660 standard, but it happens to work on many systems. Use with caution.

**−V** *volid*

Specifies the volume ID (volume name or label) to be written into the master block. There is space on the disc for 32 characters of information. This parameter can also be set in the file **.mkisofsrc** with VOLI=id. If specified in both places, the command line version is used. Note that if you assign a volume ID, this is the name that will be used as the mount point used by the Solaris volume management system and the name that is assigned to the disc on a Microsoft Win32 or Apple Mac platform.

**−volset** *ID*

Specifies the volset ID. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. This parameter can also be set in the file **.mkisofsrc** with VOLS=volset\_id. If specified in both places, the command line version is used.

**−volset−size** *\#*

Sets the volume set size to \#. The volume set size is the number of CD’s that are in a CD volume set. A volume set is a collection of one or more volumes, on which a set of files is recorded.

Volume Sets are not intended to be used to create a set numbered CD’s that are part of e.g. a Operation System installation set of CD’s. Volume Sets are rather used to record a big directory tree that would not fit on a single volume. Each volume of a Volume Set contains a description of all the directories and files that are recorded on the volumes where the sequence numbers are less than, or equal to, the assigned Volume Set Size of the current volume.

**Mkisofs** currently does not support a **−volset−size** that is larger than 1.

The option **−volset−size** must be specified before **−volset−seqno** on each command line.

**−volset−seqno** *\#*

Sets the volume set sequence number to \#. The volume set sequence number is the index number of the current CD in a CD set. The option **−volset−size** must be specified before **−volset−seqno** on each command line.

<table>
<colgroup>
<col width="33%" />
<col width="33%" />
<col width="33%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−v</strong></p></td>
<td></td>
</tr>
</tbody>
</table>

**−verbose**

Verbose execution. If given twice on the command line, extra debug information will be printed.

**−x** *path*

Exclude *path* from being written to CDROM. *path* must be the complete pathname that results from concatenating the pathname given as command line argument and the path relative to this directory. Multiple paths may be excluded. Example:

mkisofs −o cd −x /local/dir1 −x /local/dir2 /local

NOTE: The **−m** and **−x** option description should both be updated, they are wrong. Both now work identical and use filename globbing. A file is excluded if either the last component matches or the whole path matches.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−XA</strong></p></td>
<td></td>
<td><p>Generate XA iso-directory attributes with original owner and mode information. This option is required to create conforming multi session CDs as used by the Kodak Photo CD and the Kodak Picture CD. A conforming XA CD uses CD-ROM XA mode 1 sectors, see the <strong>−sector</strong> <em>xa2</em> option for more information.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−xa</strong></p></td>
<td></td>
<td><p>Generate XA iso-directory attributes with rationalized owner and mode information. User ID and group ID are set to 0. See <strong>−XA</strong> for more information.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>−z</strong></p></td>
<td></td>
<td><p>Generate special RRIP records for transparently compressed files. This is only of use and interest for hosts that support transparent decompression, such as Linux 2.4.14 or later. You must specify the <strong>−R</strong> or <strong>−r</strong> options to enable RockRidge, and generate compressed files using the <strong>mkzftree</strong> utility before running <strong>mkisofs</strong>. Note that transparent compression is a nonstandard Rock Ridge extension. The resulting disks are only transparently readable if used on Linux. On other operating systems you will need to call <strong>mkzftree</strong> by hand to decompress the files.</p></td>
</tr>
</tbody>
</table>

HFS OPTIONS []()
----------------

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−hfs</strong></p></td>
<td></td>
<td><p>Create an ISO-9660/HFS hybrid CD. This option should be used in conjunction with the <strong>−map</strong>, <strong>−magic</strong> and/or the various <em>double dash</em> options given below.</p></td>
</tr>
</tbody>
</table>

**−no−hfs**

Do not create an ISO-9660/HFS hybrid CD even though other options may imply to do so.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−apple</strong></p></td>
<td></td>
<td><p>Create an ISO-9660 CD with Apple’s extensions. Similar to the <strong>−hfs</strong> option, except that the Apple Extensions to ISO-9660 are added instead of creating an HFS hybrid volume. Former <strong>mkisofs</strong> versions did include Rock Ridge attributes by default if <strong>−apple</strong> was specified. This versions of <strong>mkisofs</strong> does not do this anymore. If you like to have Rock Ridge attributes, you need to specify this separately.</p></td>
</tr>
</tbody>
</table>

**−map** *mapping\_file*

Use the *mapping\_file* to set the CREATOR and TYPE information for a file based on the filename’s extension. A filename is mapped only if it is not one of the know Apple/Unix file formats. See the **HFS CREATOR/TYPE** section below.

**−magic** *magic\_file*

The CREATOR and TYPE information is set by using a file’s *magic number* (usually the first few bytes of a file). The *magic\_file* is only used if a file is not one of the known Apple/Unix file formats, or the filename extension has not been mapped using the **−map** option. See the **HFS CREATOR/TYPE** section below for more details.

**−hfs−creator** *CREATOR*

Set the default CREATOR for all files. Must be exactly 4 characters. See the **HFS CREATOR/TYPE** section below for more details.

**−hfs−type** *TYPE*

Set the default TYPE for all files. Must be exactly 4 characters. See the **HFS CREATOR/TYPE** section below for more details.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−probe</strong></p></td>
<td></td>
<td><p>Search the contents of files for all the known Apple/Unix file formats. See the <strong>HFS MACINTOSH FILE FORMATS</strong> section below for more about these formats. However, the only way to check for <em>MacBinary</em> and <em>AppleSingle</em> files is to open and read them. Therefore this option <em>may</em> increase processing time. It is better to use one or more <em>double dash</em> options given below if the Apple/Unix formats in use are known.</p></td>
</tr>
</tbody>
</table>

**−no−desktop**

Do not create (empty) Desktop files. New HFS Desktop files will be created when the CD is used on a Macintosh (and stored in the System Folder). By default, empty Desktop files are added to the HFS volume.

**−mac−name**

Use the HFS filename as the starting point for the ISO-9660, Joliet and Rock Ridge file names. See the **HFS MACINTOSH FILE NAMES** section below for more information.

**−boot−hfs−file** *driver\_file*

Installs the *driver\_file* that *may* make the CD bootable on a Macintosh. See the **HFS BOOT DRIVER** section below. (Alpha).

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−part</strong></p></td>
<td></td>
<td><p>Generate an HFS partition table. By default, no partition table is generated, but some older Macintosh CDROM drivers need an HFS partition table on the CDROM to be able to recognize a hybrid CDROM.</p></td>
</tr>
</tbody>
</table>

**−auto** *AutoStart\_file*

Make the HFS CD use the QuickTime 2.0 Autostart feature to launch an application or document. The given filename must be the name of a document or application located at the top level of the CD. The filename must be less than 12 characters. (Alpha).

**−cluster−size** *size*

Set the size in bytes of the cluster or allocation units of PC Exchange files. Implies the **−−exchange** option. See the **HFS MACINTOSH FILE FORMATS** section below.

**−hide−hfs** *glob*

Hide *glob* from the HFS volume. The file or directory will still exist in the ISO-9660 and/or Joliet directory. *glob* is a shell wild-card-style pattern that must match any part of the filename Multiple globs may be excluded. Example:

mkisofs −o rom −hfs −hide−hfs ’\*.o’ −hide−hfs foobar

would exclude all files ending in ".o" or called "foobar" from the HFS volume. Note that if you had a directory called "foobar" it too (and of course all its descendants) would be excluded. The *glob* can also be a path name relative to the source directories given on the command line. Example:

mkisofs −o rom −hfs −hide−hfs src/html src

would exclude just the file or directory called "html" from the "src" directory. Any other file or directory called "html" in the tree will not be excluded. Should be used with the **−hide** and/or **−hide−joliet** options. In order to match a directory name, make sure the pathname does not include a trailing ’/’ character. See README.hide for more details.

**−hide−hfs−list** *file*

A file containing a list of *globs* to be hidden as above.

**−hfs−volid** *hfs\_volid*

Volume name for the HFS partition. This is the name that is assigned to the disc on a Macintosh and replaces the *volid* used with the **−V** option

**−icon−position**

Use the icon position information, if it exists, from the Apple/Unix file. The icons will appear in the same position as they would on a Macintosh desktop. Folder location and size on screen, its scroll positions, folder View (view as Icons, Small Icons, etc.) are also preserved. This option may become set by default in the future. (Alpha).

**−root−info** *file*

Set the location, size on screen, scroll positions, folder View etc. for the root folder of an HFS volume. See README.rootinfo for more information. (Alpha)

**−prep−boot** *FILE*

PReP boot image file. Up to 4 are allowed. See README.prep\_boot (Alpha)

**−chrp-t**

Create a CHRP boot in boot partition 1. See **−prep−boot** for further information.

**−input−hfs−charset** *charset*

Input charset that defines the characters used in HFS file names when used with the *−mac−name* option. The default charset is cp10000 (Mac Roman) *cp10000* (Mac Roman) See **CHARACTER SETS** and **HFS MACINTOSH FILE NAMES** sections below for more details.

**−output−hfs−charset** *charset*

Output charset that defines the characters that will be used in the HFS file names. Defaults to the input charset. See **CHARACTER SETS** section below for more details.

**−hfs−unlock**

By default, **mkisofs** will create an HFS volume that is *locked*. This option leaves the volume unlocked so that other applications (e.g. hfsutils) can modify the volume. See the **HFS PROBLEMS/LIMITATIONS** section below for warnings about using this option.

**−hfs−bless** *folder\_name*

"Bless" the given directory (folder). This is usually the **System Folder** and is used in creating HFS bootable CDs. The name of the directory must be the whole path name as **mkisofs** sees it. e.g. if the given pathspec is ./cddata and the required folder is called System Folder, then the whole path name is "./cddata/System Folder" (remember to use quotes if the name contains spaces).

**−hfs−parms** *PARAMETERS*

Override certain parameters used to create the HFS file system. Unlikely to be used in normal circumstances. See the libhfs\_iso/hybrid.h source file for details.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−−cap</strong></p></td>
<td></td>
<td><p>Look for AUFS CAP Macintosh files. Search for CAP Apple/Unix file formats only. Searching for the other possible Apple/Unix file formats is disabled, unless other <em>double dash</em> options are given.</p></td>
</tr>
</tbody>
</table>

**−−netatalk**

Look for NETATALK Macintosh files

**−−double**

Look for AppleDouble Macintosh files

**−−ethershare**

Look for Helios EtherShare Macintosh files

**−−ushare**

Look for IPT UShare Macintosh files

**−−exchange**

Look for PC Exchange Macintosh files

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−−sgi</strong></p></td>
<td></td>
<td><p>Look for SGI Macintosh files</p></td>
<td></td>
</tr>
</tbody>
</table>

**−−xinet**

Look for XINET Macintosh files

**−−macbin**

Look for MacBinary Macintosh files

**−−single**

Look for AppleSingle Macintosh files

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>−−dave</strong></p></td>
<td></td>
<td><p>Look for Thursby Software Systems DAVE Macintosh files</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>−−sfm</strong></p></td>
<td></td>
<td><p>Look for Microsoft’s Services for Macintosh files (NT only) (Alpha)</p></td>
</tr>
</tbody>
</table>

**−−osx−double**

Look for MacOS X AppleDouble Macintosh files

**−−osx−hfs**

Look for MacOS X HFS Macintosh files

CHARACTER SETS []()
-------------------

**mkisofs** processes file names in a POSIX compliant way as strings of 8-bit characters. To represent all codings for all languages, 8-bit characters are not sufficient. Unicode or **ISO-10646** define character codings that need at least 21 bits to represent all known languages. They may be represented with **UTF-32**, **UTF-16** or **UTF-8** coding. **UTF-32** uses a plain 32-bit coding but seems to be uncommon. **UCS-2** is used by Microsoft with Win32. This coding is similar to **UTF-16** with the disadvantage that it only supports a 16 bit subset (except when surrogates are used) of all codes and that 16-bit characters are not compliant with the POSIX filesystem interface.

Modern UNIX operating systems may use **UTF-8** coding for filenames. This coding allows to use the complete Unicode code set. Each 32-bit character is represented by one or more 8-bit characters. If a character is coded in **ISO-8859-1** (used in Central Europe and North America) it maps 1:1 to a **UTF-32** or **UTF-16** coded Unicode character. If a character is coded in **7-Bit ASCII** (used in USA and other countries with limited character set) it maps 1:1 to a **UTF-32**, **UTF-16** or **UTF-8** coded Unicode character. Character codes that cannot be represented as a single byte in UTF-8 (typically if the value is &gt; 0x7F) use escape sequences that map to more than one 8-bit character.

If all operating systems would use **UTF-8** coding, **mkisofs** would not need to recode characters in file names. Unfortunately, Apple uses completely nonstandard codings and Microsoft uses a Unicode coding that is not compatible with the POSIX filename interface.

For all non **UTF-8** coded operating systems, the actual character that each byte represents, depends on the *character set* or *codepage* (which is the name used by Microsoft) used by the local operating system in use - the characters in a character set will reflect the region or natural language used by the user.

Usually character codes 0x00-0x1f are control characters, codes 0x20-0x7f are the 7 bit ASCII characters and (on PC’s and Mac’s) 0x80-0xff are used for other characters. Unfortunately even this does not follow ISO standards that reserve the range 0x80-0x9f for control characters and only allow 0xa0-0xff for other characters.

As there is a lot more than 256 characters/symbols in use, only a small subset are represented in a character set. Therefore the same character code may represent a different character in different character sets. So a file name generated, say in central Europe, may not display the same character when viewed on a machine in, say eastern Europe.

To make matters more complicated, different operating systems use different character sets for the region or language. For example the character code for "small e with acute accent" may be character code 0x82 on a PC, code 0x8e on a Macintosh and code 0xe9 on a UNIX system. Note while the codings used on a PC or Mac are nonstandard, Unicode codes this character as 0x00000000e9 which is basically the same value as the value used by most UNIX systems.

As long as not all operating systems and applications will use the Unicode character set as the basis for file names in a unique way, it may be necessary to specify which character set your file names use in and which character set the file names should appear on the CD.

There are four options to specify the character sets you want to use:
−input−charset

Defines the local character set you are using on your host machine. Any character set conversions that take place will use this character set as the staring point. The default input character sets are *cp437* on DOS based systems and *iso8859-1* on all other systems.

If the *−J* option is given, then the Unicode equivalents of the input character set will be used in the Joliet directory. Using the *−jcharset* option is the same as using the *−input−charset* and *−J* options.

−output−charset

Defines the character set that will be used with for the Rock Ridge names on the CD. Defaults to the input character set. Only likely to be useful if used on a non-Unix platform. e.g. using **mkisofs** on a Microsoft Win32 machine to create Rock Ridge CDs. If you are using **mkisofs** on a Unix machine, it is likely that the output character set will be the same as the input character set.

−input−hfs−charset

Defines the HFS character set used for HFS file names decoded from any of the various Apple/Unix file formats. Only useful when used with *−mac−name* option. See the **HFS MACINTOSH FILE NAMES** for more information. Defaults to *cp10000* (Mac Roman).

−output−hfs−charset

Defines the HFS character set used to create HFS file names from the input character set in use. In most cases this will be from the character set given with the *−input−charset* option. Defaults to the input HFS character set.

The **default** character set is built into **mkisofs**. A number of further character sets are read in from the filesystem by *mkisofs* from a directory relatively to the install path. To get a listing, use **mkisofs −input−charset help.**

Additional character sets from **iconv**(1) may be used on systems, that support **iconv**(1). In this case, call **iconv −l** to get a list of valid character sets from this coding method. To force an **iconv**(1) based coding, use **iconv:***name* instead of *name* for the character set.

If using non **iconv**(1) based character sets, additional character sets can be read from file for any of the character set options by giving a filename as the argument to the options. A given character set will be read from a file whenever the supplied name contains a ’/’.

The format of the character set files is the same as the mapping files available from http://www.unicode.org/Public/MAPPINGS The format of these files is:

Column \#1 is the input byte code (in hex as 0xXX)
Column \#2 is the Unicode (in hex as 0xXXXX)
Rest of the line is ignored.

Any blank line, line without two (or more) columns in the above format or comments lines (starting with the \# character) are ignored without any warnings. Any missing input code is mapped to Unicode character 0x0000.

Note that there is no support for 16 bit UNICODE (UTF-16) or 32 bit UNICODE (UTF-32) coding because this coding is not POSIX compliant. There should be support for UTF-8 UNICODE coding which is compatible to POSIX filenames and supported by moder UNIX implementations such as Solaris.

A 1:1 character set mapping can be defined by using the keyword *default* as the argument to any of the character set options. This is the behaviour of older (v1.12) versions of **mkisofs**.

The ISO-9660 file names generated from the input filenames are not converted from the input character set. The ISO-9660 character set is a very limited subset of the ASCII characters, so any conversion would be pointless.

Any character that **mkisofs** can not convert will be replaced with a ’\_’ character.

HFS CREATOR/TYPE []()
---------------------

A Macintosh file has two properties associated with it which define which application created the file, the *CREATOR* and what data the file contains, the *TYPE*. Both are (exactly) 4 letter strings. Usually this allows a Macintosh user to double-click on a file and launch the correct application etc. The CREATOR and TYPE of a particular file can be found by using something like ResEdit (or similar) on a Macintosh.

The CREATOR and TYPE information is stored in all the various Apple/Unix encoded files. For other files it is possible to base the CREATOR and TYPE on the filename’s extension using a *mapping* file (the **−map** option) and/or using the *magic number* (usually a *signature* in the first few bytes) of a file (the **−magic** option). If both these options are given, then their order on the command line is important. If the **−map** option is given first, then a filename extension match is attempted before a magic number match. However, if the **−magic** option is given first, then a magic number match is attempted before a filename extension match.

If a mapping or magic file is not used, or no match is found then the default CREATOR and TYPE for all regular files can be set by using entries in the **.mkisofsrc** file or using the **−hfs−creator** and/or **−hfs−type** options, otherwise the default CREATOR and TYPE are ’unix’ and ’TEXT’.

The format of the *mapping* file is the same *afpfile* format as used by *aufs*. This file has five columns for the *extension*, *file translation*, *CREATOR*, *TYPE* and *Comment*. Lines starting with the ’\#’ character are comment lines and are ignored. An example file would be like:

![Image grohtml-21521.png](grohtml-21521.png)

Where:

The first column *EXTN* defines the Unix filename extension to be mapped. The default mapping for any filename extension that doesn’t match is defined with the "\*" character.

The *Xlate* column defines the type of text translation between the Unix and Macintosh file it is ignored by **mkisofs**, but is kept to be compatible with **aufs**(1). Although **mkisofs** does not alter the contents of a file, if a binary file has its TYPE set as ’TEXT’, it *may* be read incorrectly on a Macintosh. Therefore a better choice for the default TYPE may be ’????’

The *CREATOR* and *TYPE* keywords must be 4 characters long and enclosed in single quotes.

The comment field is enclosed in double quotes - it is ignored by **mkisofs**, but is kept to be compatible with **aufs**.

The format of the *magic* file is almost identical to the **magic**(4) file used by the Linux **file**(1) command - the routines for reading and decoding the *magic* file are based on the Linux **file**(1) command.

This file has four tab separated columns for the *byte offset*, *type*, *test* and *message*. Lines starting with the ’\#’ character are comment lines and are ignored. An example file would be like:

![Image grohtml-21522.png](grohtml-21522.png)

The format of the file is described in the **magic**(4) man page. The only difference here is that for each entry in the magic file, the *message* for the initial offset **must** be 4 characters for the CREATOR followed by 4 characters for the TYPE - white space is optional between them. Any other characters on this line are ignored. Continuation lines (starting with a ’&gt;’) are also ignored i.e. only the initial offset lines are used.

Using the **−magic** option may significantly increase processing time as each file has to opened and read to find its magic number.

In summary, for all files, the default CREATOR is ’unix’ and the default TYPE is ’TEXT’. These can be changed by using entries in the *.mkisofsrc* file or by using the **−hfs−creator** and/or **−hfs−type** options.

If the a file is in one of the known Apple/Unix formats (and the format has been selected), then the CREATOR and TYPE are taken from the values stored in the Apple/Unix file.

Other files can have their CREATOR and TYPE set from their file name extension (the **−map** option), or their magic number (the **−magic** option). If the default match is used in the *mapping* file, then these values override the default CREATOR and TYPE.

A full CREATOR/TYPE database can be found at http://www.angelfire.com/il/szekely/index.html

HFS MACINTOSH FILE FORMATS []()
-------------------------------

Macintosh files have two parts called the *Data* and *Resource* fork. Either may be empty. Unix (and many other OSs) can only cope with files having one part (or fork). To add to this, Macintosh files have a number of attributes associated with them - probably the most important are the TYPE and CREATOR. Again Unix has no concept of these types of attributes.

e.g. a Macintosh file may be a JPEG image where the image is stored in the Data fork and a desktop thumbnail stored in the Resource fork. It is usually the information in the data fork that is useful across platforms.

Therefore to store a Macintosh file on a Unix filesystem, a way has to be found to cope with the two forks and the extra attributes (which are referred to as the *finder info*). Unfortunately, it seems that every software package that stores Macintosh files on Unix has chosen a completely different storage method.

The Apple/Unix formats that *mkisofs* (partially) supports are:
CAP AUFS format

Data fork stored in a file. Resource fork in subdirectory .resource with same filename as data fork. Finder info in .finderinfo subdirectory with same filename.

AppleDouble/Netatalk

Data fork stored in a file. Resource fork stored in a file with same name prefixed with "%". Finder info also stored in same "%" file. Netatalk uses the same format, but the resource fork/finderinfo stored in subdirectory .AppleDouble with same name as data fork.

AppleSingle

Data structures similar to above, except both forks and finder info are stored in one file.

Helios EtherShare

Data fork stored in a file. Resource fork and finder info together in subdirectory .rsrc with same filename as data fork.

IPT UShare

Very similar to the EtherShare format, but the finder info is stored slightly differently.

MacBinary

Both forks and finder info stored in one file.

Apple PC Exchange

Used by Macintoshes to store Apple files on DOS (FAT) disks. Data fork stored in a file. Resource fork in subdirectory resource.frk (or RESOURCE.FRK). Finder info as one record in file finder.dat (or FINDER.DAT). Separate finder.dat for each data fork directory.

Note: *mkisofs* needs to know the native FAT cluster size of the disk that the PC Exchange files are on (or have been copied from). This size is given by the **−cluster−size** option. The cluster or allocation size can be found by using the DOS utility **CHKDSK**.

May not work with PC Exchange v2.2 or higher files (available with MacOS 8.1). DOS media containing PC Exchange files should be mounted as type **msdos** (not **vfat**) when using Linux.

SGI/XINET

Used by SGI machines when they mount HFS disks. Data fork stored in a file. Resource fork in subdirectory .HSResource with same name. Finder info as one record in file .HSancillary. Separate .HSancillary for each data fork directory.

Thursby Software Systems DAVE

Allows Macintoshes to store Apple files on SMB servers. Data fork stored in a file. Resource fork in subdirectory resource.frk. Uses the AppleDouble format to store resource fork.

Services for Macintosh

Format of files stored by NT Servers on NTFS filesystems. Data fork is stored as "filename". Resource fork stored as a NTFS *stream* called "filename:AFP\_Resource". The finder info is stored as a NTFS *stream* called "filename:Afp\_AfpInfo". These streams are normally invisible to the user.

Warning: mkisofs only partially supports the SFM format. If an HFS file or folder stored on the NT server contains an *illegal* NT character in its name, then NT converts these characters to *Private Use Unicode* characters. The characters are: " \* / &lt; &gt; ?  | also a space or period if it is the last character of the file name, character codes 0x01 to 0x1f (control characters) and Apple’ apple logo.

Unfortunately, these private Unicode characters are not readable by the mkisofs NT executable. Therefore any file or directory name containing these characters will be ignored - including the contents of any such directory.

MacOS X AppleDouble

When HFS/HFS+ files are copied or saved by MacOS X on to a non-HFS file system (e.g. UFS, NFS etc.), the files are stored in AppleDouble format. Data fork stored in a file. Resource fork stored in a file with same name prefixed with ".\_". Finder info also stored in same ".\_" file.

MacOS X HFS (Alpha)

Not really an Apple/Unix encoding, but actual HFS/HFS+ files on a MacOS X system. Data fork stored in a file. Resource fork stored in a pseudo file with the same name with the suffix ’/rsrc’. The finderinfo is only available via a MacOS X library call.

Notes: (also see README.macosx)

Only works when used on MacOS X.

If a file is found with a zero length resource fork and empty finderinfo, it is assumed not to have any Apple/Unix encoding - therefore a TYPE and CREATOR can be set using other methods.

*mkisofs* will attempt to set the CREATOR, TYPE, date and possibly other flags from the finder info. Additionally, if it exists, the Macintosh filename is set from the finder info, otherwise the Macintosh name is based on the Unix filename - see the **HFS MACINTOSH FILE NAMES** section below.

When using the **−apple** option, the TYPE and CREATOR are stored in the optional System Use or SUSP field in the ISO-9660 Directory Record - in much the same way as the Rock Ridge attributes are. In fact to make life easy, the Apple extensions are added at the beginning of the existing Rock Ridge attributes (i.e. to get the Apple extensions you get the Rock Ridge extensions as well).

The Apple extensions require the resource fork to be stored as an ISO-9660 *associated* file. This is just like any normal file stored in the ISO-9660 filesystem except that the associated file flag is set in the Directory Record (bit 2). This file has the same name as the data fork (the file seen by non-Apple machines). Associated files are normally ignored by other OSs

When using the **−hfs** option, the TYPE and CREATOR plus other finder info, are stored in a separate HFS directory, not visible on the ISO-9660 volume. The HFS directory references the same data and resource fork files described above.

In most cases, it is better to use the **−hfs** option instead of the **−apple** option, as the latter imposes the limited ISO-9660 characters allowed in filenames. However, the Apple extensions do give the advantage that the files are packed on the disk more efficiently and it may be possible to fit more files on a CD - important when the total size of the source files is approaching 650MB.

HFS MACINTOSH FILE NAMES []()
-----------------------------

Where possible, the HFS filename that is stored with an Apple/Unix file is used for the HFS part of the CD. However, not all the Apple/Unix encodings store the HFS filename with the finderinfo. In these cases, the Unix filename is used - with escaped special characters. Special characters include ’/’ and characters with codes over 127.

Aufs escapes these characters by using ":" followed by the character code as two hex digits. Netatalk and EtherShare have a similar scheme, but uses "%" instead of a ":".

If mkisofs can’t find an HFS filename, then it uses the Unix name, with any %xx or :xx characters (xx == two hex digits) converted to a single character code. If "xx" are not hex digits (\[0-9a-fA-F\]), then they are left alone - although any remaining ":" is converted to "%" as colon is the HFS directory separator. Care must be taken, as an ordinary Unix file with %xx or :xx will also be converted. e.g.

![Image grohtml-21523.png](grohtml-21523.png)

Although HFS filenames appear to support upper and lower case letters, the filesystem is case insensitive. i.e. the filenames "aBc" and "AbC" are the same. If a file is found in a directory with the same HFS name, then *mkisofs* will attempt, where possible, to make a unique name by adding ’\_’ characters to one of the filenames.

If an HFS filename exists for a file, then mkisofs can use this name as the starting point for the ISO-9660, Joliet and Rock Ridge filenames using the **−mac−name** option. Normal Unix files without an HFS name will still use their Unix name. e.g.

If a *MacBinary* (or *PC Exchange*) file is stored as *someimage.gif.bin* on the Unix filesystem, but contains a HFS file called *someimage.gif*, then this is the name that would appear on the HFS part of the CD. However, as mkisofs uses the Unix name as the starting point for the other names, then the ISO-9660 name generated will probably be *SOMEIMAG.BIN* and the Joliet/Rock Ridge would be *someimage.gif.bin*. Although the actual data (in this case) is a GIF image. This option will use the HFS filename as the starting point and the ISO-9660 name will probably be *SOMEIMAG.GIF* and the Joliet/Rock Ridge would be *someimage.gif*.

Using the **−mac−name** option will not currently work with the **−T** option - the Unix name will be used in the TRANS.TBL file, not the Macintosh name.

The character set used to convert any HFS file name to a Joliet/Rock Ridge file name defaults to *cp10000* (Mac Roman). The character set used can be specified using the *−input−hfs−charset* option. Other built in HFS character sets are: cp10006 (MacGreek), cp10007 (MacCyrillic), cp10029 (MacLatin2), cp10079 (MacIcelandic) and cp10081 (MacTurkish).

Note: the character codes used by HFS file names taken from the various Apple/Unix formats will not be converted as they are assumed to be in the correct Apple character set. Only the Joliet/Rock Ridge names derived from the HFS file names will be converted.

The existing mkisofs code will filter out any illegal characters for the ISO-9660 and Joliet filenames, but as mkisofs expects to be dealing directly with Unix names, it leaves the Rock Ridge names as is. But as ’/’ is a legal HFS filename character, the **−mac−name** option converts ’/’ to a ’\_’ in Rock Ridge filenames.

If the Apple extensions are used, then only the ISO-9660 filenames will appear on the Macintosh. However, as the Macintosh ISO-9660 drivers can use *Level 2* filenames, then you can use options like **−allow−multidot** without problems on a Macintosh - still take care over the names, for example *this.file.name* will be converted to *THIS.FILE* i.e. only have one ’.’, also filename *abcdefgh* will be seen as *ABCDEFGH* but *abcdefghi* will be seen as *ABCDEFGHI.* i.e. with a ’.’ at the end - don’t know if this is a Macintosh problem or mkisofs/mkhybrid problem. All filenames will be in upper case when viewed on a Macintosh. Of course, DOS/Win3.X machines will not be able to see Level 2 filenames...

HFS CUSTOM VOLUME/FOLDER ICONS []()
-----------------------------------

To give a HFS CD a custom icon, make sure the root (top level) folder includes a standard Macintosh volume icon file. To give a volume a custom icon on a Macintosh, an icon has to be pasted over the volume’s icon in the "Get Info" box of the volume. This creates an invisible file called ’Icon\\r’ (’\\r’ is the ’carriage return’ character) in the root folder.

A custom folder icon is very similar - an invisible file called ’Icon\\r’ exits in the folder itself.

Probably the easiest way to create a custom icon that mkisofs can use, is to format a blank HFS floppy disk on a Mac, paste an icon to its "Get Info" box. If using Linux with the HFS module installed, mount the floppy using something like:

mount −t hfs /dev/fd0 /mnt/floppy

The floppy will be mounted as a CAP file system by default. Then run mkisofs using something like:

mkisofs −−cap −o output source\_dir /mnt/floppy

If you are not using Linux, then you can use the hfsutils to copy the icon file from the floppy. However, care has to be taken, as the icon file contains a control character. e.g.

hmount /dev/fd0
hdir −a
hcopy −m Icon^V^M icon\_dir/icon

Where ’^V^M’ is control−V followed by control−M. Then run **mkisofs** by using something like:

mkisofs −−macbin −o output source\_dir icon\_dir

The procedure for creating/using custom folder icons is very similar - paste an icon to folder’s "Get Info" box and transfer the resulting ’Icon\\r’ file to the relevant directory in the mkisofs source tree.

You may want to hide the icon files from the ISO-9660 and Joliet trees.

To give a custom icon to a Joliet CD, follow the instructions found at: http://www.fadden.com/cdrfaq/faq03.html\#\[3-21\]

HFS BOOT DRIVER []()
--------------------

It *may* be possible to make the hybrid CD bootable on a Macintosh.

A bootable HFS CD requires an Apple CD-ROM (or compatible) driver, a bootable HFS partition and the necessary System, Finder, etc. files.

A driver can be obtained from any other Macintosh bootable CD-ROM using the *apple\_driver* utility. This file can then be used with the **−boot−hfs−file** option.

The HFS partition (i.e. the hybrid disk in our case) must contain a suitable System Folder, again from another CD-ROM or disk.

For a partition to be bootable, it must have its *boot block* set. The boot block is in the first two blocks of a partition. For a non-bootable partition the boot block is full of zeros. Normally, when a System file is copied to partition on a Macintosh disk, the boot block is filled with a number of required settings - unfortunately I don’t know the full spec for the boot block, so I’m guessing that the following will work OK.

Therefore, the utility *apple\_driver* also extracts the boot block from the first HFS partition it finds on the given CD-ROM and this is used for the HFS partition created by **mkisofs**.
PLEASE NOTE

By using a driver from an Apple CD and copying Apple software to your CD, you become liable to obey Apple Computer, Inc. Software License Agreements.

EL TORITO BOOT INFORMATION TABLE []()
-------------------------------------

When the **−boot−info−table** option is given, **mkisofs** will modify the boot file specified by the **−b** option by inserting a 56-byte "boot information table" at offset 8 in the file. This modification is done in the source filesystem, so make sure you use a copy if this file is not easily recreated! This file contains pointers which may not be easily or reliably obtained at boot time.

The format of this table is as follows; all integers are in section 7.3.1 ("little endian") format.

Offset Name Size Meaning

<table>
<colgroup>
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
<col width="20%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p>8</p></td>
<td><p>bi_pvd</p></td>
<td><p>4 bytes</p></td>
<td><p>LBA of primary volume descriptor</p></td>
</tr>
<tr class="even">
<td></td>
<td><p>12</p></td>
<td><p>bi_file</p></td>
<td><p>4 bytes</p></td>
<td><p>LBA of boot file</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p>16</p></td>
<td><p>bi_length</p></td>
<td><p>4 bytes</p></td>
<td><p>Boot file length in bytes</p></td>
</tr>
<tr class="even">
<td></td>
<td><p>20</p></td>
<td><p>bi_csum</p></td>
<td><p>4 bytes</p></td>
<td><p>32-bit checksum</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p>24</p></td>
<td><p>bi_reserved</p></td>
<td><p>40 bytes</p></td>
<td><p>Reserved</p></td>
</tr>
</tbody>
</table>

The 32-bit checksum is the sum of all the 32-bit words in the boot file starting at byte offset 64. All linear block addresses (LBAs) are given in CD sectors (normally 2048 bytes).

CONFIGURATION []()
------------------

**mkisofs** looks for the **.mkisofsrc** file, first in the current working directory, then in the user’s home directory, and then in the directory in which the **mkisofs** binary is stored. This file is assumed to contain a series of lines of the form **TAG=***value* , and in this way you can specify certain options. The case of the tag is not significant. Some fields in the volume header are not settable on the command line, but can be altered through this facility. Comments may be placed in this file, using lines which start with a hash (\#) character.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p><strong>APPI</strong></p></td>
<td></td>
<td><p>The application identifier should describe the application that will be on the disc. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. May be overridden using the <strong>−A</strong> command line option.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>COPY</strong></p></td>
<td></td>
<td><p>The copyright information, often the name of a file on the disc containing the copyright notice. There is space in the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. May be overridden using the <strong>−copyright</strong> command line option.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>ABST</strong></p></td>
<td></td>
<td><p>The abstract information, often the name of a file on the disc containing an abstract. There is space in the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. May be overridden using the <strong>−abstract</strong> command line option.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>BIBL</strong></p></td>
<td></td>
<td><p>The bibliographic information, often the name of a file on the disc containing a bibliography. There is space in the disc for 37 characters of information. The related Joliet entry is limited to 18 characters. May be overridden using the <strong>−bilio</strong> command line option.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>PREP</strong></p></td>
<td></td>
<td><p>This should describe the preparer of the CDROM, usually with a mailing address and phone number. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. May be overridden using the <strong>−p</strong> command line option.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>PUBL</strong></p></td>
<td></td>
<td><p>This should describe the publisher of the CDROM, usually with a mailing address and phone number. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. May be overridden using the <strong>−publisher</strong> command line option.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>SYSI</strong></p></td>
<td></td>
<td><p>The System Identifier. There is space on the disc for 32 characters of information. May be overridden using the <strong>−sysid</strong> command line option.</p></td>
</tr>
<tr class="even">
<td></td>
<td><p><strong>VOLI</strong></p></td>
<td></td>
<td><p>The Volume Identifier. There is space on the disc for 32 characters of information. May be overridden using the <strong>−V</strong> command line option.</p></td>
</tr>
<tr class="odd">
<td></td>
<td><p><strong>VOLS</strong></p></td>
<td></td>
<td><p>The Volume Set Name. There is space on the disc for 128 characters of information. The related Joliet entry is limited to 64 characters. May be overridden using the <strong>−volset</strong> command line option.</p></td>
</tr>
</tbody>
</table>

**HFS\_TYPE**

The default TYPE for Macintosh files. Must be exactly 4 characters. May be overridden using the **−hfs−type** command line option.

**HFS\_CREATOR**

The default CREATOR for Macintosh files. Must be exactly 4 characters. May be overridden using the **−hfs−creator** command line option.

**mkisofs** can also be configured at compile time with defaults for many of these fields. See the file defaults.h.

EXAMPLES []()
-------------

To create a vanilla ISO-9660 filesystem image in the file *cd.iso*, where the directory *cd\_dir* will become the root directory of the CD ISO image, call:

% mkisofs −o cd.iso cd\_dir

To create a CD with Rock Ridge extensions of the source directory *cd\_dir*:

% mkisofs −o cd.iso −R cd\_dir

To create a CD with Rock Ridge extensions of the source directory *cd\_dir* where all files have at least read permission and all files are owned by *root*, call:

% mkisofs −o cd.iso −r cd\_dir

To write a tar archive directly to a CD that will later contain a simple ISO-9660 filesystem with the tar archive call:

% star −c . | mkisofs −stream−media−size 333000 | \\
cdrecord dev=b,t,l −dao tsize=333000s −

To create a HFS hybrid CD with the Joliet and Rock Ridge extensions of the source directory *cd\_dir*:

% mkisofs −o cd.iso −R −J −hfs cd\_dir

To create a HFS hybrid CD from the source directory *cd\_dir* that contains Netatalk Apple/Unix files:

% mkisofs −o cd.iso −−netatalk cd\_dir

To create a HFS hybrid CD from the source directory *cd\_dir*, giving all files CREATOR and TYPES based on just their filename extensions listed in the file "mapping".:

% mkisofs −o cd.iso −map mapping cd\_dir

To create a CD with the ’Apple Extensions to ISO-9660’, from the source directories *cd\_dir* and *another\_dir.* Files in all the known Apple/Unix format are decoded and any other files are given CREATOR and TYPE based on their magic number given in the file "magic":

% mkisofs −o cd.iso −apple −magic magic −probe \\
cd\_dir another\_dir

The following example puts different files on the CD that all have the name README, but have different contents when seen as a ISO-9660/RockRidge, Joliet or HFS CD.

Current directory contains:

% ls −F
README.hfs README.joliet README.unix cd\_dir/

The following command puts the contents of the directory *cd\_dir* on the CD along with the three README files - but only one will be seen from each of the three filesystems:

% mkisofs −o cd.iso −hfs −J −r −graft−points \\
−hide README.hfs −hide README.joliet \\
−hide−joliet README.hfs −hide−joliet README.unix \\
−hide−hfs README.joliet −hide−hfs README.unix \\
README=README.hfs README=README.joliet \\
README=README.unix cd\_dir

i.e. the file README.hfs will be seen as README on the HFS CD and the other two README files will be hidden. Similarly for the Joliet and ISO-9660/RockRidge CD.

There are probably all sorts of strange results possible with combinations of the hide options ...

To create a DVD-Audio of the DVD-Audio compliant source directory *DVD*:

% mkisofs −o dvda.iso −dvd−audio DVD

AUTHOR []()
-----------

Eric Youngdale &lt;ericy@gnu.ai.mit.edu&gt; or &lt;eric@andante.org&gt; wrote the first versions (1993 ... 1998) of the mkisofs utility. The copyright for old versions of the mkisofs utility is held by Yggdrasil Computing, Incorporated. Joerg Schilling wrote the SCSI transport library and its adaptation layer to **mkisofs** and newer parts (starting from 1997) of the utility. Joerg Schilling is the primary maintainer since 1999, this makes **mkisofs** Copyright (C) 1997-2014 Joerg Schilling.

HFS hybrid code Copyright (C) James Pearson 1997 ... 2001.

libhfs code Copyright (C) 1996, 1997 Robert Leslie.

libfile code Copyright (C) Ian F. Darwin 1986, 1987, 1989, 1990, 1991, 1992, 1994, 1995.

NOTES []()
----------

**Mkisofs** may safely be installed suid root. This may be needed to allow **mkisofs** to read the previous session when creating a multi session image.

**mkisofs** is not based on the standard mk\*fs tools for unix, because we must generate a complete copy of an existing filesystem on a disk in the ISO-9660 filesystem. The name mkisofs is probably a bit of a misnomer, since it not only creates the filesystem, but it also populates it as well. However, the appropriate tool name for a UNIX tool that creates populated filesystems - **mkproto** - is not well known.

If **mkisofs** is creating a filesystem image with Rock Ridge attributes and the directory nesting level of the source directory tree is too much for ISO-9660, **mkisofs** will do deep directory relocation. This results in a directory called **RR\_MOVED** in the root directory of the CD. You cannot avoid this directory in the directory tree that is visible with ISO-9660 but it it automatically hidden in the **Rock Ridge** tree.

The sparc boot support that is implemented with the **−sparc−boot** options completely follows the official Sparc CD boot requirements from the Boot prom in Sun Sparc systems. Some Linux distributions for Sparc systems use a boot loader called **SILO** that unfortunately is not Sparc CD boot compliant. It is annoyingly to see that the Authors of SILO don’t fix SILO but instead provide a completely unneeded "patch" to mkisofs that incorporates far more source than the fix for SILO would need.

BUGS []()
---------

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p>•</p></td>
<td></td>
<td><p>Does not properly read relocated directories in multi-session mode when adding data.</p></td>
</tr>
</tbody>
</table>

Any relocated deep directory is lost if the new session does not include the deep directory.

Repeat by: create first session with deep directory relocation then add new session with a single dir that differs from the old deep path.

<table>
<colgroup>
<col width="25%" />
<col width="25%" />
<col width="25%" />
<col width="25%" />
</colgroup>
<tbody>
<tr class="odd">
<td></td>
<td><p>•</p></td>
<td></td>
<td><p>Does not re-use RR_MOVED when doing multi-session from TRANS.TBL</p></td>
</tr>
</tbody>
</table>

There may be some other ones. Please, report them to the author.

HFS PROBLEMS/LIMITATIONS []()
-----------------------------

I have had to make several assumptions on how I expect the modified libhfs routines to work, however there may be situations that either I haven’t thought of, or come across when these assumptions fail. Therefore I can’t guarantee that mkisofs will work as expected (although I haven’t had a major problem yet). Most of the HFS features work fine, however, some are not fully tested. These are marked as *Alpha* above.

Although HFS filenames appear to support upper and lower case letters, the filesystem is case insensitive. i.e. the filenames "aBc" and "AbC" are the same. If a file is found in a directory with the same HFS name, then *mkisofs* will attempt, where possible, to make a unique name by adding ’\_’ characters to one of the filenames.

HFS file/directory names that share the first 31 characters have \_N’ (N == decimal number) substituted for the last few characters to generate unique names.

Care must be taken when "grafting" Apple/Unix files or directories (see above for the method and syntax involved). It is not possible to use a new name for an Apple/Unix encoded file/directory. e.g. If a Apple/Unix encoded file called "oldname" is to added to the CD, then you can not use the command line:

mkisofs −o output.raw −hfs −graft−points newname=oldname cd\_dir

mkisofs will be unable to decode "oldname". However, you can graft Apple/Unix encoded files or directories as long as you do not attempt to give them new names as above.

When creating an HFS volume with the multisession options, **−M** and **−C**, only files in the last session will be in the HFS volume. i.e. mkisofs can not *add* existing files from previous sessions to the HFS volume.

However, if each session is created with the **−part** option, then each session will appear as separate volumes when mounted on a Mac. In this case, it is worth using the **−V** or **−hfs−volid** option to give each session a unique volume name, otherwise each "volume" will appear on the Desktop with the same name.

Symbolic links (as with all other non-regular files) are not added to the HFS directory.

Hybrid volumes may be larger than pure ISO-9660 volumes containing the same data. In some cases (e.g. DVD sized volumes) the hybrid volume may be significantly larger. As an HFS volume gets bigger, so does the allocation block size (the smallest amount of space a file can occupy). For a 650Mb CD, the allocation block is 10Kb, for a 4.7Gb DVD it will be about 70Kb.

The maximum number of files in an HFS volume is about 65500 - although the real limit will be somewhat less than this.

The resulting hybrid volume can be accessed on a Unix machine by using the hfsutils routines. However, no changes can be made to the volume as it is set as **locked.** The option **−hfs−unlock** will create an output image that is unlocked - however no changes should be made to the contents of the volume (unless you really know what you are doing) as it’s not a "real" HFS volume.

Using the **−mac−name** option will not currently work with the **−T** option - the Unix name will be used in the TRANS.TBL file, not the Macintosh name.

Although **mkisofs** does not alter the contents of a file, if a binary file has its TYPE set as ’TEXT’, it *may* be read incorrectly on a Macintosh. Therefore a better choice for the default TYPE may be ’????’

The **−mac−boot−file** option may not work at all...

May not work with PC Exchange v2.2 or higher files (available with MacOS 8.1). DOS media containing PC Exchange files should be mounted as type **msdos** (not **vfat**) when using Linux.

The SFM format is only partially supported - see **HFS MACINTOSH FILE FORMATS** section above.

It is not possible to use the the **−sparc−boot** or **−generic−boot** options with the **−boot−hfs−file** the **−prep−boot** or **−chrp−boot** options.

**mkisofs** should be able to create HFS hybrid images over 4Gb, although this has not been fully tested.

SEE ALSO []()
-------------

**cdrecord**(1), **mkzftree**(1), **sfind**(1), **magic**(5), **apple\_driver**(8).

FUTURE IMPROVEMENTS []()
------------------------

Some sort of gui interface.

AVAILABILITY []()
-----------------

**mkisofs** is available as part of the cdrecord package from https://sourceforge.net/projects/cdrtools/files/

**hfsutils** from ftp://ftp.mars.org/pub/hfs

**mkzftree** is available as part of the zisofs-tools package from ftp://ftp.kernel.org/pub/linux/utils/fs/zisofs/

MAILING LISTS []()
------------------

If you want to actively take part on the development of mkisofs, you may join the developer mailing list via this URL:

**https://lists.sourceforge.net/lists/listinfo/cdrtools-developers**

MAINTAINER []()
---------------

Joerg Schilling
Seestr. 110
D-13353 Berlin
Germany

HFS MKHYBRID MAINTAINER []()
----------------------------

James Pearson

j.pearson@ge.ucl.ac.uk

If you have support questions, send them to:

**cdrtools-support@lists.sourceforge.net**

If you definitely found a bug, send a mail to:

**cdrtools-developers@lists.sourceforge.net**
or **joerg.schilling@fokus.fraunhofer.de**

To subscribe, use:

**https://lists.sourceforge.net/lists/listinfo/cdrtools-developers**
or **https://lists.sourceforge.net/lists/listinfo/cdrtools-support**

INTERFACE STABILITY []()
------------------------

The interfaces provided by **mkisofs** are designed for long term stability. As **mkisofs** depends on interfaces provided by the underlying operating system, the stability of the interfaces offered by **mkisofs** depends on the interface stability of the OS interfaces. Modified interfaces in the OS may enforce modified interfaces in **mkisofs**.

------------------------------------------------------------------------
