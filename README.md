# OpenWrt upgrade HOWTO

I have a TP-Link WR842ND router at home whose firmware I replaced with [OpenWrt](https://openwrt.org/).

I use the router for the following tasks:

* sharing the internet access provided by my T-Mobile 3G USB mobile stick
* as a file server (the router shares two NTFS disks over NFS and CIFS)

Lately it became necessary to upgrade the router (I needed a `8021q` kernel module for segmenting the router's switch into two separate VLANs but this module was not available for my kernel version any more).

I wrote this document to have a reference at hand when I need to go through the upgrade process again.

## Steps

1.	Make a backup of the `/overlay` directory (this contains all changes made to the default fs).

2.	Download OpenWRT 12.09 (Attitude Adjustment) upgrade image and the md5sums:
	* http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	* http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/md5sums
3.	Verify the download:

	```bash
	grep -F openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin md5sums | md5sum -c
	```

	If it says:

	```bash
	openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin: OK
	```

	then it should be OK.

4.	Download original factory image (just in case):

	http://www.tp-link.com/en/support/download/?model=TL-WR842ND&version=V1

5.	Log in to the router

6.	Stop as many services as possible to free up RAM for the upgrade.

	Other methods to free up RAM:

	```bash
	rm -rf /tmp/opkg-lists
	echo 3 > /proc/sys/vm/drop_caches
	```

7.	See `/etc/sysupgrade.conf` to check/edit the list of files/directories which will be preserved by the upgrade.

	I didn't add anything to `/etc/sysupgrade.conf` because the [docs](http://wiki.openwrt.org/doc/techref/sysupgrade) say that the files returned by `$(opkg list-changed-conffiles)` will be automatically preserved and this seemed to be enough.

8.	Copy upgrade image to /tmp on router, then do the upgrade:

	```bash
	sysupgrade -v /tmp/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	```

	When the upgrade is complete, the router shall reboot with all network settings preserved.

	If the router doesn't come up, try a cold reset (unplug-wait-replug).

9.	Install extra packages:

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

10.	Check that everything works as expected.

11.	Enjoy.

