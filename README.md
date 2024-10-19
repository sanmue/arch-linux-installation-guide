# Arch Linux Installation Guide

## Intro

This Arch Linux installation guide may be seen as a practical example following the [Arch Wiki installation guide](https://wiki.archlinux.org/title/Installation_guide).

Some examples of points covered in this guide:

- UEFI/GPT (with EFI partition) or BIOS/GPT
- GRUB bootloader
- Btrfs filesystem
- snapper
- snapper-rollback (AUR)
- AUR helper (paru)
- encryption (optional)
- swap file or zram (no hibernation)
- Display Driver AMD, Intel, (Nvidia) (but needs checking the Arch Wiki)
- Desktop Environment
- Firewall (firewalld)
- [Chaotic-AUR](https://aur.chaotic.cx/) as an example for an unofficial user repository
- additional font installation
- Virtualization (Qemu/KVM)

Using snapper + [snapper-rollback (AUR)](https://aur.archlinux.org/packages/snapper-rollback) (instead of [Timeshift](https://wiki.archlinux.org/title/Timeshift)) inspired by [mpr's video](https://www.youtube.com/watch?v=maIu1d2lAiI) because supports a more flexible btrfs subvolume layout.

[mjkstra's](https://gist.github.com/mjkstra) ['Modern Arch linux installation guide'](https://gist.github.com/mjkstra/96ce7a5689d753e7a6bdd92cdc169bae) was a kicking inspiration doing this guide.

## Table of Content

- [Arch Linux Installation Guide](#arch-linux-installation-guide)
  - [Intro](#intro)
  - [Table of Content](#table-of-content)
  - [=== Pre-installation steps ===](#-pre-installation-steps-)
  - [Keyboard layout and font](#keyboard-layout-and-font)
  - [Excursus: Connecting to the machine to be installed via ssh](#excursus-connecting-to-the-machine-to-be-installed-via-ssh)
  - [Verify Boot Mode](#verify-boot-mode)
  - [Internet connection](#internet-connection)
  - [Update the system clock](#update-the-system-clock)
  - [Disk partitioning](#disk-partitioning)
    - [List available devices](#list-available-devices)
    - [UEFI/GPT](#uefigpt)
      - [Create Partitions](#create-partitions)
      - [Current partition table UEFI/GPT](#current-partition-table-uefigpt)
    - [BIOS/GPT](#biosgpt)
      - [Create partitions](#create-partitions-1)
      - [Current partition table BIOS/GPT](#current-partition-table-biosgpt)
    - [Remark regarding partitioning (UEFI and BIOS)](#remark-regarding-partitioning-uefi-and-bios)
    - [Encryption](#encryption)
  - [Format partitions](#format-partitions)
  - [Btrfs subvolumes](#btrfs-subvolumes)
    - [Mount root partiton](#mount-root-partiton)
    - [Create subvolumes](#create-subvolumes)
    - [List subvolumes](#list-subvolumes)
    - [Unmount root subvolume](#unmount-root-subvolume)
  - [Mount the file systems](#mount-the-file-systems)
    - [Mount partition to install the system](#mount-partition-to-install-the-system)
    - [Create folders for the subvolumes (mountpoints)](#create-folders-for-the-subvolumes-mountpoints)
    - [Mount the efi partition](#mount-the-efi-partition)
    - [Mount the subvolumes](#mount-the-subvolumes)
    - [Mount btrfsroot for 'snapper-rollback'](#mount-btrfsroot-for-snapper-rollback)
    - [Mount options](#mount-options)
    - [Show current block device list](#show-current-block-device-list)
  - [Installation](#installation)
    - [Select the mirrors](#select-the-mirrors)
    - [Install essential packages](#install-essential-packages)
  - [Configure the system](#configure-the-system)
    - [Fstab (filesystem table)](#fstab-filesystem-table)
      - [UEFI/GPT with encryption](#uefigpt-with-encryption)
      - [UEFI/GPT without encryption](#uefigpt-without-encryption)
      - [BIOS/GPT with encryption](#biosgpt-with-encryption)
    - [Chroot (Change root into new system)](#chroot-change-root-into-new-system)
    - [Time](#time)
    - [Localization](#localization)
  - [Network configuration](#network-configuration)
    - [hostname file](#hostname-file)
    - [hosts file](#hosts-file)
    - [Network Management](#network-management)
  - [Initramfs (Create initial ramdisk environment)](#initramfs-create-initial-ramdisk-environment)
    - [Check mkinitcpio.conf](#check-mkinitcpioconf)
      - [Modules](#modules)
      - [Hooks](#hooks)
    - [Creating a new initramfs](#creating-a-new-initramfs)
  - [Boot loader](#boot-loader)
    - [Grub](#grub)
      - [Config default grub](#config-default-grub)
      - [Installation GRUB](#installation-grub)
      - [Configuration](#configuration)
  - [SET PASSWORD FOR root](#set-password-for-root)
  - [Reboot](#reboot)
  - [=== Post-installation steps ===](#-post-installation-steps-)
  - [System administration](#system-administration)
    - [Users and groups](#users-and-groups)
      - [Add a new user](#add-a-new-user)
      - [User directories](#user-directories)
      - [Modify sudoers file (optional)](#modify-sudoers-file-optional)
    - [INSERTION: ssh again from another machine](#insertion-ssh-again-from-another-machine)
    - [INSERTION: Snapper and btrfs snapshots](#insertion-snapper-and-btrfs-snapshots)
      - [Install packages](#install-packages)
      - [create snapper config file](#create-snapper-config-file)
      - [Config snapper snapshots](#config-snapper-snapshots)
        - [Permissions](#permissions)
        - [Automatic snapshots and cleanup](#automatic-snapshots-and-cleanup)
      - [Snapper-rollback (AUR)](#snapper-rollback-aur)
        - [Rollback example](#rollback-example)
    - [INSTERTION: Config zram as swap](#instertion-config-zram-as-swap)
      - [Disable zswap](#disable-zswap)
      - [Using zram-genrator](#using-zram-genrator)
      - [Optimizing swap on zram](#optimizing-swap-on-zram)
    - [INSTERTION: Desktop Environment](#instertion-desktop-environment)
    - [Security](#security)
      - [Enforcing strong passwords with pam\_pwquality](#enforcing-strong-passwords-with-pam_pwquality)
      - [CPU](#cpu)
      - [Memory](#memory)
      - [Storage](#storage)
      - [User setup](#user-setup)
      - [Restricting root](#restricting-root)
      - [Mandatory access control](#mandatory-access-control)
      - [Kernel hardening](#kernel-hardening)
      - [Sandboxing applications](#sandboxing-applications)
      - [Network and firewalls](#network-and-firewalls)
        - [Firewalls](#firewalls)
        - [Kernel parameters](#kernel-parameters)
        - [SSH](#ssh)
        - [DNS](#dns)
        - [Proxies](#proxies)
        - [Managing TLS certificates](#managing-tls-certificates)
    - [Service management](#service-management)
      - [Retrieve and filter the latest Pacman mirror list via Reflector](#retrieve-and-filter-the-latest-pacman-mirror-list-via-reflector)
      - [avahi, haveged, upower](#avahi-haveged-upower)
      - [Enable further services](#enable-further-services)
    - [System maintenance](#system-maintenance)
      - [pacman-contrib](#pacman-contrib)
      - [Performance and download speeds](#performance-and-download-speeds)
        - [Parallel Downloads](#parallel-downloads)
        - [Sorting Mirrors](#sorting-mirrors)
      - [Upgrading the system](#upgrading-the-system)
        - [Alerts during upgrade](#alerts-during-upgrade)
        - [Avoid certain pacman commands](#avoid-certain-pacman-commands)
        - [Deal promptly with new configuration files](#deal-promptly-with-new-configuration-files)
        - [Restart or reboot after upgrades](#restart-or-reboot-after-upgrades)
        - [Revert broken updates](#revert-broken-updates)
      - [Removing unused packages (orphans)](#removing-unused-packages-orphans)
      - [Clean the filesystem](#clean-the-filesystem)
      - [Refresh pacman files database](#refresh-pacman-files-database)
  - [Package management](#package-management)
    - [pacman](#pacman)
      - [Downloading packages in parallel](#downloading-packages-in-parallel)
      - [Cleaning the package cache](#cleaning-the-package-cache)
      - [Tips and tricks](#tips-and-tricks)
    - [Repositories](#repositories)
      - [Using 32-bit applications](#using-32-bit-applications)
      - [Unofficial user repositories](#unofficial-user-repositories)
    - [Mirrors](#mirrors)
    - [Arch Build System](#arch-build-system)
    - [Arch User Repository (AUR)](#arch-user-repository-aur)
      - [AUR helpers](#aur-helpers)
  - [Booting](#booting)
  - [Graphical user interface](#graphical-user-interface)
    - [Display drivers](#display-drivers)
      - [AMD](#amd)
      - [Intel](#intel)
      - [Nvidia](#nvidia)
    - [Desktop environments](#desktop-environments)
      - [Example installation: Gnome](#example-installation-gnome)
  - [Power management](#power-management)
  - [Multimedia](#multimedia)
    - [Sound system](#sound-system)
    - [Codecs and containers](#codecs-and-containers)
  - [Networking](#networking)
    - [Setting up a firewall](#setting-up-a-firewall)
  - [Input devices](#input-devices)
  - [Optimization](#optimization)
  - [System services](#system-services)
    - [File index and search](#file-index-and-search)
      - [locate](#locate)
    - [Local mail delivery](#local-mail-delivery)
    - [Printing](#printing)
      - [CUPS](#cups)
  - [Appearance](#appearance)
    - [Fonts](#fonts)
      - [Installing fonts](#installing-fonts)
    - [GTK and Qt themes](#gtk-and-qt-themes)
  - [Console improvements](#console-improvements)
    - [Alternative shells](#alternative-shells)
    - [Compressed files](#compressed-files)
      - [tar](#tar)
  - [Virtualization (Qemu/KVM)](#virtualization-qemukvm)
    - [Installating packages for virtualization](#installating-packages-for-virtualization)
    - [Client](#client)
    - [SPICE](#spice)
  - [Troubleshooting](#troubleshooting)
    - [Failed to synchronize all databases (unable to lock database)](#failed-to-synchronize-all-databases-unable-to-lock-database)

## === Pre-installation steps ===

Boot the live installation media.

## Keyboard layout and font

- <https://wiki.archlinux.org/title/Installation_guide#Set_the_console_keyboard_layout_and_font>
&nbsp;

- `localectl list-keymaps` # show all available keymaps, look for yours
- `loadkeys de-latin1` # load keymap
- `setfont ter-132b` # optional # set (bigger) font # `ter-122b` # `setfont - d` (double size)
  - available fonts: `ls /usr/share/kbd/consolefonts/ | less`

## Excursus: Connecting to the machine to be installed via ssh

- `passwd` # set passwort for current (root) user
&nbsp;

- `pacman -Sy` # sync + refresh package database
- `pacman -S openssh rsync sudo vim nano` # install some packages
- `systemctl start sshd` # start ssh service
- `systemctl status sshd` # check status of ssh service
&nbsp;

- `vim /etc/ssh/ssh_config` # config ssh # or use 'nano' / your prefered editor instead of 'vim'
  - uncomment: `PasswordAuthentication yes`
  - add: `PermitRootLogin yes`

(Or create a user, add to group wheel, allow wheel to execute any command and login with this user)

- `ip a` # look for the current IP address

ssh from your computer into the machine to be installed:

- `ssh -o IdentitiesOnly=yes root@IP-ADDRESS` # ssh into currently booted live environment

## Verify Boot Mode

- <https://wiki.archlinux.org/title/Installation_guide#Verify_the_boot_mode>
&nbsp;

- `cat /sys/firmware/efi/fw_platform_size`
  - If this command prints `64` or `32` then you are in UEFI mode
  - else prints:

    ```text
    cat: /sys/firmware/efi/fw_platform_size: No such file or directory
    ```

    which means you are in BIOS Boot mode

## Internet connection

- <https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet>
&nbsp;

- `ip link` # Ensure your network interface is listed and enabled / UP
  - Wi-Fi: authenticate to a wireless network using `iwctl`
  - Mobile broadband modem: connect to a mobile network with the `mmcli` utility

## Update the system clock

- <https://wiki.archlinux.org/title/Installation_guide#Update_the_system_clock>
&nbsp;

- `timedatectl`

## Disk partitioning

- <https://wiki.archlinux.org/title/Installation_guide#Partition_the_disks>
- <https://wiki.archlinux.org/title/Installation_guide#Example_layouts>
- <https://wiki.archlinux.org/title/Partitioning#Example_layouts>
- <https://wiki.archlinux.org/title/Partitioning#Choosing_between_GPT_and_MBR>
- <https://wiki.archlinux.org/title/EFI_system_partition>
- <https://wiki.archlinux.org/title/GRUB>
- <https://wiki.archlinux.org/title/Zram#Usage_as_swap>

> :memo: **Note:**
> If you want to create any stacked block devices for LVM, **system encryption** (see further below: 'Encryption') or RAID, do it now.
>
> - <https://wiki.archlinux.org/title/Install_Arch_Linux_on_LVM>
> - <https://wiki.archlinux.org/title/Dm-crypt>
> - <https://wiki.archlinux.org/title/Btrfs#Encryption>
>   - Users can encrypt the partition before running mkfs.btrfs. ...

### List available devices

`fdisk -l`

```text
Disk /dev/vda: 25 GiB, 26843545600 bytes, 52428800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[...]
```

### UEFI/GPT

#### Create Partitions

`fdisk /dev/vda`

- `,` means: press enter to accept the default value

```text
g           # gpt partition table

n
,, +1G      # efi partition, size 1 GB
t           # change type
1           # 'EFI System'

n
,,,         # system partition (takes entire rest of the disk)
t           # change type
,
23          # 'Linux root (x86-64)'

p           # print current partition table
w           # write to disk
```

#### Current partition table UEFI/GPT

```text
Disk /dev/vda: 25 GiB, 26843545600 bytes, 52428800 sectors
[...]
Disk identifier: 1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA

Device       Start      End  Sectors Size Type
/dev/vda1     2048  2099199  2097152   1G EFI System
/dev/vda2  2099200 52426751 50327552  24G Linux root (x86-64)
```

### BIOS/GPT

If you are installing on old hardware with a BIOS not supporting GPT there is a [fix](https://wiki.archlinux.org/title/Partitioning#Tricking_old_BIOS_into_booting_from_GPT).
Or use [MBR](https://wiki.archlinux.org/title/Partitioning#BIOS/MBR_layout_example) instead of [GPT](https://wiki.archlinux.org/title/Partitioning#BIOS/GPT_layout_example).

#### Create partitions

`fdisk /dev/vda`

- `,` (comma) means: press enter to accept the default value

```text
g           # gpt partition table

n
,, +1M      # BIOS boot partition
t           # change type
4           # 'BIOS boot'

n
,, -1M      # system partition # takes entire rest of the disk, excpet 1 MB)
t           # change type
,
23          # 'Linux root (x86-64)'

p           # print current partition table
w           # write to disk
```

#### Current partition table BIOS/GPT

```text
Disk /dev/vda: 35 GiB, 37580963840 bytes, 73400320 sectors
[...]
Disk identifier: 41232CF9-9C30-4ED1-B284-52D65B53A2FA

Device     Start      End  Sectors Size Type
/dev/vda1   2048     4095     2048   1M BIOS boot
/dev/vda2   4096 73396223 73392128  35G Linux root (x86-64)
```

### Remark regarding partitioning (UEFI and BIOS)

- Note your partition table, as you need the device paths later
- We will use a swap file instead of a seperate swap partition in this guide
  - The swap file will be on the encrypted root partition in its own btrfs subvolume
- If you opt for a swap partition, you may want to [encrypt](https://wiki.archlinux.org/title/Swap#Swap_encryption) it
- You could consider using [zram](https://wiki.archlinux.org/title/Zram) instead of (or [additionally to](https://wiki.archlinux.org/title/Zram#Enabling_a_backing_device_for_a_zram_block)) a swap partition or swap file

### Encryption

- <https://wiki.archlinux.org/title/Dm-crypt>
- <https://wiki.archlinux.org/title/Btrfs#Encryption>
- <https://wiki.archlinux.org/title/GRUB#Encrypted_/boot>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Btrfs_on_top_of_dmcrypt>

Encrypting our root partition:

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encrypting_devices_with_LUKS_mode>
- ~~`cryptsetup luksFormat /dev/vda2`~~
- `cryptsetup luksFormat --pbkdf pbkdf2 /dev/vda2`
  - <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encryption_options_for_LUKS_mode>
    - Argon2id (cryptsetup default) and Argon2i PBKDFs are not supported ([GRUB bug #59409](https://savannah.gnu.org/bugs/?59409)), only PBKDF2 is
    - as of 10/2024
  - <https://savannah.gnu.org/bugs/index.php?55093>
  - <https://lists.gnu.org/archive/html/grub-devel/2024-05/msg00166.html>

Open the LUKS encrypted root partition:

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Unlocking/Mapping_LUKS_partitions_with_the_device_mapper>
- `cryptsetup open /dev/vda2 cryptroot`
  - Once opened, the root partition device address would be `/dev/mapper/cryptroot` instead of the partition `/dev/vda2`; device mapper name is: `cryptroot`
  - you should make a note, we will need this info later

`lsblk -p`

```text
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/loop0                  7:0    0 790.3M  1 loop  /run/archiso/airootfs
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    25G  0 disk  
├─/dev/vda1               254:1    0     1G  0 part  
└─/dev/vda2               254:2    0    24G  0 part  
  └─/dev/mapper/cryptroot 253:0    0    24G  0 crypt 
```

&nbsp;

> :warning:
> With current setup, when starting your machine you will be asked twice for a password for decrypting: 1x bootloader, 1x cryptroot.

## Format partitions

- <https://wiki.archlinux.org/title/Installation_guide#Format_the_partitions>
- <https://wiki.archlinux.org/title/Btrfs#File_system_creation>
- <https://wiki.archlinux.org/title/EFI_system_partition#Format_the_partition>
&nbsp;

- EFI partition (skip for BIOS/GPT)
  - `mkfs.fat -F32 -n EFI /dev/vda1` # EFI partition # adjust to your device path
- root partition
  - if you enrypted the root partition:
    - `mkfs.btrfs -L ROOT /dev/mapper/cryptroot`
    - the device `/dev/mapper/cryptroot` can then be mounted like any other partition
  - if you did not enrypt root partition:
    - `mkfs.btrfs -L ROOT /dev/vda2` # main partition # adjust to your device path

## Btrfs subvolumes

- <https://wiki.archlinux.org/title/Btrfs#Subvolumes>
- <https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout>
- <https://btrfs.readthedocs.io/en/latest/Subvolumes.html>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Subvolumes>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Layout>

### Mount root partiton

- if you enrypted the root partition:
  - `mount /dev/mapper/cryptroot /mnt` # mount btrfs root # adjust to your device path
- If you did not enrypt root partition:
  - `mount /dev/vda2 /mnt` # mount btrfs root # adjust to your device path

### Create subvolumes

- in a single command:
  - `btrfs subvolume create /mnt/{@,@home,@varlog,@swap,@snapshots}`
- or individual:
  - `btrfs subvolume create /mnt/@` # root
  - `btrfs subvolume create /mnt/@home` # home folder
  - `btrfs subvolume create /mnt/@varlog` # log files
  - `btrfs subvolume create /mnt/@swap` # for swap file # can be skipped if you want to have only zram
  - `btrfs subvolume create /mnt/@snapshots` # snapshot folder
- ... and others:

> - <https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout>
>
> Tip:
>
> - Consider creating subvolumes for other directories that contain data you do not want to include in snapshots and rollbacks of the @ subvolume, such as `/var/cache`, `/var/spool`, `/var/tmp`, `/var/lib/machines` (systemd-nspawn), `/var/lib/docker` (Docker), `/var/lib/postgres` (PostgreSQL), and other data directories under `/var/lib/`.
>   - e.g.: `/var/lib/libvirt/images` (Virtual machine images), `/srv`, `/tmp`, `/opt`
> - But the pacman database in `/var/lib/pacman` must stay on the root subvolume (@).

### List subvolumes

- `btrfs subvolume list /mnt`

 ```text
 ID 256 gen 10 top level 5 path @
 ID 257 gen 10 top level 5 path @home
 ID 258 gen 10 top level 5 path @varlog
 ID 259 gen 10 top level 5 path @swap
 ID 260 gen 11 top level 5 path @snapshots
 ```

Subvolume-ID 5 (top level 5) = btrfsroot ("/")

### Unmount root subvolume

- `umount /mnt` # unmount btrfs root

## Mount the file systems

- <https://wiki.archlinux.org/title/Installation_guide#Mount_the_file_systems>
- <https://wiki.archlinux.org/title/Btrfs#Compression>
- <https://btrfs.readthedocs.io/en/latest/Administration.html#mount-options>
- <https://btrfs.readthedocs.io/en/latest/Administration.html#notes-on-generic-mount-options>
- <https://wiki.archlinux.org/title/Btrfs#Mounting_subvolumes>
- <https://wiki.archlinux.org/title/EFI_system_partition#Typical_mount_points>
- <https://wiki.archlinux.org/title/Btrfs#Swap_file>
- (<https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation>)
&nbsp;

### Mount partition to install the system

- if you enrypted the root partition:
  - `mount /dev/mapper/cryptroot -o subvol=/@,compress=zstd,noatime /mnt` # Mount the subvolume on which we want to install the system # adjust to your device path
  - ~~`mount /dev/mapper/cryptroot -o subvolid=256,compress=zstd,noatime /mnt`~~
- If you did not enrypt root partition:
  - `mount /dev/vda2 -o subvolid=/@,compress=zstd,noatime /mnt` # Mount the subvolume on which we want to install the system # adjust to your device path
  - ~~`mount /dev/vda2 -o subvolid=256,compress=zstd,noatime /mnt`~~

### Create folders for the subvolumes (mountpoints)

- in a single command:
  - `mkdir -p /mnt/{efi,home,var/log,swap,.snapshots,.btrfsroot}`
- or in individual commands:
  - `mkdir -p /mnt/efi`
  - `mkdir -p /mnt/home`
  - `mkdir -p /mnt/var/log`
  - `mkdir -p /mnt/swap`
  - `mkdir -p /mnt/.snapshots`
  - `mkdir -p /mnt/.btrfsroot` # for 'snapper-rollback'
- ... and for all other subvolumes created above

### Mount the efi partition

skip if BOOT / GPT

- `mount /dev/vda1 /mnt/efi` # adjust to your device path

### Mount the subvolumes

Adjust to your device path.

> :warning: If root partition is encrypted:
>
> - Replace `/dev/vda2` with `/dev/mapper/cryptroot`
>   - `cryptroot` = device mapper name used for enryption / opening enrypted root partition

- home
  - encryption: `mount /dev/mapper/cryptroot -o subvol=/@home,compress=zstd,noatime /mnt/home`
  - no encryption: `mount /dev/vda2 -o subvol=/@home,compress=zstd,noatime /mnt/home`
- log files
  - encryption: `mount /dev/mapper/cryptroot -o subvol=/@varlog,compress=zstd,noatime /mnt/var/log`
  - no encryption: `mount /dev/vda2 -o subvol=/@varlog,compress=zstd,noatime /mnt/var/log`
- Swap file:
  - encryption: `mount /dev/mapper/cryptroot -o subvol=/@swap,compress=zstd,noatime /mnt/swap`
  - no encryption: `mount /dev/vda2 -o subvol=/@swap,compress=zstd,noatime /mnt/swap`
  - `btrfs filesystem mkswapfile --size 4g --uuid clear /mnt/swap/swapfile` # Create a 4 GB swap file # currently without Hibernation
  - `swapon /mnt/swap/swapfile` # activate the swap file
- snapshots
  - encryption: `mount /dev/mapper/cryptroot -o subvol=/@snapshots,compress=zstd,noatime /mnt/.snapshots`
  - no encryption: `mount /dev/vda2 -o subvol=/@snapshots,compress=zstd,noatime /mnt/.snapshots`
- ... and all other subvolumes you may have created

### Mount btrfsroot for 'snapper-rollback'

- encryption: `mount /dev/mapper/cryptroot -o subvol=/,compress=zstd,noatime /mnt/.btrfsroot`
- ~~encryption: `mount /dev/mapper/cryptroot -o subvolid=5,compress=zstd,noatime /mnt/.btrfsroot`~~
- no encryption: `mount /dev/vda2 -o subvol=/,compress=zstd,noatime /mnt/.btrfsroot`
- ~~no encryption: `mount /dev/vda2 -o subvolid=5,compress=zstd,noatime /mnt/.btrfsroot`~~

### Mount options

> :memo: **Note:**
> Mount options (`-o ...`) used above are for SSDs / NVMEs and btrfs filesystem

### Show current block device list

UEFI/GPT with encryption:
`lsblk -p`

```text
/dev/loop0                  7:0    0 790.3M  1 loop  
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    25G  0 disk  
├─/dev/vda1               254:1    0     1G  0 part  /mnt/efi
└─/dev/vda2               254:2    0    24G  0 part  
  └─/dev/mapper/cryptroot 253:0    0    24G  0 crypt /mnt/.btrfsroot
                                                     /mnt/.snapshots
                                                     /mnt/swap
                                                     /mnt/var/log
                                                     /mnt/home
                                                     /mnt
```

UEFI/GPT, no encryption:
`lsblk`

```text
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0 790.3M  1 loop /run/archiso/airootfs
sr0     11:0    1   1.1G  0 rom  /run/archiso/bootmnt
vda    254:0    0    25G  0 disk 
├─vda1 254:1    0     1G  0 part /mnt/efi
└─vda2 254:2    0    24G  0 part /mnt/.btrfsroot
                                 /mnt/.snapshots
                                 /mnt/swap
                                 /mnt/var/log
                                 /mnt/home
                                 /mnt
```

BIOS/GPT, with encryption:
`lsblk -p`

```text
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/loop0                  7:0    0 790.3M  1 loop  /run/archiso/airootfs
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    35G  0 disk  
├─/dev/vda1               254:1    0     1M  0 part  
└─/dev/vda2               254:2    0    35G  0 part  
  └─/dev/mapper/cryptroot 253:0    0    35G  0 crypt /mnt/.btrfsroot
                                                     /mnt/.snapshots
                                                     /mnt/swap
                                                     /mnt/var/log
                                                     /mnt/home
                                                     /mnt
```

## Installation

### Select the mirrors

In the live system, after connecting to the internet, `reflector` updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.
`/etc/pacman.d/mirrorlist`

> :memo: **Note: reflector**
> A Python 3 module and script to retrieve and filter the latest Pacman mirror list

Update mirror list manually + set preferred countries (optional):

- `reflector --verbose --age 12 --protocol https --sort rate --country 'Austria,France,Germany,Switzerland' --save /etc/pacman.d/mirrorlist`

You could activate parallel downloads by pacman:
`vim /etc/pacman.conf` and umcomment the line with `ParallelDownlodads = 5`

### Install essential packages

The minimum would be:
`pacstrap -K /mnt base linux linux-firmware`
and install the rest while chrooted into the new system (e.g. see: Post-installation -> System administration -> after "Users and groups").

But lets already install some more package here.
We have UEFI, grub bootmanager, btrfs, using networkmanager, ...:

- `pacstrap -K /mnt base linux linux-firmware linux-headers   grub efibootmgr btrfs-progs mtools   base-devel sudo man-db man-pages texinfo   networkmanager   reflector   openssh vim rsync terminus-font`

- depending on processor add:
  - for AMD: `amd-ucode`
  - for Intel: `intel-ucode`
- for audio support add:
  - `pipewire` and maybe also:
  - `pipewire-alsa pipewire-pulse pipewire-jack wireplumber`
  - first you could install just `pipewire`, you can still install the others later if needed
- for bluetooth add: `bluez bluez-utils`
  - and mybe `sof-firmware` for onboard audio, depending on your system
- you could also add the another:
  - e.g. for zsh add: `zsh`

All packages above together:
`pacstrap -K /mnt   base linux linux-firmware linux-headers   grub efibootmgr btrfs-progs mtools   base-devel sudo man-db man-pages texinfo   networkmanager   reflector   openssh vim rsync terminus-font   amd-ucode intel-ucode   pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber sof-firmware   bluez bluez-utils   zsh`

## Configure the system

- <https://wiki.archlinux.org/title/Installation_guide#Configure_the_system>

### Fstab (filesystem table)

- `genfstab -U /mnt >> /mnt/etc/fstab` # `-U`: use UUIDs
- `cat /mnt/etc/fstab` # check

> :warning: You should delete `subvolid=<subvolid>` in the fstab:
>
> - <https://bbs.archlinux.org/viewtopic.php?id=299085>
> - <https://github.com/archlinux/arch-install-scripts/commit/added92801fac6b2aafe0362e0deca00da68ec19>:
>   - Having only one of subvol= and subvolid= is enough for mounting a btrfs subvolume.
>   - And having subvolid= set prevents things like 'snapper rollback' to work, as it
>   - updates the subvolume in-place, leaving subvol= unchanged with a different subvolid.
> - <https://btrfs.readthedocs.io/en/latest/Administration.html#mount-options>
>   - `subvolid=<subvolid>`
>   - If both subvolid and subvol are specified, they must point at the same subvolume, otherwise the mount will fail.
> - e.g. change:
>   - `UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,`**subvolid=256,**`subvol=/@ 0 0`
> - to:
>   - `UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@ 0 0`
> &nbsp;
>
> :warning: The UUIDs shown at 'with enryption' or 'without encrypion' would normally match.
> They are only different because it was a seperate new installation both times with new random UUIDs generated.

#### UEFI/GPT with encryption

```text
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@ 0 0

# /dev/vda1 LABEL=EFI
UUID=4584-08C8       /efi       vfat       rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro 0 2

# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /home      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@home 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /var/log   btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@varlog 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /swap      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@swap 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /.snapshots btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@snapshots 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /.btrfsroot btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvolid=5,subvol=/ 0 0

/swap/swapfile       none       swap       defaults   0 0
```

#### UEFI/GPT without encryption

```text
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /          btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvol=/@ 0 0

# /dev/vda1 LABEL=EFI
UUID=6155-8708       /efi       vfat       rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro 0 2

# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /home      btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvol=/@home 0 0

# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /var/log   btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvol=/@varlog 0 0

# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /swap      btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvol=/@swap 0 0

# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /.snapshots btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvol=/@snapshots 0 0

# /dev/vda2 LABEL=ROOT
UUID=a552a029-4972-4b3a-b2cd-21ddb5e2e870 /.btrfsroot btrfs      rw,noatime,compress=zstd:3,discard=async,space_cache=v2,subvolid=5,subvol=/ 0 0

/swap/swapfile       none       swap       defaults   0 0
```

#### BIOS/GPT with encryption

```text
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@ 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /home      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@home 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /var/log   btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@varlog 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /swap      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@swap 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /.snapshots btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@snapshots 0 0

# /dev/mapper/cryptroot LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /.btrfsroot btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvolid=5,subvol=/ 0 0

/swap/swapfile       none       swap       defaults   0 0
```

### Chroot (Change root into new system)

- `arch-chroot /mnt`

### Time

- <https://wiki.archlinux.org/title/System_time#Time_standard>
- <https://wiki.archlinux.org/title/System_time#Hardware_clock>
- <https://wiki.archlinux.org/title/System_time#System_clock>
- <https://wiki.archlinux.org/title/System_time#Time_zone>
- <https://wiki.archlinux.org/title/System_time#Time_synchronization>
- <https://wiki.archlinux.org/title/Systemd-timesyncd#Enable_and_start>
&nbsp;

- `tzselect` # set time zone interactively
- or manually, e.g. for Germany:
- `ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime` # set time zone # adjust to your location
~~or:~~
~~`timedatectl set-timezone Europe/Berlin`~~ # during chroot timedatectl will not work

Hardware clock:

- `hwclock --systohc` # sets the hardware clock from the system clock + update/generate `/etc/adjtime`

Start and enable systemd-timesyncd.service (when running in chroot):

- `systemctl enable systemd-timesyncd.service`

> :memo: **Note:** `timedatectl` not available in chroot
> error:
> System has not been booted with systemd as init system (PID 1). Can't operate.
> Failed to connect to bus: Host is down

~~System clock (a.k.a. the software clock):~~ # not available in chroot
~~`timedatectl` # read clock~~
~~`timedatectl set-ntp true` # synchronizing the system clock across the network (systemd-timesyncd)~~
~~`timedatectl status` # Check service~~
~~`timedatectl timesync-status` # check vorbose~~

### Localization

- <https://wiki.archlinux.org/title/Installation_guide#Localization>
&nbsp;

- `vim /etc/locale.gen` # uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 locales
- `locale-gen` # Generate the locales
&nbsp;

- `vim /etc/locale.conf` # set the LANG variable accordingly
  - `LANG=en_US.UTF-8` # insert this line in the empty new file
&nbsp;

- `vim /etc/vconsole.conf` # set the console keyboard layout (persistent)
  - `KEYMAP=de-latin1` # insert this line in the empty new file for german keyboard layout
  - ~~or~~
  - ~~`localectl set-keymap de-latin1` # corresoponding to timedatectl~~ # not available in chroot

## Network configuration

### hostname file

- `vim /etc/hostname` # Create the hostname file
  - `HOSTNAME` # insert desired hostname in this empty new file

### hosts file

- `vim /etc/hosts`
  - insert these lines into this emtpy new file, adjust HOSTNAME

```text
# Standard host addresses
127.0.0.1  localhost
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
# This host address
127.0.1.1  HOSTNAME
```

### Network Management

We installed "NetworkManager" (see: "Install essential packages")

- Enabling its systemd unit so that it starts at boot:
  - `systemctl enable NetworkManager.service`
- or the corresponding service, if you installed another one
  - e.g. dhcpcd: `systemctl enable dhcpcd.service`)

## Initramfs (Create initial ramdisk environment)

- <https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_mkinitcpio>
- <https://wiki.archlinux.org/title/Dm-crypt/System_configuration#mkinitcpio>
- <https://wiki.archlinux.org/title/Mkinitcpio#Common_hooks>
- <https://archlinux.org/packages/?name=cryptsetup>
- <https://wiki.archlinux.org/title/Dm-crypt/Specialties#The_encrypt_hook_and_multiple_disks>

### Check mkinitcpio.conf

#### Modules

- `vim /etc/mkinitcpio.conf`
- add `btrfs`: `MODULES=(btrfs)`

#### Hooks

For LVM, **system** encryption or RAID, modify mkinitcpio.conf(5) (`/etc/mkinitcpio.conf`) and recreate the initramfs image.

- Encryption (LUKS):
  - `vim /etc/mkinitcpio.conf`
  - add to `HOOKS`: **encrypt** before `filesystems`
    - results in e.g.: `HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block` **encrypt** `filesystems fsck)`

### Creating a new initramfs

- `mkinitcpio -P`

## Boot loader

- <https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader>

### Grub

- <https://wiki.archlinux.org/title/GRUB>
- <https://wiki.archlinux.org/title/GRUB#Generate_the_main_configuration_file>
- <https://wiki.archlinux.org/title/GRUB/Tips_and_tricks>
- <https://wiki.archlinux.org/title/Dm-crypt/System_configuration#cryptdevice>
- <https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_the_boot_loader>
- <https://wiki.archlinux.org/title/Kernel_parameters#GRUB>
- <https://wiki.archlinux.org/title/GRUB#GRUB_rescue_and_encrypted_/boot>

> :memo: **Note:** Seperate `/efi` directory from the `/boot` directory
> Boot related files are saved in folder `/boot`, which is a subfolder of root and therefore backed up in a snapshot
> If we roll back then the versions of the files in `/boot` will be consistent with the remaining filsystem
> Which would not necessarily be the case if boot is on the `/efi` partition

#### Config default grub

- <https://wiki.archlinux.org/title/GRUB#Encrypted_/boot>

If you enrypted the root partition (which includes `/boot` folder), we have to configure the boot loader:

- `vim /etc/default/grub`
- uncomment: `GRUB_ENABLE_CRYPTODISK=y`

We need some info (UUID of encryptet partition) for the next step:
`blkid`

```text
/dev/vda2: UUID="0daddb47-c687-4194-90e4-ac023ea72b5b" TYPE="crypto_LUKS" PARTUUID="96ebe15f-8ec3-4db4-8bd0-877798ecbb37"
[...]
/dev/mapper/cryptroot: LABEL="ROOT" UUID="7986ecc9-7656-4ccc-814f-e58f20e241a5" UUID_SUB="a27f7903-4fe4-41cb-9407-47192b592a68" BLOCK_SIZE="4096" TYPE="btrfs"
[...]
```

or more specific:

- UUID of encrypted partition: `blkid -o value -s UUID /dev/vda2`
  - `0daddb47-c687-4194-90e4-ac023ea72b5b`
- (UUID of decrypted partition: `blkid -o value -s UUID /dev/mapper/cryptroot`)
  - (`7986ecc9-7656-4ccc-814f-e58f20e241a5`)

To unlock the encrypted root partition at boot, set kernel parameters:

- search for the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`
  - which should look like this: `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"`
- add at the end e.g.:
  - `cryptdevice=UUID=0daddb47-c687-4194-90e4-ac023ea72b5b:cryptroot root=/dev/mapper/cryptroot`
    - *cryptdevice=UUID=deviceUuidOfEncryptedDevice:dmname root=pathOfDecryptedPartitionDmname*

#### Installation GRUB

UEFI/GPT:

- ~~`grub-install --target=x86_64-efi --efi-directory=/efi --boot-directory=/boot --bootloader-id=GRUB`~~
  - in this case it is not necessary to set `boot-directory` parameter (but would not be wrong either)
- `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB`

BIOS/GPT

- `grub-install --target=i386-pc /dev/vda`

#### Configuration

Create config file:

- `grub-mkconfig -o /boot/grub/grub.cfg` # Generate the main configuration file

## SET PASSWORD FOR root

- `passwd`

> :warning: Or you can not login after reboot !

## Reboot

> :warning: Check:  password set for root ?

- `exit` # Exit the chroot environment
- (`swapoff /mnt/swap/swapfile`)
- (`umount -R /mnt` # manually unmount all the partitions to check if the drive is busy)
  - also unmounts CDROM with installation iso
- `systemctl reboot`

* * *
* * *

## === Post-installation steps ===

- <https://wiki.archlinux.org/title/General_recommendations>
- <https://wiki.archlinux.org/title/List_of_applications>
&nbsp;

`login:` root
`Password:` As set a few steps above

## System administration

### Users and groups

#### Add a new user

- <https://wiki.archlinux.org/title/Users_and_groups#User_management>
- <https://wiki.archlinux.org/title/Users_and_groups#Changing_user_defaults>

- `useradd -m -G wheel USERNAME -s /bin/bash` # create user 'USERNAME', add to group 'wheel', shell would autamtically set to the default shell (should be `bash`), but I defined explicitly `bash`
- `passwd USERNAME` # set a password
&nbsp;

- optional / later: add user to some groups
  - `usermod -aG sys,adm,network,scanner,power,uucp,audio,lp,rfkill,video,storage,optical,users USERNAME`

- optional: change shell to another shell, e.g. zsh
  - `pacman -S --needed zsh` # install zsh
  - `usermod -s /bin/zsh USERNAME` # change shell to zsh

#### User directories

- When (currently) creating new users, there are no folders in its home folder (like Downloads, Documents, ...)
- When installing a Desktop Environment this will usually be done autamatically
- If you want them NOW, create them manually (as the user; or now as root: check/change permissions afterwards):
  - one command:
    - `mkdir -p /home/USERNAME/{Documents,Downloads,Music,Pictures,Templates,Videos}`
  - individual:
    - `mkdir -p /home/USERNAME/Documents`
    - `mkdir -p /home/USERNAME/Downloads`
    - `mkdir -p /home/USERNAME/Music`
    - `mkdir -p /home/USERNAME/Pictures`
    - `mkdir -p /home/USERNAME/Templates`
    - `mkdir -p /home/USERNAME/Videos`
  - ...

- <https://wiki.archlinux.org/title/Users_and_groups#Changing_user_defaults>
  - `/etc/skel`
- <https://wiki.archlinux.org/title/General_recommendations#User_directories>
  - assumes installed Desktop Environment

#### Modify sudoers file (optional)

- `EDITOR=vim visudo`
- Uncomment to allow members of group wheel to execute any command:
  - `%wheel ALL=(ALL:ALL) ALL`

### INSERTION: ssh again from another machine

- `vim /etc/ssh/ssh_config`
  - uncomment: `PasswordAuthentication yes`
&nbsp;

- `systemctl start sshd` # or permanantly: `systemctl enable --now sshd` # Start and / or enable sshd
- `systemctl status sshd` # check sshd status

Connect to VM / machine from your machine:

- `ssh -o IdentitiesOnly=yes USERNAME@IP-ADDRESS`

> :warning: If logged in as the newly created user:
> Prefix the commands in the next steps with `sudo`.

### INSERTION: Snapper and btrfs snapshots

- <https://wiki.archlinux.org/title/Btrfs#Snapshots>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Snapshots>

#### Install packages

- `sudo pacman -S --needed snapper snap-pac sudo base-devel`

#### create snapper config file

Quelle:

- <https://wiki.archlinux.org/title/Snapper#Configuration_of_snapper_and_mount_point>
&nbsp;

- `sudo umount /.snapshots`
- `sudo rm -r /.snapshots`
- `sudo snapper -c root create-config /` # create new config named 'root'
&nbsp;

- `sudo btrfs subvolume delete /.snapshots`
- `sudo mkdir /.snapshots`
- `sudo mount -a`
  - (or just: `sudo mount -o subvol=@snapshots /dev/sda2 /.snapshots` # adjust to your device path)

> :memo: **Note:**
> This will make all snapshots that snapper creates be stored outside of the @ subvolume, so that @ can easily be replaced anytime without losing the snapper snapshots.

#### Config snapper snapshots

##### Permissions

- `sudo chmod 750 /.snapshots` # give the folder 750 permissions
- `sudo chmod 750 /.btrfsroot` # also for the folder used by `snapper-rollback`
&nbsp;

Browse the snapshots directory (optional):

- <https://wiki.archlinux.org/title/Snapper#Access_for_non-root_users>
- <http://snapper.io/manpages/snapper-configs.html>
- <http://snapper.io/manpages/snapper.html>
  - PERMISSIONS section
&nbsp;

- `sudo chown -R :wheel /.snapshots` # optional # members of group 'wheel' can browse the snapshots directory
- or
- `sudo vim /etc/snapper/configs/root`
  - `ALLOW_GROUPS="wheel"` # groups allowed to work with config
  - `SYNC_ACL="yes"` # sync permissions of `ALLOW_GROUPS` (and `ALLOW_USERS`) to snapshots directory

##### Automatic snapshots and cleanup

- <https://wiki.archlinux.org/title/Snapper#Taking_snapshots>
- `sudo vim /etc/snapper/configs/root`
  - set: `TIMELINE_CREATE="no"` # optional # disable automatic timeline snapshots via cron daemon
  - (or: `sudo snapper -c root set-config "TIMELINE_CREATE=no"`)
  - We will only have automatic pre- and post-snapshots via `snap-pac`
&nbsp;

- <https://wiki.archlinux.org/title/Snapper#Set_snapshot_limits>
- `sudo sudo systemctl enable --now snapper-cleanup.timer` # periodically clean up older snapshots

#### Snapper-rollback (AUR)

Script to rollback snapper snapshots (comfortable via CLI of a booted system)

- <https://aur.archlinux.org/packages/snapper-rollback>
- <https://www.youtube.com/watch?v=maIu1d2lAiI>

> :memo: **Note:**
> If you can't boot the system anymore, you have to rollback manually via live cd
>
> TODO - out of scope / separate guide
>
> The following instructions can be used for orientation
>
> - <https://wiki.archlinux.org/title/Snapper#Restore_using_the_default_layout>
> - <https://www.dwarmstrong.org/btrfs-snapshots-rollbacks/>
>   - 11. System rollback the 'Arch Way'
>   - deviating start:
>     - boot Arch live iso
>     - mount btrfs root + arch-chroot

- `su USERNAME` # switch to the account of the new user, if not already done
- `cd` # go to user's home directory
- Download the sources for 'snapper-rollback' from AUR:
  - `curl -L -O https://aur.archlinux.org/cgit/aur.git/snapshot/snapper-rollback.tar.gz`
- `tar -xf snapper-rollback.tar.gz` # extract archive
- `cd snapper-rollback`
- `makepkg -sic` # build package, install dependencies, install (or upgrade) package, clean up leftover work files and directories
- `cd ..`
- `rm -rf snapper-rollback snapper-rollback.tar.gz` # delete the folder an file that is no longer required
&nbsp;

- `sudo btrfs subvolume list / | grep -e "@\?snapshots"` # list subvolumes with name 'snapshots'
- `sudo vim /etc/snapper-rollback.conf` # adjust config:
  - change mountpoint to: `mountpoint = /.btrfsroot` (adding '.' according to the name of our created folder)
  - no need to adjust the name of the snapshot subvolume, since we used `@snapshots`
  - when using 'archinstall' skript the name of the snapshot subvolume is `@.snapshots` and you would have to adjust it here (as of 10/2024)

##### Rollback example

- `sudo snapper list` # list snapshots
- `sudo snapper-rollback <snapshot-number> --dry-run` (just to see what would happen):

 ```text
 2024-09-30 10:30:10,562 - INFO - mv /.btrfsroot/@ /.btrfsroot/@2024-09-30T10:30
 2024-09-30 10:30:10,563 - INFO - btrfs subvolume snapshot /.btrfsroot/@.snapshots/<snapshot-number>/snapshot /.btrfsroot/@
 2024-09-30 10:30:10,563 - INFO - btrfs subvolume set-default /.btrfsroot/@
 2024-09-30 10:30:10,563 - INFO - [DRY-RUN MODE] Rollback to /.btrfsroot/@.snapshots/<snapshot-number>/snapshot complete. Reboot to finish
 ```

&nbsp;

> :memo: **Note:**
> <https://btrfs.readthedocs.io/en/latest/Subvolumes.html>:
> ... the subvolume that will be mounted by default, unless the default subvolume has been changed (see `btrfs subvolume set-default`).

- `sudo snapper-rollback <snapshot-number>` # really do the rollback now
- `sudo reboot`
&nbsp;

- <https://wiki.archlinux.org/title/Snapper#Single_snapshots>
- `sudo snapper -c root create -c number --description "after snapper-rollback <snapshot-number>"`

### INSTERTION: Config zram as swap

**Text extracts from the sources listed below:**
zram can be used for swap or as a general-purpose RAM disk.
zram creates a **compressed** block device in RAM.

**Spoiler**: If you have enough Ram / not much swapping, you do not need to concern yourself with the configuration of zram.

zram is particularly effective on machines that do not have much memory.
And / or has an advantage for devices that have flash-based memory, which has a limited lifespan due to write amplification.

**Usage as swap**: Initially the created zram block device does not reserve or use any RAM. Only as files need or want to be swapped out, they will be **compressed** and moved into the zram block device. The zram block device will then dynamically grow or shrink as required.
Even when assuming that zstd only achieves a conservative 1:2 compression ratio (real world data shows a common ratio of 1:3), zram will offer the advantage of being able to store more content in RAM than without memory compression.
So instead of having 4 GB of RAM, with zRAM you could utilize around 8-12 GB of effective system memory.
(Compression ratio depends on the data / intended use of your machine. Data that is difficult / can not be compressed would not benefit much from zram compared to data that can be easily compressed.)

The compression and decompression time is generally negligible for any relatively modern CPU (>= ~2015).

Using zram or zswap (last one is currently the default in Arch Linux) reduces swap usage, which effectively reduces the amount of wear placed on flash-based storage and makes it last longer. Using zram also results in significantly reduced I/O for Linux systems that require swapping.

- <https://wiki.archlinux.org/title/Zram>
- <https://wiki.archlinux.org/title/Swap#Compressed_block_device_in_RAM>
- <https://wiki.archlinux.org/title/Improving_performance#zram_or_zswap>
- <https://en.wikipedia.org/wiki/Zram>
- <https://fosspost.org/enable-zram-on-linux-better-system-performance>
- <https://www.maketecheasier.com/zram-zcache-zswap/>
- <https://man.archlinux.org/man/zram-generator.8>
- <https://man.archlinux.org/man/zram-generator.conf.5>
- <https://man.archlinux.org/man/zramctl.8>
- backing device
  - zram can be configured to push **incompressible** pages to a specified block device when under memory pressure.
  - Block device (e.g. swap partition):
    - <https://wiki.archlinux.org/title/Zram#Enabling_a_backing_device_for_a_zram_block>
  - Swap file - Using loopdev for zRAM writeback feature
    - <https://unix.stackexchange.com/questions/581243/using-loopdev-for-zram-writeback-feature>
    - <https://bbs.archlinux.org/viewtopic.php?pid=2087278#p2087278>
- <https://wiki.archlinux.org/title/Zswap#Toggling_zswap>
- <https://wiki.archlinux.org/title/Kernel_parameters#GRUB>
- <https://bbs.archlinux.org/viewtopic.php?id=286116>

#### Disable zswap

If the related zswap kernel feature remains enabled, it will prevent zram from being used effectively. ...

Disable zswap at runtime:

- login as root
- `echo 0 > /sys/module/zswap/parameters/enabled`

To disable zswap permanently (which is what we want), add `zswap.enabled=0` to your kernel parameters:

- `vim /etc/default/grub`
  - append the kernel option `zswap.enabled=0` between the quotes in the `GRUB_CMDLINE_LINUX_DEFAULT` line, separated by space character
  - e.g.:
  - `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=2ee5664a-fb17-4762-8e3d-d8e4b8823815:cryptroot root=/dev/mapper/cryptroot zswap.enabled=0"`
- re-generate the `grub.cfg` file:
  - `grub-mkconfig -o /boot/grub/grub.cfg`

Check if disabled:

- `grep -r . /sys/module/zswap/parameters/ | grep /enabled`
  - you should see an '`N`' after `.../enabled:`
  - if not: check again after reboot

 ```text
 /sys/module/zswap/parameters/enabled:N
 ```

- or
- `cat /sys/module/zswap/parameters/enabled`
  - you should see an '`N`
  - if not: check again after reboot

 ```text
 N
 ```

#### Using zram-genrator

- `sudo pacman -S zram-generator`
- `sudo vim /etc/systemd/zram-generator.conf`
  - for creation of zram swap using `zstd` and half of the RAM, insert in this new file:

```text
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```

- `sudo systemctl daemon-reload`
- `sudo systemctl start systemd-zram-setup@zram0.service`

 Check the swap status:

- `systemctl status systemd-zram-setup@zram0.service`
- or
- `zramctl --output-all`
- `cat /proc/swaps`

#### Optimizing swap on zram

- `sudo vim /etc/sysctl.d/99-vm-zram-parameters.conf`
  - insert into this new file:

```text
vm.swappiness = 180
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
```

### INSTERTION: Desktop Environment

If you want a Desktop Environment (e.g. Gnome, Plasma KDE, ...) now, jump to "Graphical user interface" and come back afterwards

### Security

- <https://wiki.archlinux.org/title/Security>
- <https://wiki.archlinux.org/title/List_of_applications/Security>

#### Enforcing strong passwords with pam_pwquality

- <https://wiki.archlinux.org/title/Security#Enforcing_strong_passwords_with_pam_pwquality>
&nbsp;

- `sudo pacman -S libpwquality` # Library for password quality checking and generating random passwords
- `sudo vim /etc/pam.d/passwd`
  - comment out or delete current line starting with `password`
    - `#password include  system-auth`
  - append:

  ```text
  password required pam_pwquality.so retry=2 minlen=10 difok=6 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1 [badwords=Remigration alternativlos] enforce_for_root
  password required pam_unix.so use_authtok sha512 shadow
  ```

#### CPU

Microcode install

- <https://wiki.archlinux.org/title/Security#Microcode>
- <https://wiki.archlinux.org/title/Microcode>
&nbsp;

- AMD Porcessors
  - `sudo pacman -S --needed amd-ucode` # if not already installed
- Intel Processors
  - `sudo pacman -S --needed intel-ucode` # if not already installed

#### Memory

- <https://wiki.archlinux.org/title/Security#Memory>
- skipped for now

#### Storage

- <https://wiki.archlinux.org/title/Security#Storage>
- skipped for now

#### User setup

- <https://wiki.archlinux.org/title/Security#User_setup>
&nbsp;
- User lock out after failed login attempts
  - `sudo vim /etc/security/faillock.conf` # config
    - adjust parameters, e.g:
    - `deny` # number of failed logins before lockout (default 3) # increase number if no coffee available in the morning :-)
    - `fail_interval` # time in which failed logins can cause a lockout (in seconds, default 15 minutes)
    - `unlock_time` # lockout time (in seconds, default 10 minutes)

#### Restricting root

- <https://wiki.archlinux.org/title/Security#Restricting_root>
&nbsp;
- Use `sudo` instead of `su`
- Editing files using `sudo`
  - <https://wiki.archlinux.org/title/Sudo#Editing_files>
    - `SUDO_EDITOR=vim sudoedit /path/to/file`
  - or
  - <https://wiki.archlinux.org/title/Environment_variables#Per_user>
    - Edit your shell config (e.g. `~/.bashrc`, `~/.zshrc`, ...)
    - `export SUDO_EDITOR=vim` # ajust to your preferred editor
    - source your shell config, e.g. `source ~/.bashrc`, to make the variable available in current terminal session
    - and use `sudoedit` directly

- Restricting root login
  - disable `root`, but still allowing to use `sudo`
  - Remark: you should have another user-account created with sudo privileges
    - e.g. if your account gets locked, you can unlock your account again directly
  - if you are currently logged in as `root`, switch to another account with sudo privileges
  - `sudo passwd --lock root`

- Allow only uers in the group `wheel` to login using `su` to assume root's identity
  - <https://wiki.archlinux.org/title/Su#su_and_wheel>
  - Uncomment the appropriate line in `/etc/pam.d/su` and `/etc/pam.d/su-l`
    - `auth required pam_wheel.so use_uid`

- Denying SSH login for `root` user
  - <https://wiki.archlinux.org/title/OpenSSH#Deny>
  - `sudo vim /etc/ssh/sshd_config.d/20-deny_root.conf`
  - insert: `PermitRootLogin no`
  - `sudo systemctl restart sshd` # restart the SSH daemon

#### Mandatory access control

- <https://wiki.archlinux.org/title/Security#Mandatory_access_control>

Using virtually any mandatory access control system will significantly improve the security of your computer.

- sipped for now
- \# TODO

#### Kernel hardening

- <https://wiki.archlinux.org/title/Security#Kernel_hardening>
- skipped

#### Sandboxing applications

- <https://wiki.archlinux.org/title/Security#Sandboxing_applications>
- skipped

#### Network and firewalls

- <https://wiki.archlinux.org/title/Security#Network_and_firewalls>

##### Firewalls

- <https://wiki.archlinux.org/title/Security#Network_and_firewalls>

- `sudo pacman -S iptables-nft firewalld` # or later separately / if needed
  - confirm to remove `iptables` to be replaced by `iptables-nft`
  - alternative to `firewalld`: `ufw`, ...
- `sudo systemctl enable --now firewalld.service` # enable service
- `systemctl status firewalld.service` # check status of service

> :memo: **INSERTION: config firewall**
>
> - <https://wiki.archlinux.org/title/General_recommendations#Setting_up_a_firewall>
> - <https://wiki.archlinux.org/title/Security#Network_and_firewalls>
> - <https://wiki.archlinux.org/title/Firewalld>
> - <https://man.archlinux.org/man/firewall-cmd.1>
> - <https://wiki.archlinux.org/title/Uncomplicated_Firewall>
>
> **firewalld**
> `sudo firewall-cmd --permanent --zone=public --add-service=ssh` # allow ssh # 'public' is the initial default zone
> \# config whatever appropriate for you
> `sudo firewall-cmd --check-config` # run checks on the permanent configuration
> `sudo firewall-cmd --reload` # reload firewall rules and keep state information. Current permanent configuration will become new runtime configuration.
> `sudo firewall-cmd --permanent --info-zone=public` # print information about zone 'public'
>
> **ufw** # TODO: test
> `sudo ufw default deny` # block all incoming traffic
> `sudo ufw limit ssh comment 'SSH'` # allow ssh, but limit login attempts from an IP address
> &nbsp;\# config whatever appropriate for you
> `sudo ufw reload`
> `sudo ufw status verbose`

- <https://man.archlinux.org/man/ss.8>
- <https://wiki.archlinux.org/title/Network_configuration#Investigate_sockets>
- <https://wiki.archlinux.org/title/Security#Open_ports>

List all current open ports:

- `sudo ss -l`

Show all listening processes and their numeric tcp and udp port numbers:

- `sudo ss -tulpn`

##### Kernel parameters

- <https://wiki.archlinux.org/title/Security#Kernel_parameters>
- skipped

##### SSH

- <https://wiki.archlinux.org/title/Security#SSH>
- <https://infosec.mozilla.org/guidelines/openssh.html>
- <https://wiki.archlinux.org/title/SSH_keys>
- <https://man.archlinux.org/man/ssh-keygen.1>
&nbsp;
- `ssh-keygen` # generate an ssh key # keeping defaults: just comfirm the questions with enter key
  - By default, keys are stored in the `~/.ssh/` directory and named according to the type of encryption used
- Copying the public key to the remote server on which you want to connect to via ssh
  - e.g.:
  - `ssh-copy-id -i ~/.ssh/id_ed25519.pub username@remote-server.org`
  - `ssh-copy-id -i ~/.ssh/id_ed25519.pub username@ip-address`
- <https://wiki.archlinux.org/title/OpenSSH#Force_public_key_authentication>
- enforce key-based authentication
  - <span style="color:red">WARNING:</span>
    - Before adding this to your configuration, make sure that all accounts which require SSH access have public-key authentication set up in the corresponding authorized_keys files.
    - See [SSH keys#Copying the public key to the remote server](https://wiki.archlinux.org/title/SSH_keys#Copying_the_public_key_to_the_remote_server) for more information.
  - `vim /etc/ssh/sshd_config.d/20-force_publickey_auth.conf`
    - insert:
    - `PasswordAuthentication no`
    - `AuthenticationMethods publickey`
- Fail2ban or Sshguard
  - would need own section, skipped for now
- harden authentication by using two-factor authentication
  - would need own section, skipped for now
- Denying root login
  - already done (see above:  Restricting root)

##### DNS

- <https://wiki.archlinux.org/title/Security#DNS#>
- <https://wiki.archlinux.org/title/Domain_name_resolution#Privacy_and_security>
- skipped

##### Proxies

- <https://wiki.archlinux.org/title/Security#Proxies>
- <https://wiki.archlinux.org/title/Transport_Layer_Security#Trust_management>
- skipped

##### Managing TLS certificates

- <https://wiki.archlinux.org/title/Security#Managing_TLS_certificates>
- skipped

### Service management

- <https://wiki.archlinux.org/title/General_recommendations#Service_management>

Probably not all packages are installed yet.

#### Retrieve and filter the latest Pacman mirror list via Reflector

- <https://wiki.archlinux.org/title/Reflector>
- `sudo pacman -S --needed reflector`
- `sudo systemctl start reflector.service` # retrieve and filter the latest Pacman mirror list now
- Enable `reflector.service` to run Reflector on boot:
  - `sudo systemctl enable --now reflector.service`
  - The service will run reflector with the parameters specified in `/etc/xdg/reflector/reflector.conf`
  - config service parameters:
    - `sudo vim /etc/xdg/reflector/reflector.conf`
    - or execute the following command example (adjust to your needs)
    - `sudo reflector --verbose --age 12 --protocol https --sort rate --country 'Germany,France,Austria,Switzerland' --save /etc/pacman.d/mirrorlist`
- or
- start the systemd service `reflector.service` # weekly
  - `sudo systemctl enable --now reflector.timer`

#### avahi, haveged, upower

- `sudo pacman -S --needed avahi upower` # or install later / separately if needed
- `sudo systemctl enable avahi-daemon` # Network / Service Discovery for Linux using mDNS/DNS-SD (compatible with Bonjour)
- `sudo systemctl enable upower` # abstraction for enumerating power devices, listening to device events and querying history and statistics

#### Enable further services

e.g.:

- Bluetooth
  - install: `sudo pacman -S --needed bluez bluez-utils` # optional install in a previous step (`pacstrap`)
  - start service`sudo systemctl enable bluetooth.service` # bluetooth
- Printing
  - install: `sudo pacman -S --needed cups cups-pdf` # optional install in a previous step (`pacstrap`)
  - start service on boot: `sudo systemctl enable cups.service` # OpenPrinting CUPS - daemon package
  - or only when a program wants to use the service: `sudo systemctl enable cups.socket`
- ...

### System maintenance

- <https://wiki.archlinux.org/title/General_recommendations#System_maintenance>
- <https://wiki.archlinux.org/title/System_maintenance>
- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Maintenance>

Only listing pieces of the interesting parts.

#### pacman-contrib

Contributed scripts and tools for pacman systems

- <https://archlinux.org/packages/extra/x86_64/pacman-contrib/>
- `sudo pacman -S pacman-contrib`

#### Performance and download speeds

- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Performance>
- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Download_speeds>
- <https://wiki.archlinux.org/title/Pacman#Enabling_parallel_downloads>
&nbsp;

##### Parallel Downloads

- `sudo vim /etc/pacman.conf`
- uncomment `ParallelDownloads` under `[options]`
  - you couldkeep the default number and test / adjust what works best for you

##### Sorting Mirrors

- <https://wiki.archlinux.org/title/Mirrors#Sorting_mirrors>
- e.g. with [Reflector](https://wiki.archlinux.org/title/Reflector):
  - retrieve the latest mirror list ...
  - already done, see 'Service management' above

#### Upgrading the system

- <https://wiki.archlinux.org/title/System_maintenance#Upgrading_the_system>
- Read before upgrading the system
  - [Arch Linux home page](https://archlinux.org/)
  - [RSS feed](https://archlinux.org/feeds/news/)
  - [Forum 'Technical Issues and Assistance Forum'](https://bbs.archlinux.org/)

##### Alerts during upgrade

- Act on alerts during an upgrade, pay attention to the alert notices provided by pacman

##### Avoid certain pacman commands

- **always use `pacman -Syu`**; never run `pacman -Sy`

##### Deal promptly with new configuration files

- <https://wiki.archlinux.org/title/Pacman/Pacnew_and_Pacsave>
- When pacman is invoked, `/etc/.pacnew` and `/etc/.pacsave` files can be created.
- Pacman provides notice when this happens and users must deal with these files promptly
  - `sudo find /etc -name '*.pacnew' -o -name '*.pacsave'` # Locating .pac* files

##### Restart or reboot after upgrades

- restart or reboot after upgrades

##### Revert broken updates

- see  online info-resources mentioned above ('Read before upgrading the system')
- and / or
- [downgrade the offending package](https://wiki.archlinux.org/title/Downgrading_packages)

#### Removing unused packages (orphans)

- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks#Removing_unused_packages_(orphans)>
- `sudo pacman -Qdtq | pacman -Rns -` # recursively removing orphans and their configuration files

#### Clean the filesystem

- <https://wiki.archlinux.org/title/System_maintenance#Clean_the_filesystem>

- Cleaning the package cache
  - <https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache>
  - <https://man.archlinux.org/man/paccache.8>
    - `sudo paccache -r`
  - Enable and start `paccache.timer` to discard unused packages weekly with paccache’s default options
    - `sudo systemctl enable --now paccache.timer`
- Broken symlinks
  - <https://wiki.archlinux.org/title/System_maintenance#Broken_symlinks>
  - <https://www.man7.org/linux/man-pages/man1/find.1.html>
  - `sudo find / -path /.snapshots -prune -o -xtype l -print`
    - does not search in directory `/.snapshots`

#### Refresh pacman files database

- <https://wiki.archlinux.org/title/Pacman#Search_for_a_package_that_contains_a_specific_file>
- `sudo systemctl enable --now pacman-filesdb-refresh.timer` # weekly

## Package management

- <https://wiki.archlinux.org/title/General_recommendations#Package_management>

### pacman

- <https://wiki.archlinux.org/title/General_recommendations#pacman>

#### Downloading packages in parallel

- <https://wiki.archlinux.org/title/Pacman#Enabling_parallel_downloads>
- already covered in a previous step

#### Cleaning the package cache

- <https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache>
- already covered in a previous step

#### Tips and tricks

- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks>
- already covered in a previous step
- but of course there is always more

### Repositories

- <https://wiki.archlinux.org/title/General_recommendations#Repositories>

#### Using 32-bit applications

- If you plan on using 32-bit applications, you will want to enable the `multilib` repository.
  - <https://wiki.archlinux.org/title/Official_repositories#multilib>
  - uncomment the `[multilib]` section in `/etc/pacman.conf`
  - `sudo vim /etc/pacman.conf`

  ```text
  [multilib]
  Include = /etc/pacman.d/mirrorlist
  ```

- `sudo pacman -Syu`
  - `-y`: Download a fresh copy of the master package databases (repo.db) from the server(s) defined in pacman.conf(5).

#### Unofficial user repositories

- <https://wiki.archlinux.org/title/Unofficial_user_repositories>
Binary repositories freely created and shared by the community, often providing pre-built versions of PKGBUILDs found in the Arch User Repository (AUR).

Ther are many, for example:

- chaotic-aur
  - <https://wiki.archlinux.org/title/Unofficial_user_repositories#chaotic-aur>
  - <https://aur.chaotic.cx/>
  - follow the doumentaition for setup:
    - <https://aur.chaotic.cx/docs>
  - `sudo pacman -Syu`

### Mirrors

-

- <https://wiki.archlinux.org/title/General_recommendations#Mirrors>
- <https://wiki.archlinux.org/title/Mirrors>
- <https://archlinux.org/mirrors/status/>

Routinely check the [Mirror Status](https://archlinux.org/mirrors/status/) page for a list of mirrors that have been recently synced. This can be automated with [Reflector](https://wiki.archlinux.org/title/Reflector).

- already covered in a previous step

### Arch Build System

- <https://wiki.archlinux.org/title/General_recommendations#Arch_Build_System>
- skipped

### Arch User Repository (AUR)

- <https://wiki.archlinux.org/title/General_recommendations#Arch_User_Repository>
- <https://wiki.archlinux.org/title/Arch_User_Repository>
- <https://aur.archlinux.org/>

User submitted packages

#### AUR helpers

- <https://wiki.archlinux.org/title/AUR_helpers>

AUR helpers automate usage of the Arch User Repository.

...
Pacman wrappers:

- <https://wiki.archlinux.org/title/AUR_helpers#Pacman_wrappers>
  - [paru](https://aur.archlinux.org/packages/paru)
    - `curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/paru.tar.gz`
    - `tar -xzf paru.tar.gz`
    - `cd paru`
    - `makepkg -sic` # install with dependencies and clean up work files after build
      - confirm questions with enter key (for default value)
    - `cd ..`
    - `rm -rf paru paru.tar.gz` # cleanup: delete directory and archive file
    - you can now use `paru` like / instead of `pacman`, e.g.:
      - `paru -S git` # install `git` (official repo)
      - `paru -S --needed snapper-rollback` # install `snapper-rollback` (from AUR)
  - [yay](https://aur.archlinux.org/packages/yay/)
  - ...

## Booting

- <https://wiki.archlinux.org/title/General_recommendations#Booting>
- skipped for now

## Graphical user interface

- <https://wiki.archlinux.org/title/General_recommendations#Graphical_user_interface>

[...]

### Display drivers

- <https://wiki.archlinux.org/title/General_recommendations#Display_drivers>
- <https://wiki.archlinux.org/title/Xorg#Driver_installation>
- <https://wiki.archlinux.org/title/AMDGPU>
- <https://wiki.archlinux.org/title/Hardware_video_acceleration>
- <https://wiki.archlinux.org/title/Intel_graphics>
- <https://wiki.archlinux.org/title/NVIDIA>

For installing 32-bit application support uncomment the multlib Section in `/etc/pacman.conf` and update via `pacman -Syu`.

```text
[multilib]
Include = /etc/pacman.d/mirrorlist
```

#### AMD

[Check Wiki](https://wiki.archlinux.org/title/AMDGPU) for updated / special Info!

- `pacman -S mesa vulkan-radeon libva-mesa-driver mesa-vdpau`
- `pacman -S lib32-mesa lib32-vulkan-radeon lib32-libva-mesa-driver lib32-mesa-vdpau` # 32-bit application support

#### Intel

[Check Wiki](https://wiki.archlinux.org/title/Intel_graphics) for updated / special Info!
Hardware acceleration: <https://wiki.archlinux.org/title/Hardware_video_acceleration#Intel>

- `pacman -S mesa vulkan-intel libva-mesa-driver mesa-vdpau`
- `pacman -S intel-media-driver` # Hardware acceleration
- `pacman -S lib32-mesa lib32-vulkan-intel` # 32-bit application support

#### Nvidia

[Check Wiki](https://wiki.archlinux.org/title/Intel_graphics) for updated / special Info!

### Desktop environments

- <https://wiki.archlinux.org/title/General_recommendations#Desktop_environments>
- <https://wiki.archlinux.org/title/Desktop_environment>

There are many, choose the one you like.

#### Example installation: Gnome

- <https://wiki.archlinux.org/title/GNOME#>
&nbsp;

- (`pacman -S xorg` # display server # group with additional packages and fonts)
  - optional, the `gnome` package in the next step installs what it needs
    - Gnome default is [Wayland](https://wiki.archlinux.org/title/Wayland) now
  - install if you want / need X Window System too
- `sudo pacman -S gnome` # base GNOME desktop with core apps
  - we use pipewire in this guide, so choose `pipewire-jack` when asked
  - confirm the rest with enter key to accept the default
- `sudo pacman -S gnome-tweaks` # graphical tool for many GNOME settings
- `sudo pacman -S --needed gnome-shell-extensions` # Extensions for GNOME shell + if you want to use a Shell theme # should already be installed via `gnome` package
  - <https://wiki.archlinux.org/title/GNOME#GNOME_Shell_themes>
- `sudo pacman -S gnome-extra` # optional # further GNOME applications
- `sudo systemctl enable gdm.service` # enable display manager
  - the `gdm` package (GNOME Display Manager) is included in the `gnome` group we already installed
- `sudo reboot`

## Power management

- <https://wiki.archlinux.org/title/General_recommendations#Power_management>
- <https://wiki.archlinux.org/title/Category:Power_management>
- <https://wiki.archlinux.org/title/Power_management>

skipped

## Multimedia

- <https://wiki.archlinux.org/title/General_recommendations#Multimedia>
- <https://wiki.archlinux.org/title/Category:Multimedia>

### Sound system

- <https://wiki.archlinux.org/title/General_recommendations#Sound_system>
- <https://wiki.archlinux.org/title/PipeWire>

We alreaddy installed `pipewire` package (see: Pre-installation -> Installation -> Install essential packages).

### Codecs and containers

- <https://wiki.archlinux.org/title/Codecs_and_containers>
- <https://wiki.archlinux.org/title/GStreamer>
- <https://wiki.archlinux.org/title/FFmpeg>
- <https://wiki.archlinux.org/title/Hardware_video_acceleration>

[...]

It is not always necessary to explicitly install codecs if you have installed a media player. For example, MPlayer pulls in a large number of codecs as dependencies, and also has codecs built in.

The `libavcodec` codecs are included with media players such as MPlayer and VLC, so you may not need to install the ffmpeg package itself.

[...]

GStreamer

- `pacman -S --needed gstreamer gst-libav gst-plugins-bad gst-plugins-base gst-plugins-good gst-plugin-pipewire gst-plugin-va`
- `rm ~/.cache/gstreamer-1.0/registry.*.bin`

FFmpeg

- `pacman -S --needed ffmpeg`

[...]

## Networking

- <https://wiki.archlinux.org/title/General_recommendations#Networking>
[...]

### Setting up a firewall

- <https://wiki.archlinux.org/title/General_recommendations#Setting_up_a_firewall>
- <https://wiki.archlinux.org/title/Category:Firewalls>
- <https://wiki.archlinux.org/title/Firewalld>
- <https://wiki.archlinux.org/title/Uncomplicated_Firewall>

We already installed `firewalld` (see: System administration -> Security -> Network and firewalls -> Firewalls).

## Input devices

- <https://wiki.archlinux.org/title/General_recommendations#Input_devices>
- skipped

## Optimization

- <https://wiki.archlinux.org/title/General_recommendations#Optimization>
- skipped

Discard/TRIM support for solid state drives (SSD)

- <https://wiki.archlinux.org/title/Dm-crypt/Specialties#Discard/TRIM_support_for_solid_state_drives_(SSD)>
- <https://wiki.archlinux.org/title/Solid_state_drive#TRIM>

Besides enabling discard support in dm-crypt, it is also required to periodically run fstrim(8) **or mount the filesystem (e.g. /dev/mapper/root in this example) with the discard option in /etc/fstab.** For details, please refer to the TRIM page.

We mounted with `discard=async` in fstab.

## System services

- <https://wiki.archlinux.org/title/General_recommendations#System_services>
- <https://wiki.archlinux.org/title/Systemd>

### File index and search

- CLI: <https://wiki.archlinux.org/title/General_recommendations#File_index_and_search>
- Desktop search engines: <https://wiki.archlinux.org/title/List_of_applications/Utilities#File_searching>

#### locate

- <https://wiki.archlinux.org/title/Locate>
- <https://plocate.sesse.net/>
- <https://man.archlinux.org/man/extra/plocate/plocate.1.en>
&nbsp;

- Installation
  - `sudo pacman -S plocate`
    - also activates a systemd timer unit `plocate-updatedb.timer`, which invokes a database update each day
- Preparation
  - create database: `sudo updatedb`
  - `sudo systemctl start plocate-updatedb.timer`
    - if you want to use it before reboot
- Config
  - ignore certain filesystems, e.g. `/.snapshots`
    - <https://man.archlinux.org/man/updatedb.conf.5>
    - `sudo vim /etc/updatedb.conf`
      - `PRUNEPATHS`: there should be already an intial list of paths
      - add `/.snapshots` to the list, seperated by a space character

### Local mail delivery

- <https://wiki.archlinux.org/title/General_recommendations#Local_mail_delivery>
- skipped

### Printing

- <https://wiki.archlinux.org/title/General_recommendations#Printing>
- <https://wiki.archlinux.org/title/CUPS>
- <https://wiki.archlinux.org/title/Category:Printers>

#### CUPS

standards-based, open source printing system

Already covered in a previous step (Service Managment)

- Installaton
  - `pacman -S cups cups-pdf`
    - `cups-pdf`: "print" into a PDF document
- Service
  - always starting on boot: `systemctl enable --now cups.service`
  - or
  - only start CUPS when a program wants to use the service: `systemctl enable --now cups.socket`

## Appearance

- <https://wiki.archlinux.org/title/General_recommendations#Appearance>
- <https://wiki.archlinux.org/title/Category:Eye_candy>

### Fonts

- <https://wiki.archlinux.org/title/General_recommendations#Fonts>
- <https://wiki.archlinux.org/title/Fonts#Families>
- <https://wiki.archlinux.org/title/Metric-compatible_fonts>
- <https://www.programmingfonts.org/>
- <https://wiki.archlinux.org/title/Fonts>
- <https://wiki.archlinux.org/title/Font_configuration>

#### Installing fonts

- <https://wiki.archlinux.org/title/Fonts#Installation>

- Via Pacman:
  - pick the one you like, e.g. from the [Fonts Families](https://wiki.archlinux.org/title/Fonts#Families)
  - `sudo pacman -S ttf-meslo-nerd ttf-bitstream-vera ttf-bitstream-vera-mono-nerd ttf-dejavu ttf-dejavu-nerd noto-fonts noto-fonts-emoji ttf-fira-code ttf-hack ttf-hack-nerd ttf-jetbrains-mono ttf-jetbrains-mono-nerd adobe-source-code-pro-fonts ttf-liberation ttf-liberation-mono-nerd`
  - ...
- Manual installation
  - install fonts to `~/.local/share/fonts/` # single user
  - or
  - `mkdir -p /usr/local/share/fonts` # system-wide (all users)
  - Finally, update the fontconfig cache (usually unnecessary as software using the fontconfig library does this):
    - `fc-cache`

List all installe fonts: `fc-list`

### GTK and Qt themes

- <https://wiki.archlinux.org/title/General_recommendations#GTK_and_Qt_themes>
- skipped

## Console improvements

- <https://wiki.archlinux.org/title/General_recommendations#Console_improvements>
- <https://wiki.archlinux.org/title/Category:Command-line_shells>

[...]

### Alternative shells

- <https://wiki.archlinux.org/title/General_recommendations#Alternative_shells>
- <https://wiki.archlinux.org/title/Command-line_shell#List_of_shells>

We already installed `zsh` in a previous step.

- <https://wiki.archlinux.org/title/Zsh>
- We could add some additional packages, if not already installed:
  - `sudo pacman -S --needed zsh zsh-completions zsh-autosuggestions zsh-syntax-highlighting`
- changing your default shell
  - <https://wiki.archlinux.org/title/Command-line_shell#Changing_your_default_shell>
  - as root / sudo privileges:
  - `chsh --shell /bin/zsh USERID`
  - or
  - `usermod --shell /bin/zsh USERID`

[...]

### Compressed files

- <https://wiki.archlinux.org/title/General_recommendations#Compressed_files>
- <https://wiki.archlinux.org/title/Archiving_and_compression>

#### tar

- <https://wiki.archlinux.org/title/Core_utilities>
- <https://man.archlinux.org/man/core/tar/tar.1.en>
- Install
  - `sudo pacman -S --needed coreutils`

And many others, see links above
e.g.: `sudo pacman -S --needed bzip2 bzip3 gzip zstd xz   p7zip zip unzip`

[...]

## Virtualization (Qemu/KVM)

- <https://wiki.archlinux.org/title/Category:Virtualization>
- <https://wiki.archlinux.org/title/QEMU>
- <https://wiki.archlinux.org/title/KVM>
- <https://wiki.archlinux.org/title/Libvirt>

### Installating packages for virtualization

- `pacman -S --needed qemu-full libvirt iptables-nft dnsmasq openbsd-netcat libguestfs edk2-ovmf swtpm vde2 virt-install`
  - `systemctl enable --now libvirtd.service`
  - `systemctl start virtlogd.service`

### Client

- `pacman -S --needed virt-manager virt-viewer`
- and / or cockpit with additinal features
- `pacman -S --needed  cockpit cockpit-machines cockpit-podman cockpit-storaged cockpit-packagekit udisks2 pcp` # Browser based admin tool for Linux
  - `systemctl enable --now cockpit.socket`

### SPICE

- <https://wiki.archlinux.org/title/QEMU#SPICE>

Enabling SPICE support on the **guest** (Virtual machine):

- `pacman -S spice-vdagent` # install on Virtual machine # Spice agent xorg client that enables copy and paste between client and X-session and more.

## Troubleshooting

### Failed to synchronize all databases (unable to lock database)

- <https://bbs.archlinux.org/viewtopic.php?id=249093>
- <https://wiki.archlinux.org/title/Pacman#%22Failed_to_init_transaction_(unable_to_lock_database)%22_error>

E.g. after snapper-rollback
`sudo pacman -Syu` or `sudo pacman -S <package>`, ... show this error message:

```text
error: failed to synchronize all databases (unable to lock database)
```

You can run `fuser /var/lib/pacman/db.lck` as root to verify if there is any process still using pacman's `db.lck` file:
`sudo fuser /var/lib/pacman/db.lck`

- if it shows something like:
  - `/var/lib/pacman/db.lck:   795`
  - you can not delete it yet, since `db.lck` is still used by a process (with process id `795` in this example)
- else you can delete `db.lck`:
  - `sudo rm /var/lib/pacman/db.lck`
