# Bootable USB with CentOS 7 and Kickstart

This page was vreated because I failed to find full manual of kickstart driven CentOS 7 USB installer. 

My intention was in putting all commands into script. Other manual for example tell you to use gparted of fdisk in order to fomrat USB drive. You can't use gparted in bash script.

### Can we just dd CentOS image to USB stick and put kickstart file to USB?
* No, we can't. Kickstart failed to start.

## References

Most of the instructions described below was found on the [CentOS Wiki](https://wiki.centos.org/HowTos/InstallFromUSBkey#line-30) page on Installing from USB key. 


# USB key preparation

## Partition USB

This can't be done as a disk image. As I described earlier kickstart failed to start. Below I will use /dev/sdX for the USB device.

* Create two partitions, one of type W95 FAT32 (LBA) of ~250MB, make this partition bootable. Create an ext4 partition from the remaining space.

'	sudo parted --script /dev/sdX \
	        mklabel msdos \
	        mkpart primary fat32    1MiB       250MiB \
	        mkpart primary ext4    250MiB       -1MiB \
	        \
	        set 1 boot on'

* Format partitons

'sudo mkfs -t vfat -n "BOOT" /dev/sdX1
sudo mkfs -L "DATA" /dev/sdX2'

	• Write MBR data to device
sudo dd conv=notrunc bs=440 count=1 if=/usr/share/syslinux/mbr.bin of=/dev/sdX
	• Install syslinux to first parition
sudo syslinux /dev/sdX1
Copy files to USB
	• Mount the partitions
mkdir BOOT && sudo mount /dev/sdX1 BOOT
mkdir DATA && sudo mount /dev/sdX2 DATA
mkdir DVD && sudo mount /path/to/centos/dvd.iso DVD
	• Copy DVD isolinux contents to BOOT
sudo cp DVD/isolinux/* BOOT
	• rename isolinux.cfg to syslinux.cfg
sudo mv BOOT/isolinux.cfg BOOT/syslinux.cfg
	• I also deleted a few bits from BOOT I didn't think were required, e.g. isolinux.bin, TRANS.TBL, upgrade.img, grub.conf.
	• I then copied my kickstart file to the BOOT directory and the CentOS 7 ISO to the DATA partition.
The final file structure looked something like this:
BOOT/
├── boot.cat
├── boot.msg
├── initrd.img
├── ks.cfg
├── ldlinux.sys
├── memtest
├── splash.png
├── syslinux.cfg
├── upgrade.img
├── vesamenu.c32
└── vmlinuz
DATA/
└── CentOS-7.0-1406-x86_64-Minimal.iso
Edit the syslinux.cfg
So that it points to the ISO and the kickstart
Here is the install CentOS 7 entry from the Minimal ISO isolinux.cfg (which we renamed syslinux.cfg):
label linux                                                                     
  menu label ^Install CentOS 7                                                  
  kernel vmlinuz                                                                
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet  
The append line is changed to read the following:
append initrd=initrd.img inst.stage2=hd:sdb2:/ ks=hd:sdb1:/ks.cfg
I suspect LABEL could be used here, rather than the enumerated device, which would make it safer, but I haven't tried this yet. Assuming the system you are installing on only has a single HD the USB key will be enumerated as sdb more information about this can be found in the Softpanorama article.
When you boot from the USB and select Install CentOS 7, it now installs the system as described by your kickstart.
