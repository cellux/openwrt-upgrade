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

2.	Download OpenWRT upgrade image and the md5sums:

	* http://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/generic/openwrt-15.05-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	* http://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/generic/md5sums

3.	Verify the download:

	```bash
	grep -F openwrt-15.05-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin md5sums | md5sum -c
	```

	Expected output:

	```bash
	openwrt-15.05-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin: OK
	```

4.	Download original factory image (just in case):

	http://www.tp-link.com/en/download/TL-WR842ND_V1.html

5.	Login to the router as root

6.	Stop as many services as possible to free up RAM for the upgrade.

	Other methods to free up RAM:

	```bash
	rm -rf /tmp/opkg-lists
	echo 3 > /proc/sys/vm/drop_caches
	```

7.	Upload firmware image to /tmp directory on router and run the upgrade:

	```bash
	sysupgrade -v /tmp/openwrt-15.05-ar71xx-generic-tl-wr842n-v1-squashfs-sysupgrade.bin
	```

	When the upgrade is complete, the router shall reboot.

	If the router doesn't come up, try a cold reset (unplug-wait-replug).

8.	Install packages

	After an upgrade, all extra installed packages are gone, so these
	must be reinstalled. Luckily, the configuration files under /etc
	are preserved.

	Download/refresh package meta-data:

	```bash
	opkg update
	```

	Install any extra packages you need:

	```bash
	opkg install openssh-sftp-server
	opkg install kmod-usb-storage
	opkg install ntfs-3g
	opkg install kmod-fs-ext4
	opkg install mount-utils kmod-loop losetup
	opkg install blkid block-mount
	opkg install ddns-scripts
	opkg install nfs-kernel-server nfs-kernel-server-utils
	opkg install samba36-server
	opkg install rsync screen
	```

9.	Configuration:

	```bash
	# install SSH key
	scp ~/.ssh/id_rsa.pub root@jupiter:/etc/dropbear/authorized_keys

	# test that it works
	ssh root@jupiter

	# disable SSH password auth
	vi /etc/config/dropbear

	# disable IPv6, enable SSH port on WAN interface
	vi /etc/config/firewall

	# change root password
	passwd

	# configure hostname + timezone, enable NTP server
	vi /etc/config/system

	# create mount points for external disks
	mkdir -p /mnt/shares/passport
	mkdir -p /mnt/shares/samsung
	mkdir -p /mnt/shares/bitpool

	# configure fstab
	vi /etc/config/fstab
	/etc/init.d/fstab enable
	/etc/init.d/fstab start
	# note: this is broken at the moment (in Chaos Chalmer)
	#
	# I had to mount the USB disks manually in /etc/rc.local
	# as block detect does not recognize NTFS partitions

	# configure wireless
	vi /etc/config/wireless

	# configure DynDNS
	vi /etc/config/ddns
	/etc/init.d/ddns enable
	/etc/init.d/ddns start

	# setup NFS server
	vi /etc/exports
	/etc/init.d/portmap enable
	/etc/init.d/nfsd enable
	/etc/init.d/portmap start
	/etc/init.d/nfsd start

	# install Samba
	vi /etc/config/samba
	/etc/init.d/samba enable
	/etc/init.d/samba start

	# add share user
	opkg install shadow-useradd
	useradd -d /mnt/shares -M share
	```

10.	Check that everything works as expected.

11.	Enjoy.

