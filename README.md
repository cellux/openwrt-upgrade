# OpenWrt upgrade HOWTO

I have a TP-Link WR842ND router at home whose firmware I replaced with
[OpenWrt](https://openwrt.org/).

I wrote this document to have a reference at hand when I want to reinstall the
router from scratch or upgrade to a newer OpenWrt version.

## Upgrade steps

1.	Make a backup of the `/overlay` directory (this contains all changes made to the default root fs).

	Also save the list of installed packages and changed config files:

	```bash
	opkg list-installed
	opkg list-changed-conffiles
	```

2.	Download OpenWRT trunk upgrade image and the md5sums:
	* http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	* http://downloads.openwrt.org/snapshots/trunk/ar71xx/md5sums

3.	Verify the download:

	```bash
	grep -F openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin md5sums | md5sum -c
	```

	Expected output:

	```bash
	openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin: OK
	```

4.	Download original factory image (just in case):

	http://www.tp-link.com/en/support/download/?model=TL-WR842ND&version=V1

5.	Log in to the router

6.	Stop as many services as possible to free up RAM for the upgrade.

	Other methods to free up RAM:

	```bash
	rm -rf /tmp/opkg-lists
	echo 3 > /proc/sys/vm/drop_caches
	```

7.	Upload firmware image to /tmp directory on router, start the upgrade:

	```bash
	sysupgrade -v /tmp/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	```

	If this does not work, you can do it using mtd, but in that case, *all config files will be lost*:

	```bash
	mtd -r write /tmp/openwrt-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin firmware
	```

	When the upgrade is complete, the router shall reboot.

	If the router doesn't come up, try a cold reset (unplug-wait-replug).

8.	Local configuration:

	```bash
	# install SSH key
	scp ~/.ssh/id_rsa.pub root@jupiter:/etc/dropbear/authorized_keys

	# test that it works
	ssh root@jupiter

	# disable SSH password auth
	vi /etc/config/dropbear

	# download package meta-data
	opkg update

	# install SFTP support
	opkg install openssh-sftp-server

	# disable IPv6, enable SSH port on WAN interface
	vi /etc/config/firewall

	# change root password
	passwd

	# configure hostname + timezone, enable NTP server
	vi /etc/config/system

	# create mount points for external disks
	mkdir -p /mnt/shares/passport
	mkdir -p /mnt/shares/samsung
	mkdir -p /mnt/shares/rpifs

	# install external USB storage support
	opkg install kmod-usb-storage

	# install NTFS support
	opkg install ntfs-3g

	# install Ext4 support
	opkg install kmod-fs-ext4

	# configure fstab
	opkg install block-mount
	vi /etc/config/fstab
	/etc/init.d/fstab enable
	/etc/init.d/fstab start
	# note: this is broken at the moment (in Chaos Chalmer)
	# I had to mount the USB disks manually in /etc/rc.local

	# configure wireless
	vi /etc/config/wireless

	# configure DynDNS
	opkg install ddns-scripts
	vi /etc/config/ddns
	/etc/init.d/ddns enable
	/etc/init.d/ddns start

	# setup NFS server
	opkg install nfs-kernel-server nfs-kernel-server-utils
	vi /etc/exports
	/etc/init.d/portmap enable
	/etc/init.d/nfsd enable
	/etc/init.d/portmap start
	/etc/init.d/nfsd start

	# install Samba
	opkg install samba36-server
	vi /etc/config/samba
	/etc/init.d/samba enable
	/etc/init.d/samba start

	# install real mount/umount + loop mount support
	opkg install mount-utils kmod-loop losetup

	# install some useful tools
	opkg install blkid rsync screen

	# add share user
	opkg install shadow-useradd
	useradd -d /mnt/shares -M share

9.	Check that everything works as expected.

10.	Enjoy.

