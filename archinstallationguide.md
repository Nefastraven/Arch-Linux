# Installing Arch Linux Step-By-Step

**NOTE:** When it says `edit` or `create`, edit the file with `nano`, `vi`, or `vim`. After the edit, whatever is blockquoted below is what you should have in the file.

<br>
1. Check if you are in UEFI mode:

- `ls /sys/firmware/ei/efivars`

<br>
2. Connect to the internet:

- `systemctl stop dhcpcd@eno1.service`
- `ip link` to find `DEVICE` which might be something like _wlp3s0_
- `wifi-menu -o DEVICE`

<br>
3. Partitioning drives:

- `lsblk`
- `gdisk /dev/sdx`

Create 2 partitions: a 512MiB EFI System Partition (code: `EF00`) and the rest as a Linux File System (Default). 

To create them, use 'p' to check the current partition table, 'd' to delete, and 'n' to make a new partition, 'w' to write changes

- `partprobe`
- `mkfs.ext4 /dev/sdxy` for the file system
- `mkfs.fat -F32 /dev/sdxy` for the ESP

<br>
4. Mount the partitions:

- `mkdir -p /mnt/boot`
- `mount /dev/sdxy /mnt` for the root partition (ext4 file system)
- `mount /dev/sdxy /mnt/boot` for the ESP

<br>
5. Installing base packages:

- `pacstrap /mnt`
- `pacstrap -i /mnt base base-devel btrfs-progs`

<br>
6. Generate an fstab (determines how partitions get mounted):

- `genfstab -U /mnt >> /mnt/etc/fstab`

<br>
7. Change root into the new Arch system with Bash:

- `arch-chroot /mnt /bin/bash`

<br>
8. Locale (language and regional character sets):

- `edit /etc/locale.gen`

Uncomment en_US.UTF-8 UTF-8

- `locale-gen`
- `create /etc/locale.conf`

> LANG=en_US.UTF-8

<br>
9. Setting a time zone:

- `tzselect`

ZONE and SUBZONE are determined in tzselect, use them for the following command

- `ln -s /usr/share/zoneinfo/ZONE/SUBZONE /etc/localtime`
- `hwclock --systohc --utc`

<br>
10. Initramfs:

- `mkinitcpio -p linux`

<br>
11. Boot Loader (systemd-boot, which is a UEFI bootloader that Arch is known for):

- `bootctl install`
- `pacman -S intel-ucode`
- `edit /boot/loader/loader.conf`

> timeout 4

> default arch

> editor 0

- `blkid -s PARTUUID -o value /dev/sdxy` find the PARTUUID for your root partition and write it down
- `create /boot/loader/entries/arch.conf`

> title	Arch Linux

> linux	/vmlinuz-linux

> initrd	/intel-ucode.img

> initrd	/initramfs-linux.img

> options	root=PARTUUID=_<whatever the ID was from the previous command>_ rw

<br>
12. Hostname (_myhostname_ == whatever you want your hostname/computer name on the network to be):

- `edit /etc/hostname`

> _myhostname_

Append the host name to /etc/hosts

- `edit /etc/hosts`

> 127.0.0.1	localhost.localdomain	localhost	myhostname

> ::1		localhost.localdomain	localhost	myhostname

<br>
13. Setting up wireless connections:

- `pacman -S iw wpa_supplicant dialog`

<br>
14. Bluetooth connections:

- `pacman -S bluez bluez-utils`

<br>
15. Set the root password:

- `passwd`

<br>
16. Unmount partitions and reboot:

Exit from the chroot environment with Ctrl+D or the `exit` command

- `umount -R /mnt`
- `reboot`

<br>
### POST-INSTALLATION AFTER REBOOTING (make sure you log in to root):

17. Create a user account (you don't want to use root as that is insecure and dangerous):

- `useradd -m -G wheel USER`
- `passwd USER`
- `EDITOR=nano visudo` or just `visudo` without the `EDITOR=nano` if you know how to use vim

> \#\# Uncomment to allow members of group...

> %wheel ALL=(ALL) ALL

- `edit /etc/pacman.conf` uncomment color and add ILoveCandy (this makes the installation progress bar Pacman eating white dots)

> \# Misc options

> Color

> ILoveCandy

- `pacman -Syy && reboot`

<br>
2. Install Xorg packages to use GUIs:

- `pacman -S xorg-server xorg-utils xorg-xinit xterm`

