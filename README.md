OpenWrt upgrade HOWTO
=====================

I have a TP-Link WR842ND router at home whose firmware I replaced with [OpenWrt](https://openwrt.org/).

With the help of OpenWrt, I can use the router for the following tasks:

* internet sharing for my T-Mobile 3G USB mobile stick
* file server (shares two NTFS disks over NFS and CIFS)

Lately it became necessary to upgrade the router (I needed a `8021q` kernel module which was not available for my kernel version any more).

I wrote this document to have a reference for the next time when I have to go through this process.

Steps
=====

Make a backup of the `/overlay` directory (this contains all changes made to default fs).

Download OpenWRT 12.09 (Attitude Adjustment) upgrade image and the md5sums:

http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/md5sums

Verify the download:

```bash
grep -F openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin md5sums | md5sum -c
openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin: OK
```

Download original factory image (just in case):

http://www.tp-link.com/en/support/download/?model=TL-WR842ND&version=V1

Login to the router

Stop as many services as possible to free up RAM for the upgrade.

Other methods to free up RAM:

```bash
rm -rf rm -r /tmp/opkg-lists
echo 3 > /proc/sys/vm/drop_caches
```

See `/etc/sysupgrade.conf` to check/edit the list of files/directories which will be preserved by the upgrade.

I didn't add anything to `/etc/sysupgrade.conf` because the docs say that the files returned by `$(opkg list-changed-conffiles)` are automatically preserved and I thought this would be enough.

Copy upgrade image to /tmp on router, then do the upgrade:

```bash
sysupgrade -v /tmp/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
```

Try a cold reset (unplug-wait-replug) if the router doesn't come up after the upgrade.

Install extra packages:

```bash
opkg update

# 3G modem
opkg install comgt kmod-usb-serial-option usb-modeswitch-data usbutils

# USB storage
opkg install kmod-usb-storage

# NFS
opkg install nfs-kernel-server

# NTFS + CIFS
opkg install ntfs-3g samba36-server

# misc tools
opkg install losetup mc rsync screen
```

Check that everything works as expected.

Enjoy.

