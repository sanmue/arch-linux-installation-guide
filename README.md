# Arch Linux Installation Guide - A Practical Example

## Intro

This Arch Linux installation guide may be seen as a practical example following the [Arch Wiki installation guide](https://wiki.archlinux.org/title/Installation_guide).

Some examples of points covered in this guide:

- [UEFI/GPT](https://wiki.archlinux.org/title/Partitioning#UEFI/GPT_layout_example) (with [/EFI partition](https://wiki.archlinux.org/title/EFI_system_partition)) or [BIOS/GPT](https://wiki.archlinux.org/title/Partitioning#BIOS/GPT_layout_example) - [partition layout](https://wiki.archlinux.org/title/Partitioning#Partition_scheme)
- [GRUB](https://wiki.archlinux.org/title/GRUB) bootloader and [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot)
- [Btrfs](https://wiki.archlinux.org/title/Btrfs) filesystem and [subvolumes](https://wiki.archlinux.org/title/Btrfs#Subvolumes)
- [snapper](https://wiki.archlinux.org/title/Snapper) and [snap-pac](https://wiki.archlinux.org/title/Snapper#Wrapping_pacman_transactions_in_snapshots) (for taking snapshots of the root subvolume you can rollback to)
- [snapper-rollback (AUR)](https://aur.archlinux.org/packages/snapper-rollback) for easy rollback to a snapshot
- [AUR helper](https://wiki.archlinux.org/title/AUR_helpers#Pacman_wrappers) ([paru](https://aur.archlinux.org/packages/paru)) for easy installation of packages from [AUR](https://aur.archlinux.org/)
- [Encryption](https://wiki.archlinux.org/title/Dm-crypt) with LUKS
  - GRUB bootlaoder: [Keyfile](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Keyfiles) for automatically [unlocking the root partition on boot](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Unlocking_the_root_partition_at_boot) (with [keyfile embedded in initramfs](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#With_a_keyfile_embedded_in_the_initramfs))
- [Swap file](https://wiki.archlinux.org/title/Swap#Swap_file) ([btrfs specific](https://wiki.archlinux.org/title/Btrfs#Swap_file) creation) or [zram](https://wiki.archlinux.org/title/Zram) (no [hibernation](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation))
- [Grafics card driver](https://wiki.archlinux.org/title/Xorg#Driver_installation) (AMD, Intel, Nvidia)
- [Desktop Environment](https://wiki.archlinux.org/title/Desktop_environment)
- [Firewall](https://wiki.archlinux.org/title/Category:Firewalls) ([firewalld](https://wiki.archlinux.org/title/Firewalld))
- [Chaotic-AUR](https://aur.chaotic.cx/) as an example for an [unofficial user repository](https://wiki.archlinux.org/title/Unofficial_user_repositories) besides [AUR](https://wiki.archlinux.org/title/Arch_User_Repository)
- Font [installation](https://wiki.archlinux.org/title/Fonts#Installation)
- Virtualization ([libvirt](https://wiki.archlinux.org/title/Libvirt), [Qemu/KVM](https://wiki.archlinux.org/title/QEMU))

I found [mjkstra's](https://gist.github.com/mjkstra) ['Modern Arch linux installation guide'](https://gist.github.com/mjkstra/96ce7a5689d753e7a6bdd92cdc169bae) which was a pusher for creating this guide.
But I am using snapper and [snapper-rollback (AUR)](https://aur.archlinux.org/packages/snapper-rollback) here (instead of [Timeshift](https://wiki.archlinux.org/title/Timeshift)) inspired by [mpr's video](https://www.youtube.com/watch?v=maIu1d2lAiI) because supports a more flexible btrfs subvolume layout.

## Table of Content

- [Arch Linux Installation Guide - A Practical Example](#arch-linux-installation-guide---a-practical-example)
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
      - [Encrypting our root partition](#encrypting-our-root-partition)
      - [Open the LUKS encrypted root partition](#open-the-luks-encrypted-root-partition)
      - [Show current block device list](#show-current-block-device-list)
      - [Note regarding password input at boot when using encryption](#note-regarding-password-input-at-boot-when-using-encryption)
  - [Format partitions](#format-partitions)
  - [Btrfs subvolumes](#btrfs-subvolumes)
    - [Mount root partiton](#mount-root-partiton)
    - [Create subvolumes](#create-subvolumes)
    - [List subvolumes](#list-subvolumes)
    - [Unmount root subvolume](#unmount-root-subvolume)
  - [Mount the file systems](#mount-the-file-systems)
    - [Mount partition to install the system](#mount-partition-to-install-the-system)
    - [Create folders for the subvolumes and partitions (mountpoints)](#create-folders-for-the-subvolumes-and-partitions-mountpoints)
    - [Mount the efi partition](#mount-the-efi-partition)
    - [Mount the subvolumes](#mount-the-subvolumes)
    - [Mount btrfsroot for 'snapper-rollback'](#mount-btrfsroot-for-snapper-rollback)
    - [Mount options](#mount-options)
    - [Show current block device list](#show-current-block-device-list-1)
      - [UEFI/GPT with encryption](#uefigpt-with-encryption)
      - [UEFI/GPT and no encryption](#uefigpt-and-no-encryption)
      - [BIOS/GPT with encryption](#biosgpt-with-encryption)
  - [Installation](#installation)
    - [Select the mirrors](#select-the-mirrors)
    - [Install essential packages](#install-essential-packages)
      - [Minimal package install](#minimal-package-install)
      - [Extended package install](#extended-package-install)
      - [Summarized package list](#summarized-package-list)
  - [Configure the system](#configure-the-system)
    - [Fstab (filesystem table)](#fstab-filesystem-table)
      - [Create fstab](#create-fstab)
      - [UEFI/GPT with encryption](#uefigpt-with-encryption-1)
      - [UEFI/GPT without encryption](#uefigpt-without-encryption)
      - [BIOS/GPT with encryption](#biosgpt-with-encryption-1)
    - [=== Chroot start ===](#-chroot-start-)
    - [Chroot (Change root into new system)](#chroot-change-root-into-new-system)
    - [Time](#time)
    - [Localization](#localization)
  - [Network configuration](#network-configuration)
    - [hostname file](#hostname-file)
    - [hosts file](#hosts-file)
    - [Network Management](#network-management)
  - [INSERTION: keyfile for decrypting root partition on boot](#insertion-keyfile-for-decrypting-root-partition-on-boot)
  - [Initramfs (Create initial ramdisk environment)](#initramfs-create-initial-ramdisk-environment)
    - [Check mkinitcpio.conf](#check-mkinitcpioconf)
      - [Modules](#modules)
      - [FILES: Include the keyfile for decrypting root partition on boot](#files-include-the-keyfile-for-decrypting-root-partition-on-boot)
      - [Hooks](#hooks)
    - [Creating new initramfs](#creating-new-initramfs)
  - [Boot loader](#boot-loader)
    - [Grub](#grub)
      - [Config default grub](#config-default-grub)
        - [Encrypted the root partition](#encrypted-the-root-partition)
        - [Preparing next step: get UUID of encrypted partition](#preparing-next-step-get-uuid-of-encrypted-partition)
        - [Unlock the encrypted root partition on boot by setting kernel parameters](#unlock-the-encrypted-root-partition-on-boot-by-setting-kernel-parameters)
      - [Installation GRUB](#installation-grub)
      - [Configuration](#configuration)
    - [Systemd-boot](#systemd-boot)
      - [Installing the UEFI boot manager](#installing-the-uefi-boot-manager)
      - [Automatic update via systemd service](#automatic-update-via-systemd-service)
      - [Copy kernel and initramfs to directory on the efi partition](#copy-kernel-and-initramfs-to-directory-on-the-efi-partition)
        - [Preparation step](#preparation-step)
        - [Automated copy of kernel and initramfs using systemd](#automated-copy-of-kernel-and-initramfs-using-systemd)
        - [Verify the syntax of the Systemd Unit files](#verify-the-syntax-of-the-systemd-unit-files)
        - [Enable and start efistub-update.path](#enable-and-start-efistub-updatepath)
        - [Copy of kernel and initramfs using systemd service](#copy-of-kernel-and-initramfs-using-systemd-service)
      - [Loader configuration](#loader-configuration)
      - [Adding loaders](#adding-loaders)
        - [Standard loaders](#standard-loaders)
        - [Fallback loader](#fallback-loader)
        - [Additional options](#additional-options)
      - [Check config](#check-config)
  - [Set password for root user](#set-password-for-root-user)
  - [Reboot](#reboot)
    - [=== Chroot end ===](#-chroot-end-)
    - [Optionally manually unmount all the partitions](#optionally-manually-unmount-all-the-partitions)
    - [Restart the machine](#restart-the-machine)
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
        - [Function for rollback](#function-for-rollback)
        - [Rollback example](#rollback-example)
        - [Manual Snapshot after Rollback](#manual-snapshot-after-rollback)
    - [INSERTION: Config zram as swap](#insertion-config-zram-as-swap)
      - [Disable zswap](#disable-zswap)
      - [Using zram-generator](#using-zram-generator)
      - [Optimizing swap on zram](#optimizing-swap-on-zram)
    - [INSERTION: Desktop Environment](#insertion-desktop-environment)
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
- `setfont ter-132b` # optional # set (bigger) font # `ter-122b` (maybe big enough) # `setfont -d` (double size, maybe looks kind of blurred)
  - available fonts: `ls /usr/share/kbd/consolefonts/ | less`

## Excursus: Connecting to the machine to be installed via ssh

- `passwd` # set passwort for current (root) user
&nbsp;

- `pacman -Sy` # sync + refresh package database
- `pacman -S --needed openssh rsync sudo vim nano` # install some packages
- `systemctl start sshd` # start ssh service
- `systemctl status sshd` # check status of sshd service
&nbsp;

- `vim /etc/ssh/ssh_config` # config ssh # or use 'nano' / your prefered editor instead of 'vim'
  - uncomment: `PasswordAuthentication yes`
  - add: `PermitRootLogin yes`
&nbsp;

- `ip a` # get current IP address

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
    - which means you are in BIOS Boot mode

> :warning: **`systemd-boot`** bootloader supports only UEFI boot mode.
>
> For BIOS boot mode you have to use Grub bootloader (or another bootlaoder supporting BIOS boot mode).
> Only Grub and systemd-boot are addressed in this guide.

## Internet connection

- <https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet>
&nbsp;

- connect to network
  - Wi-Fi: authenticate to a wireless network using `iwctl`
    - <https://wiki.archlinux.org/title/Iwd#iwctl>
  - Mobile broadband modem: connect to a mobile network with the `mmcli` utility
    - <https://wiki.archlinux.org/title/Mobile_broadband_modem#ModemManager>
- `ip link` # Ensure your network interface is listed and enabled / UP, e.g.:
```text
[...]
2: enp14s0: <BROADCAST,MULTICAST, [...] state UP [...]
[...]
```
- verify connection: `ping -c 3 archlinux.org` # send 3 pings

## Update the system clock

- <https://wiki.archlinux.org/title/Installation_guide#Update_the_system_clock>
&nbsp;

- ensure the system clock is synchronized:
  - `timedatectl`
  - `timedatectl set-timezone Europe/Berlin` # optional # set timezone, e.g. for Germany

## Disk partitioning

- <https://wiki.archlinux.org/title/Installation_guide#Partition_the_disks>
- <https://wiki.archlinux.org/title/Installation_guide#Example_layouts>
- <https://wiki.archlinux.org/title/Partitioning#Example_layouts>
- <https://wiki.archlinux.org/title/Partitioning#Choosing_between_GPT_and_MBR>
- <https://wiki.archlinux.org/title/EFI_system_partition>
- <https://wiki.archlinux.org/title/GRUB>
- <https://wiki.archlinux.org/title/Systemd-boot#Installation_using_XBOOTLDR>
- <https://uapi-group.org/specifications/specs/boot_loader_specification/#the-partitions>
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

> :memo: **Size of the efi partition**
> If you plan to install multiple kernels (e.g. linux, linux-lts, linux-zen, ...), ... increase the size of the efi partition accordingly.
> Systemd-boot: Also take the size requirements into account if you want [Grml on ESP](https://wiki.archlinux.org/title/Systemd-boot#Grml_on_ESP) and/or [Archiso on ESP](https://wiki.archlinux.org/title/Systemd-boot#Archiso_on_ESP).

#### Create Partitions

`fdisk /dev/vda`

- `,` means: press enter to accept the default value

**Grub and systemd-boot:**
```text
g           # gpt partition table

n           # create a new partition
,, +1G      # efi partition, size 1 GB (increase the size according to your needs)
t           # change type
1           # 'EFI System'

n
,,,         # system partition (takes entire rest of the disk)
t           # change type
,           # accept default (partition number 2)
23          # 'Linux root (x86-64)'

p           # print current partition table
w           # write to disk and exit
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

**Only Grub:**

`fdisk /dev/vda`

- `,` (comma) means: press enter to accept the default value

```text
g           # gpt partition table

n           # create a new partition
,, +1M      # size: 1 MB
t           # change type
4           # 'BIOS boot'

n
,, -1M      # system partition # takes entire rest of the disk, excpet 1 MB)
t           # change type
,           # accept default (partition number 2)
23          # 'Linux root (x86-64)'

p           # print current partition table
w           # write to disk and exit
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

#### Encrypting our root partition

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encrypting_devices_with_LUKS_mode>
- **Systemd-boot + XBOOTLDR**:
  - `cryptsetup luksFormat /dev/vda3`
- **Grub**:
  - `cryptsetup luksFormat --pbkdf pbkdf2 /dev/vda2`
  - <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Encryption_options_for_LUKS_mode>
    - Argon2id (cryptsetup default) and Argon2i PBKDFs are not supported ([GRUB bug #59409](https://savannah.gnu.org/bugs/?59409)), only PBKDF2 is
    - as of 10/2024
  - <https://savannah.gnu.org/bugs/index.php?55093>
  - <https://lists.gnu.org/archive/html/grub-devel/2024-05/msg00166.html>

#### Open the LUKS encrypted root partition

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Unlocking/Mapping_LUKS_partitions_with_the_device_mapper>
&nbsp;

- `cryptsetup open /dev/vda2 root`
  - Once opened, the root partition device address would be `/dev/mapper/root` (device mapper name is `root`) instead of the partition `/dev/vda2`.
  - you should make a note, we will need this info later

> :warning: **Encryption: device mapper name and root partition label**
> - <https://uapi-group.org/specifications/specs/discoverable_partitions_specification/#defined-partition-type-uuids>
>   - On systems with matching architecture, the first partition with this type UUID on the disk containing the active EFI ESP is automatically mounted to the root directory `/`.
>   - If the partition is **encrypted** with LUKS or has dm-verity integrity data ..., the device mapper file will be named **`/dev/mapper/root`**.
> - <https://uapi-group.org/specifications/specs/discoverable_partitions_specification/#why-are-you-taking-my-etcfstab-away>
> - <https://wiki.archlinux.org/title/Systemd#systemd.mount_-_mounting>
> - <https://man.archlinux.org/man/systemd.mount.5#FSTAB>
> - <https://man.archlinux.org/man/systemd-fstab-generator.8.en>
> 
> I used to set the device mapper name to `cryptroot`.
> Systemd-boot sets device mapper name to `root` by default (and seems to like the partition label to be named corresponding, if set).
> To harmonize the naming I changed the device mapper name for `Grub` und `systemd-boot` to `root` and the root partition label (see: Format partitions).
>
> Remark:
> When using device-mapper name and root partition label `cryptroot`, systemd-boot still works, but it looks a bit ugly:
> 
> **fstab:**
> ```text
> NAME                      FSTYPE      FSVER LABEL     UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
> [...]
> └─/dev/vda3               crypto_LUKS 2               f3fcab95-75b4-471d-ab22-39f11537701d                
>   ├─/dev/mapper/cryptroot btrfs             CRYPTROOT 67f9435b-2f88-41ce-826f-928577e17519                
>   └─/dev/mapper/root      btrfs             CRYPTROOT 67f9435b-2f88-41ce-826f-928577e17519   26.2G    18% /var/log
>                                                                                                           /swap
>                                                                                                           [...]
> [...]
> ```
>
> **ls -la /dev/mapper**
> ```
> [...]
> lrwxrwxrwx  1 root root       7 Nov  6 17:47 cryptroot -> ../dm-0
> lrwxrwxrwx  1 root root       7 Nov  6 17:47 root -> ../dm-1
> ```
>
> **ls -la /dev/disk/by-uuid**
> ```
> [...]
> lrwxrwxrwx 1 root root  10 Nov  6 17:47 67f9435b-2f88-41ce-826f-928577e17519 -> ../../dm-1
> lrwxrwxrwx 1 root root  10 Nov  6 17:47 B2CE-79C8 -> ../../vda1
> lrwxrwxrwx 1 root root  10 Nov  6 17:47 B438-9424 -> ../../vda2
> lrwxrwxrwx 1 root root  10 Nov  6 17:47 f3fcab95-75b4-471d-ab22-39f11537701d -> ../../vda3
> ```

#### Show current block device list

`lsblk -p`

```text
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/loop0                  7:0    0 790.3M  1 loop  /run/archiso/airootfs
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    25G  0 disk
├─/dev/vda1               254:1    0     1G  0 part
└─/dev/vda2               254:2    0    24G  0 part
  └─/dev/mapper/root 253:0    0    24G  0 crypt
```

#### Note regarding password input at boot when using encryption

**Grub:** When not using a keyfile to decrypt the encrypted root partition: when starting your machine you will be asked twice for a password for decrypting: 1x bootloader, 1x encrypted root partition.
**Systemd-boot:** We do not use a keyfile in this guide (/efi partition is not encrypted). You will currently only be asked once for the password to decrypt the root partition.

> :memo: **keyfile**
> You can generate and add a keyfile to [unlock encrypted root at boot](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#With_a_keyfile_embedded_in_the_initramfs), which has to be accessible only by root user.
>
> The steps will be described at a later point in this guide (for Grub), see: 'Initramfs (Create initial ramdisk environment)', and 'BOOT LOADER -> GRUB' when in chroot environment.
>
> Grub: Since you will still have to enter a secure passphrase once on boot, and `/boot` is encrypted (on encrypted root partition), and for UEFI: ESP ist mounted to `/efi`, there should be no real security disadvantages.

- Unlocking the root partition at boot, with a keyfile embedded in the initramfs
  - <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#With_a_keyfile_embedded_in_the_initramfs>
- EFI system partition (ESP), typical mount points, mount the ESP to `/efi`:
  - <https://wiki.archlinux.org/title/EFI_system_partition#Typical_mount_points>

## Format partitions

- <https://wiki.archlinux.org/title/Installation_guide#Format_the_partitions>
- <https://wiki.archlinux.org/title/Btrfs#File_system_creation>
- <https://wiki.archlinux.org/title/EFI_system_partition#Format_the_partition>
&nbsp;

- EFI partition (only UEFI boot mode, skip for BIOS boot mode)
  - `mkfs.fat -F32 -n EFI /dev/vda1` # EFI partition # adjust to your device path
- Root partition
  - encrypted: `mkfs.btrfs -L ROOT /dev/mapper/root`
    - the device `/dev/mapper/root` can then be mounted like any other partition
  - not encrypted: `mkfs.btrfs -L ROOT /dev/vda2` # main partition # adjust to your device path, e.g. /dev/vda2 (for Grub bootloader in our example)

## Btrfs subvolumes

- <https://wiki.archlinux.org/title/Btrfs#Subvolumes>
- <https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout>
- <https://btrfs.readthedocs.io/en/latest/Subvolumes.html>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Subvolumes>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Layout>

### Mount root partiton

- encrypted: `mount /dev/mapper/root /mnt` # mount btrfs root
- not encrypted: `mount /dev/vda2 /mnt` # mount btrfs root # adjust to your device path

### Create subvolumes

- <https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout>
&nbsp;

- in a single command:
  - `btrfs subvolume create /mnt/{@,@home,@varlog,@swap,@snapshots}`
- or individual commands:
  - `btrfs subvolume create /mnt/@` # root
  - `btrfs subvolume create /mnt/@home` # home folder
  - `btrfs subvolume create /mnt/@varlog` # log files
  - `btrfs subvolume create /mnt/@swap` # for swap file # can be skipped if you want to have only zram
  - `btrfs subvolume create /mnt/@snapshots` # snapshot folder
- ... and others you may want to create:

> **Tip:**
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

- root partition:
  - encrypted: `mount /dev/mapper/root -o subvol=/@,compress=zstd,noatime /mnt` # Mount the subvolume on which we want to install the system # adjust to your device path
    - ~~`mount /dev/mapper/root -o subvolid=256,compress=zstd,noatime /mnt`~~ do not use subvolid
  - not encrypted: `mount /dev/vda2 -o subvol=/@,compress=zstd,noatime /mnt` # Mount the subvolume on which we want to install the system # adjust to your device path

### Create folders for the subvolumes and partitions (mountpoints)

- in a single command:
  - `mkdir -p /mnt/{efi,home,var/log,swap,.snapshots,.btrfsroot}`
- or in individual commands:
  - `mkdir -p /mnt/efi` # only for UEFI boot mode
  - `mkdir -p /mnt/home`
  - `mkdir -p /mnt/var/log`
  - `mkdir -p /mnt/swap`
  - `mkdir -p /mnt/.snapshots`
  - `mkdir -p /mnt/.btrfsroot` # for 'snapper-rollback'
- ... and for all other subvolumes and partitions you may have created

### Mount the efi partition

- <https://wiki.archlinux.org/title/Talk:EFI_system_partition#Mountpoint_umask>
  - `fmask` + `dmask` mount options are used by EndeavourOS (as of 10/2024)

Only UEFI, skip if BIOS boot mode

- `mount /dev/vda1 -o fmask=0137,dmask=0027 /mnt/efi` # adjust to your device path
  - mount options (`-o`): -> `drwxr-x---`

### Mount the subvolumes

Adjust to your device path.

> :warning: **If root partition is encrypted:**
> Replace `/dev/vda2` (root partition) with `/dev/mapper/root` (`root` = device mapper name used for enryption / opening encrypted root partition)
> The mount command ist the same for Grub and systemd-boot.

- home
  - encryption: `mount /dev/mapper/root -o subvol=/@home,compress=zstd,noatime /mnt/home`
  - no encryption: `mount /dev/vda2 -o subvol=/@home,compress=zstd,noatime /mnt/home`
- log files
  - encryption: `mount /dev/mapper/root -o subvol=/@varlog,compress=zstd,noatime /mnt/var/log`
  - no encryption: `mount /dev/vda2 -o subvol=/@varlog,compress=zstd,noatime /mnt/var/log`
- swap file:
  - encryption: `mount /dev/mapper/root -o subvol=/@swap,compress=zstd,noatime /mnt/swap`
  - no encryption: `mount /dev/vda2 -o subvol=/@swap,compress=zstd,noatime /mnt/swap`
  - create and activate swap:
    - `btrfs filesystem mkswapfile --size 4g --uuid clear /mnt/swap/swapfile` # Create a 4 GB swap file (currently without Hibernation support)
    - `swapon /mnt/swap/swapfile` # activate the swap file
- snapshots
  - encryption: `mount /dev/mapper/root -o subvol=/@snapshots,compress=zstd,noatime /mnt/.snapshots`
  - no encryption: `mount /dev/vda2 -o subvol=/@snapshots,compress=zstd,noatime /mnt/.snapshots`
- ... and all other subvolumes you may have created

### Mount btrfsroot for 'snapper-rollback'

- encryption: `mount /dev/mapper/root -o subvol=/,compress=zstd,noatime /mnt/.btrfsroot`
- no encryption: `mount /dev/vda2 -o subvol=/,compress=zstd,noatime /mnt/.btrfsroot`

### Mount options

> :memo: **Note:**
> Mount options (`-o ...`) used above fit for (NVME) SSDs and btrfs filesystem

### Show current block device list

#### UEFI/GPT with encryption

`lsblk -p`

```text
/dev/loop0                  7:0    0 790.3M  1 loop
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    25G  0 disk
├─/dev/vda1               254:1    0     1G  0 part  /mnt/efi
└─/dev/vda2               254:2    0    24G  0 part
  └─/dev/mapper/root      253:0    0    24G  0 crypt /mnt/.btrfsroot
                                                     /mnt/.snapshots
                                                     /mnt/swap
                                                     /mnt/var/log
                                                     /mnt/home
                                                     /mnt
```

#### UEFI/GPT and no encryption

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

#### BIOS/GPT with encryption

`lsblk -p`

```text
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/loop0                  7:0    0 790.3M  1 loop  /run/archiso/airootfs
/dev/sr0                   11:0    1   1.1G  0 rom   /run/archiso/bootmnt
/dev/vda                  254:0    0    35G  0 disk
├─/dev/vda1               254:1    0     1M  0 part
└─/dev/vda2               254:2    0    35G  0 part
  └─/dev/mapper/root      253:0    0    35G  0 crypt /mnt/.btrfsroot
                                                     /mnt/.snapshots
                                                     /mnt/swap
                                                     /mnt/var/log
                                                     /mnt/home
                                                     /mnt
```

## Installation

- <https://wiki.archlinux.org/title/Installation_guide#Installation>

### Select the mirrors

- <https://wiki.archlinux.org/title/Installation_guide#Select_the_mirrors>

> :memo: **Note: reflector**
> A Python 3 module and script to retrieve and filter the latest Pacman mirror list

In the live system, after connecting to the internet, `reflector` updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.
`/etc/pacman.d/mirrorlist`

Update mirror list manually + set preferred countries (optional, only temporary since we are in an arch live system):

- `reflector --verbose --age 12 --protocol https --sort rate --country 'Austria,France,Germany,Switzerland' --save /etc/pacman.d/mirrorlist`

You could activate parallel downloads by pacman (optional, temporary since we are in an arch live system):
`vim /etc/pacman.conf` and umcomment the line containing `ParallelDownlodads = 5`

### Install essential packages

- <https://wiki.archlinux.org/title/Installation_guide#Install_essential_packages>

#### Minimal package install

The minimum would be: `pacstrap -K /mnt base linux linux-firmware` # using 'linux' kernel
And install the rest while chrooted into the new system (e.g. see: Post-installation -> System administration; after "Users and groups").

> :memo: You could consider installing a second kernel (e.g. lts version), which may bemome handy if you have problems booting with your standard kernel.
> `linux-lts` and as an "extended package" `linux-lts-headers`

#### Extended package install

But we could also install some more packages here:

- e.g. Grub + UEFI: `pacstrap -K /mnt base linux linux-firmware linux-headers   grub efibootmgr   btrfs-progs mtools   base-devel sudo man-db man-pages texinfo   networkmanager   reflector   openssh vim rsync terminus-font`
- e.g. Systemd-boot: same as above, but without the `grub` package
&nbsp;

- for Ext2/3/4 filesystem utilities add: `e2fsprogs`
- depending on processor add:
  - for AMD: `amd-ucode`
  - for Intel: `intel-ucode`
- for audio support add:
  - `pipewire` and maybe also:
  - `pipewire-alsa pipewire-pulse pipewire-jack wireplumber`
  - first you could install just `pipewire`, you can still install the others later if needed
- and maybe `sof-firmware` for onboard audio, depending on your system
- for bluetooth add: `bluez bluez-utils`
- other shells, e.g. for zsh add: `zsh`

#### Summarized package list

All packages above together:
- e.g. Grub + UEFI: `pacstrap -K /mnt   base linux linux-headers linux-lts linux-lts-headers linux-firmware   grub efibootmgr   btrfs-progs mtools e2fsprogs   base-devel sudo man-db man-pages texinfo   networkmanager   reflector   openssh vim rsync terminus-font   amd-ucode intel-ucode   pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber sof-firmware   bluez bluez-utils   zsh`
- e.g. Systemd-boot (UEFI only): same as above, but without the `grub` package

## Configure the system

- <https://wiki.archlinux.org/title/Installation_guide#Configure_the_system>

### Fstab (filesystem table)

- <https://wiki.archlinux.org/title/Installation_guide#Fstab>

#### Create fstab
- `genfstab -U /mnt >> /mnt/etc/fstab` # `-U`: use UUIDs
- `cat /mnt/etc/fstab` # check

> :warning: You should delete `subvolid=<subvolid>` in the mount options in fstab if present:
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

> :memo: The UUIDs shown at 'with enryption' or 'without encrypion' within following points would match.
> They are only different because it was a seperate new installation each time with new random UUIDs generated.
ge

#### UEFI/GPT with encryption

```text
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@ 0 0

# /dev/vda1 LABEL=EFI
UUID=4584-08C8       /efi       vfat       rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro 0 2

# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /home      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@home 0 0

# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /var/log   btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@varlog 0 0

# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /swap      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@swap 0 0

# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /.snapshots btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@snapshots 0 0

# /dev/mapper/root LABEL=ROOT
UUID=7986ecc9-7656-4ccc-814f-e58f20e241a5 /.btrfsroot btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvolid=5,subvol=/ 0 0

# Example with swapfile:
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

# Example with swapfile:
/swap/swapfile       none       swap       defaults   0 0
```

#### BIOS/GPT with encryption

```text
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /          btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@ 0 0

# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /home      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@home 0 0

# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /var/log   btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@varlog 0 0

# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /swap      btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@swap 0 0

# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /.snapshots btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvol=/@snapshots 0 0

# /dev/mapper/root LABEL=ROOT
UUID=53791c43-d7ad-48fd-bbc9-8f15451d94f0 /.btrfsroot btrfs      rw,noatime,compress=zstd:3,space_cache=v2,subvolid=5,subvol=/ 0 0

# Example with swapfile:
/swap/swapfile       none       swap       defaults   0 0
```

### === Chroot start ===

### Chroot (Change root into new system)

- <https://wiki.archlinux.org/title/Installation_guide#Chroot>

- `arch-chroot /mnt`

### Time

- <https://wiki.archlinux.org/title/Installation_guide#Time>
- <https://wiki.archlinux.org/title/System_time#Time_standard>
- <https://wiki.archlinux.org/title/System_time#Hardware_clock>
- <https://wiki.archlinux.org/title/System_time#System_clock>
- <https://wiki.archlinux.org/title/System_time#Time_zone>
- <https://wiki.archlinux.org/title/System_time#Time_synchronization>
- <https://wiki.archlinux.org/title/Systemd-timesyncd#Enable_and_start>
&nbsp;

- `tzselect` # set time zone interactively
- or manually, e.g. for Germany:
- `ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime` # set time zone (symbolic link) # adjust to your location
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

- `vim /etc/locale.gen` # uncomment `en_US.UTF-8 UTF-8` and other UTF-8 locales you need
- `locale-gen` # Generate the locales
&nbsp;

- `vim /etc/locale.conf` # set the LANG variable accordingly
  - `LANG=en_US.UTF-8` # insert this line (for US english) in the empty new file
&nbsp;

- `vim /etc/vconsole.conf` # set the console keyboard layout (persistent)
  - `KEYMAP=de-latin1` # insert this line in the empty new file (german keyboard layout)
  - ~~or~~
  - ~~`localectl set-keymap de-latin1` # corresoponding to timedatectl~~ # not available in chroot

## Network configuration

- <https://wiki.archlinux.org/title/Installation_guide#Network_configuration>
- <https://wiki.archlinux.org/title/Network_configuration>

### hostname file

- <https://wiki.archlinux.org/title/Network_configuration#Set_the_hostname>
&nbsp;

- `vim /etc/hostname` # Create the hostname file
  - `HOSTNAME` # insert desired hostname in this empty new file

### hosts file

- <https://wiki.archlinux.org/title/Network_configuration#Local_network_hostname_resolution>
- <https://man.archlinux.org/man/hosts.5>
&nbsp;

- `vim /etc/hosts`
  - insert these lines into this emtpy new file, adjust HOSTNAME to your chosen hostname

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

- Enable its systemd unit to start autamatically:
  - `systemctl enable NetworkManager.service`
- or the corresponding service, if you installed another one
  - e.g. for "dhcpcd" instead of "NetworkManager": `systemctl enable dhcpcd.service`

## INSERTION: keyfile for decrypting root partition on boot

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Creating_a_keyfile_with_random_characters>
- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Storing_the_keyfile_on_a_file_system>
- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#Configuring_LUKS_to_make_use_of_the_keyfile>
&nbsp;

- **Skip for systemd-boot**

If the root partition is encrypted and you want it to be dercrypted automatically on boot, you can use a keyfile.
Skip this step, if you no not use enryption or do not want to use keyfile.

> :memo: You will still be asked to enter the passphrase at boot for decrypting bootloader
> (the passphrase you set when encryptig the root partition, since `/boot` is on the encrypted root partition)

- Creating a keyfile and storing it on the file system:
  - `dd bs=512 count=4 if=/dev/random iflag=fullblock | install -m 0600 /dev/stdin /etc/cryptsetup-keys.d/crypto_keyfile.key`
- Configuring LUKS to make use of the keyfile:
  - e.g.: `cryptsetup luksAddKey /dev/vda2 /etc/cryptsetup-keys.d/crypto_keyfile.key` # `/dev/vda2` is our root partition
    - you will be asked to enter the passphrase you set when encryptig the root partition

The next steps to do are

- Include the key in mkinitcpio's FILES array (in `mkinitcpio.conf`)
- Specify the keyfile with the `cryptkey=` kernel parameter (in `/etc/default/grub`)

and are described in the following sections.

## Initramfs (Create initial ramdisk environment)

- <https://wiki.archlinux.org/title/Installation_guide#Initramfs>
- <https://wiki.archlinux.org/title/Mkinitcpio>
- <https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_mkinitcpio>
- <https://wiki.archlinux.org/title/Dm-crypt/System_configuration#mkinitcpio>
- <https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#Configuring_mkinitcpio>
- <https://wiki.archlinux.org/title/Mkinitcpio#Common_hooks>
- <https://archlinux.org/packages/?name=cryptsetup>
- <https://wiki.archlinux.org/title/Dm-crypt/Specialties#The_encrypt_hook_and_multiple_disks>

### Check mkinitcpio.conf

#### Modules

- `vim /etc/mkinitcpio.conf`
- add `btrfs`: `MODULES=(btrfs)`

#### FILES: Include the keyfile for decrypting root partition on boot

- <https://wiki.archlinux.org/title/Dm-crypt/Device_encryption#With_a_keyfile_embedded_in_the_initramfs>

**Skip this step if you no not use enryption or do not want to use a keyfile (e.g. when using systemd-boot in this guide).**

- Include the key in mkinitcpio's FILES array:
  - `vim /etc/mkinitcpio.conf`
  - set: `FILES=(/etc/cryptsetup-keys.d/crypto_keyfile.key)`

#### Hooks

For LVM, **system** encryption or RAID, modify mkinitcpio.conf(5) (`/etc/mkinitcpio.conf`) and recreate the initramfs image.

- `vim /etc/mkinitcpio.conf`
- No Encryption:
  - Grub:
    - `HOOKS`: default should be ok, nothing to do
  - Systemd-boot:
    - `HOOKS`: replace `udev` with `systemd` and `keymap consolefont` with `sd-vconsole`
      - results in e.g.: `HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)`
- Encryption (LUKS):
  - Grub:
    - `HOOKS`: add **encrypt** before `filesystems`
      - results in e.g.: `HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block` **encrypt** `filesystems fsck)`
  - Systemd-boot:
    - `HOOKS`: add **sd-encrypt** before `filesystems`
      - results in e.g.: `HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block` **sd-encrypt** `filesystems fsck)`

### Creating new initramfs

- `mkinitcpio -P` 
  - `-P`: process all presets contained in `/etc/mkinitcpio.conf`

## Boot loader

- <https://wiki.archlinux.org/title/Installation_guide#Boot_loader>
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
> Boot related files are saved in folder `/boot`, which is a subfolder of root and therefore are backed up in snapshots.
> If we roll back then the versions of the files in `/boot` will be consistent with the remaining filsystem.
> Which would not necessarily be the case if `/boot` is on the `/efi` partition or a seperate `/boot` partition, which are not snapshotted.

#### Config default grub

##### Encrypted the root partition

- <https://wiki.archlinux.org/title/GRUB#Encrypted_/boot>

If you encrypted the root partition (which includes `/boot` folder), we have to configure the boot loader:

- `vim /etc/default/grub`
- uncomment: `GRUB_ENABLE_CRYPTODISK=y`

##### Preparing next step: get UUID of encrypted partition

We need some info (UUID of encrypted partition) for the next step:
`blkid`

```text
/dev/vda2: UUID="0daddb47-c687-4194-90e4-ac023ea72b5b" TYPE="crypto_LUKS" PARTUUID="96ebe15f-8ec3-4db4-8bd0-877798ecbb37"
[...]
/dev/mapper/root: LABEL="ROOT" UUID="7986ecc9-7656-4ccc-814f-e58f20e241a5" UUID_SUB="a27f7903-4fe4-41cb-9407-47192b592a68" BLOCK_SIZE="4096" TYPE="btrfs"
[...]
```

or more specific:

- UUID of encrypted partition: `blkid -o value -s UUID /dev/vda2`
  - `0daddb47-c687-4194-90e4-ac023ea72b5b`
- (UUID of decrypted partition: `blkid -o value -s UUID /dev/mapper/root`)
  - (`7986ecc9-7656-4ccc-814f-e58f20e241a5`)

##### Unlock the encrypted root partition on boot by setting kernel parameters

- search for the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`
  - which should look like this: `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"`
- add at the end (separated by a space character):
  - *cryptdevice=UUID=deviceUuidOfEncryptedDevice:dmname root=pathOfDecryptedPartitionDmname*
  - e.g.: `cryptdevice=UUID=0daddb47-c687-4194-90e4-ac023ea72b5b:root root=/dev/mapper/root`

If you also dediced to use a keyfile to **automatically** unlock root partition on boot:

- additionally add at the end (separated by a space character):
  - `cryptkey=rootfs:/etc/cryptsetup-keys.d/crypto_keyfile.key`

Altogether it would look like this in our example:
`GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=0daddb47-c687-4194-90e4-ac023ea72b5b:root root=/dev/mapper/root cryptkey=rootfs:/etc/cryptsetup-keys.d/crypto_keyfile.key"`

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

### Systemd-boot

`systemd-boot` is shipped with the systemd package which is a dependency of the `base` meta package, so no additional packages need to be installed manually.

#### Installing the UEFI boot manager

- <https://wiki.archlinux.org/title/Systemd-boot#Installing_the_UEFI_boot_manager>
- <https://man.archlinux.org/man/bootctl.1#OPTIONS>

`bootctl --entry-token=machine-id --make-entry-directory=yes install`

#### Automatic update via systemd service

- <https://wiki.archlinux.org/title/Systemd-boot#systemd_service>

To autamatically update systemd-boot enable its update service:
`systemctl enable systemd-boot-update.service`

#### Copy kernel and initramfs to directory on the efi partition

##### Preparation step

Kernel and initramfs files have to be copied to the (unencrypted) efi partition's subdirectory named after the machine-ID. This directory has been autamatically created when installing the boot manager.

We need the machine-ID first: `cat /etc/machine-id`

- prints e.g.: af36e2965f254d26a73f1eb3e6049a8c

Now we know the destination path on the efi partition: `/efi/af36e2965f254d26a73f1eb3e6049a8c/`
&nbsp;

> :memo: **Access to kernel and initramfs**
> Systemd-boot needs access to the kernel and initramfs (unencrypted), which are generated by `mkinitcpio` (according to its current config) into the `/boot` folder on the encrypted root partition / root subvolume.
> 
> This has the advantage that `/boot` is included in the snapshots of the root subvolume. So we have the kernel and initramfs available which match the status of the snapshot if we need it (e.g. if we make a rollback).
> 
> The drawback is that we need to copy kernel and initramfs to a directory on the efi partition and config the systemd-boot loader entries to point there.

> :memo: **Automated copy of kernel and initramfs**
> Copy of kernel and initramfs after every kernel update to the directory on the efi partition will be automated via systemd (see further below).

> :memo: **Kernel and initramfs when rolling back to a desired snapshot**
> In this guide we will use `snapper-rollback` (AUR) for rolling back to a desired snapshot.
> 
> When installing and configuring `snapper-rollback` at a later step (Post-install step after reboot), we will not use snapper-rollback directly, but have a `rollback` function which also copies the kernel and initramfs files matching the state of the snapshot to rollback to.
> 
> This prevents possible problems between a newer kernel and the packages at the rolled-back state, if there were kernel updates after the snapshot we roollback to.

&nbsp;

##### Automated copy of kernel and initramfs using systemd

- <https://wiki.archlinux.org/title/EFI_system_partition#Using_systemd>

Creating the systemd path and service files:

- `vim /etc/systemd/system/efistub-update.path`

  ```text
  [Unit]
  Description=Copy EFISTUB Kernel to EFI system partition

  [Path]
  PathChanged=/boot/initramfs-linux-fallback.img
  Unit=efistub-update.service

  #if you have multiple kernels installed, you can add them here too, e.g. for lts kernel:
  [Path]
  PathChanged=/boot/initramfs-linux-lts-fallback.img
  Unit=efistub-update.service
  #you have to add 'ExecStart's in the efistub-update.service file to copy this kernel and initramfs files
  #or create a separate service file for each kernel

  [Install]
  WantedBy=multi-user.target
  WantedBy=system-update.target
  ```

- `vim /etc/systemd/system/efistub-update.service`

  ```text
  [Unit]
  Description=Copy EFISTUB Kernel to EFI system partition

  [Service]
  Type=oneshot
  #linux:
  ExecStart=/usr/bin/cp -af /boot/vmlinuz-linux /efi/af36e2965f254d26a73f1eb3e6049a8c/
  ExecStart=/usr/bin/cp -af /boot/initramfs-linux.img /efi/af36e2965f254d26a73f1eb3e6049a8c/
  ExecStart=/usr/bin/cp -af /boot/initramfs-linux-fallback.img /efi/af36e2965f254d26a73f1eb3e6049a8c/
  #linux-lts:
  ExecStart=/usr/bin/cp -af /boot/vmlinuz-linux-lts /efi/af36e2965f254d26a73f1eb3e6049a8c/
  ExecStart=/usr/bin/cp -af /boot/initramfs-linux-lts.img /efi/af36e2965f254d26a73f1eb3e6049a8c/
  ExecStart=/usr/bin/cp -af /boot/initramfs-linux-lts-fallback.img /efi/af36e2965f254d26a73f1eb3e6049a8c/
  ```

> :memo: **Secure Boot**
> For Secure Boot with your own keys, you can set up the service to also sign the image using `sbsigntools`.

##### Verify the syntax of the Systemd Unit files

`systemd-analyze verify /etc/systemd/system/efistub-update.*`

##### Enable and start efistub-update.path

`systemctl enable efistub-update.path`

##### Copy of kernel and initramfs using systemd service

`systemctl start efistub-update.service`

#### Loader configuration

- <https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration>
- <https://man.archlinux.org/man/loader.conf.5#OPTIONS>
&nbsp;

machine-ID in our example is: af36e2965f254d26a73f1eb3e6049a8c (see further above)

- `vim /efi/loader/loader.conf`

  ```text
  timeout      4
  default      af36e2965f254d26a73f1eb3e6049a8c-*
  console-mode max
  editor       no
  ```

> :memo: **Delimiter**
> Use **spaces** (NOT tabs) as delimiter.

- if you have set `timeout 0`, the boot menu can be accessed by pressing Space.
- `default`: a glob pattern to select the default entry...
  - you may also set `default` to one prefered loader entry, e.g.:
  - `default af36e2965f254d26a73f1eb3e6049a8c-arch.conf`

#### Adding loaders

- <https://wiki.archlinux.org/title/Systemd-boot#Adding_loaders>
- <https://uapi-group.org/specifications/specs/boot_loader_specification/>

##### Standard loaders

**for 'linux' kernel e.g.:**

- `vim /efi/loader/entries/af36e2965f254d26a73f1eb3e6049a8c-arch.conf`
- no encryption:

  ```text
  title      Arch Linux
  linux      /vmlinuz-linux
  initrd     /initramfs-linux.img
  options    rootflags=subvol=/@ root=UUID=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA rw
  ```

- with encryption:
  - <https://wiki.archlinux.org/title/Dm-crypt/System_configuration#Using_systemd-cryptsetup-generator>

  ```text
  title      Arch Linux
  linux      /vmlinuz-linux
  initrd     /initramfs-linux.img
  options    rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw
  ```

- and optional (encryption and no encryption) add to conf file:

  ```text
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  ```

- **systemd-boot only** (encryption and no encryption): optional add to `options` in conf file:
  - `systemd.machine-id=46ccd99c37fa4e3cb5bfe076152df18f`
  - e.g. with enryption:

    ```text
    `options rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw`
    ```

> :memo: **Note:**
> When using `rd.luks.name` parameter you can omit `rd.luks.uuid` and vice versa.
>
> - `rd.luks.name`: Specify the name of the mapped device after the LUKS partition is open, the UUID ist the UUID of the LUKS partition
>   - This is equivalent to the second parameter of encrypt's cryptdevice
> - `rd.luks.uuid`: Specify the UUID of the device to be decrypted on boot:
>   - `rootflags=subvol=/@ rd.luks.uuid=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA root=/dev/mapper/root rw`

&nbsp;

**for 'linux-lts' kernel e.g.:**

- `vim /efi/loader/entries/af36e2965f254d26a73f1eb3e6049a8c-arch-lts.conf`
- no encryption:

  ```text
  title      Arch Linux LTS
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux-lts
  initrd     /initramfs-linux-lts.img
  options    rootflags=subvol=/@ root=UUID=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA rw
  ```

- with encryption:

  ```text
  title      Arch Linux LTS
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux-lts
  initrd     /initramfs-linux-lts.img
  options    rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw
  ```

##### Fallback loader

**for 'linux' kernel e.g.:**

- `vim /efi/loader/entries/af36e2965f254d26a73f1eb3e6049a8c-arch-fallback.conf`
- no encryption:

  ```text
  title      Arch Linux (fallback initramfs)
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux
  initrd     /initramfs-linux-fallback.img
  options    rootflags=subvol=/@ root=UUID=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA rw
  ```

- with encryption:

  ```text
  title      Arch Linux (fallback initramfs)
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux
  initrd     /initramfs-linux-fallback.img
  options    rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw
  ```

**for 'linux-lts' kernel e.g.:**

- `vim /efi/loader/entries/af36e2965f254d26a73f1eb3e6049a8c-arch-fallback.conf`
- no encryption:

  ```text
  title      Arch Linux LTS (fallback initramfs)
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux-lts
  initrd     /initramfs-linux-lts-fallback.img
  options    rootflags=subvol=/@ root=UUID=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA rw
  ```

- with encryption:

  ```text
  title      Arch Linux LTS (fallback initramfs)
  machine-id 46ccd99c37fa4e3cb5bfe076152df18f
  linux      /vmlinuz-linux-lts
  initrd     /initramfs-linux-lts-fallback.img
  options    rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw
  ```

##### Additional options

In the loader entries add to `options` (separated by space):

- `nvme_load=YES` # if you have an (NVME) SSD
- `nowatchdog`
  - <https://wiki.archlinux.org/title/Improving_performance#Watchdogs>
  - ok to set for non-critical home / desktop use case
- **systemd-boot only**, options: `systemd.machine_id=<MACHINEID>`
  - e.g.: `systemd.machine_id=46ccd99c37fa4e3cb5bfe076152df18f`
  - options (encryption): `options    rootflags=subvol=/@ rd.luks.name=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA=root root=/dev/mapper/root rw systemd.machine_id=46ccd99c37fa4e3cb5bfe076152df18f`
  - options (no encryption): `options    rootflags=subvol=/@ root=UUID=1C2A3274-4C0B-4146-A5B7-EC8C5235E1FA rw systemd.machine_id=46ccd99c37fa4e3cb5bfe076152df18f`

#### Check config

- `bootctl`

> :memo: **Tip:**
> After changing the configuration, run `bootctl` (without any arguments) to make sure that systemd-boot will be able to parse it properly.

## Set password for root user

- <https://wiki.archlinux.org/title/Installation_guide#Root_password>
&nbsp;

- `passwd`

> :warning: If you do not set a password for the root user you can not login after reboot !

## Reboot

- <https://wiki.archlinux.org/title/Installation_guide#Reboot>

> :warning: Check: Did you set a password for the root user ?!

### === Chroot end ===

- `exit` # Exit the chroot environment

### Optionally manually unmount all the partitions

- ~~(`swapoff /mnt/swap/swapfile`)~~
- `umount -R /mnt` # manually unmount all the partitions, allows noticing any "busy" partitions
  - also unmounts "CDROM", USB device ... arch installation medium

### Restart the machine

> :warning: Check: Did you set a password for the root user when in chroot environment ?!

- `systemctl reboot` # or just: `reboot`

* * *
* * *

## === Post-installation steps ===

- <https://wiki.archlinux.org/title/Installation_guide#Post-installation>
- <https://wiki.archlinux.org/title/General_recommendations>
- <https://wiki.archlinux.org/title/List_of_applications>

`login:` root
`Password:` enter password for root user

## System administration

### Users and groups

#### Add a new user

- <https://wiki.archlinux.org/title/Users_and_groups#User_management>
- <https://wiki.archlinux.org/title/Users_and_groups#Changing_user_defaults>
&nbsp;

- `useradd -m -G wheel -s /bin/bash USERNAME` # create user 'USERNAME', add to group 'wheel', shell would autamtically set to the default shell (should be `bash`), but we defined explicitly `bash`
- `passwd USERNAME` # set a password
&nbsp;

- optional / later: add user to some groups
  - `usermod -aG sys,adm,network,scanner,power,uucp,audio,lp,rfkill,video,storage,optical,users USERNAME`

- optional: change standard shell of the user to another shell, e.g. zsh
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
- e.g. uncomment this line to allow members of group `wheel` to execute any command:
  - `%wheel ALL=(ALL:ALL) ALL`

> :warning: If logged in as the newly created user: prefix the commands in the next steps with `sudo`.

### INSERTION: ssh again from another machine

- `vim /etc/ssh/ssh_config`
  - uncomment: `PasswordAuthentication yes`
&nbsp;

- `systemctl start sshd` # or permanantly: `systemctl enable --now sshd` # Start and / or enable sshd
- `systemctl status sshd` # check sshd serivce status

Connect to (virtual) machine:

- `ssh -o IdentitiesOnly=yes USERNAME@IP-ADDRESS`

### INSERTION: Snapper and btrfs snapshots

- <https://wiki.archlinux.org/title/Btrfs#Snapshots>
- <https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Snapshots>

#### Install packages

- `sudo pacman -S --needed snapper snap-pac sudo base-devel`

#### create snapper config file

- <https://wiki.archlinux.org/title/Snapper#Configuration_of_snapper_and_mount_point>
&nbsp;

- `sudo umount /.snapshots`
- `sudo rm -r /.snapshots` # # delete **folder** `/.snapshots`, since it will be created when creating snapper config (which would complain otherwise) in the next step
- `sudo snapper -c root create-config /` # # create new config named 'root'
- `sudo btrfs subvolume delete /.snapshots` # # delete **subvolume** `.snappshots` created with the last command (we already have subvolume `@snapshots`)
- `sudo mkdir /.snapshots` # # create **folder** `/.snapshots` again, since was also deleted with the last command
- `sudo mount -a` # # mount all filesystems mentioned in fstab
  - or specify explicitly, e.g. no encryption: `sudo mount /dev/vda2 -o subvol=@snapshots,compress=zstd,noatime /.snapshots` # adjust to your device path

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

Script to rollback snapper snapshots.

- <https://aur.archlinux.org/packages/snapper-rollback>
- <https://www.youtube.com/watch?v=maIu1d2lAiI>
&nbsp;

- `su USERNAME` # switch to the account of the new user, if not already done
- `cd` # go to user's home directory
- Download the sources for 'snapper-rollback' from AUR:
  - `curl -L -O https://aur.archlinux.org/cgit/aur.git/snapshot/snapper-rollback.tar.gz`
- `tar -xf snapper-rollback.tar.gz` # extract archive
- `cd snapper-rollback`
- `makepkg -sic` # build package, install dependencies, install (or upgrade) package, clean up leftover work files and directories
- `cd ..`
- `rm -rf snapper-rollback snapper-rollback.tar.gz` # delete the folder an file which is no longer required
&nbsp;

- `sudo btrfs subvolume list / | grep -e "@\?snapshots"` # list subvolumes with name 'snapshots'
- `sudo vim /etc/snapper-rollback.conf` # adjust config:
  - change mountpoint: `mountpoint = /.btrfsroot` (adding '.' according to the name of our created folder)
  - no need to adjust the name of the snapshot subvolume, since we used `@snapshots`
  - when using 'archinstall' skript the name of the snapshot subvolume is `@.snapshots` (as of 10/2024) and you would have to adjust it here

> :memo: **Note:**
> If you can't boot the system anymore, you have to rollback manually via live cd.
>
> The following instructions can be used for orientation in such case:
> 
> - <https://wiki.archlinux.org/title/Snapper#Restore_using_the_default_layout>
> - <https://www.dwarmstrong.org/btrfs-snapshots-rollbacks/>
>   - `11.` System rollback the 'Arch Way'
>   - deviating start:
>     - boot Arch live iso
>     - mount btrfs root and arch-chroot

##### Function for rollback

Since we want to rollback and also copy the kernel and initramfs files matching the snapshot to the efi partition, we will use a function (`rollback`).

Insert into your shell config file (e.g.: `vim ~/.bashrc` (for bash) and/or `vim ~/.zshrc` (for zsh), ...):

```bash
# function to rollback to a specified snapshot ID
# takes 1 parameter: snapshot ID to rollback to
# example: 'rollback 8'
rollback() {
  local snapshotsSubvolPath="/.snapshots"
  local machineID=$(cat /etc/machine-id)
  local efiPartitionPath="/efi"
  local efiPartition_targetPath="${efiPartitionPath}/${machineID}/" # destination for kernel and initramfs matching the snapshot ID to rollback to
  
  if [ "$#" -eq 1 ] && [ "${1}" -ge 1 ]; then # if exactly 1 parameter passed and is an integer >= 1
    local snapshotID="${1}" # Parameter 1: snapshot ID to rollback to
    local snapshot_bootFolder="${snapshotsSubvolPath}/${snapshotID}/snapshot/boot/" # source path of kernel and initramfs matching the snapshot ID to rollback to
  else
    echo "Parameter error: pass exactly one parameter (snapshot ID) which has to be an integer value >= 1"
    echo -e "Example: 'rollback 8'\nTo find the snapshot ID execute 'sudo snapper list'"
    return
  fi
 
  if [[ $(command -v snapper-rollback) ]]; then
    echo "Installing 'rsync' if not available..."
    sudo pacman -S --needed --noconfirm rsync # ensure rsync is installed
    
    local mode="test"
    read -rp "Do you want to just test or execute the rollback? ('e' = execute, other input = test): " mode
    local rollbackParameter=""
    local rsyncParameter=""
    if [ "${mode}" = "e" ]; then
        sudo snapper-rollback "${snapshotID}"
        
        echo -e "\nCopy kernel and initramfs from snapshot ${snapshotID} to '${efiPartition_targetPath}'..."
        sudo rsync -aPhEv --delete "${snapshot_bootFolder}" "${efiPartition_targetPath}"
    else
        rollbackParameter="--dry-run"
        rsyncParameter="--dry-run"

        echo -e "\nTESTING rollback, this is gonna be just a ${rollbackParameter}..."
        sudo snapper-rollback "${rollbackParameter}" "${snapshotID}"

        echo -e "\nCopy kernel and initramfs from snapshot ${snapshotID} to '${efiPartition_targetPath}' ${rsyncParameter}..."
        sudo rsync -aPhEv --delete "${rsyncParameter}" "${snapshot_bootFolder}" "${efiPartition_targetPath}"
    fi
  else
    echo "'snapper-rollback' not available, rollback not possible"
    return
  fi
}
```

- `source ~/.bashrc`

##### Rollback example

- `sudo snapper list` # list snapshots
&nbsp;

  ```text
  # │ Type   │ Pre # │ Date                            │ User │ Cleanup │ Description                                                              │ Userdata
  ───┼────────┼───────┼─────────────────────────────────┼──────┼─────────┼──────────────────────────────────────────────────────────────────────────┼─────────
  [...]
  7 │ pre    │       │ Sat 09 Nov 2024 06:43:04 PM CET │ root │ number  │ pacman -S --needed nano                                                  │
  8 │ post   │     7 │ Sat 09 Nov 2024 06:43:04 PM CET │ root │ number  │ nano   
  9 │ pre    │       │ Sat 09 Nov 2024 06:46:26 PM CET │ root │ number  │ pacman -Syu linux-zen linux-zen-headers                                  │
  10 │ post  │      9 │ Sat 09 Nov 2024 06:46:42 PM CET │ root │ number  │ linux-zen linux-zen-headers 
  ```

- Test:
  - `rollback 7`
&nbsp;

    ```text
    Installing 'rsync' if not available...
    [sudo] password for USER: 
    warning: rsync-3.3.0-2 is up to date -- skipping
    there is nothing to do
    Do you want to just test or execute the rollback? ('e' = execute, other input = test): 

    TESTING rollback, this is gonna be just a --dry-run...
    Are you SURE you want to rollback? Type 'CONFIRM' to continue: CONFIRM
    2024-11-11 01:22:31,896 - INFO - mv /.btrfsroot/@ /.btrfsroot/@2024-11-11T01:22
    2024-11-11 01:22:31,896 - INFO - btrfs subvolume snapshot /.btrfsroot/@snapshots/7/snapshot /.btrfsroot/@
    2024-11-11 01:22:31,896 - INFO - btrfs subvolume set-default /.btrfsroot/@
    2024-11-11 01:22:31,896 - INFO - [DRY-RUN MODE] Rollback to /.btrfsroot/@snapshots/7/snapshot complete. Reboot to finish

    Copy kernel and initramfs from snapshot 7 to '/efi/af36e2965f254d26a73f1eb3e6049a8c/' --dry-run...
    sending incremental file list
    deleting vmlinuz-linux-zen
    deleting initramfs-linux-zen.img
    deleting initramfs-linux-zen-fallback.img
    ./
    initramfs-linux.img
    vmlinuz-linux
    vmlinuz-linux-lts

    sent 247 bytes  received 117 bytes  728.00 bytes/sec
    total size is 267.43M  speedup is 734,700.43 (DRY RUN)
    ```

- Rollback:
  - `rollback 7`
&nbsp;

    ```text
    Installing 'rsync' if not available...
    warning: rsync-3.3.0-2 is up to date -- skipping
    there is nothing to do
    Do you want to just test or execute the rollback? ('e' = execute, other input = test): e
    Are you SURE you want to rollback? Type 'CONFIRM' to continue: CONFIRM
    2024-11-11 01:26:45,597 - INFO - Rollback to /.btrfsroot/@snapshots/7/snapshot complete. Reboot to finish

    Copy kernel and initramfs from snapshot 7 to '/efi/af36e2965f254d26a73f1eb3e6049a8c/'...
    sending incremental file list
    deleting vmlinuz-linux-zen
    deleting initramfs-linux-zen.img
    deleting initramfs-linux-zen-fallback.img
    ./
    initramfs-linux.img
            16.22M 100%  514.67MB/s    0:00:00 (xfr#1, to-chk=2/7)
    vmlinuz-linux
            13.48M 100%  217.90MB/s    0:00:00 (xfr#2, to-chk=1/7)
    vmlinuz-linux-lts
            13.00M 100%  145.87MB/s    0:00:00 (xfr#3, to-chk=0/7)

    sent 42.72M bytes  received 169 bytes  85.43M bytes/sec
    total size is 267.43M  speedup is 6.26
    ```

  - `sudo reboot`

> :memo: **Default subvolume**
> <https://btrfs.readthedocs.io/en/latest/Subvolumes.html>
> ... the subvolume that will be mounted by default, unless the default subvolume has been changed (see `btrfs subvolume set-default`).

&nbsp;

##### Manual Snapshot after Rollback
- <https://wiki.archlinux.org/title/Snapper#Single_snapshots>
- You could manually create a new snapshot (after rollback and reboot) to show up in the snapshot list, for better unterstanding what has been done, e.g.:
  - `sudo snapper -c root create -c number --description "after rollback 7"`
  - `sudo snapper list`
&nbsp;

    ```text
    # │ Type   │ Pre # │ Date                            │ User │ Cleanup │ Description                                                              │ Userdata
    ───┼────────┼───────┼─────────────────────────────────┼──────┼─────────┼──────────────────────────────────────────────────────────────────────────┼─────────
    [...]
    11 │ single │       │ Mon 11 Nov 2024 01:38:36 AM CET │ root │ number  │ after rollback 7
    ```

### INSERTION: Config zram as swap

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

> :memo: **Text extracts from the sources listed above:**
> zram can be used for swap or as a general-purpose RAM disk.
> zram creates a **compressed** block device in RAM.
> 
> **Starting remark**: If you have enough RAM / not much swapping, you do not need to concern yourself with the configuration of zram.
> 
> zram is particularly effective on machines that do not have much memory.
> And / or has an advantage for devices that have flash-based memory, which has a limited lifespan due to write amplification.
> 
> **Usage as swap**: Initially the created zram block device does not reserve or use any RAM. Only as files need or want to be swapped out, they will be **compressed** (if possible) and moved into the zram block device. The zram block device will then dynamically grow or shrink as required.
> 
> Even when assuming that zstd only achieves a conservative 1:2 compression ratio (real world data shows a common ratio of 1:3), zram will offer the advantage of being able to store more content in RAM than without memory compression.
> So instead of having 4 GB of RAM, with zRAM you could utilize around 8-12 GB of effective system memory.
> 
> Compression ratio depends on the data / intended use of your machine. Data that is difficult / can not be compressed would not benefit much from zram compared to data that can be easily compressed.
> 
> The compression and decompression time is generally negligible for any relatively modern CPU (>= ~2015).
> 
> Using zram or zswap (as of 10/2024: zswap is the default in Arch Linux) reduces swap usage, which effectively reduces the amount of wear placed on flash-based storage and makes it last longer.
> Using zram also results in significantly reduced I/O for Linux systems that require swapping.

#### Disable zswap

> :memo: If the related zswap kernel feature remains enabled, it will prevent zram from being used effectively. ...

To disable zswap permanently add `zswap.enabled=0` to your kernel parameters:

- **Grub**
  - `vim /etc/default/grub`
    - append `zswap.enabled=0` between the quotes in the `GRUB_CMDLINE_LINUX_DEFAULT` line, separated by space character
    - e.g.:
    - `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=2ee5664a-fb17-4762-8e3d-d8e4b8823815:root root=/dev/mapper/root zswap.enabled=0"`
  - re-generate the `grub.cfg` file:
    - `grub-mkconfig -o /boot/grub/grub.cfg`
- **Systemd-boot**
  - modify options for the loader entries config files: `sudo vim /boot/loader/entries/arch.conf` and `sudo vim /boot/loader/entries/arch-fallback.conf`
    - e.g.: `rootflags=subvol=/@ rd.luks.name=2ee5664a-fb17-4762-8e3d-d8e4b8823815=root root=/dev/mapper/root rw zswap.enabled=0`

Check if disabled (after a reboot):

- `cat /sys/module/zswap/parameters/enabled`
  - this should print `N` if deactivating zswap was successful

> :memo: Disable zswap at runtime (temporary):
> Login as root user and execute: `echo 0 > /sys/module/zswap/parameters/enabled`

#### Using zram-generator

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

### INSERTION: Desktop Environment

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
