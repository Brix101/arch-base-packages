# Arch Install Guide

## Internet connection

### Ethernet

To make sure you have an internet connection, you have to ask Mr. Google:

```bash
    ip a
    ping 8.8.8.8
```

### Wifi card

use [iwtcl](https://man.archlinux.org/man/community/iwd/iwctl.1.en)

## Add best Arch mirrors

To install arch you have to download packages. It's a good idea to download
them from the best connection mirror.
select your country mirrorlist or remove country

```bash
   pacman -Sy
   pacman -S reflector
   reflector --country SG --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

> Color & Parallel Download Settings (Optional)

To enable color & adjust pacman parallel download

```bash
   nano /etc/pacman.conf
```

### Uncomment

```
   #Color
   #ParallelDownloads = 5
```

## Update the system clock

```bash
   timedatectl set-ntp true
   timedatectl status
```

## Synchronize mirror

```bash
   pacman -Syy
```

## Partition disk

Your primary disk will be known from now on as `sda`. You can check if
this is really your primary disk:

```bash
   lsblk
```

Feel free to adapt the rest of the guide to `sda` or any other if you
want to install Arch on a secondary hard drive.

This guide will use a 250GB hard disk and will have only Arch Linux installed.
You'll create 5 partitions of the disk (feel free to suit this to your needs).

- `/dev/sda1` boot partition (+512M).
- `/dev/sda2` data partition (remaining disk space).

You're going to start by removing all the previous partitions and creating
the new ones.

```bash
   gdisk /dev/sda
```

This interactive CLI program allows you to enter commands for managing your HD.
I'm going to show you only the commands you need to enter.

#### Clear partitions table

```bash
   Command: O
   Y
```

#### EFI partition (BOOT)

```bash
   Command: n
   ENTER
   ENTER
   +512M
   EF00
   Command: c
   Command: 1
   Enter name: BOOT
```

#### Root partition (ROOT)

```bash
   Command: N
   ENTER
   ENTER
   ENTER
   ENTER
   Command: c
   Command: 2
   Enter name: ROOT
```

#### Save changes and exit

```bash
   Command: w
   Y
```

### Format partitions

```bash
   mkfs.vfat -n BOOT /dev/sda1
   mkfs.btrfs -L ROOT /dev/sda2
```

### Mount partitions

```bash
   mount /dev/sda2 /mnt
```

### Btrfs Create Subvol

```bash
   cd /mnt

   btrfs su cr @
   btrfs su cr @home
   btrfs su cr @snapshots
   btrfs su cr @var_log

   umount /mnt
```

### Mount partitions on btrfs partitions

```bash
   mount -o compress=zstd:1,noatime,subvol=@ /dev/sda2 /mnt

   mkdir -p /mnt/{boot/efi,home,.snapshots,var/log}

   mount -o compress=zstd:1,noatime,subvol=@home /dev/sda2 /mnt/home
   mount -o compress=zstd:1,noatime,subvol=@snapshots /dev/sda2 /mnt/.snapshots
   mount -o compress=zstd:1,noatime,subvol=@var_log /dev/sda2 /mnt/var/log

   mount /dev/sda1 /mnt/boot/efi
```

If you run the `lsblk` command you should see something like this:

```
   NAME   MAJ:MIN RM   SIZE  RO TYPE MOUNTPOINT
   sda     254:0   0  232.9G  0 disk
   ├─sda1  254:1   0    512G  0 part /mnt/boot
   └─sda2  254:2   0  231.5G  0 part /mnt/.snapshots
                                     /mnt/var/log
                                     /mnt/home
                                     /mnt
```

## Installation

### Install Arch packages

```bash
   pacstrap -K /mnt base base-devel linux linux-firmware vim nano git reflector rsync
```

### Generate fstab file

```bash
   genfstab -U /mnt >> /mnt/etc/fstab
```

## Add basic configuration

### Enter the new system

```bash
   arch-chroot /mnt
```

### Configure timezone

For this example I'll use "Asia/Manila", but adapt it to your zone.

```bash
   ln -sf /usr/share/zoneinfo/Asia/Manila /etc/localtime
   hwclock --systohc


   reflector --country SG --latest 5 --sort rate --save /etc/pacman.d/mirrorlist

   pacman -Syy
```

### Language-related settings

```bash
   vim /etc/locale.gen
```

Now you have to uncomment the language of your choice, for example
`en_US.UTF-8 UTF-8`.

```bash
   locale-gen
```

Set locale:

```bash
   echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### Choose a name for your computer

Assuming your computer is known as `my-hostname`:

```bash
   echo my-hostname > /etc/hostname
```

### Adding content to the hosts file

```bash
   vim /etc/hosts
```

And add this content to the file:

```
   127.0.0.1    localhost
   ::1          localhost
   127.0.1.1    my-hostname.localdomain   my-hostname
```

> Replace "my-hostname" with your hostname.

## Install packages

Enable color & adjust pacman parallel download

```bash
   nano /etc/pacman.conf
```

### Uncomment

```
   #Color
   #ParallelDownloads = 5
```

Clone and Install packages

```bash
   git clone https://github.com/Brix101/arch-install-guide.git
   cd arch-i3-btrfs-guid
   pacman -S --needed - < LIST.txt
```

### Update root password

```bash
   passwd
```

### Add your user

Assuming your chosen user is "brix":

```bash
   useradd -m -G network,docker,power,rfkill,users,video,storage,audio,wheel,input -s /bin/zsh brix
   passwd brix
```

### Grant root access to our user

```bash
   visudo /etc/sudoers
```

If you prefer not to be prompted for a password every time you run a command
with "sudo" privileges you need to uncomment this line:

```
   %wheel ALL=(ALL) NOPASSWD: ALL
```

Or if you prefer the standard behavior of most Linux distros you need to
uncomment this line:

```
   %wheel ALL=(ALL) ALL
```

### Install bootloader

```bash
   grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinux
   grub-mkconfig -o /boot/grub/grub.cfg
```

### Add btrfs on mkinitcpio

add on binaries for system repair

```
   BINARIES=(btrfs)
```

### Mkinitcpio

```bash
   mkinitcpio -p linux
```

You can replace "ArchLinux" with the id of your choice.

### Enable SSH, NetworkManager and DHCP

These services will be started automatically when the system boots up.

```bash
   systemctl enable avahi-daemon
   systemctl enable bluetooth
   systemctl enable haveged
   systemctl enable fstrim.timer
   systemctl enable sshd
   systemctl enable dhcpcd
   systemctl enable NetworkManager
   systemctl enable reflector.timer
   systemctl enable upower
   systemctl enable acpid
   systemctl enable firewalld
```

### Final steps

```bash
   exit
   umount -a
   reboot
```

## Post-install configuration

Now your computer has restarted and in the login window on the tty1 console you
can log in with the root user and the password chosen in the previous step.

### Login into newly created user

```
   # su - thinkpad
   $ xdg-user-dirs-update
```

### Install AUR package manager

In this guide we'll install [yay](https://github.com/Jguer/yay) as the
AUR package manager. More about [AUR](https://aur.archlinux.org/).

TL;DR AUR is a Community-driven package repository.

```
   $ mkdir Sources
   $ cd Sources
   $ git clone https://aur.archlinux.org/paru.git
   $ cd paru
   $ makepkg -si
```

# `Install desktop manager /display manager `

### The coolest Pacman

If you want to make Pacman look cooler you can edit the configuration file and
uncomment the `Color` option and add just below the `ILoveCandy` option.

```
   $ sudo nvim /etc/pacman.conf
```

### PulseAudio applet

If you want to manage your computer's volume from a small icon in the systray:

```
   $ yay -S pa-applet-git
```

### Manage Bluetooth

```
   $ sudo pacman -S bluez bluez-utils blueman
   $ sudo systemctl enable bluetooth
```

### Improve laptop battery consumption

```
   $ sudo pacman -S tlp tlp-rdw powertop acpi
   $ sudo systemctl enable tlp
   $ sudo systemctl mask systemd-rfkill.service
   $ sudo systemctl mask systemd-rfkill.socket
```

If your laptop is a ThinkPad, also run this:

```
   $ sudo pacman -S acpi_call
```

## i3-gaps related steps

### Install graphical environment and i3

```
   $ sudo pacman -S xorg-server xorg-apps xorg-xinit
   $ sudo pacman -S i3-wm i3blocks i3lock i3Status numlockx
```

### Install display manager

The display manager allows us to log in to the system graphically and also
to automate the startup of some services.
[LightDM](https://wiki.archlinux.org/index.php/LightDM) is one of the most
lightweight display managers.

```
   $ sudo pacman -S lightdm lightdm-gtk-greeter --needed
   $ sudo systemctl enable lightdm
```

### Install some basic fonts

```
   $ sudo pacman -S noto-fonts ttf-ubuntu-font-family ttf-dejavu ttf-freefont
   $ sudo pacman -S ttf-liberation ttf-droid ttf-roboto terminus-font
```

### Install some useful tools on i3

```
   $ sudo pacman -S rxvt-unicode ranger rofi dmenu --needed
```

### Install some GUI programs

```
   $ sudo pacman -S firefox vlc --needed
```

### Apply previous settings

```
   $ sudo reboot
```

## Rice i3-gaps

"Rice" is how we know to make visual improvements and customizations
to the desktop and its programs.

### Console

The program we use the most is the console emulator. I use
[kitty](https://github.com/kovidgoyal/kitty) as a replacement for urxvt,
the default terminal emulator on i3. However, these improvements can be
applied to any terminal emulator.

#### zsh

`zsh` is an alternative to `bash` shell I particularly love.
You can also have a look at the `fish` shell, because even though I haven't
tried it, it looks very cool.

```
   $ sudo pacman -S zsh
```

### LXAppearance

With LXAppearance you can change themes, icons, cursors or fonts.

```
   $ sudo pacman -S lxappearance
```

Once you have installed LXAppearance, you can start exploring the many possible
customization options by installing the great
[Arc theme](https://github.com/arc-design/arc-theme) and the
[Papirus](https://github.com/PapirusDevelopmentTeam/papirus-icon-theme) icons
theme.

```
   $ sudo pacman -S arc-gtk-theme
   $ sudo pacman -S papirus-icon-theme
```

### Customize LightDM

At this point you can also customize the look of LigthDM. You can blow your mind
by adding Papirus icons and Arc theme in LightDM, just by editing its config file.

```
   $ sudo nvim /etc/lightdm/lightdm-gtk-greeter.conf
```

In this file you have to add these lines

```
   [greeter]
   theme-name = Arc-Dark
   icon-theme-name = Papirus-Dark
   background = #2f343f
```

You can also custom the font with the same one you added in LXAppearance,
just by adding `font-name = Whatever` to this file.
