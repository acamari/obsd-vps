# OpenBSD on Cloudcone.com

## Updated as 2022-11-29

These instructions have been validated on a SC2 instance with 1vCPU + 1G RAM +
250 GB disk.
The instance was created selecting Operating system as Arch Linux 2021 Server -
64 Bit.

# TLDR

I provide a miniroot72.img file modified to run the vanilla OpenBSD installer,
you can download it [here](https://github.com/acamari/obsd-vps/blob/master/cloudcone/miniroot72.img).
You should `dd` that image to your `/dev/vda` device, you can use cloudcone's
*Recovery mode* to do that.


# Overview

The modified miniroot72.img file was produced with the following instructions.

Cloudcone uses a CDROM boot device for the vm, that CDROM uses grub2 to search
for a grub configuration file on a partition on the regular disk that should be the
number 1 in a MBR partition scheme.

Specifically configuration in cdrom is:

```
# cat /mnt/cdrom/boot/grub/grub.cfg
#
# Sample GRUB configuration file
# GRUB 2.03 (2.02-81.el8)
#

# Boot automatically after 30 secs.
set timeout=0

# By default boot Grub 2
set default=0

# Fallback to Grub Legacy
set fallback=1

# For booting from HDD with Grub2
menuentry "Grub 2" --id grub2 {
        set root=(hd0,msdos1)
        configfile /boot/grub2/grub.cfg
}

# For booting from HDD with Grub Legacy
menuentry "Grub Legacy" --id legacy {
        set root=(hd0,msdos1)
        legacy_configfile /boot/grub/grub.conf
}
```

So it looks for file `/boot/grub2/grub.cfg` (with grub2 format) if not exists
looks for `/boot/grub/grub.conf` (in grub legacy format)

You could use the OpenBSD official miniroot72.img image and add the required
file to the efi partition at `boot/grub2/grub.cfg`.

## 1. Boot into Recovery mode

Recovery mode is a Centos 7 Live image with unknown cloudcone modifications (at
least modified to use a different root password for each client).

## 2. Log in vm via ssh or vnc

(Refer to cloudcone docs)

## 2. Burn miniroot image to your disk

```
# wget -O /dev/vda https://cdn.openbsd.org/pub/OpenBSD/7.2/amd64/miniroot72.img
```

## 3. Add chainload in EFI partition

```
# mkdir /mnt/tmp
# mount /dev/vda1 /mnt/tmp/
# cd /mnt/tmp/
# mkdir -p boot/grub2
# cat > boot/grub2/grub.cfg < END
set timeout=5

menuentry "OpenBSD" {
 set root=(hd0,msdos4)
 chainloader +1
}
END
#
```

## 4. Reboot in normal mode (disable recovery mode in http console)

Log via VNC and you should see a regular `bsd.rd` installer. Install as usual.

