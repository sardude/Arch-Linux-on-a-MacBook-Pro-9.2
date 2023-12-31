<h1 align="center">
    <a>
    <img src="https://raw.githubusercontent.com/sardude/Arch-Linux-on-a-MacBook-Pro-9.2/main/asset/macbook-pro.png" width="400">
  </a>
  <br> macbook pro 9,2  >>  +   +   <br>
</h1>

## ✨ Features

⚠️Batterry Issue: I'v used arch and artix for couple year on my MBP, everything is fine, smooth, but there is a problem, because OSX is apple stuff, so they make it very well, when using linux on Macbook, you must accept the risk. The only problem i've ever faced is that my battery degrades very quickly.

Here are some sneak picks of what **Mac** can work:
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

- [Install arch triple boot](#install-arch-triple-boot)
    - [Make space for Arch]
    - [Make installer USB]
    - [Boot it up]
    - [Connect wifi](#connect-wifi)
    - [disk](#disk)
    - [Format & Mount](#format-and-mount)
    - [Install base packages & generate fstab](#install-base-packages-&-generate-fstab)
    - [System config](#system-config)
    - [Install the bootloader](#install-the-bootloader)
    - [Make arch triple bootable](#make-arch-triple-bootable)
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


Install arch triple boot
<details>
  <summary>Install arch triple boot</summary>  
    # Make space for Arch
    from macbookpro; open terminal

➜ diskutil list

/dev/disk0 (internal, physical):

|#:  |TYPE NAME                       |SIZE          |IDENTIFIER|
|--- |---                             |---           |---|
|0:  |GUID_partition_scheme           |*500.1 GB     |disk0|
|1:  |EFI NO NAME                     |209.7 MB       |disk0s1|
|2:  |Microsoft Basic Data darkstar   |454.9 GB   |disk0s2|
|3:  |Linux Filesystem      |45.0 GB |disk0s3|

/dev/disk1 (internal, physical):

|#:  |TYPE NAME                       |SIZE          |IDENTIFIER|
|--- |---                             |---           |---|
|0:  |GUID_partition_scheme           |*250.1 GB     |disk1|
|1:  | EFI EFI                     |209.7 MB      |disk1s1|
|2:  |Apple_APFS Container disk2   |184.0 GB      |disk1s3|
|3:  |Microsoft Basic Data OS                |65.8 GB       |disk1s3|


Use Disk Utility Partition feature to add new Partition for Arch, follow [this guide](https://wiki.archlinux.org/index.php/Mac#Arch_Linux_with_OS_X_or_other_operating_systems)

Or if you already know how to use Disk Utility, then create a partition with FAT32 format.


# Make installer USB

Download arch's iso [here](https://www.archlinux.org/download/)

Find usb by using `diskutil list`, then:

```
# Assume usb disk is /dev/diskX
diskutil unmountDisk /dev/diskX
dd if=path/to/arch.iso of=/dev/diskX bs==1m
```

# Boot it up

Hold bold `alt/option` when system bootup, then choose boot from USB

:warning: get larger font, [connect to wifi](connect-wifi) and run these commands:

```
sudo pacman -Sy terminus-font
```

```
setfont ter-p18n
```

# Connect wifi

Use `wifi-menu` then choose wifi to connect, then check connection with:

```
ping -c 3 google.com
```

# Disk
<details>
<summary>Partitions</summary>
Assuming you have this when run `fdisk -l`:

```zsh title="Terminal"
 fdisk -list
```
|Size    |Type              |Description|
|---     |---               |---        |
|200MB   |EFI System        |This is required in order to make boot with OSX|
|423.7G   |Microsoft basic data  |NTFS file system|
|41.9G     |Linux filesystem       |root|

</details>

# Format and mount
<details>
<summary>format</summary>
Now let format and mount partition:
    
```zsh title="Terminal"
 mkfs.ext4 /dev/sda4
```
</details>

<details>
<summary>mount</summary>
Now let format and mount partition:
    
```zsh title="Terminal"
mount /dev/sda /mnt
```

```zsh title="Terminal"
mkdir /mnt/boot && mount /dev/sda4 /mnt/boot
```
</details>


# Install base packages & generate fstab

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

# System config

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

# Make arch triple bootable

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
</details>
Now when reboot and welcome to arch world :hearts:

<details>
    <summary>Install to make arch usable</summary>  
    <details>
        <summary> tty default font</summary> 
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
</details>

<details>
 <summary>Install drivers</summary>  
```
sudo pacman -S xf86-video-intel xf86-input-libinput mesa
```
</details>


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
</details>

#improvement
<details>
  <summary>improvement</summary>  

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

</details>

Post-installation
See General recommendations for system management directions and post-installation tutorials (like creating unprivileged user accounts, setting up a graphical user interface, sound or a touchpad).

For a list of applications that may be of interest, see List of applications.

