# Install Arch Linux

1.  Download arch iso image from https://www.archlinux.org/download/ and copy to a USB drive.

        # dd if=arch.iso of=/dev/sdb

1.  Connect the USB drive and boot from the Arch Linux ISO.

1.  Make sure the system is booted in UEFI mode.
    The following command should display the directory contents without error.

        ls /sys/firmware/efi/efivars

1.  Connect to the internet.
    A wired connection is preferred since it's easier to connect.
    [More info](https://wiki.archlinux.org/index.php/Installation_guide#Connect_to_the_internet)

1.  Run `fdisk` to create Linux partitions.

        fdisk /dev/<your-disk>

    If you have installed Windows, you already have a GPT partition table.
    Otherwise, create an empty GPT partition table using the `g` command.
    (**WARNING:** This will erase the entire disk.)

        # WARNING: This will erase the entire disk.

        Command (m for help): g
        Created a new GPT disklabel (GUID: ...).

    Create the EFI partition (`/dev/<your-disk-efi>`):

        Command (m for help): n
        Partition number: <Press Enter>
        First sector: <Press Enter>
        Last sector, +/-sectors or +/-size{K,M,G,T,P}: +100M

        Command (m for help): t
        Partition type or alias (type L to list all): uefi

    Create the Boot partition (`/dev/<your-disk-boot>`):

        Command (m for help): n
        Partition number: <Press Enter>
        First sector: <Press Enter>
        Last sector, +/-sectors or +/-size{K,M,G,T,P}: +512M

        Command (m for help): t
        Partition type or alias (type L to list all): linux

    Create the LUKS partition (`/dev/<your-disk-luks>`):

        Command (m for help): n
        Partition number: <Press Enter>
        First sector: <Press Enter>
        Last sector, +/-sectors or +/-size{K,M,G,T,P}: <Press Enter>

        Command (m for help): t
        Partition type or alias (type L to list all): linux

    Print the partition table using the `p` command and check that everything is OK:

        Command (m for help): p

    Write changes to the disk using the `w` command.
    (Make sure you know what you're doing before running this command).

        Command (m for help): w

1.  Format the EFI and Boot Partitions.

        mkfs.fat -F 32 /dev/<your-disk-efi>
        mkfs.ext4 /dev/<your-disk-boot>

1.  Set up the encrypted partition.
    You can choose any other name instead of `cryptlvm`.

        cryptsetup luksFormat /dev/<your-disk-luks>
        cryptsetup luksOpen /dev/<your-disk-luks> cryptlvm

1.  Create an LVM volume group.
    You can choose any other name instead of `vg0`.

        pvcreate /dev/mapper/cryptlvm
        vgcreate vg0 /dev/mapper/cryptlvm

1.  Create LVM partitions (logical volumes).

    We create logical volumes for swap, root (`/`), and optionnaly home (`/home`).
    Leave 256MiB of free space in the volume group because the `e2scrub` command requires
    the LVM volume group to have at least 256MiB of unallocated space to dedicate
    to the snapshot.

        lvcreate --size 8G vg0 --name swap
        lvcreate --size 100G vg0 --name root
        lvcreate -l +100%FREE vg0 --name home
        lvreduce --size -256M vg0/home

1.  Format logical volumes.

        mkswap /dev/vg0/swap
        mkfs.ext4 /dev/vg0/root
        mkfs.ext4 /dev/vg0/home

1.  Mount new filesystems.

        mount /dev/vg0/root /mnt
        mount --mkdir /dev/<your-disk-efi> /mnt/efi
        mount --mkdir /dev/<your-disk-boot> /mnt/boot
        mount --mkdir /dev/vg0/home /mnt/home
        swapon /dev/vg0/swap

1.  Install the base system.
    We also install some useful packages like `git`, `vim`, and `sudo`.

        pacstrap -K /mnt base base-devel linux linux-firmware e2fsprogs dhcpcd networkmanager sof-firmware git neovim man-db man-pages texinfo sudo zsh

⚠️ If you get errors due to key then do the following:

1.  Initialize _pacman_ keys and populate them:

        pacman-key --init
        pacman-key --populate

1.  Synchronize Arch keyring:

        archlinux-keyring-wkd-sync

1.  Generate `/etc/fstab`. This file can be used to define how disk partitions,
    various other block devices, or remote filesystems should be mounted into the
    filesystem.

        genfstab -U /mnt >> /mnt/etc/fstab

1.  Enter the new system.

        arch-chroot /mnt /bin/bash

1.  Set TimeZone.

        # See available timezones:
        ls /usr/share/zoneinfo/

        # Set timezone:
        ln -s /usr/share/zoneinfo/France/Paris /etc/localtime

1.  Run hwclock(8) to generate `/etc/adjtime`.

        hwclock --systohc

1.  Set Locale.

        sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen
        locale-gen
        echo LANG=en_US.UTF-8 > /etc/locale.conf

1.  Create `/etc/vconsole.conf` and set the following variables according to your preferred language:

        KEYMAP=de_CH-latin1
        FONT=Lat2-Terminus16

1.  Set hostname.

        echo yourhostname > /etc/hostname

1.  Edit `/etc/hosts` like this:

        127.0.0.1 localhost
        ::1 localhost
        127.0.1.1 yourhostname

1.  Create a user.

        useradd -m -G wheel,storage,power,video,audio,input --shell /bin/bash yourusername
        passwd yourusername
        sed -i 's/^# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/' /etc/sudoers

1.  Configure `mkinitcpio` with modules needed to create the initramfs image.

        pacman -S lvm2
        vim /etc/mkinitcpio.conf
        sed -i 's/^HOOKS=($.*$filesystems$.*$)$/HOOKS=(\1encrypt lvm2 filesystems\2)/' /etc/mkinitcpio.conf

    Recreate the initramfs image:

        mkinitcpio -P

1.  Setup GRUB.

          pacman -S grub efibootmgr
          grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB

    In `/etc/default/grub` edit the line GRUB_CMDLINE_LINUX as follows.
    Don't forget to replace `/dev/<your-disk-luks>` with the appropriate path.

          GRUB_CMDLINE_LINUX="cryptdevice=/dev/<your-disk-luks>:cryptlvm root=/dev/vg0/root"

    Now generate the main GRUB configuration file:

          grub-mkconfig -o /boot/grub/grub.cfg

1.  Install `networkmanager` package and enable `NetworkManager` service
    to ensure you have Internet connectivity after rebooting.

        pacman -S networkmanager
        systemctl enable NetworkManager

1.  Exit new system and unmount all filesystems.

        exit
        umount -R /mnt
        swapoff -a

    Arch is now installed 🎉. Reboot.

        reboot

# Notes

## Backup LUKS Header

It is important to make a backup of LUKS header so that you can access your data in case of emergency
(if your LUKS header somehow gets damaged).

Create a backup file:

    sudo cryptsetup luksHeaderBackup /dev/<your-disk-luks> --header-backup-file luks-header-backup-$(date -I)

Store the backup file in a safe place, such as a USB drive.
If something bad happens, you can restore the backup header:

    sudo cryptsetup luksHeaderRestore /dev/<your-disk-luks> --header-backup-file /path/to/backup_header_file

### Drivers

**Intel**:

    sudo pacman -S mesa intel-media-driver libva-intel-driver vulkan-intel

**NVIDIA**:

    sudo pacman -S nvidia

VirtualBox:

    sudo pacman -S virtualbox-guest-utils

### Beautify Pacman

    sudo nvim /etc/pacman.conf

Uncomment `Color` and add below it `ILoveCandy`.

If you have a good internet connection, you can uncomment the option `ParallelDownloads = 5`.

### GTK Dark Theme

To make GTK applications (e.g. _nemo_) use dark theme, execute the following commands:

    gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark'
    gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

## Run LARBS to install Desktop Environment

You can now install a Desktop Environment manually or run larbs.sh to install mine (see [dotfiles](https://github.com/sudo-Tiz/dotfiles))

    sudo su
    curl -LO https://raw.githubusercontent.com/sudo-Tiz/LARBRE/main/larbs.sh
    sh larbs.sh
