# Arch installation
## Preparation
On the drive you want to install on, you'll have to wipe it with something like `blkdiscard /dev/X`. You can find the drive with `lsblk`. 

## Partitioning and formatting
Use something like `cfdisk` to partition. I personally have a boot partition of 2G, SWAP with 36G and the rest is a root partition of type Linux x86-64. In cfdisk, you can easily change the type of each partition. You'll have to format the partitions with these commands:
```
mkfs.fat -F 32 /dev/efi_system_partition
mkswap /dev/swap_partition
mkfs.xfs /dev/root_partition
```
For a ext4 filesystem, replace mkfs.xfs with mkfs.ext4

Overview: 
> ![image](https://github.com/user-attachments/assets/06de0820-613b-439c-9fbb-d3911e1852d6)

## Mounting 
Using this partition scheme, this is super simple. Make sure to mount root before boot:
```
mount /dev/root_partition /mnt
swapon /dev/swap_partition
mount --mkdir /dev/efi_system_partition /mnt/boot
```

## Installing system
We'll keep it simple for now:
```
pacstrap -K /mnt base base-devel linux linux-headers linux-firmware amd-ucode networkmanager nano xfsprogs
```
Tips: Check out https://wiki.archlinux.org/title/Linux_firmware to see what firmware you need. However, the meta-package is in general recommended. 
If you use ext4, you don't need xfsprogs. We'll install everything else after chrooting! In order to generate fstab file, use `genfstab`: `genfstab -U /mnt >> /mnt/etc/fstab`

## Chroot, enabling multilib, nVidia and enabling internet
The system is essentially installed, and now you just have to configure it. This is just a matter of following the simple steps [here](https://wiki.archlinux.org/title/Installation_guide#Chroot). Enable multilib by uncommenting the relevant lines in `/etc/pacman.conf`. For nVidia users, you should install the nVidia-drivers. Here is a summary of things you'll probably want: `nvidia-open-dkms nvidia-utils nvidia-settings`. Remember to enable NetworkManager `systemctl enable NetworkManager` ;)

## Bootloader
To start, I simply use systemd-boot. First run `bootctl install`. You'll have to manually create an entry in `/boot/loader/entries`. For reference, this is how my `/boot/loader/entries/arch.conf` looks like: 
```
title	Arch
linux	/vmlinuz-linux
initrd	/initramfs-linux.img
options root=PARTUUID=x rw
```
I recommend adding the last options line with the following command `echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/root_partition) rw" >> /boot/loader/entries/arch.conf`, tips taken from this [video](https://www.youtube.com/watch?v=_JYIAaLrwcY) (about 30 min mark). Edit loader.conf in /boot/loader directory. This Is what mine looks like:
```
default arch.conf
timeout 5
#console-mode keep
```
Tips: default @saved can also be used to boot into to the last picked kernel. 

## Make an account before restarting
This is the final stretch. First create your mkinitcpio: `mkinitcpio -P`.
I suggest setting the root password by simply typing `passwd`. To make a user account:
```
useradd -m -g users -G wheel,storage,power -s /bin/bash name
passwd name
```
Edit the wheel group (the top one) by uncommenting it with `EDITOR=nano visudo`. Now reboot into your brand new system!
```
exit
umount -R /mnt
reboot
```
