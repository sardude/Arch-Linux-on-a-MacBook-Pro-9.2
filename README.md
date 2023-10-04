# arch-on-mbp9-2
install arch on mbp9,2 triple booting

![Screen Shot 2023-10-04 at 16 13 37](https://github.com/sardude/arch-on-mbp9-2/assets/9292055/68b43de2-a441-4de7-8e9c-60aaae41b66d)


Installation
============

Internet
---------
# ping -c 3 -4 archlinux.org

Format and mount
-----------------
Assuming you have this when run fdisk

#fdisk -l:

Device			Size	Type
/dev/sdb1	209.7 MB	boot
/dev/sdb3	45.0 GB 	Linux filesystem


	Formatting partitions
	----------------------
mkfs.fat -F32 /dev/sdb1
mkfs.ext4 /dev/sdb3

	Mounting
	--------
mount /dev/<rootpartition> /mnt
mkdir /mnt/boot
mount /dev/<appleEFIpartition> /mnt/boot

Base Install
------------
This will install the base Arch Linux packages.
#pacstrap -i /mnt/ base


