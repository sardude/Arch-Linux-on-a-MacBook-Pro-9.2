Arch Linux on a MacBook Pro 9.2
===============================

- :ballot_box_with_check: display
- :ballot_box_with_check: audio
- :ballot_box_with_check: internet connection, wifi
- :ballot_box_with_check: keyboard (work perfectly as normal keyboard)
- :ballot_box_with_check: trackpad & external mouse
- :ballot_box_with_check: screen backlight
- :ballot_box_with_check: keyboard backlight
- :ballot_box_with_check: fan
- :ballot_box_with_check: battery (not best but quite good)
- :black_square_button: webcam: not tested but someone say it work perfectly with [bcwc-pcie](https://aur.archlinux.org/packages/bcwc-pcie-git/), [more detail here](https://wiki.archlinux.org/index.php/MacBookPro11,x#Web_cam)

gathered information and installed Arch on a MacBook:

- [Before you start](#before-you-start)
  - [Install arch dual boot](#install-arch-dual-boot)
    - [Make space for Arch](#make-space-for-arch)
    - [Make installer USB](#make-installer-usb)
    - [Boot it up](#boot-it-up)
    - [Connect wifi](#connect-wifi)
    - [Partition](#partitioning)
    - [Format & Mount](#format-and-mount)
    - [Install base packages & generate fstab](#install-base-packages-&-generate-fstab)
    - [System config](#system-config)
    - [Install the bootloader](#install-the-bootloader)
    - [Make arch duo bootable](#make-arch-duo-bootable)
  - [Install to make arch usable](#install-to-make-arch-usable)
    - [Set tty default font](#set-tty-default-font)
    - [Install drivers](#install-drivers)
    - [Install require packages](#install-require-packages)
      - [Required packages for display](#required-packages-for-display)
      - [Required packages for window manager](#required-packages-for-window-manager)
    - [Keyboard](#keyboard)
    - [Screen display](#screen-display)
    - [Window management](#window-management)
    - [Better network management](#better-network-management)
    - [Trackpad](#trackpad)
    - [Sound](#sound)
    - [Fan](#fan)
    - [Screen backlight](#screen-backlight)
    - [Keyboard backlight](#keyboard-backlight)
    - [Webcam](#webcam)
  - [Improvement](#improvement)
    - [Improve DHCP connection speed](#improve-dhcp-connection-speed)
    - [Turn on firewall](#turn-on-firewall)
    - [Enable Trim for SSD](#enable-trim-for-ssd)
    - [Fixing lid closing to suspend](#fixing-lid-closing-to-suspend)
    - [Power saving for Intel chip](#power-saving-for-intel-chip)



# Before you start
from macbookpro; open terminal

➜ diskutil list
|Device             |Size            |Type|
|---                |---             |---|
|/dev/sda3          |128MB           |Apple HFS+|
|/dev/sda4          |256MB           |Linux filesystem|
|/dev/sda5          |16GB            |Linux Swap|
|/dev/sda6          |64GB            |Linux filesystem|

/dev/disk0 (internal, physical):
|Device             |Size            |Type|
|#:                 |TYPE NAME                    |SIZE       |IDENTIFIER|

|#:                  |TYPE NAME                    |SIZE       |IDENTIFIER|
   0:      GUID_partition_scheme                        *500.1 GB   disk0
   1:                        EFI NO NAME                 209.7 MB   disk0s1
   2:       Microsoft Basic Data darkstar                454.9 GB   disk0s2
   3:           Linux Filesystem                         45.0 GB    disk0s3

/dev/disk1 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *250.1 GB   disk1
   1:                        EFI EFI                     209.7 MB   disk1s1
   2:                 Apple_APFS Container disk2         184.0 GB   disk1s2
   3:       Microsoft Basic Data OS                      65.8 GB    disk1s3

/dev/disk2 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +184.0 GB   disk2
                                 Physical Store disk1s2
   1:                APFS Volume MacOS - Data            140.0 GB   disk2s1
   2:                APFS Volume Preboot                 28.5 MB    disk2s2
   3:                APFS Volume Recovery                526.8 MB   disk2s3
   4:                APFS Volume VM                      1.1 GB     disk2s4
   5:                APFS Volume MacOS                   11.3 GB    disk2s5


It's seem [Macbook with T2 Security](https://support.apple.com/en-us/HT208862) isn't support linux very well, view this [discussions](https://discussions.apple.com/thread/251087440?answerId=252062188022#252062188022)

⚠️Batterry Issue: I'v used arch and artix for couple year on my MBP, everything is fine, smooth, but there is a problem, because OSX is apple stuff, so they make it very well, when using linux on Macbook, you must accept the risk. The only problem i've ever faced is that my battery degrades very quickly.


# Install arch dual boot

## Make space for Arch

Use Disk Utility Partition feature to add new Partition for Arch, follow [this guide](https://wiki.archlinux.org/index.php/Mac#Arch_Linux_with_OS_X_or_other_operating_systems)

Or if you already know how to use Disk Utility, then create a partition with FAT32 format.


## Make installer USB

Download arch's iso [here](https://www.archlinux.org/download/)

Find usb by using `diskutil list`, then:

```
# Assume usb disk is /dev/diskX
diskutil unmountDisk /dev/diskX
dd if=path/to/arch.iso of=/dev/diskX bs==1m
```

## Boot it up

Hold bold `alt/option` when system bootup, then choose boot from USB

:warning: If you are using Retina Macbook, tty font will be very small. To get larger font, [connect to wifi](connect-wifi) and run these commands:

```
sudo pacman -Sy terminus-font
setfont /usr/share/kbd/consolefonts/ter-132b.psf.gz
```

## Connect wifi

Use `wifi-menu` then choose wifi to connect, then check connection with:

```
ping -c 3 google.com
```

## Partition

View all your patitions to choose correct one

```
use fdisk -list
```

Open cgdisk with:

```
cgdisk /dev/sdaX
```

Create these new partitions:

|Size    |Type              |Description|
|---     |---               |---        |
|128MB   |Apple HFS+        |This is required in order to make Arch dual boot with OSX|
|256MB   |Linux filesystem  |Arch file system|
|xMB     |Linux Swap        |If you have space, try to make it double size of your ram size|
|xGB     |Linux filesystem  |This is our home|

## Format and mount

Assuming you have this when run `fdisk -l`:

|Device             |Size            |Type|
|---                |---             |---|
|/dev/sda3          |128MB           |Apple HFS+|
|/dev/sda4          |256MB           |Linux filesystem|
|/dev/sda5          |16GB            |Linux Swap|
|/dev/sda6          |64GB            |Linux filesystem|

Now let format and mount partition:

```
mkfs.ext4 /dev/sda4
mkswap /dev/sda5
mkfs.ext4 /dev/sda6
mount /dev/sda6 /mnt
mkdir /mnt/boot && mount /dev/sda4 /mnt/boot
swapon /dev/sda5
```

## Install base packages & generate fstab

Run these commands:

```
pacstrap /mnt base base-devel
genfstab -p /mnt >> /mnt/etc/fstab
```

:warning: If you are using SSD drive. Open fstab config file:

```
vi /mnt/etc/fstab
```

If you are using ssd, remove all discard in all lines, and make sure it look like:

```sh
/dev/sda4   /boot   ext2   defaults,relatime,stripe=4        0 2
/dev/sda6   /       ext4   defaults,noatime,data=writeback   0 1
```

## System config

```
arch-chroot /mnt /bin/bash
echo arch > /etc/hostname   (Change arch with your hostname)
ln -s /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime  (Use tab to select your zoneinfo easier)
useradd -m -G wheel -s /bin/bash your_username
passwd your_username   (Create your password)
```

Open /etc/sudoers file (with sudo) and uncomment this line to get sudo right for our user:
```
%wheel ALL=(ALL) ALL
```

Uncomment `en_US.UTF-8 UTF-8` (Or what ever locale you want) line in `/etc/locale.gen` file, then run:

```
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

Modify your `/etc/mkinitcpio.conf` file to insert `keyboard` after `autodetect` in the HOOK section (If it not exist). Then run:

```
mkinitcpio -p linux
```

## Install the bootloader

We will boot using OSX native EFI boot loader, so install this:

```
pacman -S grub-efi-x86_64
```

Change `/etc/default/grub` to look like:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback"
```

Then create boot.efi with GRUB

```
grub-mkconfig -o boot/grub/grub.cfg
grub-mkstandalone -o boot.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi --compress xz boot/grub/grub.cfg
```

:heavy_exclamation_mark: Important: Copy boot.efi to your usb or save it somewhere, we'll need this to duo boot.

To copy it to usb, use:

```
mkdir /mnt/myusb && mount /dev/sdb /mnt/myusb
cp boot.efi /mnt/myusb/
```

Or upload it to file.io:

```
curl -F "file=boot.efi" https://file.io
```

:heavy_exclamation_mark: Important: If you have only wifi, please install those below, if not you'll not able to use `wifi-menu` after reboot

```
pacman -S iw wireless_tools wpa_supplicant dialog
```

Exit chroot and reboot (back to mac world)

```
exit
reboot
```

## Make arch duo bootable

When OSX loaded. Using Disk Utility to format `/dev/sda3` (128MB HFS+ we have created before) with Journaled format.

Then create this file structure:

```
|___mach_kernel
|___System
    |___Library
        |___CoreServices
            |___SystemVersion.plist
            |___boot.efi              (Is the file we'v copy, upload in the previous step)
```

Edit SystemVersion.plist content:

```xml
<xml version="1.0" encoding="utf-8"?>
<plist version="1.0">
<dict>
    <key>ProductBuildVersion</key>
    <string></string>
    <key>ProductName</key>
    <string>Linux</string>
    <key>ProductVersion</key>
    <string>Arch Linux</string>
</dict>
</plist>
```

To make arch auto boot with out holding `alt/option`, run this command:

```
sudo bless --device /dev/disk0s4 --setBoot
```

Now when reboot and welcome to arch world :hearts:

# Install to make arch usable

## Set tty default font

:warning: Font in retina screen is very small, so this maybe the first step you do after install arch. If you can see the font clearly, you don't have to do this step.

Install terminus-font: `sudo pacman -S terminus-font`

create file `/etc/vconsole.conf` with content:

```
FONT=ter-132b
```

Install these default fonts (to make browser look suckless):

```
yay -S ttf-dejavu ttf-linux-libertine ttf-mac-fonts ttf-ms-fonts ttf-opensans ttf-ubuntu-font-family ttf-symbola
```

## Install drivers

```
sudo pacman -S xf86-video-intel xf86-input-libinput mesa
```

## Install require packages

### Required packages for display

Using pacman to install all these packages:

xorg-server: graphical server
xorg-xinit: starts graphical server
xorg-xrandr: resize & rotate utility for X
xorg-xwininfo: querying windows infomation on X server
xorg-xprop: detecting window properties tool
xorg-xdpyinfo: display infomation unity
xorg-xset: to configure keyboard
xorg-xev: indentifying keycodes
xcompmgr: remove screen-tearing, add shadow, transparent...
xwallpaper: set wallpaper
arandr: UI for screen adjustment

### Required package for window manager

Install these packages with pacman:

i3-gaps:  main graphical user interface and window manager
i3status: generate status for i3bar
i3blocks: status bar
dmenu: minimal app launcher

## Keyboard

If you want normally keyboard instead macbook's keyboad, install [this patch](https://aur.archlinux.org/packages/hid-apple-patched-git-dkms/)

Read more in [wiki](https://wiki.archlinux.org/index.php/Apple_Keyboard#Use_a_patch_to_hid-apple)

After install run these commands:

```
mkinitcpio -p linux
sudo modprobe -r hid_apple; sudo modprobe hid_apple
```

Remapping other keys, i like to switch caplocks with esc keys

Create `~/.Xmodmap` file with this content:

```
# Swap ESC and CAPLOCKS
remove Lock = Caps_Lock
remove Esc   = Escape
keysym Escape = Caps_Lock
keysym Caps_Lock = Escape
add Lock = Caps_Lock
add Esc   = Escape
```

then add:

```
if [ -f $HOME/.Xmodmap ]; then
  xmodmap ~/.Xmodmap
fi
```

to `~/.xinitrc`

## Screen display

~/.Xresources
```
Xft.dpi: 96
Xft.autohint: 0
Xft.lcdfilter:  lcddefault
Xft.hintstyle:  hintfull
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb
```

~/.xinitrc
```
# Adjust keyboard typematic delay and rate
xset r rate 270 30

# Start X at 220 DPI
xrandr --output eDP1 --mode 2880x1800 --scale 0.5x0.5
xrandr --dpi 220

# Serve Xmodmap
if [ -f $HOME/.Xmodmap ]; then
  xrdb -merge ~/.Xmodmap
fi

# Merge & load configuration from .Xresources
if [ -f $HOME/.Xresources ]; then
  xrdb -merge ~/.Xresources
fi

# Let QT and GTK autodetect retina screen and autoadjust
export QT_AUTO_SCREEN_SCALE_FACTOR=1
export GDK_SCALE=2
export GDK_DPI_SCALE=0.5

# Finally start i3wm
exec i3
```

For multiple display view [this](https://www.reddit.com/r/archlinux/comments/b8wuai/beginner_needs_help_setting_and_scaling_xrandr/)

## Window management

Edit `~/.bash_profile`, add this:

```
if [ -z "$DISPLAY" ] && [ -n "$XDG_VTNR" ] && [ "$XDG_VTNR" -eq 1 ]; then
  exec startx
fi
```

## Better network management

Install these:

```
sudo pacman -S wpa_supplicant wireless_tools networkmanager
```

For GUI support, install:

```
sudo pacman -S nm-connection-editor network-manager-applet
```

Start NetworkManager service and disable default dhcpcd service to prevent conflict:

```
sudo systemctl enable NetworkManager.service
sudo systemctl enable wpa_supplicant.service
sudo systemctl disable dhcpcd.service
```

Start the service

```
sudo systemctl start NetworkManager.service
```

## Trackpad

Install [xf86-input-mtrack-git](https://aur.archlinux.org/packages/xf86-input-mtrack-git/) to get trackpad gestures like in osx:

A few more config to make touchpad and mouse work better:

Natural scrolling:

/etc/X11/xorg.conf.d/30-touchpad.conf
```
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "NaturalScrolling" "true"
    Option "ClickMethod" "clickfinger"
    Option "AccelProfile" "flat"
    Option "TransformationMatrix" "1 0 0 0 1 0 0 0 0.3"
EndSection
```

Natural scrolling for external mouse:


/etc/X11/xorg.conf.d/30-pointer.conf
```
Section "InputClass"
    Identifier "pointer"
    Driver "libinput"
    MatchIsPointer "on"
    Option "NaturalScrolling" "true"
    Option "AccelProfile" "flat"
    Option "TransformationMatrix" "1 0 0 0 1 0 0 0 0.3"
EndSection
```

## Sound

Install: `pacman -S alsa-utils pulseaudio`

Create `/etc/modprobe.d/snd_hda_intel.conf` with content:

```
# Switch audio output from HDMI to PCH and Enable sound chipset powersaving
options snd-hda-intel index=1,0 power_save=1
```

Then restart and use `speaker-test -c 2` to test:

To adjust volume:

```
amixer set Master 2+
amixer set Master 2-
```

## Fan

Use [mbpfan](https://github.com/dgraziotin/mbpfan#arch-linux)
 yay mbpfan
```

Config

/etc/mbpfan.conf
```
[general]
# put the *lowest* value of "cat /sys/devices/platform/applesmc.768/fan*_min"
min_fan_speed = 2000
# put the *highest* value of "cat /sys/devices/platform/applesmc.768/fan*_max"
max_fan_speed = 6100

# try ranges 50-63, default is 63
low_temp = 50
# try ranges 58-66, default is 66
high_temp = 65
# take highest number returned by
# "cat /sys/devices/platform/coretemp.*/hwmon/hwmon*/temp*_max"
# divide by 1000
max_temp = 84
# default is 1 seconds
polling_interval = 1
```

Then run these commands:

```
systemctl enable mbpfan.service
systemctl start mbpfan.service
```

## Screen backlight
/etc/udev/rules.d/backlight.rules
Use [light](https://github.com/haikarainen/light)

```
light -S 50  # sets brightness to 50%
light -U 10 # decrease by 10%
light -A 10 # increase by 10%
```

## Keyboard backlight

Use [kbdlight](https://github.com/WhyNotHugo/kbdlight)

Then use this to adjust keyboard backlight:

```
kbdlight up [<percentage>]|down [<percentage>]|off|max|get|set <value>
```

## Webcam

https://github.com/aur-packages/bcwc-pcie-git

# Improvement

## Improve DHCP connection speed

In `/etc/dhcpcd.conf` add to the end:

```
# Disable IP ARP checking
noarp
```

## Turn on firewall

Run these commands:

```
sudo pacman -S ufw
sudo systemctl enable ufw
sudo ufw enable
```

## Enable Trim for SSD

Run these commands:

```
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

## Fixing lid closing to suspend

In `/etc/systemd/logind.conf`, add those at bottom:

```
HandlePowerKey=suspend
HandleLidSwitch=suspend
```

## Power Saving for Intel chip

Thermald is a deamon regulating the CPU speed, when your CPU runs too hot.

```
yaourt -S thermald
sudo systemctl enable thermald
sudo systemctl start thermald

sudo pacman -S tlp
sudo systemctl enable tlp.service
sudo systemctl enable tlp-sleep.service
sudo systemctl start tlp.service
sudo systemctl start tlp-sleep.service

sudo pacman -S cpupower
```

Mine is MJLQ2 so set this

/etc/default/cpupower
```
max_freq="2.2GHz"
```








# Install arch triple boot

## Make space for Arch

Some boot keycodes:
-------------------

  Hold alt while booting to display the startup manager (you can select your boot device here).
  Hold ctrl while selecting a boot device (choose your EFI partition) to permanently boot from that one.
  Hold cmd+R while starting to start a recovery. 
  If you have a OS X recovery partition, this is a recovery wizard (restore from time machine, reinstall OS X, disk   utility). If you don’t, it starts a complete internet recovery ((almost?) restore factory defaults). It even   restores your EFI NVRAM thingy, I think.

Boot the live environment
Note: Arch Linux installation images do not support Secure Boot. You will need to disable Secure Boot to boot the installation medium. If desired, Secure Boot can be set up after completing the installation.
Point the current boot device to the one which has the Arch Linux installation medium. Typically it is achieved by pressing a key during the POST phase, as indicated on the splash screen. Refer to your motherboard's manual for details.
When the installation medium's boot loader menu appears, select Arch Linux install medium and press Enter to enter the installation environment.
Tip:
The installation image uses GRUB for UEFI and syslinux for BIOS booting. Use respectively e or Tab to enter the boot parameters. See README.bootparams for a list.
A common example of manually defined boot parameter would be the font size. For better readability on HiDPI screens—when they are not already recognized as such—using fbcon=font:TER16x32 can help. See HiDPI#Linux console (tty) for a detailed explanation.
You will be logged in on the first virtual console as the root user, and presented with a Zsh shell prompt.
To switch to a different console—for example, to view this guide with Lynx alongside the installation—use the Alt+arrow shortcut. To edit configuration files, mcedit(1), nano and vim are available. See pkglist.x86_64.txt for a list of the packages included in the installation medium.

Set the console keyboard layout and font
The default console keymap is US. Available layouts can be listed with:

# ls /usr/share/kbd/keymaps/**/*.map.gz
To set the keyboard layout, pass a corresponding file name to loadkeys(1), omitting path and file extension. For example, to set a German keyboard layout:

# loadkeys de-latin1
Console fonts are located in /usr/share/kbd/consolefonts/ and can likewise be set with setfont(8). For example, to use one of the largest fonts suitable for HiDPI screens, run:

# setfont ter-132b
Verify the boot mode
To verify the boot mode, check the UEFI bitness:

# cat /sys/firmware/efi/fw_platform_size
If the command returns 64, then system is booted in UEFI mode and has a 64-bit x64 UEFI. If the command returns 32, then system is booted in UEFI mode and has a 32-bit IA32 UEFI; while this is supported, it will limit the boot loader choice to systemd-boot. If the file does not exist, the system may be booted in BIOS (or CSM) mode. If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual.

Connect to the internet
To set up a network connection in the live environment, go through the following steps:

Ensure your network interface is listed and enabled, for example with ip-link(8):
# ip link
For wireless and WWAN, make sure the card is not blocked with rfkill.
Connect to the network:
Ethernet—plug in the cable.
Wi-Fi—authenticate to the wireless network using iwctl.
Mobile broadband modem—connect to the mobile network with the mmcli utility.
Configure your network connection:
DHCP: dynamic IP address and DNS server assignment (provided by systemd-networkd and systemd-resolved) should work out of the box for Ethernet, WLAN, and WWAN network interfaces.
Static IP address: follow Network configuration#Static IP address.
The connection may be verified with ping:
# ping archlinux.org
Note: In the installation image, systemd-networkd, systemd-resolved, iwd and ModemManager are preconfigured and enabled by default. That will not be the case for the installed system.
Update the system clock
In the live environment systemd-timesyncd is enabled by default and time will be synced automatically once a connection to the internet is established.

Use timedatectl(1) to ensure the system clock is accurate:

# timedatectl
Partition the disks
When recognized by the live system, disks are assigned to a block device such as /dev/sda, /dev/nvme0n1 or /dev/mmcblk0. To identify these devices, use lsblk or fdisk.

# fdisk -l
Results ending in rom, loop or airoot may be ignored.

Tip: Check that your NVMe drives and Advanced Format hard disk drives are using the optimal logical sector size before partitioning.
The following partitions are required for a chosen device:

One partition for the root directory /.
For booting in UEFI mode: an EFI system partition.
If you want to create any stacked block devices for LVM, system encryption or RAID, do it now.

Use fdisk or parted to modify partition tables. For example:

# fdisk /dev/the_disk_to_be_partitioned
Note:
If the disk does not show up, make sure the disk controller is not in RAID mode.
If the disk from which you want to boot already has an EFI system partition, do not create another one, but use the existing partition instead.
Swap space can be set on a swap file for file systems supporting it.
Example layouts
UEFI with GPT
Mount point	Partition	Partition type	Suggested size
/mnt/boot1	/dev/efi_system_partition	EFI system partition	At least 300 MiB. If multiple kernels will be installed, then no less than 1 GiB.
[SWAP]	/dev/swap_partition	Linux swap	More than 512 MiB
/mnt	/dev/root_partition	Linux x86-64 root (/)	Remainder of the device
Other mount points, such as /mnt/efi, are possible, provided that the used boot loader is capable of loading the kernel and initramfs images from the root volume. See the warning in Arch boot process#Boot loader.
BIOS with MBR
Mount point	Partition	Partition type	Suggested size
[SWAP]	/dev/swap_partition	Linux swap	More than 512 MiB
/mnt	/dev/root_partition	Linux	Remainder of the device
See also Partitioning#Example layouts.

Format the partitions
Once the partitions have been created, each newly created partition must be formatted with an appropriate file system. See File systems#Create a file system for details.

For example, to create an Ext4 file system on /dev/root_partition, run:

# mkfs.ext4 /dev/root_partition
If you created a partition for swap, initialize it with mkswap(8):

# mkswap /dev/swap_partition
Note: For stacked block devices replace /dev/*_partition with the appropriate block device path.
If you created an EFI system partition, format it to FAT32 using mkfs.fat(8).

Warning: Only format the EFI system partition if you created it during the partitioning step. If there already was an EFI system partition on disk beforehand, reformatting it can destroy the boot loaders of other installed operating systems.
# mkfs.fat -F 32 /dev/efi_system_partition
Mount the file systems
Mount the root volume to /mnt. For example, if the root volume is /dev/root_partition:

# mount /dev/root_partition /mnt
Create any remaining mount points (such as /mnt/boot) and mount the volumes in their corresponding hierarchical order.

Tip: Run mount(8) with the --mkdir option to create the specified mount point. Alternatively, create it using mkdir(1) beforehand.
For UEFI systems, mount the EFI system partition:

# mount --mkdir /dev/efi_system_partition /mnt/boot
If you created a swap volume, enable it with swapon(8):

# swapon /dev/swap_partition
genfstab(8) will later detect mounted file systems and swap space.

Installation
Select the mirrors
Packages to be installed must be downloaded from mirror servers, which are defined in /etc/pacman.d/mirrorlist. On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.

Install essential packages
Note: No software or configuration (except for /etc/pacman.d/mirrorlist) get carried over from the live environment to the installed system.
Use the pacstrap(8) script to install the base package, Linux kernel and firmware for common hardware:

# pacstrap -K /mnt base linux linux-firmware
Tip:
You can substitute linux with a kernel package of your choice, or you could omit it entirely when installing in a container.
You could omit the installation of the firmware package when installing in a virtual machine or container.
The base package does not include all tools from the live installation, so installing more packages may be necessary for a fully functional base system. To install other packages or package groups, append the names to the pacstrap command above (space separated) or use pacman to install them while chrooted into the new system. In particular, consider installing:

userspace utilities for file systems that will be used on the system—for the purposes of e.g. file system creation and fsck,
utilities for accessing and managing RAID or LVM if they will be used on the system,
specific firmware for other devices not included in linux-firmware (e.g. sof-firmware for onboard audio, linux-firmware-marvell for Marvell wireless and any of the multiple firmware packages for Broadcom wireless),
software necessary for networking (e.g. a network manager or a standalone DHCP client, authentication software for Wi-Fi, ModemManager for mobile broadband connections),
a text editor,
packages for accessing documentation in man and info pages: man-db, man-pages and texinfo.
For comparison, packages available in the live system can be found in pkglist.x86_64.txt.

Configure the system
Fstab
Generate an fstab file (use -U or -L to define by UUID or labels, respectively):

# genfstab -U /mnt >> /mnt/etc/fstab
Check the resulting /mnt/etc/fstab file, and edit it in case of errors.

Chroot
Change root into the new system:

# arch-chroot /mnt
Time zone
Set the time zone:

# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
Run hwclock(8) to generate /etc/adjtime:

# hwclock --systohc
This command assumes the hardware clock is set to UTC. See System time#Time standard for details.

Localization
Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed UTF-8 locales. Generate the locales by running:

# locale-gen
Create the locale.conf(5) file, and set the LANG variable accordingly:

/etc/locale.conf
LANG=en_US.UTF-8
If you set the console keyboard layout, make the changes persistent in vconsole.conf(5):

/etc/vconsole.conf
KEYMAP=de-latin1
Network configuration
Create the hostname file:

/etc/hostname
yourhostname
Complete the network configuration for the newly installed environment. That may include installing suitable network management software, configuring it if necessary and enabling its systemd unit so that it starts at boot.

Initramfs
Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap.

For LVM, system encryption or RAID, modify mkinitcpio.conf(5) and recreate the initramfs image:

# mkinitcpio -P
Root password
Set the root password:

# passwd
Boot loader
Choose and install a Linux-capable boot loader. If you have an Intel or AMD CPU, enable microcode updates in addition.

Reboot
Exit the chroot environment by typing exit or pressing Ctrl+d.

Optionally manually unmount all the partitions with umount -R /mnt: this allows noticing any "busy" partitions, and finding the cause with fuser(1).

Finally, restart the machine by typing reboot: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation medium and then login into the new system with the root account.

Post-installation
See General recommendations for system management directions and post-installation tutorials (like creating unprivileged user accounts, setting up a graphical user interface, sound or a touchpad).

For a list of applications that may be of interest, see List of applications.

